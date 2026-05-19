# Chapter 01: 배포란 무엇인가 — AWS Lambda로 첫 배포하기

## 이 장에서 배우는 것

- "배포"가 왜 필요한지, 내 컴퓨터 실행과 무엇이 다른지
- AWS Lambda가 초보자에게 왜 좋은 첫 번째 선택인지
- AWS 콘솔에서 Lambda 함수를 직접 만들고 실행하는 방법
- 배포가 성공했는지 CloudWatch 로그와 curl로 확인하는 방법
- 배포 성공 기준 5가지를 직접 체크하는 방법

---

## 왜 필요한가 — 배포를 배워야 하는 이유

### "내 컴퓨터에서는 되는데요"

개발을 처음 배우면 이런 상황을 자주 겪는다.

- 내 컴퓨터에서는 프로그램이 잘 돌아간다
- 친구 컴퓨터에서 실행해 보려고 파일을 보냈더니 안 된다
- "Python이 없다", "라이브러리가 없다", "버전이 다르다" 같은 오류가 뜬다

이 문제의 근본 원인은 **내 컴퓨터에만 실행 환경이 갖춰져 있기 때문**이다.

배포는 이 문제를 해결한다. 프로그램을 **인터넷 어딘가의 서버**에 올려두면, 누구든 그 서버에 요청을 보내서 결과를 받아볼 수 있다. 내 컴퓨터가 꺼져 있어도, 상대방이 Python을 설치하지 않아도 된다.

### 카페 주문으로 이해하는 배포

배포를 이해하는 가장 쉬운 비유는 카페다.

**내 컴퓨터에서 실행하는 것** = 집에서 직접 커피를 내리는 것
- 내가 마실 수는 있다
- 다른 사람이 먹으려면 우리 집에 직접 와야 한다
- 내가 없으면 아무도 커피를 못 마신다

**배포한 것** = 카페를 차린 것
- 주소(URL)만 알면 누구든 주문할 수 있다
- 내가 자리에 없어도 카페(서버)가 주문을 받는다
- 주문이 들어오면 카페(서버)가 커피(응답)를 내준다

Lambda는 이 비유에서 **아주 작고 효율적인 카페 주방**에 해당한다. 주문(요청)이 들어올 때만 주방이 돌아가고, 주문이 없으면 멈춰 있어서 전기세(비용)가 거의 나오지 않는다.

### 로컬 실행 vs 배포 실행 — 실질적인 차이

| 항목 | 내 컴퓨터에서 실행 | 배포 후 실행 |
|---|---|---|
| 실행 주소 | 없음 (직접 `python app.py` 실행) | `https://xxxx.lambda-url.ap-northeast-1.on.aws/` |
| 접근 가능 대상 | 나만 (내 컴퓨터 앞에 있을 때만) | URL을 아는 누구나, 언제든 |
| 내 컴퓨터가 꺼지면 | 프로그램도 종료됨 | 서버는 계속 동작 |
| 환경 설정 필요 | Python, 라이브러리 설치 필요 | AWS가 환경을 대신 관리 |
| 비용 | 없음 | Lambda 무료 티어: 월 100만 건까지 무료 |

---

## Lambda가 초보자에게 좋은 3가지 이유

### 이유 1: 서버를 직접 관리할 필요가 없다

일반적인 서버를 운영하려면 이런 것들을 알아야 한다.

- EC2 인스턴스 생성 및 보안 그룹 설정
- SSH로 서버에 접속하는 방법
- Nginx, Gunicorn 같은 웹 서버 설정
- 서버가 다운됐을 때 재시작 방법

Lambda는 이 모든 것을 AWS가 대신 처리한다. 나는 **코드만 붙여 넣으면** 된다.

### 이유 2: 비용이 거의 0이다

Lambda는 함수가 실행될 때만 비용이 발생한다. 무료 티어 기준으로:

- 월 **1,000,000건** 요청까지 무료
- 월 **400,000 GB-초** 컴퓨팅 시간까지 무료

학습 중에 하루에 수백 번 테스트해도 비용이 0원이다.

### 이유 3: 설정이 최소화되어 있다

아래 3단계면 첫 Lambda 함수가 만들어진다.

1. AWS 콘솔에서 함수 생성 (클릭 5번)
2. 코드 붙여 넣기
3. Deploy 버튼 클릭

이 과정에서 서버, 네트워크, 운영체제를 전혀 신경 쓰지 않아도 된다.

---

## 핵심 개념 3가지

### 1. 배포 (Deployment)

내 컴퓨터 밖의 환경(AWS 서버)에서 프로그램이 실행될 수 있도록 코드를 올리는 행위다.

