## 이 장에서 배우는 것

- AWS Lambda가 무엇인지, 왜 서버가 없어도 코드를 실행할 수 있는지 이해한다
- Python 함수를 Lambda에 배포하는 기본 흐름을 순서대로 따라 할 수 있다
- IAM 역할(Role)이 왜 필요한지, 어떻게 만드는지 설명할 수 있다
- Lambda 함수를 zip으로 패키징하고 업로드하는 방법을 직접 해본다
- 배포 후 콘솔에서 테스트하고 로그를 확인할 수 있다

---

## 먼저 쉬운 설명

카페를 운영한다고 상상해 보세요. 손님이 올 때만 바리스타가 일하고, 손님이 없으면 바리스타도 없는 카페예요. 월급도 손님이 올 때만 나가죠.

AWS Lambda가 딱 이런 방식입니다. 여러분의 코드는 **요청이 들어올 때만 실행**되고, 아무것도 없을 땐 비용이 0원입니다. EC2처럼 서버를 24시간 켜놓을 필요가 없어요.

처음에는 설정해야 할 것들이 많아 보여서 복잡하게 느껴질 수 있어요. 하지만 이 장에서는 딱 **핵심 세 단계**만 집중합니다.

1. 코드를 준비한다
2. Lambda에 올린다
3. 테스트한다

나머지는 나중에 천천히 배워도 됩니다.

---

## 1. Lambda 함수의 구조 이해하기

Lambda 함수는 특별한 형식을 따릅니다. 반드시 `lambda_handler`라는 이름의 함수가 있어야 하고, `event`와 `context` 두 개의 매개변수를 받아야 합니다.

```python
# hello_lambda.py

def lambda_handler(event, context):
    # event: 함수를 호출할 때 전달되는 데이터 (딕셔너리)
    # context: AWS가 자동으로 넣어주는 실행 환경 정보 (지금은 몰라도 됨)

    name = event.get("name", "세계")
    message = f"안녕하세요, {name}님!"

    return {
        "statusCode": 200,
        "body": message
    }
```

> **포인트:** `event`는 우리가 보내는 데이터, `context`는 AWS가 자동으로 채워줘서 지금 당장은 신경 쓰지 않아도 됩니다.

---

## 2. IAM 역할(Role) 만들기

Lambda는 AWS의 다른 서비스(예: S3, DynamoDB)를 사용할 때 "이 Lambda가 정말 권한이 있는 녀석이야"라는 증명서가 필요합니다. 그게 바로 **IAM 역할**입니다.

처음에는 로그만 쓸 수 있는 기본 역할을 만드는 것으로 충분합니다.

**AWS 콘솔에서 만드는 순서:**

```
IAM 콘솔 → 역할(Roles) → 역할 만들기(Create role)
  → 신뢰할 수 있는 엔터티: AWS 서비스
  → 사용 사례: Lambda
  → 정책 추가: AWSLambdaBasicExecutionRole  ← 이것만 선택
  → 역할 이름: my-lambda-basic-role
  → 역할 만들기(Create role)
```

`AWSLambdaBasicExecutionRole`은 CloudWatch에 로그를 쓸 수 있는 최소 권한입니다. 처음 배울 때는 이것만 있어도 충분합니다.

---

## 3. 코드를 zip으로 패키징하기

Lambda에 코드를 올리는 가장 간단한 방법은 **zip 파일**로 묶는 것입니다.

```bash
# 터미널에서 실행 (hello_lambda.py가 있는 폴더에서)

# 방법 1: 파일 하나만 있을 때
zip function.zip hello_lambda.py

# 방법 2: 여러 파일이 있을 때
zip function.zip hello_lambda.py utils.py

# 방법 3: 폴더 전체를 zip으로 묶을 때
cd my_lambda_project
zip -r ../function.zip .
```

zip 파일 안에 `lambda_function.py`가 있어야 하고, 폴더로 감싸지 않아야 합니다.

**올바른 zip 구조:**
```
function.zip
├── hello_lambda.py   ← 바로 이 위치에 있어야 함
└── utils.py
```

