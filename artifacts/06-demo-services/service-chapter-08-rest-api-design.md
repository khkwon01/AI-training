## 이 장에서 배우는 것

- REST API가 무엇인지, 왜 사용하는지 이해한다
- HTTP 메서드(GET, POST, PUT, DELETE)의 역할을 구분할 수 있다
- AWS Lambda 함수로 간단한 REST API 엔드포인트를 만들 수 있다
- API Gateway와 Lambda를 연결하는 구조를 이해한다
- 요청(Request)과 응답(Response)의 JSON 구조를 직접 작성할 수 있다
- 상태 코드(200, 400, 404, 500)를 올바르게 반환할 수 있다

---

## 먼저 쉬운 설명

카페에서 음료를 주문할 때를 떠올려 보세요.

1. 손님이 점원에게 "아이스 아메리카노 한 잔 주세요"라고 **요청(Request)** 합니다.
2. 점원은 주문을 받아 음료를 만들고 "여기 있습니다"라고 **응답(Response)** 합니다.
3. 만약 재료가 없으면 "죄송합니다, 품절입니다"라고 **오류 응답** 을 돌려줍니다.

REST API도 똑같습니다. 앱이나 웹사이트가 서버에게 "이 데이터 줘", "이 데이터 저장해 줘"라고 요청하면, 서버가 결과를 돌려줍니다.

AWS Lambda를 사용하면 서버를 직접 운영하지 않아도 됩니다. 코드만 올려두면 요청이 들어올 때만 실행되고, 비용도 실행한 만큼만 냅니다. 규모가 작은 프로젝트나 처음 배우는 단계에서 매우 적합합니다.

> **핵심 한 줄 요약:** REST API는 클라이언트와 서버가 약속된 규칙으로 대화하는 방법이고, Lambda는 그 서버 역할을 클라우드에서 대신 해주는 서비스입니다.

---

## 1. REST API 기본 개념 — HTTP 메서드와 URL 설계

REST API에서 URL은 **"무엇을"**, HTTP 메서드는 **"어떻게 할지"** 를 나타냅니다.

| HTTP 메서드 | 의미 | 예시 URL | 설명 |
|------------|------|----------|------|
| GET | 조회 | `/users` | 사용자 목록 가져오기 |
| GET | 조회 | `/users/42` | ID가 42인 사용자 가져오기 |
| POST | 생성 | `/users` | 새 사용자 만들기 |
| PUT | 전체 수정 | `/users/42` | ID가 42인 사용자 전체 수정 |
| DELETE | 삭제 | `/users/42` | ID가 42인 사용자 삭제 |

**잘 설계된 URL vs 잘못 설계된 URL**

```
# 좋은 URL (명사 사용, 소문자, 복수형)
GET  /products
GET  /products/5
POST /products

# 나쁜 URL (동사 사용, 일관성 없음)
GET  /getProducts
POST /createNewProduct
GET  /Product/5
```

---

## 2. Lambda 이벤트 구조 이해하기

API Gateway가 Lambda를 호출할 때 `event` 딕셔너리로 요청 정보를 전달합니다. 이 구조를 반드시 이해해야 합니다.

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    # event 딕셔너리에서 요청 정보를 꺼냅니다
    http_method = event.get('httpMethod', '')        # 'GET', 'POST' 등
    path        = event.get('path', '')              # '/users', '/users/42' 등
    path_params = event.get('pathParameters') or {}  # URL 경로 변수
    query_params = event.get('queryStringParameters') or {}  # ?name=kim 같은 쿼리
    body_raw    = event.get('body', '{}') or '{}'   # POST/PUT 본문 (문자열)

    # body는 문자열로 오기 때문에 딕셔너리로 변환해야 합니다
    body = json.loads(body_raw)

    print(f"메서드: {http_method}, 경로: {path}")
    print(f"경로 파라미터: {path_params}")
    print(f"쿼리 파라미터: {query_params}")
    print(f"본문: {body}")

    # 응답은 반드시 이 형식이어야 합니다
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({'message': '요청을 받았습니다'}, ensure_ascii=False)
    }
```

> **중요:** `body`는 항상 문자열(string)로 전달됩니다. 반드시 `json.loads()`로 딕셔너리로 변환해서 사용하세요.

---

## 3. HTTP 상태 코드 — 결과를 숫자로 말하기

상태 코드는 응답이 성공인지 실패인지를 숫자로 전달합니다. 올바른 상태 코드를 반환하는 것이 좋은 API 설계의 기본입니다.

| 상태 코드 | 의미 | 사용 시점 |
|----------|------|----------|
| 200 OK | 성공 | GET 조회 성공, PUT 수정 성공 |
| 201 Created | 생성 성공 | POST로 새 리소스 생성 성공 |
| 400 Bad Request | 잘못된 요청 | 필수 필드 누락, 형식 오류 |
| 404 Not Found | 없음 | 해당 ID가 존재하지 않을 때 |
| 500 Internal Server Error | 서버 오류 | 예상치 못한 서버 내부 오류 |

```python
# helpers.py — 응답 생성 헬퍼 함수
import json

