## 이 장에서 배우는 것

- AI(Claude)를 활용해서 Python AWS Lambda 서비스의 E2E(End-to-End) 테스트 코드를 자동 생성하는 방법
- `pytest`로 Lambda 핸들러 함수를 직접 호출하여 검증하는 방법
- `moto` 라이브러리로 AWS 서비스(DynamoDB, S3 등)를 가짜(mock)로 대체하는 방법
- AI에게 명확한 테스트 요구사항을 설명하고 코드를 받아 수정하는 방법
- 실제 배포 없이 로컬에서 Lambda 전체 흐름을 테스트하는 방법

---

## 먼저 쉬운 설명

여러분이 음식점을 연다고 상상해 보세요. 주방에서 요리가 잘 되는지 확인하는 것(단위 테스트)도 중요하지만, 손님이 주문하고 → 주방이 받고 → 요리가 나오고 → 계산까지 이어지는 **전체 흐름**이 정상인지 확인하는 것이 E2E 테스트입니다.

Lambda 서비스도 마찬가지입니다. API Gateway에서 요청이 오면 → Lambda가 처리하고 → DynamoDB에 저장하거나 → S3에 파일을 올리는 흐름이 있습니다. 어느 한 단계가 고장 나면 전체가 망가집니다.

문제는 이 테스트 코드를 처음부터 작성하는 것이 꽤 번거롭다는 점입니다. **AI를 활용하면 "이런 Lambda 함수가 있는데 E2E 테스트 코드 짜줘"라고 말하는 것만으로 초안을 받을 수 있습니다.** 이 장에서는 그 방법을 단계별로 익힙니다.

---

## 1. E2E 테스트가 단위 테스트와 다른 이유

단위 테스트는 함수 하나만 검사합니다. E2E 테스트는 **실제 사용자가 겪는 시나리오 전체**를 검사합니다.

```python
# tests/unit/test_calc.py  ← 단위 테스트: 함수 하나만 확인
from app.utils import calculate_price

def test_calculate_price():
    assert calculate_price(1000, 0.1) == 1100
```

```python
# tests/e2e/test_order_flow.py  ← E2E 테스트: 전체 흐름 확인
# 1) API Gateway 이벤트 생성
# 2) Lambda 핸들러 호출
# 3) DynamoDB에 데이터가 저장됐는지 확인
# 4) 응답 상태 코드가 200인지 확인
```

E2E 테스트는 "사용자 입장에서 이 서비스가 정말 동작하는가?"를 묻습니다.

---

## 2. 테스트 환경 설정하기

실제 AWS에 배포하지 않고 로컬에서 테스트하려면 두 가지 도구가 필요합니다.

**필요한 패키지 설치:**

```bash
pip install pytest moto boto3
```

```
# requirements-dev.txt
pytest==8.2.0
moto[dynamodb,s3]==5.0.2
boto3==1.34.0
```

**프로젝트 폴더 구조:**

```
my-lambda-service/
├── src/
│   └── handler.py        ← Lambda 핸들러
├── tests/
│   ├── conftest.py       ← 공통 설정 (fixture)
│   └── e2e/
│       └── test_order_api.py
└── requirements-dev.txt
```

`conftest.py`는 pytest가 자동으로 읽는 설정 파일입니다. AWS 가짜 환경을 여기서 준비합니다.

```python
# tests/conftest.py
import os
import boto3
import pytest
from moto import mock_aws

# 테스트 중에는 가짜 AWS 자격증명 사용
os.environ["AWS_ACCESS_KEY_ID"] = "testing"
os.environ["AWS_SECRET_ACCESS_KEY"] = "testing"
os.environ["AWS_DEFAULT_REGION"] = "ap-northeast-2"
os.environ["ORDERS_TABLE"] = "orders-test"

@pytest.fixture
def aws_mock():
    """모든 테스트에서 재사용할 AWS mock 환경"""
    with mock_aws():
        # 가짜 DynamoDB 테이블 생성
        dynamodb = boto3.resource("dynamodb", region_name="ap-northeast-2")
        dynamodb.create_table(
            TableName="orders-test",
            KeySchema=[{"AttributeName": "order_id", "KeyType": "HASH"}],
            AttributeDefinitions=[{"AttributeName": "order_id", "AttributeType": "S"}],
            BillingMode="PAY_PER_REQUEST",
        )
        yield  # 테스트 실행
        # with 블록이 끝나면 가짜 환경도 자동으로 정리됨
```

---

## 3. 테스트할 Lambda 핸들러 이해하기

AI에게 테스트 코드를 요청하기 전에, **테스트 대상 코드를 명확히 파악**해야 합니다.

