# AWS Lambda 배포 후 검증 실습 01

## 학습 목표

배포된 Lambda 함수가 정상적으로 동작하는지 5가지 방법으로 확인하는 습관을 만든다. 실습 시나리오로 의도적인 오류 코드를 배포하고, 탐지하고, 수정하는 전 과정을 따라 해 본다.

---

## 전제 조건

- AWS 콘솔에 로그인된 상태
- Lambda 함수가 이미 하나 배포되어 있는 상태
- 함수 이름 예시: `hello-world-function`

아직 Lambda 함수가 없다면, 아래 기본 코드로 콘솔에서 직접 만들 수 있다.

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({
            "status": "ok",
            "message": "hello from aws"
        })
    }
```

---

## 검증 방법 1: AWS 콘솔에서 Test 이벤트 실행

가장 빠르고 기본적인 확인 방법이다. 콘솔에서 바로 함수를 실행해볼 수 있다.

### 순서

1. AWS 콘솔 → Lambda → Functions → 함수 이름 클릭
2. 상단 탭에서 **Test** 탭 클릭
3. **Create new event** 선택
4. Event name: `test-event` 입력
5. Event JSON에 아래 내용 입력:

```json
{
  "key1": "hello",
  "key2": "world"
}
```

6. **Save** 클릭
7. **Test** 버튼(주황색) 클릭

### 정상 응답 예시

```json
{
  "statusCode": 200,
  "body": "{\"status\": \"ok\", \"message\": \"hello from aws\"}"
}
```

콘솔에서 **Details** 를 펼치면 더 자세한 정보가 나온다:

```
Duration: 1.23 ms
Billed Duration: 2 ms
Memory Size: 128 MB
Max Memory Used: 37 MB
```

### 오류 응답 예시

```json
{
  "errorMessage": "name 'undefined_variable' is not defined",
  "errorType": "NameError",
  "stackTrace": [
    "  File \"/var/task/lambda_function.py\", line 5, in lambda_handler\n    return undefined_variable\n"
  ]
}
```

빨간색 배경으로 표시되며 `errorType`, `stackTrace`가 포함된다.

### 체크포인트 1

- [ ] Test 이벤트가 정상적으로 실행되었는가?
- [ ] `statusCode: 200`이 응답에 포함되어 있는가?
- [ ] `body`에 `"status": "ok"`가 있는가?

---

## 검증 방법 2: Function URL로 curl 테스트

Lambda Function URL이 활성화되어 있으면 HTTP로 직접 호출할 수 있다.

### Function URL 확인 방법

1. Lambda 함수 페이지 → **Configuration** 탭 → **Function URL**
2. URL이 있으면 복사, 없으면 **Create function URL** 클릭
3. Auth type: `NONE` (학습용 — 실제 서비스에서는 AWS_IAM 권장)
4. **Save** 클릭

URL 형태 예시:
```
https://abcdefghij.lambda-url.ap-northeast-1.on.aws/
```

### curl로 GET 요청 보내기

```bash
curl https://abcdefghij.lambda-url.ap-northeast-1.on.aws/
```

예상 출력:
```json
{"status": "ok", "message": "hello from aws"}
```

### curl로 POST 요청 보내기

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "Mina"}' \
  https://abcdefghij.lambda-url.ap-northeast-1.on.aws/
```

### 응답 상태 코드 확인

```bash
curl -o /dev/null -s -w "%{http_code}\n" https://abcdefghij.lambda-url.ap-northeast-1.on.aws/
```

출력:
```
200
```

200이 나오면 정상. 그 외 코드의 의미:

| 코드 | 의미 |
|------|------|
| 200 | 정상 응답 |
| 403 | 권한 없음 (Auth 설정 확인) |
| 500 | 함수 내부 오류 |
| 502 | Lambda 응답 형식 오류 |
| 504 | 타임아웃 (30초 초과) |

### curl이 없는 경우 (Windows)

PowerShell에서:
```powershell
Invoke-WebRequest -Uri "https://abcdefghij.lambda-url.ap-northeast-1.on.aws/" -Method GET
```

또는 브라우저 주소창에 URL을 직접 입력하면 GET 요청을 보낼 수 있다.

### 체크포인트 2

- [ ] Function URL이 존재하는가?
- [ ] `curl` 요청에 200 응답이 왔는가?
- [ ] 응답 body에 기대한 JSON이 포함되어 있는가?

---

## 검증 방법 3: CloudWatch Logs에서 로그 확인

