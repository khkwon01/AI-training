# Chapter 04: 로깅과 모니터링

## 이 장에서 배우는 것

- 배포된 코드에서 로그가 왜 유일한 디버깅 수단인지
- CloudWatch Logs의 구조 (Log group → Log stream → Log event)
- Lambda에서 `print()`가 CloudWatch에 저장되는 원리
- 로그 레벨(INFO / WARNING / ERROR)을 구분하는 이유
- 실제 오류 로그를 읽고 원인을 파악하는 방법
- CloudWatch Metrics(Invocations, Errors, Duration)의 의미
- CloudWatch 알람 만들기
- 실습 3개: 로그 추가 → 실행 확인 → 오류 로그 분석

---

## 왜 필요한가 — 배포 후 디버깅의 유일한 수단

로컬에서 코드를 실행하면 터미널에 오류가 바로 보인다.

```
$ python memo.py
Traceback (most recent call last):
  File "memo.py", line 12, in add_memo
    memos[text] = timestamp   # KeyError: 딕셔너리 접근 오류
TypeError: unhashable type: 'list'
```

화면 앞에 앉아 있으니 즉시 볼 수 있다. 그런데 Lambda는 다르다.

Lambda 함수는 AWS 서버 어딘가에서 실행된다. 내 컴퓨터 터미널이 아니다. 오류가 나도 아무도 그 자리에서 보지 않는다. 사용자는 500 오류만 받고, 나는 무슨 일이 벌어졌는지 알 수 없다.

**이때 유일한 단서가 로그다.**

로그가 없으면:
- 오류가 어디서 났는지 알 수 없다
- 어떤 입력이 들어왔는지 알 수 없다
- 얼마나 자주 오류가 나는지 알 수 없다
- 고칠 방법을 모른다

로그를 잘 남기는 것은 선택이 아니라 배포된 서비스를 운영하기 위한 필수 기술이다.

---

## 1. CloudWatch Logs 구조 이해하기

Lambda 로그는 AWS CloudWatch Logs에 저장된다. 저장 구조를 이해하면 필요한 로그를 빠르게 찾을 수 있다.

### 3단계 계층 구조

```
CloudWatch Logs
└── Log group (로그 그룹)
    └── Log stream (로그 스트림)
        └── Log event (로그 이벤트)
```

각 계층의 의미:

| 계층 | 의미 | 예시 |
|------|------|------|
| **Log group** | 함수 단위 로그 묶음 | `/aws/lambda/hello-python` |
| **Log stream** | 실행 환경 인스턴스별 로그 묶음 | `2026/05/21/[$LATEST]abc123def456` |
| **Log event** | 개별 로그 한 줄 | `[INPUT] event: {"name": "Mina"}` |

### Log group

Lambda 함수 하나당 Log group이 하나 생성된다. 이름 형식은 항상 다음과 같다.

```
/aws/lambda/함수이름
```

예: 함수 이름이 `hello-python`이라면 Log group은 `/aws/lambda/hello-python`이다.

Log group은 함수가 처음 실행될 때 자동으로 생성된다. 한 번도 실행하지 않으면 Log group 자체가 없다.

### Log stream

Lambda는 요청을 처리할 때 실행 환경(컨테이너)을 만든다. Log stream은 이 실행 환경 인스턴스별로 생성된다.

이름 형식:
```
2026/05/21/[$LATEST]a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
```

앞부분이 날짜, 뒷부분이 실행 환경의 고유 ID다.

실제로 CloudWatch를 열어보면 여러 개의 Log stream이 보인다. 가장 최신 것을 열면 된다.

### Log event

Log stream 안에 있는 각각의 로그 줄이다. `print()`로 남긴 내용이 바로 Log event로 기록된다.

---

## 2. Lambda에서 print()가 CloudWatch에 저장되는 원리

Lambda가 실행될 때 내부적으로 일어나는 일:

```
Lambda 실행 시작
    ↓
표준 출력(stdout)을 CloudWatch Logs에 연결
    ↓
print()로 출력된 모든 내용 → CloudWatch Log event로 자동 저장
    ↓
오류(stderr)도 자동으로 CloudWatch에 저장
    ↓
Lambda 실행 종료 + 실행 요약(REPORT) 자동 기록
```