**잘못된 zip 구조:**
```
function.zip
└── my_lambda_project/   ← 폴더가 감싸고 있으면 안 됨
    ├── hello_lambda.py
    └── utils.py
```

---

## 4. Lambda 함수 만들고 코드 올리기

**AWS 콘솔에서:**

```
Lambda 콘솔 → 함수 생성(Create function)
  → 처음부터 작성(Author from scratch) 선택
  → 함수 이름: hello-lambda-practice
  → 런타임(Runtime): Python 3.12
  → 권한(Permissions): 기존 역할 사용 → my-lambda-basic-role 선택
  → 함수 생성(Create function)
```

함수가 만들어지면 코드를 올립니다:

```
코드 소스(Code source) 영역
  → 업로드 위치(Upload from): .zip 파일
  → 업로드 → function.zip 선택
  → 저장(Save)
```

그리고 **핸들러 설정**을 확인합니다:

```
런타임 설정(Runtime settings) → 편집(Edit)
  → 핸들러: hello_lambda.lambda_handler
             ↑파일명       ↑함수명
```

---

## 5. 테스트 이벤트로 확인하기

코드를 올렸으면 바로 테스트해볼 수 있습니다.

```json
// 테스트 이벤트 JSON (콘솔의 Test 탭에서 입력)
{
  "name": "김민준"
}
```

**콘솔에서 테스트하는 방법:**

```
Test 탭 → 새 이벤트 생성(Create new event)
  → 이벤트 이름: test-hello
  → 이벤트 JSON에 위 내용 입력
  → Test 버튼 클릭
```

**성공했을 때 결과:**

```json
{
  "statusCode": 200,
  "body": "안녕하세요, 김민준님!"
}
```

**로그 확인:**

```
테스트 결과 하단의 'Log output' 클릭
또는
모니터링(Monitor) 탭 → 로그 보기(View CloudWatch logs)
```

---

## 따라 하기 실습

### 실습 1 — 첫 번째 Lambda 함수 만들고 배포하기

아래 코드를 `greet_lambda.py`라는 이름으로 저장하세요.

```python
# greet_lambda.py

def lambda_handler(event, context):
    first_name = event.get("first_name", "이름없음")
    last_name = event.get("last_name", "")
    full_name = f"{last_name}{first_name}"

    return {
        "statusCode": 200,
        "body": f"환영합니다, {full_name}님! Lambda 배포에 성공하셨어요!"
    }
```

1. `greet_lambda.py`를 `function.zip`으로 묶는다
2. Lambda 콘솔에서 `greet-practice` 함수를 만들고 zip을 업로드한다
3. 핸들러를 `greet_lambda.lambda_handler`로 설정한다
4. 아래 JSON으로 테스트한다

```json
{
  "first_name": "지수",
  "last_name": "박"
}
```

기대 결과: `"환영합니다, 박지수님! Lambda 배포에 성공하셨어요!"`

---

### 실습 2 — 계산 기능 추가하고 재배포하기

실습 1의 `greet_lambda.py`에 간단한 계산 기능을 추가합니다.

```python
# greet_lambda.py (수정 버전)

def lambda_handler(event, context):
    first_name = event.get("first_name", "이름없음")
    last_name = event.get("last_name", "")
    full_name = f"{last_name}{first_name}"

    # 새로 추가: 나이 계산
    birth_year = event.get("birth_year", 2000)
    current_year = 2026
    age = current_year - birth_year

    return {
        "statusCode": 200,
        "body": f"환영합니다, {full_name}님! 올해 {age}살이시군요!"
    }
```

1. 파일을 수정하고 다시 `function.zip`으로 묶는다
2. Lambda 콘솔에서 기존 `greet-practice` 함수에 새 zip을 업로드한다
3. 아래 JSON으로 테스트한다

```json
{
  "first_name": "지수",
  "last_name": "박",
  "birth_year": 1998
}
```

---

### 실습 3 — 오류 상황 만들어보고 로그 읽기