배포가 완료된다고 해서 끝이 아니다. 배포 후에는 반드시 **실제로 동작하는지 확인**해야 한다.

### 2. 검증 (Validation)

배포된 프로그램이 의도한 대로 동작하는지 확인하는 과정이다. 검증 없는 배포는 "올린 것 같다"에 그친다. 검증을 해야 "올렸고, 잘 동작한다"고 말할 수 있다.

검증에서 확인할 것:
- 요청을 보냈을 때 응답이 오는가
- 응답에 기대한 값이 들어 있는가
- 문제가 생겼을 때 로그를 볼 수 있는가

### 3. 되돌리기 (Rollback)

배포 후 문제가 생겼을 때 이전 상태로 복구하는 방법이다. Lambda에서는 이전 버전의 코드를 다시 붙여 넣고 Deploy하면 된다.

---

## 오늘 사용할 시나리오

이 장 전체에서 아래 한 가지 Python 함수를 기준으로 실습한다.

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"status": "ok", "message": "hello from aws"})
    }
```

이 코드에서 우리가 확인할 것은 세 가지뿐이다.

- 요청을 보냈을 때 `{"status": "ok", "message": "hello from aws"}` 응답이 오는가
- 응답이 오지 않으면 CloudWatch 로그에서 원인을 찾을 수 있는가
- 코드를 수정한 뒤 Deploy하면 새 코드가 즉시 반영되는가

---

## 실습 1: AWS 콘솔에서 Lambda 함수 만들고 테스트 실행하기

### 준비물

- AWS 계정 (없으면 aws.amazon.com에서 무료 계정 생성)
- 브라우저 (Chrome, Firefox, Edge 모두 가능)

### 단계 1: Lambda 서비스로 이동

1. 브라우저에서 `https://console.aws.amazon.com` 접속
2. 로그인 후 상단 검색창에 `Lambda` 입력
3. 검색 결과에서 **Lambda** 클릭

> **막히는 지점:** 처음 로그인하면 리전(지역)이 `us-east-1 (버지니아 북부)`로 설정되어 있을 수 있다. 오른쪽 상단에서 `아시아 태평양 (서울) ap-northeast-2`로 변경하는 것을 권장한다. 리전은 서버의 물리적 위치다. 한국에서 사용하면 서울이 가장 빠르다.

### 단계 2: 함수 생성

1. Lambda 메인 화면에서 주황색 **함수 생성** 버튼 클릭
2. 다음 화면에서 아래처럼 설정한다

| 항목 | 선택값 |
|---|---|
| 함수 생성 방법 | **새로 작성 (Author from scratch)** |
| 함수 이름 | `my-first-lambda` |
| 런타임 | **Python 3.12** |
| 아키텍처 | x86_64 (기본값 유지) |

3. 나머지는 기본값 그대로 두고, 화면 아래쪽 주황색 **함수 생성** 버튼 클릭
4. "함수를 성공적으로 생성했습니다"라는 초록색 알림이 뜨면 성공

> **막히는 지점:** "역할이 없다" 또는 "권한이 부족하다"는 오류가 뜨는 경우가 있다. 이는 AWS 계정의 IAM 권한 설정 문제다. 루트 계정으로 로그인했다면 이 오류는 나타나지 않는다. IAM 사용자로 로그인 중이라면 관리자에게 `AWSLambdaFullAccess` 권한 부여를 요청해야 한다.

### 단계 3: 코드 입력

1. 함수 상세 화면이 열리면 아래쪽 **코드** 탭을 클릭
2. 화면 가운데 코드 편집기(lambda_function.py 파일)가 보인다
3. 기존 코드를 모두 지우고, 아래 코드를 그대로 붙여 넣는다

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"status": "ok", "message": "hello from aws"})
    }
```

4. 오른쪽 상단 주황색 **Deploy** 버튼 클릭
5. "함수 my-first-lambda을(를) 성공적으로 업데이트했습니다" 알림이 뜨면 성공

> **막히는 지점:** Deploy 버튼을 누르지 않으면 코드가 저장되지 않는다. 코드를 수정한 뒤에는 반드시 Deploy를 눌러야 한다. Deploy 없이 테스트를 실행하면 이전 코드가 실행된다.

### 단계 4: 콘솔에서 테스트 실행

1. 코드 탭 오른쪽 상단 **Test** 버튼 클릭
2. 처음에는 "테스트 이벤트 구성" 팝업이 뜬다
3. 아래처럼 설정한다

| 항목 | 값 |
|---|---|
| 이벤트 이름 | `MyFirstTest` |
| 이벤트 JSON | `{}` (빈 중괄호, 기본값 유지) |

4. **저장** 버튼 클릭
5. 다시 **Test** 버튼 클릭

### 단계 5: 실행 결과 확인

테스트를 실행하면 코드 편집기 아래에 실행 결과가 표시된다.

성공 시 화면:
```
Status: Succeeded

