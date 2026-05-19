## 이 장에서 배우는 것

- API Gateway에서 경로(route)를 용도별로 나누는 방법
- 경로 파라미터와 쿼리 파라미터를 Lambda에서 읽는 방법
- 요청이 잘못되었을 때 의미 있는 에러 메시지를 돌려주는 방법
- 인증이 필요한 경로와 공개 경로를 분리하는 방법
- AWS CDK로 위 설정을 코드로 관리하는 방법

---

## 먼저 쉬운 설명

음식 배달 앱을 생각해 보세요. 앱에는 여러 종류의 요청이 옵니다.

- `/menu` → 메뉴 목록 보기 (누구나 가능)
- `/order` → 주문하기 (로그인한 사람만 가능)
- `/admin/stats` → 통계 보기 (관리자만 가능)

이 세 경로는 **목적도 다르고, 접근 권한도 다릅니다.** 하나의 Lambda 함수가 모든 요청을 받아서 직접 구분해도 되지만, 그러면 함수가 점점 복잡해집니다.

API Gateway의 **고급 라우팅**은 이 분류를 입구에서 처리합니다. 잘못된 요청은 Lambda에 도달하기도 전에 400 에러로 돌려보냅니다. 덕분에 Lambda는 자기 할 일만 할 수 있습니다.

이 장을 마치면 여러분은 경로를 체계적으로 나누고, 에러가 발생했을 때 "무엇이 왜 잘못됐는지" 알 수 있는 API를 만들 수 있습니다.

---

## 1. 경로 파라미터와 쿼리 파라미터 읽기

### 경로 파라미터 (Path Parameter)

`/items/{itemId}` 처럼 경로 안에 값을 넣는 방식입니다. 특정 하나의 자원을 가리킬 때 씁니다.

```python
# handlers/get_item.py

import json

def handler(event, context):
    # API Gateway가 경로 파라미터를 여기에 넣어줍니다
    path_params = event.get("pathParameters") or {}
    item_id = path_params.get("itemId")

    if not item_id:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "itemId가 경로에 없습니다"}, ensure_ascii=False)
        }

    # 실제로는 DB에서 조회하겠지만 여기서는 가짜 데이터를 씁니다
    item = {"id": item_id, "name": f"상품 {item_id}", "price": 9900}

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(item, ensure_ascii=False)
    }
```

### 쿼리 파라미터 (Query Parameter)

`/items?category=food&limit=10` 처럼 `?` 뒤에 붙는 값입니다. 목록 필터링이나 페이징에 씁니다.

```python
# handlers/list_items.py

import json

def handler(event, context):
    # 쿼리 파라미터는 여기서 읽습니다
    query_params = event.get("queryStringParameters") or {}

    category = query_params.get("category", "all")
    limit_raw = query_params.get("limit", "20")

    # 숫자 변환 시 에러 처리를 꼭 해야 합니다
    try:
        limit = int(limit_raw)
        if limit < 1 or limit > 100:
            raise ValueError("범위 초과")
    except ValueError:
        return {
            "statusCode": 400,
            "body": json.dumps(
                {"error": "limit은 1~100 사이 숫자여야 합니다"},
                ensure_ascii=False
            )
        }

    items = [
        {"id": str(i), "category": category, "name": f"상품 {i}"}
        for i in range(1, limit + 1)
    ]

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"items": items, "total": len(items)}, ensure_ascii=False)
    }
```

---

## 2. 공통 에러 응답 형식 만들기

에러 메시지가 API마다 형식이 다르면 프론트엔드 개발자가 힘들어집니다. 처음부터 형식을 통일합시다.

```python
# handlers/common.py

import json

def error_response(status_code: int, code: str, message: str) -> dict:
    """
    모든 에러 응답을 같은 형식으로 만들어 줍니다.
    code: 프로그램이 읽는 짧은 영문 코드 (예: ITEM_NOT_FOUND)
    message: 사람이 읽는 한국어 설명
    """
    return {
        "statusCode": status_code,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(
            {"error": {"code": code, "message": message}},
            ensure_ascii=False
        )
    }

def success_response(data: dict, status_code: int = 200) -> dict:
    return {
        "statusCode": status_code,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(data, ensure_ascii=False)
    }
```