실수로 오류가 나는 코드를 배포해보고, CloudWatch 로그에서 오류를 읽는 연습을 합니다.

```python
# broken_lambda.py (일부러 오류를 만든 버전)

def lambda_handler(event, context):
    number = event["number"]          # .get() 없이 접근
    result = 100 / number             # 0이 들어오면 오류 발생!
    return {"statusCode": 200, "body": f"결과: {result}"}
```

1. 위 코드를 `broken_lambda.py`로 저장하고 배포한다
2. `{"number": 0}` 으로 테스트한다 (0으로 나누기 오류 발생)
3. CloudWatch 로그에서 `ZeroDivisionError` 메시지를 찾아본다
4. `{"number": 10}` 으로도 테스트해서 정상 동작을 확인한다

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|------|------------|-----------|
| 핸들러 이름을 잘못 설정함 | `Runtime.HandlerNotFound: module 'lambda_function' has no attribute 'lambda_handler'` | 핸들러를 `파일명.lambda_handler` 형식으로 정확히 입력 |
| zip 안에 폴더가 한 겹 더 있음 | `Unable to import module 'lambda_function'` | zip 파일 안에 `.py` 파일이 바로 있어야 함. 폴더 안에 넣지 말 것 |
| IAM 역할을 설정 안 함 | `The role defined for the function cannot be assumed by Lambda` | 신뢰 관계(Trust policy)에 `lambda.amazonaws.com`이 있는지 확인 |
| `event` 키가 없을 때 `.get()` 안 씀 | `KeyError: 'name'` | `event["name"]` 대신 `event.get("name", "기본값")` 사용 |
| Python 버전이 안 맞는 외부 라이브러리 포함 | `Runtime.ImportModuleError` | 라이브러리를 Lambda와 같은 Python 버전 환경에서 설치 후 zip |
| 코드 수정 후 저장(Deploy)을 안 누름 | 이전 코드가 그대로 실행됨 | 코드 수정 후 반드시 **Deploy** 버튼 클릭 |
| 타임아웃이 너무 짧음 | `Task timed out after 3.00 seconds` | 구성(Configuration) → 일반 구성 → 제한 시간을 늘림 (기본값 3초) |

---

## 확인 체크리스트

- [ ] `lambda_handler(event, context)` 함수 이름과 매개변수를 정확히 썼다
- [ ] zip 파일 안에 `.py` 파일이 폴더 없이 바로 들어있다
- [ ] Lambda 함수의 핸들러 설정이 `파일명.함수명` 형식으로 되어있다
- [ ] IAM 역할에 `AWSLambdaBasicExecutionRole` 정책이 붙어있다
- [ ] 코드 수정 후 **Deploy** 버튼을 눌렀다
- [ ] 테스트 이벤트 JSON이 올바른 형식인지 확인했다
- [ ] 테스트 후 로그 출력(Log output)에서 오류가 없는지 확인했다
- [ ] `event.get("키", "기본값")` 방식으로 안전하게 값을 꺼내고 있다

---

## 한 번 더 생각해 보기

1. Lambda 함수가 `event` 딕셔너리에서 없는 키를 `event["없는키"]`로 꺼내려 하면 어떤 일이 생길까요? 그리고 이 문제를 막으려면 코드를 어떻게 바꿔야 할까요?

2. 같은 Lambda 함수를 하루에 100번 호출하는 서비스와, EC2 서버를 24시간 켜두는 서비스를 비교한다면 각각 어떤 상황에서 더 비용 효율적일까요?

3. zip 파일을 업로드하는 방법 말고, AWS 콘솔의 인라인 편집기(브라우저에서 직접 코드 수정)를 쓰는 건 어떤 경우에 편리하고, 어떤 경우에 불편할까요?

---

## 다음 장

다음 장에서는 Lambda 함수에 **환경 변수**를 설정하는 방법을 배우고, API Gateway와 연결해서 HTTP 요청을 받을 수 있는 진짜 API를 만들어봅니다.