```python
# src/handler.py
import json
import os
import uuid
import boto3

dynamodb = boto3.resource("dynamodb")

def lambda_handler(event, context):
    """
    POST /orders 요청을 처리합니다.
    body: {"item": "아메리카노", "quantity": 2}
    """
    try:
        body = json.loads(event.get("body", "{}"))
        item = body.get("item")
        quantity = body.get("quantity", 1)

        if not item:
            return {
                "statusCode": 400,
                "body": json.dumps({"error": "item은 필수 항목입니다"}),
            }

        order_id = str(uuid.uuid4())
        table = dynamodb.Table(os.environ["ORDERS_TABLE"])
        table.put_item(
            Item={"order_id": order_id, "item": item, "quantity": quantity}
        )

        return {
            "statusCode": 201,
            "body": json.dumps({"order_id": order_id, "message": "주문 완료"}),
        }

    except Exception as e:
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)}),
        }
```

이 핸들러가 하는 일:
1. API Gateway 이벤트에서 `body`를 꺼냄
2. `item`이 없으면 400 반환
3. DynamoDB에 주문 저장
4. 성공하면 201과 `order_id` 반환

---

## 4. AI에게 테스트 코드 요청하는 방법

AI(Claude)에게 테스트를 요청할 때는 **맥락을 충분히 제공**해야 좋은 코드를 받을 수 있습니다.

**좋지 않은 프롬프트:**
```
Lambda 테스트 코드 짜줘
```

**좋은 프롬프트 예시:**
```
아래 Python Lambda 핸들러에 대한 pytest E2E 테스트를 작성해줘.

조건:
- AWS는 moto로 mock 처리
- 테스트 시나리오: 정상 주문, item 누락 시 400, DynamoDB 저장 확인
- conftest.py에 DynamoDB fixture가 이미 있음 (테이블명: orders-test)
- handler.py의 lambda_handler 함수를 직접 import해서 호출

[src/handler.py 코드 붙여넣기]
```

AI가 생성한 코드 초안:

```python
# tests/e2e/test_order_api.py  ← AI가 생성한 초안 (검토 필요)
import json
import pytest
from src.handler import lambda_handler

def make_event(body: dict) -> dict:
    """API Gateway 이벤트 형식을 만들어주는 도우미 함수"""
    return {"body": json.dumps(body)}

def test_정상_주문_생성(aws_mock):
    event = make_event({"item": "아메리카노", "quantity": 2})
    response = lambda_handler(event, {})

    assert response["statusCode"] == 201
    body = json.loads(response["body"])
    assert "order_id" in body
    assert body["message"] == "주문 완료"

def test_item_없을때_400_반환(aws_mock):
    event = make_event({"quantity": 1})
    response = lambda_handler(event, {})

    assert response["statusCode"] == 400
    body = json.loads(response["body"])
    assert "error" in body

def test_dynamodb에_실제로_저장됨(aws_mock):
    import boto3
    item_name = "카페라떼"
    event = make_event({"item": item_name, "quantity": 1})
    response = lambda_handler(event, {})

    order_id = json.loads(response["body"])["order_id"]

    # DynamoDB에서 직접 조회해서 확인
    dynamodb = boto3.resource("dynamodb", region_name="ap-northeast-2")
    table = dynamodb.Table("orders-test")
    result = table.get_item(Key={"order_id": order_id})

    assert "Item" in result
    assert result["Item"]["item"] == item_name
```

AI 코드를 받은 후 **반드시 확인할 것:**
- import 경로가 실제 프로젝트와 맞는가?
- fixture 이름(`aws_mock`)이 `conftest.py`와 일치하는가?
- 테스트 시나리오가 실제 비즈니스 요구사항을 반영하는가?

---

## 5. 테스트 실행하고 결과 해석하기

```bash
# 전체 E2E 테스트 실행
pytest tests/e2e/ -v

# 특정 테스트만 실행
pytest tests/e2e/test_order_api.py::test_정상_주문_생성 -v
```

**성공했을 때 출력:**

```
tests/e2e/test_order_api.py::test_정상_주문_생성 PASSED
tests/e2e/test_order_api.py::test_item_없을때_400_반환 PASSED
tests/e2e/test_order_api.py::test_dynamodb에_실제로_저장됨 PASSED

3 passed in 1.23s
```

**실패했을 때 출력 예시:**

```
FAILED tests/e2e/test_order_api.py::test_정상_주문_생성
AssertionError: assert 500 == 201

  response = {'statusCode': 500, 'body': '{"error": "..."}'}
```

실패 메시지에서 `statusCode: 500`이 보이면 Lambda 내부에서 예외가 발생했다는 뜻입니다. `body`의 `error` 내용을 읽어서 원인을 파악하세요.

---

## 따라 하기 실습

### 실습 1 — 환경 준비 및 첫 테스트 실행

1. 새 폴더를 만들고 파일을 구성합니다.

```bash
mkdir order-lambda-test && cd order-lambda-test
mkdir -p src tests/e2e
pip install pytest moto boto3
```

2. 위의 `src/handler.py`, `tests/conftest.py` 내용을 그대로 복사합니다.

3. `tests/e2e/test_order_api.py`에 `test_정상_주문_생성` 하나만 먼저 작성합니다.

```bash
pytest tests/e2e/test_order_api.py::test_정상_주문_생성 -v
```

`PASSED`가 출력되면 성공입니다.