이 공통 함수를 사용하면 에러 응답이 언제나 같은 JSON 구조를 가집니다.

```json
{
  "error": {
    "code": "ITEM_NOT_FOUND",
    "message": "해당 상품을 찾을 수 없습니다"
  }
}
```

---

## 3. CDK로 라우팅 규칙 코드로 작성하기

콘솔에서 클릭하는 대신, CDK로 라우팅을 코드로 관리하면 변경 이력이 Git에 남습니다.

```python
# cdk/api_stack.py

import aws_cdk as cdk
from aws_cdk import (
    aws_apigatewayv2 as apigw,
    aws_apigatewayv2_integrations as integrations,
    aws_lambda as _lambda,
)
from constructs import Construct


class ApiStack(cdk.Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs):
        super().__init__(scope, construct_id, **kwargs)

        # --- Lambda 함수 정의 ---
        list_fn = _lambda.Function(
            self, "ListItemsFunction",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="list_items.handler",
            code=_lambda.Code.from_asset("handlers"),
        )

        get_fn = _lambda.Function(
            self, "GetItemFunction",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="get_item.handler",
            code=_lambda.Code.from_asset("handlers"),
        )

        # --- HTTP API 생성 ---
        api = apigw.HttpApi(
            self, "ItemApi",
            api_name="item-api",
            cors_preflight=apigw.CorsPreflightOptions(
                allow_origins=["*"],
                allow_methods=[apigw.CorsHttpMethod.GET, apigw.CorsHttpMethod.POST],
                allow_headers=["Content-Type", "Authorization"],
            ),
        )

        # --- 라우팅 설정 ---
        # GET /items          → 목록 조회
        api.add_routes(
            path="/items",
            methods=[apigw.HttpMethod.GET],
            integration=integrations.HttpLambdaIntegration("ListIntegration", list_fn),
        )

        # GET /items/{itemId} → 단건 조회
        api.add_routes(
            path="/items/{itemId}",
            methods=[apigw.HttpMethod.GET],
            integration=integrations.HttpLambdaIntegration("GetIntegration", get_fn),
        )

        # 배포 후 URL 출력
        cdk.CfnOutput(self, "ApiUrl", value=api.api_endpoint)
```

---

## 4. 인가(Authorization) 없는 경로와 있는 경로 분리하기

모든 경로에 인증이 필요하지는 않습니다. CDK에서 `authorizer` 옵션으로 경로별로 다르게 설정할 수 있습니다.

```python
# cdk/api_stack.py (인가 설정 추가 버전)

from aws_cdk import aws_apigatewayv2_authorizers as authorizers

# JWT 인가자 (예: Cognito User Pool 사용)
jwt_authorizer = authorizers.HttpJwtAuthorizer(
    "CognitoAuthorizer",
    jwt_issuer="https://cognito-idp.ap-northeast-2.amazonaws.com/ap-northeast-2_XXXXXXX",
    jwt_audience=["your-app-client-id"],
)

# 공개 경로: 인가자 없음
api.add_routes(
    path="/items",
    methods=[apigw.HttpMethod.GET],
    integration=integrations.HttpLambdaIntegration("ListIntegration", list_fn),
    # authorizer를 지정하지 않으면 누구나 접근 가능
)

# 보호된 경로: JWT 인가자 필요
create_fn = _lambda.Function(
    scope_ref, "CreateItemFunction",
    runtime=_lambda.Runtime.PYTHON_3_12,
    handler="create_item.handler",
    code=_lambda.Code.from_asset("handlers"),
)

api.add_routes(
    path="/items",
    methods=[apigw.HttpMethod.POST],
    integration=integrations.HttpLambdaIntegration("CreateIntegration", create_fn),
    authorizer=jwt_authorizer,  # 토큰이 없으면 API Gateway가 401을 반환합니다
)
```