Response:
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "application/json"
  },
  "body": "{\"status\": \"ok\", \"message\": \"hello from aws\"}"
}
```

"Succeeded"라고 나오면 Lambda 함수가 정상적으로 실행된 것이다.

> **막히는 지점:** Response에서 `body` 값이 문자열로 표시된다 (`"{\"status\": ...}"`). 이것이 맞다. Lambda의 HTTP 응답에서 `body`는 JSON 문자열이어야 한다. 딕셔너리로 반환하면 502 오류가 발생한다.

---

## 실습 2: CloudWatch Logs에서 로그 확인하기

배포된 함수가 실행될 때마다 AWS는 자동으로 실행 기록을 CloudWatch Logs에 저장한다. 오류가 발생했을 때 원인을 찾는 핵심 도구다.

### 로그를 보는 방법 1: Lambda 콘솔에서 바로 이동

1. Lambda 함수 상세 화면에서 **모니터링** 탭 클릭
2. 화면 오른쪽 위 **CloudWatch Logs에서 보기** 버튼 클릭
3. 로그 그룹 화면이 열린다

### 로그를 보는 방법 2: CloudWatch 콘솔에서 직접 이동

1. 상단 검색창에 `CloudWatch` 입력 후 클릭
2. 왼쪽 메뉴 **로그** → **로그 그룹** 클릭
3. `/aws/lambda/my-first-lambda` 로그 그룹 클릭
4. 최신 로그 스트림(맨 위 항목) 클릭

### 로그 내용 읽기

로그 스트림을 열면 이런 내용이 보인다.

```
START RequestId: abc-123 Version: $LATEST
END RequestId: abc-123
REPORT RequestId: abc-123	Duration: 1.23 ms	Billed Duration: 2 ms	Memory Size: 128 MB	Max Memory Used: 37 MB
```

각 줄의 의미:

| 줄 | 의미 |
|---|---|
| `START` | 함수 실행 시작 |
| `END` | 함수 실행 종료 |
| `REPORT` | 실행 시간, 메모리 사용량 요약 |
| `Duration: 1.23 ms` | 실제 실행 시간 (짧을수록 좋음) |
| `Billed Duration: 2 ms` | 요금 계산 기준 시간 |
| `Max Memory Used: 37 MB` | 실제 사용된 메모리 |

### 오류 로그는 어떻게 생겼나

만약 코드에 오류가 있다면 로그에 아래처럼 빨간색 텍스트가 찍힌다.

```
[ERROR] NameError: name 'json' is not defined
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 5, in lambda_handler
    "body": json.dumps({"status": "ok"})
NameError: name 'json' is not defined
```

이 로그를 보면 "5번째 줄에서 `json`을 import하지 않았다"는 것을 바로 알 수 있다.

### 직접 해보기: 의도적으로 오류를 만들고 로그 확인

1. Lambda 코드에서 첫 줄 `import json`을 지운다
2. Deploy 클릭
3. Test 실행
4. "Function Invocation Error" 또는 "Failed"가 나타난다
5. CloudWatch Logs로 이동해서 어떤 오류 메시지가 찍혔는지 확인한다
6. 다시 `import json`을 추가하고 Deploy 후 Test — 성공으로 돌아오는 것을 확인

> **이 연습이 중요한 이유:** 실제 개발에서 오류는 반드시 발생한다. 오류가 났을 때 당황하지 않고 CloudWatch Logs를 열어 원인을 찾는 습관을 지금 들여야 한다.

---

## 실습 3: 배포 체크리스트 — 성공 기준 5가지 직접 확인

배포가 끝났다고 생각될 때, 아래 5가지 기준을 직접 확인한다. 모두 통과하면 진짜 성공이다.

### 성공 기준 1: 함수가 오류 없이 실행된다

Lambda 콘솔 Test 탭에서 실행 결과가 **"Succeeded"** 로 표시되는지 확인한다.

확인 방법:
1. Lambda 함수 → 코드 탭 → Test 버튼 클릭
2. 실행 결과에 "Succeeded" 문구가 보이면 통과

### 성공 기준 2: 응답에 기대한 값이 있다

응답의 `body` 안에 `"status": "ok"`와 `"message": "hello from aws"`가 있는지 확인한다.

확인 방법:
1. Test 실행 후 Response 영역 확인
2. `body` 값을 찾아 `status`와 `message`가 올바른지 확인

### 성공 기준 3: 같은 결과가 반복해서 나온다

한 번 성공했다고 끝이 아니다. Test를 3번 더 눌러 매번 같은 응답이 나오는지 확인한다.

이것이 중요한 이유: 간헐적 오류(네트워크 문제, 타임아웃 등)를 잡기 위해서다.

### 성공 기준 4: CloudWatch 로그에 오류가 없다

CloudWatch Logs에서 방금 실행된 로그 스트림을 열어 빨간색 ERROR 텍스트가 없는지 확인한다.

확인 방법:
1. Lambda → 모니터링 탭 → CloudWatch Logs에서 보기 클릭
2. 최신 로그 스트림 클릭
3. START, END, REPORT 줄만 보이고 ERROR 줄이 없으면 통과

### 성공 기준 5: 성공 기준을 한 줄로 설명할 수 있다

아래 빈칸을 직접 채워본다. 채울 수 있다면 이 장을 완전히 이해한 것이다.

```text
이 Lambda 함수의 성공 기준:
"[확인 방법]으로 호출했을 때 [기대하는 응답]이 오면 성공이다."

