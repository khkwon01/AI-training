# Chapter 18: Lambda Function URL — 가장 빠른 첫 번째 배포 방법

## 이 장에서 배우는 것

- Lambda Function URL이 무엇인지, API Gateway와 어떻게 다른지 이해한다
- 초보자가 Function URL을 먼저 배워야 하는 3가지 이유
- AWS 콘솔에서 Function URL을 단계별로 만든다
- CORS 설정을 포함해 브라우저에서 호출할 수 있도록 구성한다
- Python Lambda 핸들러를 작성하고 `curl`로 직접 테스트한다
- 자주 겪는 오류(403, CORS, timeout)를 스스로 해결한다
- API Gateway로 넘어가야 할 시점을 판단한다

---

## 왜 필요한가 — Function URL을 먼저 배워야 하는 이유

Lambda 함수를 만들면 가장 먼저 드는 의문이 있다.

> "이 함수를 어떻게 외부에서 호출하지? 브라우저나 앱에서 부르려면 URL이 있어야 하는데."

Lambda 함수 자체는 AWS 내부에서만 실행된다. 외부에서 HTTP 요청으로 호출하려면 HTTP 진입점이 필요하다. 가장 널리 알려진 방법은 **API Gateway**이지만, 처음 보는 사람에게는 설정이 복잡하다. 리소스 생성, 메서드 정의, 통합 설정, 스테이지 배포 — 단계가 많고 용어가 낯설다.

**Lambda Function URL**은 그 복잡함을 제거한 대안이다. Lambda 함수에 고유한 HTTPS 주소를 즉시 붙여주는 기능으로, 설정은 클릭 몇 번으로 끝난다.

### 초보자에게 Function URL이 먼저인 3가지 이유

**이유 1: 설정이 3분 안에 끝난다**

API Gateway는 처음 설정하는 데 15~30분이 걸리고, 잘못 설정하면 403이나 502 오류가 자주 발생한다. Function URL은 Lambda 함수 화면에서 버튼 하나로 활성화된다. 배포의 본질인 "코드를 외부에서 호출하는 경험"에 집중할 수 있다.

**이유 2: 추가 비용이 없다**

API Gateway는 요청 수에 따라 별도 요금이 발생한다. Function URL은 Lambda 실행 비용만 발생한다. 무료 티어(월 100만 건) 안에서 학습하면 비용이 0원이다.

**이유 3: 단일 엔드포인트 서비스에 충분하다**

웹훅 수신, 간단한 API, Slack 봇 연동, 폼 데이터 처리 — 경로가 하나인 서비스라면 Function URL만으로 충분하다. 학습 단계에서 필요하지 않은 복잡성을 미리 추가할 필요가 없다.

---

## 1. Function URL vs API Gateway 비교

| 항목 | Function URL | API Gateway |
|---|---|---|
| 설정 복잡도 | 매우 낮음 (클릭 3번) | 높음 (리소스/메서드/스테이지 설정 필요) |
| 추가 비용 | 없음 (Lambda 호출 비용만) | 별도 요금 발생 (요청 100만 건당 약 $3.50) |
| 커스텀 도메인 | 지원 (Route 53 연결 가능) | 지원 |
| 경로 라우팅 (`/users`, `/orders`) | 불가능 (단일 엔드포인트) | 가능 |
| 인증·권한 제어 | IAM 또는 없음 (NONE) | IAM, Cognito, Lambda Authorizer |
| CORS 설정 | 콘솔에서 간단히 설정 | 별도 OPTIONS 메서드 설정 필요 |
| WebSocket 지원 | 불가능 | 가능 |
| 사용량 제한 (Rate Limiting) | 불가능 | 가능 |
| 초보자 진입 장벽 | 낮음 | 높음 |
| 권장 사용 시점 | 학습, 단순 웹훅, 1개 엔드포인트 | 실서비스, 다중 경로, 세밀한 권한 |

**결론:** 처음에는 Function URL로 시작한다. 경로가 여러 개 필요하거나, 인증 정책이 복잡해지거나, 사용량 제한이 필요해지면 그때 API Gateway로 넘어간다.

---

## 2. AWS 콘솔에서 Function URL 만들기 (단계별)

### 단계 1: Lambda 함수 생성

1. AWS 콘솔(`console.aws.amazon.com`) 접속 후 상단 검색창에 `Lambda` 입력
2. Lambda 서비스 클릭
3. 오른쪽 상단 주황색 **함수 생성** 버튼 클릭
4. 다음 화면에서 아래처럼 설정

| 항목 | 선택값 |
|---|---|
| 함수 생성 방법 | **새로 작성 (Author from scratch)** |
| 함수 이름 | `hello-function-url` |
| 런타임 | **Python 3.12** |
| 아키텍처 | x86_64 (기본값 유지) |