인가에 실패했을 때 API Gateway는 자동으로 이런 응답을 돌려줍니다.

```json
{"message": "Unauthorized"}
```

---

## 5. Lambda 안에서 구조적 에러 처리하기

Lambda가 예상치 못한 에러를 만났을 때 500 응답 대신 의미 있는 메시지를 주도록 합니다.

```python
# handlers/create_item.py

import json
import boto3
from botocore.exceptions import ClientError
from common import error_response, success_response

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("Items")

REQUIRED_FIELDS = ["name", "price", "category"]

def handler(event, context):
    # 1. 요청 본문 파싱
    try:
        body = json.loads(event.get("body") or "{}")
    except json.JSONDecodeError:
        return error_response(400, "INVALID_JSON", "요청 본문이 올바른 JSON 형식이 아닙니다")

    # 2. 필수 필드 검증
    missing = [f for f in REQUIRED_FIELDS if f not in body]
    if missing:
        return error_response(
            400,
            "MISSING_FIELDS",
            f"필수 항목이 없습니다: {', '.join(missing)}"
        )

    # 3. 타입 검증
    if not isinstance(body["price"], (int, float)) or body["price"] <= 0:
        return error_response(400, "INVALID_PRICE", "price는 0보다 큰 숫자여야 합니다")

    # 4. DB 저장
    import uuid
    item_id = str(uuid.uuid4())
    item = {"id": item_id, **body}

    try:
        table.put_item(Item=item)
    except ClientError as e:
        # AWS SDK 에러는 error_code로 구분합니다
        error_code = e.response["Error"]["Code"]
        if error_code == "ProvisionedThroughputExceededException":
            return error_response(503, "DB_THROTTLED", "잠시 후 다시 시도해 주세요")
        # 그 외 예상치 못한 AWS 에러
        raise  # CloudWatch에 스택 트레이스가 남습니다

    return success_response({"id": item_id}, status_code=201)
```

---

## 따라 하기 실습

### 실습 1 — 기본 프로젝트 구조 만들기

아래 명령어로 실습 디렉터리를 만듭니다.

```bash
mkdir api-routing-lab && cd api-routing-lab
python -m venv .venv && source .venv/bin/activate
pip install aws-cdk-lib constructs boto3

# 디렉터리 구조 만들기
mkdir handlers cdk

touch handlers/__init__.py
touch handlers/common.py
touch handlers/list_items.py
touch handlers/get_item.py
touch handlers/create_item.py
touch cdk/api_stack.py
touch app.py
```

`handlers/common.py`에 2번 섹션의 `error_response`, `success_response` 함수를 복사해 붙여 넣으세요.

---

### 실습 2 — Lambda 핸들러 작성 후 로컬 테스트

`handlers/get_item.py`에 1번 섹션의 코드를 복사합니다. 그다음 로컬에서 Lambda 이벤트를 흉내 내어 테스트합니다.

```python
# test_local.py  (프로젝트 루트에 생성)

import json
from handlers.get_item import handler

# 정상 요청 테스트
event_ok = {
    "pathParameters": {"itemId": "42"},
    "queryStringParameters": None
}
result = handler(event_ok, {})
assert result["statusCode"] == 200
body = json.loads(result["body"])
assert body["id"] == "42"
print("정상 요청 통과 ✓")

# 에러 요청 테스트
event_bad = {
    "pathParameters": None,
    "queryStringParameters": None
}
result = handler(event_bad, {})
assert result["statusCode"] == 400
print("에러 요청 통과 ✓")
```

```bash
python test_local.py
# 기대 출력:
# 정상 요청 통과 ✓
# 에러 요청 통과 ✓
```

---

### 실습 3 — CDK 배포 후 curl로 검증

`cdk/api_stack.py`와 `app.py`를 3번 섹션 코드로 채운 뒤 배포합니다.

```python
# app.py

import aws_cdk as cdk
from cdk.api_stack import ApiStack

app = cdk.App()
ApiStack(app, "ApiRoutingLab")
app.synth()
```

