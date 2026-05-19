# Chapter 03: API Gateway로 URL 만들기

## 이 장에서 배우는 것

- API Gateway가 무엇인지, Lambda와 어떻게 연결되는지
- HTTP API를 만들고 Lambda에 연결하는 방법
- URL로 Lambda 함수를 호출하는 방법
- 브라우저와 Python 코드에서 API를 테스트하는 방법
- 기본 보안 설정 이해하기

---

## 먼저 쉬운 설명

앞 장에서 Lambda 함수를 만들었다. 그런데 이 함수는 AWS 콘솔에서만 테스트할 수 있다.

외부에서 호출하려면 URL이 필요하다.

**API Gateway**는 Lambda 함수에 URL을 붙여주는 서비스다.

```
사용자/앱 → https://abc123.execute-api.ap-northeast-2.amazonaws.com/hello
                            ↓
                     API Gateway
                            ↓
                     Lambda 함수 실행
                            ↓
                     응답 반환
```

URL이 생기면 브라우저, Python 코드, 모바일 앱 등 어디서든 Lambda를 호출할 수 있다.

---

## 1. Lambda Function URL (가장 빠른 방법)

API Gateway 없이 Lambda에 직접 URL을 붙이는 방법이다. 초보자에게 가장 간단하다.

### 설정 방법

1. Lambda 함수 페이지 → **Configuration** 탭 → **Function URL**
2. **Create function URL** 클릭
3. Auth type: **NONE** 선택 (누구나 접근 가능, 테스트용)
4. **Save** 클릭

URL이 생성된다:
```
https://abc123def456.lambda-url.ap-northeast-2.on.aws/
```

### 브라우저에서 테스트

생성된 URL을 브라우저 주소창에 붙여넣으면 함수 실행 결과가 표시된다.

앞 장 `hello-python` 함수의 경우:
```json
{"statusCode": 200, "body": "안녕하세요, World님!"}
```

---

## 2. API Gateway HTTP API 만들기

더 세밀한 제어가 필요할 때 API Gateway를 사용한다.

### 콘솔에서 만들기

1. AWS 콘솔 검색창에 `API Gateway` 입력 → **API Gateway** 클릭
2. **Create API** 클릭
3. **HTTP API** → **Build** 선택

   (REST API보다 HTTP API가 더 단순하고 저렴하다)

4. **Add integration** → **Lambda** 선택
5. Lambda function 선택: `hello-python`
6. API name: `hello-api`
7. **Next** → **Next** → **Create**

### 엔드포인트 확인

API가 만들어지면 URL이 표시된다:
```
https://abc123.execute-api.ap-northeast-2.amazonaws.com
```

기본적으로 `GET /{proxy+}` 라우트가 설정되어 있어 모든 경로에서 Lambda를 호출한다.

---

## 3. URL로 호출 테스트

### 브라우저에서

```
https://abc123.execute-api.ap-northeast-2.amazonaws.com/hello
```

### curl로 (터미널)

```bash
curl https://abc123.execute-api.ap-northeast-2.amazonaws.com/hello
```

### Python 코드로

```python
import urllib.request
import json

url = "https://abc123.execute-api.ap-northeast-2.amazonaws.com/hello"

with urllib.request.urlopen(url) as response:
    data = json.loads(response.read())
    print(data)
```

또는 `requests` 라이브러리 사용 시:

```bash
pip install requests
```

```python
import requests

url = "https://abc123.execute-api.ap-northeast-2.amazonaws.com/hello"
response = requests.get(url)
print(response.json())
```

---

## 4. 이름을 전달하는 API 만들기

쿼리 파라미터로 이름을 전달하고 Lambda에서 읽는 방법이다.

### Lambda 함수 수정

```python
def lambda_handler(event, context):
    # 쿼리 파라미터에서 name 읽기
    query_params = event.get("queryStringParameters") or {}
    name = query_params.get("name", "World")

    message = f"안녕하세요, {name}님!"
    print(f"실행됨: {message}")

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": message}, ensure_ascii=False)
    }
```

`import json` 을 파일 상단에 추가한다.

**Deploy** 클릭 후 URL로 테스트:

```
https://abc123.execute-api.ap-northeast-2.amazonaws.com/?name=Mina
```

응답:
```json
{"message": "안녕하세요, Mina님!"}
```

---

## 5. 비용과 무료 범위

| 서비스 | 무료 범위 |
|--------|----------|
| Lambda | 월 100만 요청, 400,000 GB-초 |
| API Gateway HTTP API | 월 100만 요청 (12개월) |
| Lambda Function URL | 무료 (Lambda 요금만 적용) |

테스트 목적으로는 무료 범위를 초과하지 않는다.

---

## 6. 따라 하기 실습

### 실습 1. Lambda Function URL 설정

1. `hello-python` Lambda 함수 → Configuration → Function URL
2. Auth type: NONE으로 URL 생성
3. 브라우저에서 URL 접속 → 응답 확인

### 실습 2. 이름 파라미터 추가

Lambda 함수 코드를 위의 수정된 버전으로 교체하고 Deploy.

```
URL?name=내이름
```

으로 테스트해서 개인화된 응답이 오는지 확인.

### 실습 3. Python 코드로 호출

```python
import urllib.request
import json

url = "Lambda Function URL 주소?name=Python학습자"

with urllib.request.urlopen(url) as res:
    data = json.loads(res.read())
    print(data["message"])
```

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| Deploy 안 하고 테스트 | 이전 코드가 실행됨 | 코드 수정 후 반드시 **Deploy** |
| 리전이 달라 URL이 안 보임 | API 목록이 비어있음 | 콘솔 오른쪽 위 리전 확인 |
| CORS 오류 | 브라우저에서 API 호출 실패 | API Gateway에서 CORS 설정 활성화 |
| `queryStringParameters`가 None | `get()` 호출 시 오류 | `or {}` 처리 패턴 사용 |

---

## 확인 체크리스트

- [ ] Lambda Function URL을 만들고 브라우저에서 접속할 수 있는가
- [ ] API Gateway HTTP API를 만들고 Lambda에 연결할 수 있는가
- [ ] 쿼리 파라미터로 값을 전달하는 Lambda 함수를 만들 수 있는가
- [ ] Python 코드로 API URL을 호출할 수 있는가

---

## 한 번 더 생각해 보기

1. Lambda Function URL과 API Gateway의 차이는 무엇인가?
2. Auth type을 NONE으로 설정하면 어떤 보안 위험이 있을까?
3. URL에 이름을 넣어서 개인화된 응답을 만드는 것을 어디에 응용할 수 있을까?

---

## 다음 장

다음 장에서는 Lambda 함수의 로그를 확인하고 문제를 진단하는 방법을 배운다.

---

## 참고 자료

- AWS Lambda Function URL — https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html
- API Gateway HTTP API — https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api.html