즉, Lambda에서 `print()`를 쓰면 자동으로 CloudWatch에 저장된다. 별도 설정이 필요 없다.

이것이 가능한 이유는 Lambda의 기본 실행 역할(AWSLambdaBasicExecutionRole)에 CloudWatch Logs에 쓰는 권한이 포함되어 있기 때문이다.

---

## 3. 로그 구조 읽기

CloudWatch Logs에서 Log stream을 열면 다음과 같은 형식이 보인다.

```
START RequestId: a1b2c3d4-e5f6-7890-abcd-ef1234567890 Version: $LATEST
[INPUT] event: {"queryStringParameters": {"name": "Mina"}}
[PROCESS] name=Mina
[OUTPUT] result: {'message': '안녕하세요, Mina님!'}
END RequestId: a1b2c3d4-e5f6-7890-abcd-ef1234567890
REPORT RequestId: a1b2c3d4-e5f6-7890-abcd-ef1234567890  Duration: 2.34 ms  Billed Duration: 3 ms  Memory Size: 128 MB  Max Memory Used: 36 MB
```

각 줄의 의미:

| 줄 | 의미 |
|----|------|
| `START RequestId: ...` | 이 요청 처리가 시작됨. RequestId는 요청 고유 번호 |
| 중간 줄들 | `print()`로 남긴 로그들 |
| `END RequestId: ...` | 이 요청 처리가 끝남 |
| `REPORT RequestId: ...` | 처리 요약: 실행 시간, 과금 시간, 메모리 사용량 |

### REPORT 줄 읽는 법

```
REPORT RequestId: abc-123
  Duration: 2.34 ms          ← 실제 실행 시간
  Billed Duration: 3 ms      ← 과금 기준 시간 (1ms 단위로 올림)
  Memory Size: 128 MB        ← 설정한 메모리 한도
  Max Memory Used: 36 MB     ← 실제 사용한 메모리
```

- Duration이 타임아웃 한도(기본 3초)에 가깝다면 최적화 필요
- Max Memory Used가 Memory Size에 가깝다면 메모리 설정을 늘려야 함

---

## 4. 로그 레벨 — INFO / WARNING / ERROR

로그에 레벨을 붙이면 나중에 검색하거나 필터링하기 쉽다.

| 레벨 | 의미 | 언제 사용하는가 |
|------|------|--------------|
| `INFO` | 정상 동작 중에 기록하는 일반 정보 | 입력값 확인, 처리 단계 기록 |
| `WARNING` | 문제는 아니지만 주의가 필요한 상황 | 기본값으로 대체됨, 예상치 못한 입력 |
| `ERROR` | 오류 발생, 요청 처리 실패 | 예외 발생, 외부 API 실패 |

### 로그 레벨을 붙인 코드 예시

```python
import json
import logging

# logging 모듈 설정
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    # INFO: 정상적인 처리 기록
    logger.info(f"요청 수신: {json.dumps(event)}")

    params = event.get("queryStringParameters") or {}
    name = params.get("name")

    if not name:
        # WARNING: 문제는 아니지만 기본값으로 대체
        logger.warning("name 파라미터 없음, 기본값 'World' 사용")
        name = "World"

    try:
        result = {"message": f"안녕하세요, {name}님!"}
        logger.info(f"처리 완료: {result}")
        return {
            "statusCode": 200,
            "body": json.dumps(result, ensure_ascii=False)
        }
    except Exception as e:
        # ERROR: 실제 오류 발생
        logger.error(f"처리 실패: {type(e).__name__}: {e}", exc_info=True)
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Internal Server Error"})
        }
```

CloudWatch Logs에서 `[ERROR]`로 필터링하면 오류만 빠르게 볼 수 있다.

### print() vs logging 모듈

초보자는 `print()`로 충분하다. 규칙만 지키면 된다.

```python
# print()로 로그 레벨 표시하기 (초보자용)
print("[INFO] 요청 수신")
print("[WARNING] 파라미터 없음, 기본값 사용")
print("[ERROR] 처리 실패")
```

나중에 팀 프로젝트나 실제 서비스를 운영할 때 `logging` 모듈로 업그레이드하면 된다.

---

## 5. 유용한 로그 남기기 — 패턴