---

### 실습 2 — AI에게 추가 시나리오 요청하기

실습 1의 테스트가 통과된 상태에서, AI(Claude)에게 다음 프롬프트로 추가 테스트를 요청합니다.

```
지금까지 작성된 test_order_api.py에 아래 시나리오를 추가해줘:
1. quantity가 음수일 때 400을 반환하는 테스트
2. 같은 item으로 두 번 주문하면 DynamoDB에 두 개의 row가 생기는 테스트

기존 fixture와 make_event 함수를 재사용해줘.
```

받은 코드를 `test_order_api.py`에 추가하고 실행합니다.

```bash
pytest tests/e2e/ -v
```

만약 `handler.py`가 quantity 검증을 하지 않는다면 테스트가 실패합니다. 이때 **테스트가 틀린 것이 아니라 핸들러에 로직을 추가해야** 한다는 신호입니다.

---

### 실습 3 — CI처럼 전체 테스트 스위트 점검하기

로컬에서 배포 전 최종 점검을 시뮬레이션합니다.

```bash
# 실패 즉시 멈추고 짧은 요약 출력
pytest tests/ -v --tb=short -x

# 커버리지 확인 (선택)
pip install pytest-cov
pytest tests/ --cov=src --cov-report=term-missing
```

커버리지 결과 예시:

```
Name              Stmts   Miss  Cover
-------------------------------------
src/handler.py       20      2    90%
```

`Miss` 2줄이 어디인지 확인하고, AI에게 "handler.py의 이 부분을 테스트하는 케이스 추가해줘"라고 요청하여 커버리지를 높입니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|-----------|
| `conftest.py`에서 mock을 활성화하기 전에 `boto3` 클라이언트를 모듈 수준에서 생성 | `botocore.exceptions.NoCredentialsError` | `boto3` 클라이언트 생성을 함수 안으로 이동하거나 fixture 안에서 생성 |
| `ORDERS_TABLE` 환경 변수를 설정하지 않음 | `KeyError: 'ORDERS_TABLE'` | `conftest.py` 최상단에 `os.environ["ORDERS_TABLE"] = "orders-test"` 추가 |
| moto를 import했지만 `@mock_aws` 데코레이터나 `with mock_aws()` 없이 사용 | `ResourceNotFoundException` (실제 AWS 호출 시도) | fixture에 `with mock_aws():` 블록이 감싸고 있는지 확인 |
| import 경로 오류 | `ModuleNotFoundError: No module named 'src'` | 프로젝트 루트에서 `pytest`를 실행하거나 `pyproject.toml`에 `pythonpath = ["."]` 추가 |
| AI가 생성한 테이블 키 스키마가 실제와 다름 | `ValidationException: One or more parameter values were invalid` | `conftest.py`의 `create_table`에서 `KeySchema`와 `AttributeDefinitions`가 `handler.py`의 실제 키와 일치하는지 확인 |
| `make_event`에서 body를 dict로 전달 (json.dumps 누락) | `TypeError: the JSON object must be str` | `make_event` 안에서 `json.dumps(body)` 호출 확인 |

---

## 확인 체크리스트

- [ ] `pytest`, `moto`, `boto3`가 설치되어 있다
- [ ] `conftest.py`에 가짜 AWS 자격증명 환경 변수가 설정되어 있다
- [ ] `conftest.py`의 fixture가 `mock_aws()` 컨텍스트 안에서 테이블을 생성한다
- [ ] 테스트 함수 파라미터에 fixture 이름(`aws_mock`)이 올바르게 들어가 있다
- [ ] `lambda_handler`를 `src.handler`에서 직접 import하고 있다
- [ ] 정상 케이스(201), 에러 케이스(400), DB 저장 확인 세 가지 시나리오가 모두 있다
- [ ] `pytest tests/e2e/ -v`를 실행했을 때 모든 테스트가 `PASSED`다
- [ ] AI가 생성한 코드를 눈으로 읽고 의도를 이해했다

---

## 한 번 더 생각해 보기

1. **moto로 mock 처리한 테스트와 실제 AWS에서 돌리는 테스트의 차이는 무엇일까요?** mock 테스트가 통과해도 실제 배포 후 실패할 수 있는 상황을 한 가지 떠올려 보세요.

2. **AI가 생성한 테스트 코드를 그대로 커밋해도 될까요?** 어떤 기준으로 AI 코드를 검토하고 수정해야 할지 팀원에게 설명해 본다면 어떻게 말할 건가요?

3. **테스트가 실패했을 때 "테스트를 고쳐야 한다"와 "핸들러를 고쳐야 한다" 중 어느 쪽을 먼저 판단해야 할까요?** 실습 2에서 quantity 음수 테스트가 실패했을 때 여러분은 어느 쪽을 선택했나요?

---

## 다음 장

다음 장에서는 작성한 E2E 테스트를 GitHub Actions CI 파이프라인에 연결해서, 코드를 push할 때마다 자동으로 테스트가 실행되도록 설정하는 방법을 배웁니다.