Lambda는 실행할 때마다 자동으로 CloudWatch Logs에 로그를 남긴다. 오류가 발생했을 때 원인을 찾는 가장 중요한 방법이다.

### 접근 방법 1: Lambda 콘솔에서 바로 이동

1. Lambda 함수 페이지 → **Monitor** 탭
2. **View CloudWatch logs** 버튼 클릭

### 접근 방법 2: CloudWatch 콘솔에서 직접 찾기

1. AWS 콘솔 → CloudWatch → Logs → Log groups
2. `/aws/lambda/hello-world-function` 이름의 그룹 클릭
3. 최신 Log stream 클릭 (날짜/시간 기준)

### 로그 구조 이해하기

```
START RequestId: abc-123-def-456 Version: $LATEST
상태 체크 요청 받음
END RequestId: abc-123-def-456
REPORT RequestId: abc-123-def-456 Duration: 1.23 ms Billed Duration: 2 ms Memory Size: 128 MB Max Memory Used: 37 MB
```

각 라인 설명:
- `START` — 함수 실행 시작. RequestId는 요청마다 고유한 ID
- 중간 라인들 — `print()` 로 출력한 내용들이 여기 나타남
- `END` — 함수 실행 완료
- `REPORT` — 실행 시간, 메모리 사용량 요약

### 오류 로그 예시

```
START RequestId: xyz-789 Version: $LATEST
[ERROR] NameError: name 'undefined_variable' is not defined
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 5, in lambda_handler
    return undefined_variable
NameError: name 'undefined_variable' is not defined
END RequestId: xyz-789
REPORT RequestId: xyz-789 Duration: 0.89 ms Billed Duration: 1 ms Memory Size: 128 MB Max Memory Used: 36 MB
```

`[ERROR]`로 시작하는 라인과 Traceback이 오류의 핵심이다.

### 로그에 내 메시지 추가하기

함수 코드에 `print()`를 추가하면 CloudWatch에 기록된다.

```python
import json

def lambda_handler(event, context):
    print("함수 실행 시작")
    print("받은 이벤트:", json.dumps(event))

    result = {
        "status": "ok",
        "message": "hello from aws"
    }

    print("응답 준비 완료:", json.dumps(result))
    return {
        "statusCode": 200,
        "body": json.dumps(result)
    }
```

### 체크포인트 3

- [ ] CloudWatch Log group을 찾아서 열었는가?
- [ ] 최신 실행의 Log stream을 확인했는가?
- [ ] START, END, REPORT 라인이 보이는가?
- [ ] 오류가 있다면 `[ERROR]` 라인을 찾았는가?

---

## 검증 방법 4: CloudWatch Metrics에서 지표 확인

Metrics는 함수가 시간이 지남에 따라 어떻게 동작하는지 숫자로 보여준다. 단발성 확인이 아니라 경향(trend)을 보는 데 유용하다.

### 접근 방법

1. Lambda 함수 페이지 → **Monitor** 탭
2. 여러 그래프가 자동으로 표시된다

### 주요 지표 3가지

#### Invocations (호출 횟수)
함수가 몇 번 실행됐는지를 보여준다.

```
정상: 테스트할 때마다 숫자가 1씩 늘어난다
이상: 아예 0이라면 함수가 호출 자체가 안 되고 있다
```

#### Errors (오류 횟수)
오류가 발생한 횟수를 보여준다.

```
정상: 0 (오류 없음)
이상: 빨간색 막대가 보인다 = 오류 발생
```

Error rate(오류율) 계산:
```
오류율 = (Errors / Invocations) × 100%
예: 10번 호출 중 2번 오류 = 20% 오류율
```

#### Duration (실행 시간)
함수 실행에 걸린 시간(밀리초).

```
정상: 수 ms ~ 수백 ms (함수 복잡도에 따라 다름)
이상: 타임아웃 설정(기본 3초)에 가깝거나 초과
```

### Lambda 기본 설정 확인

**Configuration 탭 → General configuration:**

| 설정 항목 | 기본값 | 의미 |
|-----------|--------|------|
| Memory | 128 MB | 함수에 할당된 메모리 |
| Timeout | 3초 | 이 시간 초과 시 오류 발생 |
| Ephemeral storage | 512 MB | /tmp 폴더 용량 |

Duration이 Timeout에 자주 근접한다면 Timeout 값을 늘리거나 코드를 최적화해야 한다.

### 체크포인트 4

- [ ] Monitor 탭에서 Invocations 그래프를 확인했는가?
- [ ] Errors가 0인가?
- [ ] Duration이 Timeout 설정보다 충분히 짧은가?