5. 나머지 기본값 유지 → 주황색 **함수 생성** 버튼 클릭
6. "함수를 성공적으로 생성했습니다" 초록색 알림 확인

> **막히는 지점:** 리전이 어디로 설정되어 있는지 확인한다. 오른쪽 상단에서 `아시아 태평양 (서울) ap-northeast-2`를 선택하는 것을 권장한다. 리전마다 Function URL 주소가 다르게 생성되므로, 나중에 헷갈리지 않도록 처음부터 고정한다.

### 단계 2: 코드 입력 및 배포

1. 함수 상세 화면 → 아래쪽 **코드** 탭 클릭
2. 코드 편집기(`lambda_function.py`)에서 기존 코드를 모두 지운다
3. 아래 코드를 붙여 넣는다

```python
import json

def lambda_handler(event, context):
    # 쿼리 파라미터 추출 (없으면 기본값 사용)
    params = event.get("queryStringParameters") or {}
    name = params.get("name", "세상")

    # HTTP 메서드 확인
    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")

    body = {
        "message": f"안녕하세요, {name}!",
        "method": method,
        "status": "ok"
    }

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps(body, ensure_ascii=False)
    }
```

4. 코드 편집기 오른쪽 상단 주황색 **Deploy** 버튼 클릭
5. "함수 hello-function-url을(를) 성공적으로 업데이트했습니다" 알림 확인

> **막히는 지점:** `ensure_ascii=False`를 빠뜨리면 한글이 `안녕하세요`처럼 유니코드 이스케이프로 출력된다. 한글이 포함된 응답에는 반드시 이 옵션을 추가한다.

### 단계 3: Function URL 활성화

1. 함수 상세 화면 상단 탭에서 **구성** 클릭
2. 왼쪽 메뉴에서 **함수 URL** 클릭
3. 오른쪽 **함수 URL 생성** 버튼 클릭
4. 팝업에서 아래처럼 설정

| 항목 | 선택값 |
|---|---|
| 인증 유형 | **NONE** |
| CORS 구성 | 아래 "단계 4" 참고 |

> **NONE 인증이란?** 이 URL을 아는 누구나 호출할 수 있다는 뜻이다. 학습 환경에서는 NONE으로 시작하되, 실서비스에는 IAM 인증 또는 API Gateway의 인증 레이어를 사용해야 한다.

5. **저장** 클릭
6. 구성 화면에 표시되는 URL 복사

URL 형태 예시:
```
https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/
```

### 단계 4: CORS 설정 (브라우저에서 호출할 때 필요)

브라우저에서 JavaScript로 이 Lambda를 호출하려면 CORS(Cross-Origin Resource Sharing) 설정이 필요하다.

**CORS가 필요한 이유:** 브라우저는 보안상 다른 출처(도메인)의 서버로 요청을 보낼 때 서버가 허용 여부를 응답 헤더로 명시하도록 요구한다. CORS 설정이 없으면 브라우저가 응답을 차단한다. `curl`은 이 제한이 없기 때문에 `curl`로는 되는데 브라우저에서는 안 되는 상황이 생긴다.

CORS 설정 방법:
1. 함수 URL 생성 또는 편집 화면에서 **CORS 구성 추가** 체크박스 활성화
2. 아래처럼 설정

| 항목 | 값 |
|---|---|
| 허용된 출처 | `*` (모든 출처 허용, 학습용) |
| 허용된 메서드 | `GET`, `POST` |
| 허용된 헤더 | `Content-Type` |

3. **저장** 클릭

> **실서비스 주의:** 허용된 출처를 `*`로 설정하면 모든 사이트에서 이 Lambda를 호출할 수 있다. 실서비스에서는 `https://myapp.example.com`처럼 특정 도메인만 허용한다.

---

## 3. Python Lambda 핸들러 코드 설명

```python
import json

def lambda_handler(event, context):
    # 1. 쿼리 파라미터 추출
    # URL에 ?name=지수 처럼 붙으면 params = {"name": "지수"}
    # URL에 파라미터가 없으면 event.get("queryStringParameters")는 None 반환
    # 그래서 "or {}"로 빈 딕셔너리를 기본값으로 설정
    params = event.get("queryStringParameters") or {}
    name = params.get("name", "세상")

    # 2. HTTP 메서드 추출 (GET, POST 등)
    # Function URL 호출 시 event에 requestContext가 포함됨
    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")

    # 3. 응답 바디 구성
    body = {
        "message": f"안녕하세요, {name}!",
        "method": method,
        "status": "ok"
    }

    # 4. HTTP 응답 반환
    # statusCode: HTTP 상태 코드 (200 = 성공)
    # headers: 응답 헤더 (Content-Type을 application/json으로 지정)
    # body: 반드시 문자열이어야 함 (json.dumps로 변환)
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps(body, ensure_ascii=False)
    }
```