```bash
cdk bootstrap   # 계정당 처음 한 번만 실행
cdk deploy

# 출력에서 ApiUrl 값을 복사한 뒤 아래 명령 실행
API_URL="https://xxxxxxxxxx.execute-api.ap-northeast-2.amazonaws.com"

# 목록 조회
curl "$API_URL/items?category=food&limit=3"

# 단건 조회
curl "$API_URL/items/42"

# 잘못된 limit 값 → 400 에러 확인
curl "$API_URL/items?limit=abc"
```

마지막 명령에서 아래와 같은 응답이 오면 성공입니다.

```json
{"error": {"code": "INVALID_PARAM", "message": "limit은 1~100 사이 숫자여야 합니다"}}
```

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 원인 | 해결 방법 |
|---|---|---|---|
| `pathParameters`를 바로 인덱싱 | `TypeError: 'NoneType' object is not subscriptable` | 경로 파라미터가 없을 때 `None` 반환 | `event.get("pathParameters") or {}` 로 방어 |
| `body`를 파싱 없이 사용 | `TypeError: string indices must be integers` | `body`는 문자열이므로 `json.loads` 필요 | `json.loads(event.get("body") or "{}")` |
| CDK에서 경로 끝에 `/` 추가 | `{"message":"Not Found"}` | `/items/` 와 `/items` 는 다른 경로 | CDK `path` 값에서 끝 슬래시 제거 |
| `ensure_ascii=False` 누락 | `{"name": "\\uc0c1\\ud488 1"}` | 한글이 유니코드 이스케이프로 출력됨 | `json.dumps(..., ensure_ascii=False)` 항상 사용 |
| 인가자 설정 후 OPTIONS 요청 실패 | `{"message":"Unauthorized"}` | CORS preflight도 인가자 통과 요구 | CORS preflight(`OPTIONS`)는 인가 제외하도록 API GW 설정 |
| `cdk deploy` 전 `cdk bootstrap` 미실행 | `❌  ApiRoutingLab failed: Error: This stack uses Assets...` | CDK가 파일을 업로드할 S3 버킷이 없음 | `cdk bootstrap` 먼저 실행 |

---

## 확인 체크리스트

- [ ] `pathParameters`와 `queryStringParameters`를 모두 `or {}`로 방어적으로 읽을 수 있다
- [ ] `error_response` 함수를 만들어 모든 에러 응답에서 일관된 JSON 형식을 사용한다
- [ ] CDK 코드에서 같은 경로(`/items`)에 GET과 POST를 다른 Lambda 함수에 연결할 수 있다
- [ ] 공개 경로와 인가 필요 경로를 CDK에서 `authorizer` 옵션으로 구분할 수 있다
- [ ] `ClientError`를 잡아서 AWS SDK 에러 코드별로 다른 응답을 줄 수 있다
- [ ] `curl`로 실제 API를 호출해 400, 401, 200 응답을 각각 확인했다
- [ ] CloudWatch 로그에서 Lambda 실행 로그를 찾아볼 수 있다

---

## 한 번 더 생각해 보기

1. `/items`(목록)와 `/items/{itemId}`(단건) 중 어느 경로에서 페이징(`page`, `limit`)이 필요할까요? 반대 경로에 이 파라미터가 오면 어떻게 처리하는 것이 좋을까요?

2. 인가에 실패했을 때 API Gateway가 돌려주는 `{"message":"Unauthorized"}`는 여러분이 만든 `error_response` 형식과 다릅니다. 프론트엔드 개발자가 일관된 형식을 받으려면 어떤 방법이 있을까요?

3. Lambda 함수 안에서 `raise`를 호출하면 500 응답이 내려갑니다. 어떤 경우에 에러를 잡아서 처리하고, 어떤 경우에 그냥 `raise`를 쓰는 게 좋을까요?

---

## 다음 장

다음 장에서는 이 API에 **CloudWatch 알람과 X-Ray 트레이싱**을 붙여서, 에러가 발생했을 때 어디서 시간이 걸리고 무엇이 실패했는지 한눈에 파악하는 방법을 배웁니다.