def make_response(status_code, data):
    """일관된 API 응답 형식을 만들어 줍니다."""
    return {
        'statusCode': status_code,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(data, ensure_ascii=False)
    }

def make_error(status_code, message):
    """오류 응답을 만들어 줍니다."""
    return make_response(status_code, {'error': message})
```

---

## 4. 실제 CRUD API Lambda 함수 작성하기

상품(Product) 데이터를 다루는 간단한 REST API를 만들어 봅니다. 실제 DB 대신 메모리 딕셔너리를 사용합니다.

```python
# product_handler.py
import json

# 임시 데이터 저장소 (실제 서비스에서는 DynamoDB 등을 사용합니다)
PRODUCTS = {
    '1': {'id': '1', 'name': '노트북', 'price': 1200000},
    '2': {'id': '2', 'name': '마우스', 'price': 35000},
}

def lambda_handler(event, context):
    method       = event.get('httpMethod', '')
    path_params  = event.get('pathParameters') or {}
    product_id   = path_params.get('id')          # /products/{id} 의 {id} 값
    body_raw     = event.get('body') or '{}'
    body         = json.loads(body_raw)

    # 라우팅: 메서드와 경로 파라미터 존재 여부로 분기합니다
    if method == 'GET' and not product_id:
        return get_all_products()

    elif method == 'GET' and product_id:
        return get_product(product_id)

    elif method == 'POST':
        return create_product(body)

    elif method == 'PUT' and product_id:
        return update_product(product_id, body)

    elif method == 'DELETE' and product_id:
        return delete_product(product_id)

    else:
        return make_error(400, '지원하지 않는 요청입니다.')


def get_all_products():
    """GET /products — 전체 상품 목록 반환"""
    products = list(PRODUCTS.values())
    return make_response(200, {'products': products, 'count': len(products)})


def get_product(product_id):
    """GET /products/{id} — 특정 상품 반환"""
    product = PRODUCTS.get(product_id)
    if not product:
        return make_error(404, f'ID {product_id}에 해당하는 상품이 없습니다.')
    return make_response(200, product)


def create_product(body):
    """POST /products — 새 상품 생성"""
    name  = body.get('name')
    price = body.get('price')

    # 필수 필드 검증
    if not name or price is None:
        return make_error(400, 'name과 price는 필수 항목입니다.')

    new_id = str(len(PRODUCTS) + 1)
    new_product = {'id': new_id, 'name': name, 'price': price}
    PRODUCTS[new_id] = new_product

    return make_response(201, new_product)


def update_product(product_id, body):
    """PUT /products/{id} — 상품 전체 수정"""
    if product_id not in PRODUCTS:
        return make_error(404, f'ID {product_id}에 해당하는 상품이 없습니다.')

    name  = body.get('name')
    price = body.get('price')

    if not name or price is None:
        return make_error(400, 'name과 price는 필수 항목입니다.')

    PRODUCTS[product_id] = {'id': product_id, 'name': name, 'price': price}
    return make_response(200, PRODUCTS[product_id])


def delete_product(product_id):
    """DELETE /products/{id} — 상품 삭제"""
    if product_id not in PRODUCTS:
        return make_error(404, f'ID {product_id}에 해당하는 상품이 없습니다.')

    deleted = PRODUCTS.pop(product_id)
    return make_response(200, {'message': f'상품 "{deleted["name"]}"이 삭제되었습니다.'})


# ── 헬퍼 함수 ──────────────────────────────────────────────────────────────

def make_response(status_code, data):
    return {
        'statusCode': status_code,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(data, ensure_ascii=False)
    }

def make_error(status_code, message):
    return make_response(status_code, {'error': message})
```

---

## 5. Lambda 로컬 테스트 방법

실제 AWS에 배포하기 전에 로컬에서 테스트하는 방법입니다.

```python
# test_product_handler.py
import json
from product_handler import lambda_handler

def make_event(method, path, path_params=None, body=None):
    """테스트용 가짜 API Gateway 이벤트를 만듭니다."""
    return {
        'httpMethod': method,
        'path': path,
        'pathParameters': path_params,
        'queryStringParameters': None,
        'body': json.dumps(body) if body else None,
    }

# 테스트 1: 전체 목록 조회
event = make_event('GET', '/products')
response = lambda_handler(event, {})
print('=== GET /products ===')
print(f"상태 코드: {response['statusCode']}")
print(f"응답 본문: {response['body']}")