단순히 `print("실행됨")`보다 더 구체적인 정보를 남기는 패턴이다.

```python
import json

def lambda_handler(event, context):
    # 패턴 1: 입력 전체 기록
    print(f"[INPUT] event: {json.dumps(event)}")

    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    # 패턴 2: 처리 중 중요 변수 기록
    print(f"[PROCESS] name={name!r}")

    result = {"message": f"안녕하세요, {name}님!"}

    # 패턴 3: 출력 기록
    print(f"[OUTPUT] statusCode=200, message={result['message']!r}")

    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

CloudWatch 로그 출력 예:
```
[INPUT] event: {"queryStringParameters": {"name": "Mina"}}
[PROCESS] name='Mina'
[OUTPUT] statusCode=200, message='안녕하세요, Mina님!'
```

### 로그에 넣으면 좋은 정보

- 입력값 (어떤 요청이 들어왔는지)
- 분기 조건 (어느 경로로 실행됐는지)
- 외부 API 호출 결과 (성공/실패, 응답 코드)
- 처리 완료 결과

### 로그에 절대 넣으면 안 되는 정보

- 비밀번호
- API 키 / 토큰
- 신용카드 번호
- 주민등록번호

CloudWatch Logs도 기록이 남기 때문에 민감 정보가 있으면 보안 사고가 된다.

---

## 6. 실제 오류 로그 읽는 법

오류가 발생하면 CloudWatch Logs에 다음과 같은 traceback이 남는다.

```
START RequestId: abc-123 Version: $LATEST
[INPUT] event: {}
[ERROR] KeyError: 'name'
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in lambda_handler
    name = event["name"]
KeyError: 'name'
END RequestId: abc-123
REPORT RequestId: abc-123  Duration: 1.23 ms  Billed Duration: 2 ms
```

읽는 순서:

**1단계: 오류 종류 확인**
```
KeyError: 'name'
```
`KeyError`는 딕셔너리에서 없는 키에 접근했을 때 나오는 오류다.

**2단계: 오류가 난 위치 확인**
```
File "/var/task/lambda_function.py", line 8, in lambda_handler
    name = event["name"]
```
`lambda_function.py` 파일의 8번째 줄, `lambda_handler` 함수 안에서 `event["name"]`을 실행하다 오류가 났다.

**3단계: 원인 파악**
`event`에 `"name"` 키가 없는데 `event["name"]`으로 직접 접근했다. `event.get("name", "World")`로 바꿔야 한다.

### 자주 보이는 오류 유형

| 오류 메시지 | 의미 | 주로 발생하는 상황 |
|------------|------|------------------|
| `KeyError: 'xxx'` | 딕셔너리에 없는 키 접근 | `event["name"]` 직접 접근 |
| `TypeError: 'NoneType' object is not subscriptable` | None에 인덱스 접근 | `event["queryStringParameters"]["name"]`에서 queryStringParameters가 None |
| `AttributeError: 'NoneType' object has no attribute 'get'` | None에 메서드 호출 | `event.get("queryStringParameters").get("name")` |
| `JSONDecodeError` | JSON 파싱 실패 | `json.loads()` 호출 시 잘못된 문자열 |
| `Task timed out after 3.00 seconds` | Lambda 타임아웃 | 외부 API 호출 지연, 무한 루프 |

---

## 7. try/except로 오류 로그 개선하기

오류가 나도 500 응답을 반환하고, 상세 정보를 로그에 남기는 패턴이다.

```python
import json
import traceback

def lambda_handler(event, context):
    print(f"[INFO] 요청 시작: {json.dumps(event)}")

    try:
        params = event.get("queryStringParameters") or {}
        name = params.get("name", "World")

        print(f"[INFO] 이름 추출: name={name!r}")

        result = {"message": f"안녕하세요, {name}님!"}

        print(f"[INFO] 처리 완료: statusCode=200")
        return {
            "statusCode": 200,
            "body": json.dumps(result, ensure_ascii=False)
        }

    except Exception as e:
        # 오류 타입, 메시지, 전체 traceback 기록
        print(f"[ERROR] {type(e).__name__}: {e}")
        print(f"[ERROR] Traceback:\n{traceback.format_exc()}")

        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Internal Server Error"})
        }
