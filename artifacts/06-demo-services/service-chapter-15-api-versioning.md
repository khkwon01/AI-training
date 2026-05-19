## 이 장에서 배우는 것

- API 버저닝이 왜 필요한지 이해한다
- URL 경로, 쿼리 파라미터, 헤더 방식의 차이를 설명할 수 있다
- FastAPI로 간단한 버전별 API를 직접 만들 수 있다
- 버전 전환 시 하위 호환성을 유지하는 방법을 안다
- 실제 현업에서 자주 쓰는 버저닝 패턴을 코드로 구현한다

---

## 먼저 쉬운 설명

앱을 만들다 보면 반드시 이런 상황이 옵니다.

> "사용자 정보 API를 바꿔야 하는데… 이미 수십 개의 앱이 이 API를 쓰고 있어요."

전화번호부를 예로 들어볼게요. 처음에는 이름과 전화번호만 저장했는데, 나중에 이메일도 추가하고 싶어졌습니다. 기존 사용자는 그냥 쓰던 대로 써야 하고, 새 사용자는 이메일도 받고 싶습니다.

이럴 때 **API 버저닝**이 필요합니다.

버저닝은 쉽게 말해 "이 API는 1버전, 저 API는 2버전"처럼 **계약서에 날짜를 찍는 것**과 같습니다. 오래된 계약서(v1)도 유효하게 유지하면서, 새 계약서(v2)도 동시에 운영하는 거죠.

---

## 1. URL 경로 버저닝 — 가장 직관적인 방법

가장 많이 쓰이는 방식입니다. URL 자체에 버전 번호를 넣습니다.

```
/api/v1/users
/api/v2/users
```

### 실제 코드로 만들어 보기

```python
# app/main.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="버저닝 예제")

# --- v1 모델: 이름과 전화번호만 ---
class UserV1(BaseModel):
    id: int
    name: str
    phone: str

# --- v2 모델: 이메일 필드 추가 ---
class UserV2(BaseModel):
    id: int
    name: str
    phone: str
    email: str  # v2에서 새로 추가됨

# 가짜 DB
users_db = [
    {"id": 1, "name": "김철수", "phone": "010-1234-5678", "email": "cs@example.com"},
    {"id": 2, "name": "이영희", "phone": "010-9876-5432", "email": "yh@example.com"},
]

# v1 엔드포인트 — 이메일 없이 반환
@app.get("/api/v1/users", response_model=list[UserV1])
def get_users_v1():
    return [{"id": u["id"], "name": u["name"], "phone": u["phone"]} for u in users_db]

# v2 엔드포인트 — 이메일 포함해서 반환
@app.get("/api/v2/users", response_model=list[UserV2])
def get_users_v2():
    return users_db
```

```bash
# 실행
uvicorn app.main:app --reload

# v1 호출 결과
curl http://localhost:8000/api/v1/users
# [{"id":1,"name":"김철수","phone":"010-1234-5678"}, ...]

# v2 호출 결과
curl http://localhost:8000/api/v2/users
# [{"id":1,"name":"김철수","phone":"010-1234-5678","email":"cs@example.com"}, ...]
```

> **장점**: URL만 봐도 버전을 알 수 있어서 디버깅이 쉽습니다.  
> **단점**: URL이 길어지고, v1/v2 코드가 중복될 수 있습니다.

---

## 2. 쿼리 파라미터 버저닝 — 유연하지만 잊기 쉬운 방법

URL 끝에 `?version=1` 처럼 버전을 붙이는 방식입니다.

```
/api/users?version=1
/api/users?version=2
```

### 실제 코드로 만들어 보기

```python
# app/query_version.py
from fastapi import FastAPI, Query, HTTPException

app = FastAPI()

users_db = [
    {"id": 1, "name": "박지수", "phone": "010-1111-2222", "email": "js@example.com"},
]

@app.get("/api/users")
def get_users(version: int = Query(default=1, ge=1, le=2)):
    """
    version=1 이면 이메일 제외,
    version=2 이면 이메일 포함
    """
    if version == 1:
        return [{"id": u["id"], "name": u["name"], "phone": u["phone"]} for u in users_db]
    elif version == 2:
        return users_db
    else:
        raise HTTPException(status_code=400, detail=f"지원하지 않는 버전입니다: {version}")
```

```bash
# 버전 1 요청
curl "http://localhost:8000/api/users?version=1"

# 버전 2 요청
curl "http://localhost:8000/api/users?version=2"

# 버전 생략 시 기본값(1) 사용
curl "http://localhost:8000/api/users"
```

> **장점**: URL 구조가 깔끔합니다.  
> **단점**: 버전을 안 써도 에러가 안 나서 실수를 놓치기 쉽습니다.

---

## 3. 헤더 버저닝 — 전문가들이 선호하는 방법

요청 헤더에 버전 정보를 담는 방식입니다. URL은 그대로이지만, `Accept` 또는 커스텀 헤더로 버전을 구분합니다.