---

## 검증 방법 5: AI에게 오류 로그 분석 요청하기

오류 메시지를 처음 보면 무슨 뜻인지 모를 수 있다. AI(ChatGPT, Claude 등)에게 붙여넣으면 빠르게 설명을 들을 수 있다.

### 효과적인 질문 방법

**나쁜 예시:**
```
오류가 났어요. 어떻게 해야 하나요?
```

**좋은 예시:**
```
AWS Lambda 함수에서 아래 오류가 발생했습니다.
Python으로 작성된 함수이고, JSON을 반환하는 간단한 함수입니다.
오류 원인과 수정 방법을 알려주세요.

--- 오류 로그 ---
[ERROR] NameError: name 'undefined_variable' is not defined
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 5, in lambda_handler
    return undefined_variable
NameError: name 'undefined_variable' is not defined
```

### 자주 묻는 오류 패턴

**패턴 1: NameError**
```
[ERROR] NameError: name 'xxx' is not defined
```
의미: 정의되지 않은 변수를 사용했다.
질문 예시: "NameError: name 'xxx' is not defined 가 무슨 뜻인가요?"

**패턴 2: ImportError**
```
[ERROR] Runtime.ImportModuleError: Unable to import module 'lambda_function': No module named 'requests'
```
의미: 필요한 라이브러리가 Lambda에 없다.
질문 예시: "Lambda에서 requests 라이브러리를 사용하려면 어떻게 해야 하나요?"

**패턴 3: JSONDecodeError**
```
[ERROR] JSONDecodeError: Expecting value: line 1 column 1 (char 0)
```
의미: JSON 형식이 아닌 문자열을 JSON으로 파싱하려 했다.

### 체크포인트 5

- [ ] 오류가 발생했을 때 로그를 복사해서 AI에게 질문할 수 있는가?
- [ ] AI의 설명을 바탕으로 코드를 수정했는가?

---

## 실습 시나리오: 오류 코드 배포 → 탐지 → 수정 → 재확인

실제로 오류를 만들고, 찾고, 고치는 과정을 직접 경험해 본다.

### 1단계: 오류 있는 코드로 업데이트

Lambda 콘솔 → 함수 → Code 탭에서 아래 코드로 교체 후 **Deploy** 클릭:

```python
import json

def lambda_handler(event, context):
    # 의도적인 오류: 정의되지 않은 변수 사용
    result = {
        "status": undefined_variable,   # ← 여기서 오류 발생
        "message": "hello from aws"
    }
    return {
        "statusCode": 200,
        "body": json.dumps(result)
    }
```

### 2단계: 오류 탐지

Test 이벤트를 실행한다.

예상 결과 (오류 응답):
```json
{
  "errorMessage": "name 'undefined_variable' is not defined",
  "errorType": "NameError",
  "stackTrace": [
    "  File \"/var/task/lambda_function.py\", line 4, in lambda_handler\n    \"status\": undefined_variable,\n"
  ]
}
```

콘솔에서 빨간색 배경이 나타나고 `errorType: NameError` 가 표시된다.

### 3단계: CloudWatch Logs에서 로그 확인

Monitor 탭 → View CloudWatch logs → 최신 스트림 열기

```
START RequestId: ...
[ERROR] NameError: name 'undefined_variable' is not defined
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 4, in lambda_handler
    "status": undefined_variable,
NameError: name 'undefined_variable' is not defined
END RequestId: ...
```

오류 위치: `lambda_function.py` 4번째 줄

### 4단계: CloudWatch Metrics 확인

Monitor 탭에서 Errors 그래프에 빨간 막대가 생긴 것을 확인한다.

### 5단계: 코드 수정

Code 탭으로 돌아가서 오류를 수정한다:

```python
import json

def lambda_handler(event, context):
    # 수정: 실제 문자열 값으로 교체
    result = {
        "status": "ok",              # ← 수정됨
        "message": "hello from aws"
    }
    return {
        "statusCode": 200,
        "body": json.dumps(result)
    }
```

**Deploy** 버튼 클릭 → "Successfully updated the function" 메시지 확인

### 6단계: 재확인

Test 이벤트 재실행:

```json
{
  "statusCode": 200,
  "body": "{\"status\": \"ok\", \"message\": \"hello from aws\"}"
}
```

초록색 배경과 함께 정상 응답이 나온다.

### 시나리오 체크포인트