```

이 패턴의 장점:
- 사용자는 "Internal Server Error"만 보고 상세 오류는 숨겨짐 (보안)
- 개발자는 CloudWatch에서 전체 traceback 확인 가능

---

## 8. CloudWatch Logs 콘솔 사용법

### 경로 1: Lambda 함수에서 바로 이동

```
AWS 콘솔 로그인
  → Lambda 서비스 선택
  → 함수 목록에서 함수 클릭
  → [Monitor] 탭 클릭
  → [View CloudWatch logs] 버튼 클릭
```

이 경로가 가장 빠르다.

### 경로 2: CloudWatch 직접 접근

```
AWS 콘솔 로그인
  → CloudWatch 서비스 선택
  → 왼쪽 메뉴: [Logs] → [Log groups]
  → 검색창에 함수 이름 입력
  → /aws/lambda/함수이름 클릭
  → Log stream 목록에서 가장 위(최신) 클릭
```

### 로그 검색 필터 사용법

Log stream 화면 상단에 필터 입력창이 있다.

```
[ERROR]         ← 오류만 보기
[WARNING]       ← 경고만 보기
"KeyError"      ← 특정 오류 종류로 검색
"Mina"          ← 특정 이름이 포함된 로그 검색
```

---

## 9. CloudWatch Metrics (지표) 확인

Lambda 함수 페이지 → **Monitor** 탭에서 그래프로 확인한다.

### 주요 지표

| 지표 | 의미 | 언제 확인하는가 |
|------|------|--------------|
| **Invocations** | Lambda 실행 횟수 | 예상보다 많으면 무한 루프나 잘못된 자동 호출 의심 |
| **Duration** | 실행 시간 (ms) | 타임아웃(기본 3000ms)의 80% 이상이면 최적화 필요 |
| **Errors** | 오류 횟수 | 0이 아니면 즉시 CloudWatch Logs 확인 |
| **Throttles** | 동시 실행 제한 초과 횟수 | 서비스 규모가 커지면 확인 필요 |
| **ConcurrentExecutions** | 현재 동시 실행 수 | 동시 실행 한도(기본 1000)에 가까우면 확인 |

### Duration 그래프 읽는 법

```
Duration 그래프 예시:
평소: 5~15ms
특정 시점: 갑자기 2800ms (타임아웃 3000ms에 근접)
```

이런 패턴이 보이면:
- 외부 API 호출이 느려진 것
- 처리할 데이터가 갑자기 많아진 것
- 타임아웃 한도를 늘리거나 코드를 최적화해야 함

### Errors 그래프 읽는 법

Errors가 0이 아닐 때:
1. Errors 그래프의 시작 시점 확인 (언제부터 오류가 났는가)
2. 같은 시점의 최근 배포 여부 확인 (배포 직후라면 배포가 원인)
3. CloudWatch Logs에서 그 시점의 오류 내용 확인

---

## 10. CloudWatch 알람 만들기

오류가 발생했을 때 자동으로 알림을 받는 기능이다. 수시로 CloudWatch를 확인하지 않아도 된다.

### 알람 생성 경로

```
CloudWatch → Alarms → Create alarm
```

### Errors > 0 알람 만들기 (가장 중요한 알람)

**Step 1. 지표 선택**
```
Select metric
  → Lambda → By Function Name
  → 함수 이름 체크
  → Metric name: Errors 선택
  → Select metric 클릭
```

**Step 2. 조건 설정**
```
Conditions:
  Threshold type: Static
  Whenever Errors is: Greater than
  than: 0
```

**Step 3. 알림 설정 (선택)**
```
Notification:
  In alarm → Send a notification to:
  → Create new topic (처음이라면)
  → Topic name: lambda-alerts
  → Email endpoints: 내 이메일 입력
  → Create topic