**가장 중요한 규칙:** `body`는 반드시 **문자열**이어야 한다. 딕셔너리를 그대로 반환하면 Lambda가 내부적으로 502 오류를 돌려준다. `json.dumps()`로 문자열로 변환한 뒤 반환한다.

---

## 따라 하기 실습

### 실습 1 — 기본 응답 curl로 확인하기

복사한 Function URL로 `curl` 명령을 실행한다.

```bash
# <YOUR_URL> 자리에 복사한 Function URL을 붙여 넣는다
curl "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/"
```

예상 출력:
```json
{"message": "안녕하세요, 세상!", "method": "GET", "status": "ok"}
```

**curl이 없을 때:** Windows에서는 PowerShell의 `Invoke-RestMethod` 또는 `curl.exe`를 사용한다.

```powershell
# Windows PowerShell
Invoke-RestMethod "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/"
```

브라우저에서도 가능하다. Function URL을 브라우저 주소창에 그대로 입력하면 JSON 응답이 표시된다.

> **막히는 지점:** `curl: (6) Could not resolve host` 오류가 나오면 URL을 잘못 복사한 것이다. AWS 콘솔 → 구성 → 함수 URL 에서 URL을 다시 확인한다.

### 실습 2 — 쿼리 파라미터 전달하기

이름을 쿼리 파라미터로 넘겨본다.

```bash
curl "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/?name=지수"
```

예상 출력:
```json
{"message": "안녕하세요, 지수!", "method": "GET", "status": "ok"}
```

이름을 바꿔가며 몇 번 더 테스트해본다.

```bash
curl "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/?name=민준"
curl "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/?name=ChatGPT"
```

### 실습 3 — POST 요청 보내기

JSON 바디를 포함한 POST 요청을 보낸다.

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"greeting": "hello"}' \
  "https://abcdefghij1234567890.lambda-url.ap-northeast-2.on.aws/?name=민준"
```

예상 출력:
```json
{"message": "안녕하세요, 민준!", "method": "POST", "status": "ok"}
```

`"method": "POST"`가 응답에 포함되면 Lambda가 HTTP 메서드를 올바르게 읽은 것이다.

---

## 직접 해보기: POST 바디 읽기 기능 추가

현재 핸들러는 POST 바디를 읽지 않는다. 아래 코드로 바꾸면 POST로 보낸 JSON 바디도 응답에 포함된다.

```python
import json

def lambda_handler(event, context):
    params = event.get("queryStringParameters") or {}
    name = params.get("name", "세상")

    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")

    # POST 바디 읽기
    raw_body = event.get("body") or "{}"
    try:
        request_body = json.loads(raw_body)
    except json.JSONDecodeError:
        request_body = {}

    greeting = request_body.get("greeting", "안녕")

    body = {
        "message": f"{greeting}, {name}!",
        "method": method,
        "status": "ok",
        "received_body": request_body
    }

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(body, ensure_ascii=False)
    }
```

이 코드를 Lambda에 붙여 넣고 Deploy → POST 요청을 보내 결과를 확인한다.

---

## 자주 겪는 문제와 해결 방법

### 문제 1: 403 Forbidden

**증상:**
```bash
$ curl "https://xxxx.lambda-url.ap-northeast-2.on.aws/"
{"message":"Forbidden"}
```

**원인과 해결:**

| 원인 | 확인 방법 | 해결 |
|---|---|---|
| 인증 유형이 IAM으로 설정됨 | 구성 → 함수 URL에서 인증 유형 확인 | NONE으로 변경 후 저장 |
| URL이 잘못됨 | URL에 오타 또는 잘못된 리전 포함 여부 확인 | 콘솔에서 URL 다시 복사 |
| URL 끝 슬래시 누락 | URL 끝에 `/`가 있는지 확인 | URL 끝에 `/` 추가 |

### 문제 2: CORS 오류 (브라우저에서만 발생)

**증상:**
브라우저 개발자 도구(F12) 콘솔에서:
```
Access to fetch at 'https://xxxx.lambda-url...' from origin 'http://localhost:3000' 
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present
```

**원인:** Function URL에 CORS가 설정되지 않았거나, 허용된 출처에 현재 페이지 주소가 포함되지 않음.

**해결:**
1. Lambda → 구성 → 함수 URL → 편집 클릭
2. CORS 구성 활성화
3. 허용된 출처에 `*` 또는 현재 페이지 주소(`http://localhost:3000`) 추가
4. 허용된 메서드: `GET`, `POST` 추가
5. 허용된 헤더: `Content-Type` 추가
6. 저장