# 테스트 2: 새 상품 생성
event = make_event('POST', '/products', body={'name': '키보드', 'price': 89000})
response = lambda_handler(event, {})
print('\n=== POST /products ===')
print(f"상태 코드: {response['statusCode']}")  # 201 이어야 합니다
print(f"응답 본문: {response['body']}")

# 테스트 3: 없는 상품 조회
event = make_event('GET', '/products/999', path_params={'id': '999'})
response = lambda_handler(event, {})
print('\n=== GET /products/999 (없는 상품) ===')
print(f"상태 코드: {response['statusCode']}")  # 404 이어야 합니다
print(f"응답 본문: {response['body']}")
```

터미널에서 실행:

```bash
python test_product_handler.py
```

예상 출력:

```
=== GET /products ===
상태 코드: 200
응답 본문: {"products": [{"id": "1", "name": "노트북", "price": 1200000}, ...], "count": 2}

=== POST /products ===
상태 코드: 201
응답 본문: {"id": "3", "name": "키보드", "price": 89000}

=== GET /products/999 (없는 상품) ===
상태 코드: 404
응답 본문: {"error": "ID 999에 해당하는 상품이 없습니다."}
```

---

## 따라 하기 실습

### 실습 1 — 기본 Lambda 핸들러 만들고 이벤트 구조 확인하기

`01_event_inspector.py` 파일을 만들고 아래 코드를 작성하세요.

```python
# 01_event_inspector.py
import json

def lambda_handler(event, context):
    """들어오는 이벤트를 그대로 출력하는 디버그용 핸들러"""
    print("받은 이벤트:", json.dumps(event, ensure_ascii=False, indent=2))

    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({
            'method': event.get('httpMethod'),
            'path': event.get('path'),
            'body_received': event.get('body')
        }, ensure_ascii=False)
    }

# 로컬 실행 테스트
if __name__ == '__main__':
    test_event = {
        'httpMethod': 'POST',
        'path': '/hello',
        'pathParameters': None,
        'body': json.dumps({'name': '홍길동', 'age': 25})
    }
    result = lambda_handler(test_event, {})
    print("응답:", json.dumps(json.loads(result['body']), ensure_ascii=False, indent=2))
```

실행 후 출력 결과를 확인하고, `httpMethod`, `path`, `body` 값이 올바르게 추출되는지 확인하세요.

---

### 실습 2 — 메모 API 만들기 (GET, POST 구현)

실습 1을 바탕으로 `02_memo_api.py` 파일을 만드세요. 메모를 저장하고 조회하는 API를 구현합니다.

```python
# 02_memo_api.py
import json

MEMOS = {}  # {'1': {'id': '1', 'title': '제목', 'content': '내용'}}
_next_id = 1

def lambda_handler(event, context):
    global _next_id
    method = event.get('httpMethod', '')
    path_params = event.get('pathParameters') or {}
    memo_id = path_params.get('id')
    body = json.loads(event.get('body') or '{}')

    if method == 'GET' and not memo_id:
        # TODO: 전체 메모 목록을 반환하세요
        # 힌트: list(MEMOS.values()) 를 활용하세요
        pass

    elif method == 'POST':
        # TODO: body에서 'title'과 'content'를 꺼내 새 메모를 만드세요
        # 힌트: 필수 필드 검증 후 MEMOS에 저장하고 201을 반환하세요
        pass

    return make_error(400, '구현되지 않은 요청입니다.')


def make_response(status_code, data):
    return {
        'statusCode': status_code,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps(data, ensure_ascii=False)
    }

def make_error(status_code, message):
    return make_response(status_code, {'error': message})
```

`pass` 부분을 채워서 아래 테스트가 통과하도록 만드세요.

```python
# 02_memo_api_test.py
import json
from memo_api_02 import lambda_handler

# POST 테스트
res = lambda_handler({'httpMethod': 'POST', 'pathParameters': None,
                      'body': json.dumps({'title': '장보기', 'content': '우유, 계란'})}, {})
assert res['statusCode'] == 201, f"예상: 201, 실제: {res['statusCode']}"

# GET 테스트
res = lambda_handler({'httpMethod': 'GET', 'pathParameters': None, 'body': None}, {})
assert res['statusCode'] == 200
data = json.loads(res['body'])
assert data['count'] == 1, f"예상: 1, 실제: {data['count']}"

print("모든 테스트 통과!")
```

---

### 실습 3 — DELETE 기능 추가하고 전체 CRUD 완성하기

실습 2의 `02_memo_api.py`를 복사해 `03_memo_api_full.py`를 만들고 DELETE와 PUT도 추가하세요.

```python
# 03_memo_api_full.py 에 추가할 함수 뼈대