```

이메일로 구독 확인 메일이 온다. 확인 링크를 클릭해야 알람이 발송된다.

**Step 4. 알람 이름**
```
Alarm name: hello-python-errors
```

이제 `hello-python` 함수에서 오류가 발생하면 이메일로 알림이 온다.

### Duration 알람 만들기 (타임아웃 방지)

위와 같은 과정으로, Metric name: Duration으로 설정하고 조건을 다음과 같이 설정:

```
Whenever Duration is: Greater than
than: 2500   ← 타임아웃(3000ms)의 83%
```

타임아웃이 임박하면 미리 알림을 받아 최적화할 수 있다.

---

## 11. 따라 하기 실습

### 실습 1. 로그 추가하고 CloudWatch에서 확인하기

**목표**: `print()` 로그를 추가하고, CloudWatch에서 로그를 직접 찾아서 읽는다.

**Step 1. 코드 수정**

Lambda 편집기에서 기존 함수를 아래로 교체한다.

```python
import json

def lambda_handler(event, context):
    print("[INFO] === 함수 실행 시작 ===")
    print(f"[INFO] 수신한 event: {json.dumps(event)}")

    params = event.get("queryStringParameters") or {}
    print(f"[INFO] queryStringParameters: {params}")

    name = params.get("name", "World")
    print(f"[INFO] 추출된 name: {name!r}")

    result = {"message": f"안녕하세요, {name}님!"}
    print(f"[INFO] 생성된 result: {result}")
    print("[INFO] === 함수 실행 완료 ===")

    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

**Step 2. Deploy**

코드 편집 후 오른쪽 위 [Deploy] 버튼 클릭.

**Step 3. 함수 URL 호출**

```bash
# 브라우저 또는 터미널에서
curl "https://여기에함수URL입력/?name=Mina"
```

**Step 4. CloudWatch에서 로그 확인**

```
Lambda 함수 페이지 → Monitor 탭 → View CloudWatch logs → 최신 Log stream 클릭
```

다음 내용이 모두 보이는지 확인한다:
- `[INFO] === 함수 실행 시작 ===`
- `[INFO] 수신한 event: ...`
- `[INFO] 추출된 name: 'Mina'`
- `REPORT` 줄의 Duration

---

### 실습 2. WARNING 로그 확인하기

**목표**: 파라미터가 없을 때 WARNING이 기록되는 것을 확인한다.

**Step 1. 코드 수정**

```python
import json

def lambda_handler(event, context):
    print("[INFO] 요청 수신")

    params = event.get("queryStringParameters") or {}
    name = params.get("name")

    if not name:
        print("[WARNING] name 파라미터 없음 → 기본값 'World' 사용")
        name = "World"
    else:
        print(f"[INFO] name 파라미터 수신: {name!r}")

    result = {"message": f"안녕하세요, {name}님!"}
    print(f"[INFO] 응답 생성 완료")

    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

**Step 2. Deploy 후 두 가지 방식으로 호출**

```bash
# name 파라미터 있는 경우
curl "https://함수URL/?name=Mina"

# name 파라미터 없는 경우
curl "https://함수URL/"
```

**Step 3. CloudWatch에서 두 Log stream 비교**

- 첫 번째 호출: `[INFO] name 파라미터 수신: 'Mina'`
- 두 번째 호출: `[WARNING] name 파라미터 없음 → 기본값 'World' 사용`

두 로그의 차이를 직접 눈으로 확인한다.

---

### 실습 3. 의도적으로 오류를 만들고 오류 로그 분석하기

**목표**: 오류가 발생했을 때 CloudWatch 로그에 어떻게 기록되는지 직접 보고, 로그에서 원인을 파악하는 연습을 한다.

**Step 1. 오류가 나는 코드 배포**

```python
import json

def lambda_handler(event, context):
    print("[INFO] 요청 수신")

    # 의도적 오류: .get()을 쓰지 않고 직접 접근
    # queryStringParameters가 None일 때 TypeError 발생
    name = event["queryStringParameters"]["name"]

    result = {"message": f"안녕하세요, {name}님!"}
    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

Deploy 클릭.

**Step 2. 오류 유발하기**

```bash
# name 파라미터 없이 호출 → 오류 유발
curl "https://함수URL/"
```

500 오류 응답이 오는 것을 확인한다.

**Step 3. CloudWatch에서 오류 로그 찾기**

Lambda → Monitor → View CloudWatch logs → 최신 Log stream 클릭

다음과 같은 오류 로그를 찾는다:

```
START RequestId: ...
[INFO] 요청 수신
[ERROR] TypeError: 'NoneType' object is not subscriptable
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in lambda_handler
    name = event["queryStringParameters"]["name"]
TypeError: 'NoneType' object is not subscriptable
END RequestId: ...
```

**Step 4. 로그를 읽고 원인 파악하기**

아래 질문에 답해본다:
- 오류 종류는 무엇인가? (`TypeError`)
- 오류가 난 줄 번호는? (8번째 줄)
- 왜 `'NoneType' object`라고 나왔는가? (queryStringParameters가 None이기 때문)
- 어떻게 고치면 되는가? (`event.get("queryStringParameters") or {}` 사용)

**Step 5. 올바른 코드로 복원**

```python
import json

def lambda_handler(event, context):
    print("[INFO] 요청 수신")

    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    result = {"message": f"안녕하세요, {name}님!"}
    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

Deploy 후 URL 재호출 → 정상 응답 확인.

---

## 자주 막히는 지점

### 막히는 지점 1. "CloudWatch에 로그가 안 보여요"

**원인**: Lambda 함수를 한 번도 실행하지 않으면 Log group 자체가 생성되지 않는다.

**해결**: 함수를 먼저 한 번 실행(URL 호출)하면 Log group이 자동으로 생성된다.

### 막히는 지점 2. "Log stream이 여러 개라 어떤 걸 봐야 하는지 모르겠어요"

**해결**: 제일 위에 있는 것이 가장 최근 것이다. 가장 위 항목을 클릭하면 된다.

### 막히는 지점 3. "로그가 너무 많아서 찾기 어려워요"

**해결**: Log stream 화면 상단 필터창에 `[ERROR]`를 입력하면 오류만 보인다.

### 막히는 지점 4. "Lambda에서 실행했는데 CloudWatch에 로그가 안 나타나요"

**원인**: Lambda 실행 역할에 CloudWatch 권한이 없는 경우.

**확인**: Lambda → Configuration → Permissions → Execution role 클릭 → AWSLambdaBasicExecutionRole 정책이 있는지 확인.

없다면 Attach policy에서 `AWSLambdaBasicExecutionRole` 추가.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| 로그 그룹이 없음 | CloudWatch에 로그가 안 보임 | 함수를 한 번이라도 실행해야 로그 그룹 생성 |
| 로그가 너무 많음 | 필요한 로그 찾기 어려움 | 최근 Log stream만 확인, 필터 사용 |
| 민감 정보를 로그에 남김 | API 키 등이 로그에 노출 | 비밀번호, API 키는 절대 print()로 출력하지 않음 |
| 로그 없이 디버깅 시도 | 원인 파악 불가 | print()를 충분히 추가한 후 재배포 |
| Deploy 없이 로그 확인 | 수정 전 코드의 로그가 나옴 | 코드 수정 후 반드시 Deploy 클릭 |

---

## 확인 체크리스트

- [ ] CloudWatch Logs에서 Lambda 로그를 찾을 수 있는가 (Log group → Log stream → Log event)
- [ ] `[INFO]`, `[WARNING]`, `[ERROR]` 접두사로 로그를 구분해서 남길 수 있는가
- [ ] 오류 로그에서 파일명, 줄 번호, 오류 종류, 오류 내용을 읽을 수 있는가
- [ ] Monitor 탭에서 Invocations, Duration, Errors 지표를 확인할 수 있는가
- [ ] CloudWatch 알람을 하나 이상 만들 수 있는가

---

## 한 번 더 생각해 보기

1. 로그를 너무 많이 남기면 어떤 문제가 생길까? (CloudWatch Logs는 저장량에 따라 비용이 발생한다)
2. 민감한 정보(비밀번호, 카드 번호)를 로그에 남기면 왜 위험한가?
3. 오류가 났을 때 traceback을 로그에 남기는 것이 왜 중요한가?
4. Duration이 갑자기 높아졌다면 어떤 원인을 먼저 의심해야 할까?

---

## 다음 장

다음 장에서는 API 키와 같은 민감한 값을 코드 밖에서 안전하게 관리하는 환경 변수와 보안 설정을 배운다.

---

## 참고 자료

- AWS CloudWatch Logs 공식 문서 — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html
- Lambda 모니터링 지표 — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html
- CloudWatch 알람 설정 — https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html