```
GET /api/users
Accept-Version: v1

GET /api/users
Accept-Version: v2
```

### 실제 코드로 만들어 보기

```python
# app/header_version.py
from fastapi import FastAPI, Header, HTTPException
from typing import Annotated

app = FastAPI()

users_db = [
    {"id": 1, "name": "최민준", "phone": "010-3333-4444", "email": "mj@example.com"},
]

@app.get("/api/users")
def get_users(accept_version: Annotated[str | None, Header()] = "v1"):
    """
    헤더 Accept-Version 값에 따라 다른 응답 반환
    헤더가 없으면 v1 기본값 사용
    """
    if accept_version == "v1":
        return [{"id": u["id"], "name": u["name"], "phone": u["phone"]} for u in users_db]
    elif accept_version == "v2":
        return users_db
    else:
        raise HTTPException(
            status_code=400,
            detail=f"알 수 없는 API 버전: {accept_version}. v1 또는 v2를 사용하세요."
        )
```

```bash
# v1 헤더 요청
curl -H "accept-version: v1" http://localhost:8000/api/users

# v2 헤더 요청
curl -H "accept-version: v2" http://localhost:8000/api/users
```

> **장점**: URL이 깔끔하고, REST 원칙에 충실합니다.  
> **단점**: 브라우저 주소창에서 직접 테스트하기 어렵습니다.

---

## 4. APIRouter로 버전 분리하기 — 실무에서 쓰는 구조

실제 프로젝트에서는 버전별로 파일을 분리합니다. 이렇게 하면 v1 코드를 건드리지 않고 v2를 개발할 수 있습니다.

### 폴더 구조

```
my_api/
├── main.py
├── routers/
│   ├── v1/
│   │   ├── __init__.py
│   │   └── users.py
│   └── v2/
│       ├── __init__.py
│       └── users.py
```

### 코드 작성

```python
# routers/v1/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/users")
def get_users_v1():
    return [{"id": 1, "name": "홍길동", "phone": "010-0000-0001"}]

@router.get("/users/{user_id}")
def get_user_v1(user_id: int):
    return {"id": user_id, "name": "홍길동", "phone": "010-0000-0001"}
```

```python
# routers/v2/users.py
from fastapi import APIRouter

router = APIRouter()

@router.get("/users")
def get_users_v2():
    # v2는 email 포함, 페이지네이션 메타 추가
    return {
        "data": [{"id": 1, "name": "홍길동", "phone": "010-0000-0001", "email": "gd@example.com"}],
        "total": 1,
        "page": 1,
    }
```

```python
# main.py
from fastapi import FastAPI
from routers.v1 import users as users_v1
from routers.v2 import users as users_v2

app = FastAPI()

# prefix로 버전 경로 자동 추가
app.include_router(users_v1.router, prefix="/api/v1", tags=["v1 - Users"])
app.include_router(users_v2.router, prefix="/api/v2", tags=["v2 - Users"])
```

이렇게 하면 `/api/v1/users`와 `/api/v2/users`가 자동으로 만들어지고, Swagger UI(`/docs`)에서도 깔끔하게 분리되어 보입니다.

---

## 따라 하기 실습

### 실습 1 — URL 버저닝 API 만들기

`my_api/` 폴더를 만들고 아래 파일을 작성하세요.

```python
# my_api/main.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ProductV1(BaseModel):
    id: int
    name: str
    price: int

class ProductV2(BaseModel):
    id: int
    name: str
    price: int
    discount_rate: float  # 할인율 추가

products = [
    {"id": 1, "name": "노트북", "price": 1200000, "discount_rate": 0.1},
    {"id": 2, "name": "마우스", "price": 35000, "discount_rate": 0.05},
]

@app.get("/api/v1/products", response_model=list[ProductV1])
def list_products_v1():
    return products

@app.get("/api/v2/products", response_model=list[ProductV2])
def list_products_v2():
    return products
```

```bash
# 서버 실행
cd my_api
uvicorn main:app --reload

# 브라우저 또는 curl로 확인
curl http://localhost:8000/api/v1/products
curl http://localhost:8000/api/v2/products
```

v1에는 `discount_rate`가 없고, v2에는 있는 것을 확인하세요.

---

### 실습 2 — Router 분리 구조로 리팩터링

실습 1의 코드를 파일 분리 구조로 바꿔보세요.

```bash
# 폴더 구조 만들기
mkdir -p my_api/routers/v1 my_api/routers/v2
touch my_api/routers/__init__.py
touch my_api/routers/v1/__init__.py
touch my_api/routers/v2/__init__.py
```

```python
# my_api/routers/v1/products.py
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class ProductV1(BaseModel):
    id: int
    name: str
    price: int

_products = [
    {"id": 1, "name": "노트북", "price": 1200000},
    {"id": 2, "name": "마우스", "price": 35000},
]

@router.get("/products", response_model=list[ProductV1])
def list_products():
    return _products
```