def delete_memo(memo_id):
    """DELETE /memos/{id} 구현"""
    # TODO:
    # 1. memo_id가 MEMOS에 없으면 404 반환
    # 2. MEMOS에서 해당 항목을 pop()으로 제거
    # 3. 삭제된 메모 제목을 포함한 성공 메시지를 200으로 반환
    pass

def update_memo(memo_id, body):
    """PUT /memos/{id} 구현"""
    # TODO:
    # 1. memo_id가 MEMOS에 없으면 404 반환
    # 2. body에서 title, content 추출 후 필수 필드 검증
    # 3. MEMOS[memo_id]를 갱신하고 200으로 반환
    pass
```

완성 후 직접 테스트 케이스를 작성해 보세요. 없는 ID를 삭제하려 할 때 404가 반환되는지 확인하는 테스트를 반드시 포함하세요.

---

## 자주 하는 실수

| 실수 | 발생하는 오류 메시지 | 원인 | 해결 방법 |
|------|-------------------|------|----------|
| `body`를 그냥 딕셔너리처럼 사용 | `TypeError: string indices must be integers` | `body`는 문자열로 전달됨 | `json.loads(event['body'])` 로 변환 후 사용 |
| `pathParameters`가 `None`일 때 `.get()` 호출 | `AttributeError: 'NoneType' object has no attribute 'get'` | 경로 파라미터가 없으면 `None`이 옴 | `event.get('pathParameters') or {}` 로 기본값 처리 |
| `body`가 `None`일 때 `json.loads()` 호출 | `TypeError: the JSON object must be str, bytes or bytearray, not NoneType` | GET 요청엔 보통 body가 없음 | `event.get('body') or '{}'` 로 기본값 처리 |
| 응답에 `headers` 누락 | API Gateway에서 CORS 오류 또는 Content-Type 오류 | Lambda 응답 형식 불완전 | 반드시 `'headers': {'Content-Type': 'application/json'}` 포함 |
| `json.dumps()`에서 한글 깨짐 | 응답 본문이 `\uac00\ub098\ub2e4` 형태로 출력 | 기본 설정이 ASCII escape | `ensure_ascii=False` 옵션 추가 |
| 존재하지 않는 키에 `[]` 접근 | `KeyError: 'name'` | 요청 본문에 키가 없음 | `body.get('name')` 으로 접근하고 `None` 검증 추가 |
| 상태 코드를 항상 200으로 반환 | 클라이언트가 오류 여부를 알 수 없음 | 상태 코드 설계 미흡 | 생성 시 201, 없을 때 404, 오류 시 400/500 구분 |

---

## 확인 체크리스트

- [ ] HTTP 메서드 GET, POST, PUT, DELETE의 차이를 말로 설명할 수 있다
- [ ] `event['httpMethod']`로 메서드를 꺼내고, 조건문으로 분기할 수 있다
- [ ] `event['body']`가 문자열임을 알고 `json.loads()`로 변환할 수 있다
- [ ] `pathParameters`가 `None`일 수 있음을 알고 안전하게 처리할 수 있다
- [ ] GET 조회 성공은 200, 생성 성공은 201, 없는 리소스는 404를 반환한다
- [ ] `json.dumps(..., ensure_ascii=False)`를 사용해 한글이 깨지지 않게 할 수 있다
- [ ] `make_response()`, `make_error()` 같은 헬퍼 함수로 응답 형식을 통일할 수 있다
- [ ] 로컬에서 가짜 이벤트를 만들어 Lambda 함수를 테스트할 수 있다
- [ ] 실습 3의 CRUD API 전체를 직접 완성했다

---

## 한 번 더 생각해 보기

1. **URL 설계 선택의 이유:** `/getUser`와 `/users/42`는 같은 사용자를 가져오지만 설계 방식이 다릅니다. REST 원칙에서 왜 `/users/42`가 더 좋은 설계인지 설명해 보세요. 다른 개발자가 처음 이 API를 볼 때 어떤 URL이 더 직관적일까요?

2. **상태 코드의 중요성:** Lambda 함수가 항상 `statusCode: 200`을 반환하더라도 `body`에 오류 메시지를 담아 보내면 안 될까요? 모바일 앱이나 다른 서비스가 이 API를 사용할 때 어떤 문제가 생길 수 있을지 생각해 보세요.

3. **메모리 데이터의 한계:** 이 장의 예제는 Lambda 메모리 안의 딕셔너리에 데이터를 저장합니다. Lambda 함수가 재시작되거나 여러 요청이 동시에 들어오면 데이터가 어떻게 될까요? 실제 서비스에서는 어디에 데이터를 저장해야 할지 생각해 보세요.

---

## 다음 장

다음 장에서는 이 Lambda API에 **AWS DynamoDB**를 연결해 데이터를 영구적으로 저장하고, 실제 서비스 수준의 CRUD를 완성하는 방법을 배웁니다.