> **curl로는 되는데 브라우저에서 안 되는 경우:** 거의 대부분 CORS 문제다. curl은 CORS 제한을 적용하지 않는다.

### 문제 3: 502 Bad Gateway

**증상:**
```json
{"message": "Internal server error"}
```
그리고 HTTP 상태 코드가 502.

**원인과 해결:**

| 원인 | 확인 방법 | 해결 |
|---|---|---|
| `body`를 딕셔너리로 반환 | 코드에서 `body` 값 확인 | `json.dumps(body)`로 문자열 변환 |
| `statusCode` 오타 | 코드에서 키 이름 확인 | `statusCode` (대소문자 정확히) |
| 코드에 Python 문법 오류 | CloudWatch Logs 확인 | 오류 메시지 보고 코드 수정 |

### 문제 4: 타임아웃

**증상:**
```json
{"errorMessage": "Task timed out after 3.00 seconds"}
```

**원인:** Lambda 기본 타임아웃이 3초다. 함수 실행이 3초를 넘으면 강제 종료된다.

**해결:**
1. Lambda → 구성 → 일반 구성 → 편집 클릭
2. 제한 시간(Timeout)을 늘린다 (예: 10초, 30초)
3. 저장

> **주의:** 타임아웃을 무한정 늘리는 것은 좋지 않다. Lambda 최대 타임아웃은 15분이다. 오래 걸리는 작업은 타임아웃을 적절히 설정하고, 실제로 오래 걸리는 이유(느린 외부 API 호출, 무한 루프 등)를 코드에서 해결해야 한다.

---

## 언제 API Gateway로 넘어가야 하는가

아래 기준 중 하나라도 해당되면 API Gateway를 검토할 시점이다.

| 상황 | 이유 |
|---|---|
| 경로가 2개 이상 필요 (`/users`, `/products`) | Function URL은 단일 엔드포인트만 지원 |
| 사용자 인증이 필요 (로그인, 토큰 검증) | Cognito, Lambda Authorizer 연동은 API Gateway에서 가능 |
| API 요청 수를 제한해야 함 (Rate Limiting) | API Gateway의 사용량 계획 기능 필요 |
| 여러 Lambda 함수를 하나의 API로 묶어야 함 | API Gateway가 라우팅 허브 역할을 함 |
| WebSocket이 필요 | Function URL은 WebSocket 미지원 |
| 캐싱이 필요 | API Gateway에서 응답 캐싱 지원 |

**반대로, 아래 상황이라면 Function URL로 충분하다:**

- 웹훅을 하나 받는 서비스 (GitHub 웹훅, Slack 이벤트 등)
- 단순한 API 엔드포인트 하나 (폼 데이터 수신, 간단한 계산)
- 학습 목적의 실험 및 프로토타입
- 내부 도구 또는 개인 프로젝트

---

## 확인 체크리스트

- [ ] `hello-function-url` Lambda 함수를 생성했다
- [ ] `handler.py` 코드를 인라인 편집기에 붙여 넣고 **Deploy**를 눌렀다
- [ ] 함수 URL을 생성하고 HTTPS 주소를 복사했다
- [ ] `curl` 기본 호출로 `{"status": "ok"}` 응답을 받았다
- [ ] `?name=이름` 쿼리 파라미터가 응답 메시지에 반영되는 것을 확인했다
- [ ] POST 요청 시 `"method": "POST"`가 응답에 나타나는 것을 확인했다
- [ ] CORS 설정이 무엇인지, 왜 필요한지 설명할 수 있다
- [ ] Function URL과 API Gateway의 차이를 한 문장으로 설명할 수 있다
- [ ] API Gateway로 넘어가야 할 기준을 2가지 이상 말할 수 있다

---

## 한 번 더 생각해 보기

1. Function URL 인증 유형을 **NONE**으로 설정했다. 이 URL을 다른 사람이 알게 되면 어떤 일이 생길까? 학습 환경이 아닌 실서비스라면 어떻게 해야 할까?

2. 현재 핸들러는 경로(`/users`, `/products`)를 구분하지 못한다. 경로마다 다른 동작이 필요한 서비스를 만들려면 무엇이 부족하고, 어떤 방법으로 해결할 수 있을까?

3. Lambda 기본 타임아웃은 3초다. 외부 API를 호출하는 함수라면 타임아웃을 얼마나 설정하는 것이 적절할까? 타임아웃을 15분으로 설정하면 어떤 문제가 생길까?

---

## 다음 장

다음 장(Chapter 19)에서는 여러 경로와 인증이 필요한 실서비스 수준의 API를 구성하기 위해 **API Gateway와 Lambda를 연동하는 방법**을 배운다.