```python
# my_api/routers/v2/products.py
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter()

class ProductV2(BaseModel):
    id: int
    name: str
    price: int
    discount_rate: float

_products = [
    {"id": 1, "name": "노트북", "price": 1200000, "discount_rate": 0.1},
    {"id": 2, "name": "마우스", "price": 35000, "discount_rate": 0.05},
]

@router.get("/products", response_model=list[ProductV2])
def list_products():
    return _products
```

```python
# my_api/main.py (리팩터링 후)
from fastapi import FastAPI
from routers.v1 import products as products_v1
from routers.v2 import products as products_v2

app = FastAPI()

app.include_router(products_v1.router, prefix="/api/v1", tags=["v1"])
app.include_router(products_v2.router, prefix="/api/v2", tags=["v2"])
```

---

### 실습 3 — Deprecated 경고 추가하기

v1 API는 앞으로 없어질 거라고 사용자에게 알려주는 헤더를 추가해 보세요.

```python
# my_api/routers/v1/products.py (수정)
from fastapi import APIRouter, Response
from pydantic import BaseModel

router = APIRouter()

class ProductV1(BaseModel):
    id: int
    name: str
    price: int

_products = [
    {"id": 1, "name": "노트북", "price": 1200000},
]

@router.get("/products", response_model=list[ProductV1])
def list_products(response: Response):
    # 클라이언트에게 v1이 곧 종료됨을 알림
    response.headers["Deprecation"] = "true"
    response.headers["Sunset"] = "2027-01-01"
    response.headers["Link"] = '</api/v2/products>; rel="successor-version"'
    return _products
```

```bash
# 헤더 확인
curl -I http://localhost:8000/api/v1/products
# Deprecation: true
# Sunset: 2027-01-01
# Link: </api/v2/products>; rel="successor-version"
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| `include_router` 순서 착각 | v1과 v2가 뒤바뀌어 작동함 | `prefix="/api/v1"`과 `prefix="/api/v2"` 값을 다시 확인하세요 |
| 헤더 이름에 대문자 사용 | `422 Unprocessable Entity` | FastAPI 헤더는 소문자로 변환됩니다. `Accept-Version` → `accept_version` |
| 쿼리 파라미터 버전 검증 누락 | 잘못된 버전(`version=99`)을 넣어도 응답이 옴 | `Query(ge=1, le=2)` 또는 `HTTPException`으로 명시적 검증 추가 |
| v1 코드를 v2에서 직접 수정 | 기존 클라이언트가 갑자기 동작하지 않음 | v1 파일은 절대 수정하지 말고, v2 파일을 새로 만드세요 |
| `ModuleNotFoundError: No module named 'routers'` | 서버 실행 시 import 에러 | `uvicorn`을 `my_api/` 폴더 안에서 실행하세요: `cd my_api && uvicorn main:app` |
| Pydantic 모델 누락 필드 | `ValidationError: field required` | v2 모델에 새 필드를 추가했으면, DB 데이터에도 해당 키가 있는지 확인하세요 |

---

## 확인 체크리스트

- [ ] URL 경로 버저닝(`/v1/`, `/v2/`)으로 동작하는 API를 만들었다
- [ ] v1과 v2가 서로 다른 응답 구조를 반환함을 curl로 확인했다
- [ ] `APIRouter`와 `include_router`로 버전별 파일을 분리했다
- [ ] v1 API에 `Deprecation` 헤더를 추가하고 curl로 헤더를 확인했다
- [ ] `/docs` (Swagger UI)에서 v1과 v2 엔드포인트가 태그로 분리되어 보인다
- [ ] 잘못된 버전 요청 시 명확한 에러 메시지가 반환된다

---

## 한 번 더 생각해 보기

1. **v1 API를 언제까지 유지해야 할까요?** 당장 v2를 만들었다고 v1을 바로 없애면 어떤 문제가 생길까요? 실제 서비스에서는 폐기(deprecation) 기간을 얼마나 두는 것이 적당할까요?

2. **URL 버저닝, 쿼리 파라미터, 헤더 버저닝 중 어느 것을 언제 쓸까요?** 모바일 앱 개발자와 협업하는 경우, 브라우저에서 직접 테스트하는 경우, 마이크로서비스 간 통신의 경우 각각 어떤 방식이 더 편리할지 생각해 보세요.

3. **같은 데이터를 두 곳(v1 파일, v2 파일)에서 관리하면 유지보수가 힘들어집니다.** 실습의 `_products` 리스트처럼 데이터가 중복될 때, 이를 어떻게 하나의 공유 계층으로 합칠 수 있을까요?

---

## 다음 장

다음 장에서는 API에 **인증(Authentication)과 권한 부여(Authorization)** 를 추가하는 방법을 배웁니다 — JWT 토큰으로 로그인 기능을 구현하고, 특정 API를 인증된 사용자만 호출할 수 있도록 보호하는 실습을 진행합니다.