내가 쓴 기준:
_________________________________________________
```

예시 답:
```text
"Lambda 콘솔 Test로 실행했을 때 status가 ok이고 message가 hello from aws인 JSON 응답이 오면 성공이다."
```

---

## 초보자가 자주 막히는 지점 모음

### 막히는 지점 1: Deploy 버튼을 눌렀는데 변경이 반영 안 됨

원인: Deploy 버튼을 눌렀지만 "Successfully updated" 알림을 확인하지 않은 경우.

해결: 코드 수정 → Deploy 클릭 → 초록색 "함수를 성공적으로 업데이트했습니다" 알림 확인 → Test 실행.

### 막히는 지점 2: 로그가 CloudWatch에 안 보임

원인: Lambda 함수에 CloudWatch Logs 쓰기 권한이 없는 경우.

해결: Lambda → 구성 탭 → 권한 탭에서 실행 역할(Execution role)을 클릭. IAM 역할에 `AWSLambdaBasicExecutionRole` 정책이 붙어 있는지 확인한다. 없으면 해당 정책을 추가한다.

### 막히는 지점 3: Response에서 `body`가 이상하게 나옴

원인: `body`를 딕셔너리로 반환했을 때.

잘못된 코드:
```python
return {
    "statusCode": 200,
    "body": {"status": "ok"}  # 딕셔너리를 그대로 반환
}
```

올바른 코드:
```python
import json

return {
    "statusCode": 200,
    "body": json.dumps({"status": "ok"})  # 문자열로 변환
}
```

### 막히는 지점 4: 함수는 성공인데 실제 브라우저에서 안 열림

원인: Lambda 함수는 만들었지만 Function URL을 아직 생성하지 않았기 때문. Lambda 함수 자체는 콘솔에서만 테스트할 수 있고, 외부에서 HTTP로 호출하려면 Function URL 또는 API Gateway가 필요하다.

해결: Chapter 18에서 Function URL 설정을 배운다.

---

## 확인 체크리스트

- [ ] AWS 콘솔에서 Lambda 함수를 생성했다
- [ ] `import json`과 `lambda_handler` 함수를 코드에 붙여 넣고 Deploy를 눌렀다
- [ ] Test 실행 결과가 "Succeeded"로 표시됐다
- [ ] Response에서 `"status": "ok"`와 `"message": "hello from aws"`를 확인했다
- [ ] CloudWatch Logs에서 이번 실행의 로그 스트림을 열어봤다
- [ ] 의도적으로 오류를 만들고, 오류 로그가 CloudWatch에 어떻게 찍히는지 확인했다
- [ ] 배포 성공 기준을 한 줄로 직접 써봤다

---

## 한 번 더 생각해 보기

1. Lambda 함수에서 Deploy를 누르기 전과 후의 차이는 무엇일까? Deploy를 누르지 않으면 어떤 일이 생길까?

2. CloudWatch Logs가 없다면 배포 후 오류가 났을 때 어떻게 원인을 찾을 수 있을까?

3. "성공"의 기준을 미리 정해두는 것이 왜 중요할까? 기준 없이 배포하면 어떤 문제가 생길까?

---

## 다음 장

다음 장(Chapter 02)에서는 Lambda 함수에 환경변수를 설정하는 방법과, 환경에 따라 다른 동작을 하는 코드를 작성하는 방법을 배운다.