- [ ] 오류 코드를 배포하고 Test에서 오류 응답을 확인했는가?
- [ ] CloudWatch Logs에서 오류 메시지를 찾았는가?
- [ ] Metrics에서 Errors 그래프가 증가했는가?
- [ ] 코드를 수정하고 재배포 후 정상 응답을 확인했는가?

---

## 배포 후 체크리스트 (10개 항목)

배포할 때마다 이 목록을 확인하는 습관을 만든다.

1. [ ] Test 이벤트가 `statusCode: 200`을 반환하는가?
2. [ ] 응답 body에 기대한 JSON 구조가 있는가?
3. [ ] CloudWatch Logs에 `[ERROR]` 라인이 없는가?
4. [ ] Function URL (있다면) curl 테스트에서 200 응답인가?
5. [ ] Metrics에서 Errors가 0인가?
6. [ ] Duration이 Timeout 설정의 절반 이하인가?
7. [ ] 환경변수(Environment variables)가 올바르게 설정되어 있는가?
8. [ ] Memory 사용량이 설정값의 80% 이하인가?
9. [ ] 로그에서 함수가 예상대로 실행됐음을 나타내는 print 메시지가 있는가?
10. [ ] 이전 배포 대비 Duration이 크게 늘지 않았는가?

---

## 자주 겪는 오류와 해결 방법

### 오류 1: 403 Forbidden

```bash
curl https://abcdef.lambda-url.ap-northeast-1.on.aws/
# 응답: {"message":"Forbidden"}
```

원인: Function URL의 Auth type이 `AWS_IAM`으로 설정되어 있어서 인증 없이는 호출 불가

해결:
1. Lambda 함수 → Configuration → Function URL → Edit
2. Auth type을 `NONE`으로 변경 (학습 목적)
3. 또는 AWS 서명이 포함된 요청을 보내야 함

---

### 오류 2: 502 Bad Gateway

```json
{"message": "Internal server error"}
```

원인: Lambda가 응답을 반환했지만 형식이 잘못됨. Function URL은 특정 형식을 요구한다.

올바른 응답 형식:
```python
return {
    "statusCode": 200,                   # 필수
    "headers": {                         # 선택 (보통 필요)
        "Content-Type": "application/json"
    },
    "body": json.dumps({"status": "ok"}) # 문자열이어야 함 (dict 아님)
}
```

흔한 실수:
```python
# 잘못된 예 - body가 dict
return {
    "statusCode": 200,
    "body": {"status": "ok"}   # ← 이렇게 하면 502 오류
}

# 올바른 예 - body는 문자열
return {
    "statusCode": 200,
    "body": json.dumps({"status": "ok"})  # ← json.dumps() 필요
}
```

---

### 오류 3: Task timed out

```
START RequestId: ...
END RequestId: ...
REPORT RequestId: ... Duration: 3000.12 ms ... Max Memory Used: 45 MB
[ERROR] Task timed out after 3.00 seconds
```

원인: 함수 실행이 Timeout 설정(기본 3초)을 초과

해결:
1. Lambda → Configuration → General configuration → Edit
2. Timeout을 늘린다 (예: 10초, 최대 15분)
3. 또는 코드 안에서 오래 걸리는 부분을 최적화

일반적인 원인:
- 외부 API 호출이 응답하지 않음
- 큰 데이터를 처리하는 루프
- 데이터베이스 쿼리가 너무 느림

---

### 오류 4: Runtime.ImportModuleError

```
[ERROR] Runtime.ImportModuleError: Unable to import module 'lambda_function': No module named 'requests'
```

원인: `requests` 같은 외부 라이브러리를 설치하지 않고 코드에서 사용

해결 방법 (두 가지 중 하나):

방법 A - Lambda Layer 사용:
1. 로컬에서 패키지 다운로드: `pip install requests -t python/`
2. `python/` 폴더를 zip으로 압축
3. Lambda → Layers → Create layer → zip 업로드
4. 함수에 Layer 추가

방법 B - 패키지를 코드와 함께 업로드:
1. `lambda_function.py`와 같은 폴더에 패키지 설치: `pip install requests -t .`
2. 전체 폴더를 zip으로 묶어서 Upload from .zip file

---

## 이해 확인 질문

1. Test 이벤트를 직접 만드는 이유는 무엇인가?
2. CloudWatch Logs에서 `RequestId`가 하는 역할은?
3. Duration이 Timeout에 가까워지면 어떻게 해야 하는가?
4. `body`에 `json.dumps()`를 사용하는 이유는 무엇인가?
5. 오류가 났을 때 가장 먼저 확인해야 할 곳은 어디인가?
