# Chapter 17: 입력 검증과 살균 (Input Validation)

## 이 장에서 배우는 것

- 사용자 입력이 왜 위험한지 이해하기
- `pydantic`으로 입력 타입과 범위를 검증하는 방법
- 문자열을 안전하게 정제(sanitize)하는 패턴
- FastAPI에서 검증 오류를 깔끔하게 처리하는 방법
- SQL 인젝션과 XSS 같은 기본 보안 위협 막기

---

## 먼저 쉬운 설명

여러분이 카페에서 주문을 받는다고 상상해 보세요. 손님이 "아메리카노 -5잔 주세요"라고 하거나, 이름란에 `'; DROP TABLE orders;--` 를 적어 넣는다면 어떻게 될까요?

API도 똑같습니다. 외부에서 들어오는 모든 데이터는 **믿을 수 없습니다.** 실수로 잘못된 값을 보낼 수도 있고, 악의적으로 시스템을 망가뜨리려는 시도일 수도 있습니다.

입력 검증(validation)은 "이 값이 올바른 형식인가?"를 확인하는 것이고, 정제(sanitization)는 "위험한 문자를 제거하거나 무력화하는 것"입니다. 이 두 가지가 API 보안의 첫 번째 방어선입니다.

---

## 1. Pydantic으로 입력 타입 검증하기

Pydantic은 Python 데이터 클래스에 타입 힌트를 붙이면 자동으로 값을 검증해 주는 라이브러리입니다. FastAPI는 내부적으로 Pydantic을 사용합니다.

```python
# models/user.py
from pydantic import BaseModel, Field, EmailStr
from typing import Optional

class UserCreateRequest(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    email: EmailStr
    age: int = Field(ge=0, le=150)  # ge=이상, le=이하
    bio: Optional[str] = Field(default=None, max_length=500)
```

```python
# 올바른 데이터 → 통과
user = UserCreateRequest(
    username="kihyuk",
    email="kihyuk@example.com",
    age=30,
)

# 잘못된 데이터 → ValidationError 발생
user = UserCreateRequest(
    username="k",          # 너무 짧음 (min_length=3 위반)
    email="not-an-email",  # 이메일 형식 아님
    age=200,               # 범위 초과 (le=150 위반)
)
```

실제 오류 메시지:
```
pydantic_core._pydantic_core.ValidationError: 3 validation errors for UserCreateRequest
username
  String should have at least 3 characters [type=string_too_short]
email
  value is not a valid email address [type=value_error]
age
  Input should be less than or equal to 150 [type=less_than_equal]
```

---

## 2. FastAPI 라우터에서 검증 오류 처리하기

FastAPI는 Pydantic 오류를 자동으로 `422 Unprocessable Entity`로 변환하지만, 응답 형식을 직접 제어하고 싶을 때는 예외 핸들러를 등록합니다.

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(
    request: Request,
    exc: RequestValidationError,
):
    # 오류 목록을 사람이 읽기 쉬운 형태로 변환
    errors = []
    for error in exc.errors():
        field = " → ".join(str(loc) for loc in error["loc"])
        errors.append({
            "field": field,
            "message": error["msg"],
        })
    return JSONResponse(
        status_code=422,
        content={"ok": False, "errors": errors},
    )
```

```python
# routers/users.py
from fastapi import APIRouter
from models.user import UserCreateRequest

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/")
async def create_user(body: UserCreateRequest):
    # 여기까지 왔다면 body는 이미 검증된 상태입니다
    return {"ok": True, "username": body.username}
```

---

## 3. 문자열 정제(Sanitization) 패턴

검증을 통과한 값이라도 위험한 문자가 포함될 수 있습니다. 특히 HTML이나 SQL을 다룰 때는 반드시 정제가 필요합니다.

```python
# utils/sanitize.py
import re
import html

def sanitize_text(value: str) -> str:
    """HTML 특수문자를 이스케이프하고 제어 문자를 제거합니다."""
    # 1단계: HTML 이스케이프 (<script> → &lt;script&gt;)
    value = html.escape(value)
    # 2단계: 출력 불가능한 제어 문자 제거 (탭·줄바꿈 제외)
    value = re.sub(r"[\x00-\x08\x0b\x0c\x0e-\x1f\x7f]", "", value)
    return value.strip()

def sanitize_filename(value: str) -> str:
    """파일 이름에서 경로 탐색 문자를 제거합니다."""
    # ../, ..\, 절대 경로 시도를 모두 제거
    value = re.sub(r"[/\\]", "", value)
    value = re.sub(r"\.{2,}", "", value)
    return value.strip()
```

```python
# 사용 예시
from utils.sanitize import sanitize_text

user_input = "<script>alert('해킹')</script> 안녕하세요"
safe = sanitize_text(user_input)
print(safe)
# 출력: &lt;script&gt;alert(&#x27;해킹&#x27;)&lt;/script&gt; 안녕하세요
```

---

## 4. SQL 인젝션 방지 — ORM과 파라미터 바인딩

직접 SQL 문자열을 조합하는 것은 매우 위험합니다. SQLAlchemy ORM 또는 파라미터 바인딩을 항상 사용하세요.

```python
# ❌ 절대 이렇게 하지 마세요
async def bad_search(keyword: str, db):
    # keyword = "' OR '1'='1" 이면 전체 데이터가 노출됩니다
    query = f"SELECT * FROM users WHERE username = '{keyword}'"
    result = await db.execute(query)

# ✅ ORM을 사용하는 올바른 방법
from sqlalchemy import select
from models.db import User

async def safe_search(keyword: str, db):
    stmt = select(User).where(User.username == keyword)
    result = await db.execute(stmt)
    return result.scalars().all()

# ✅ 원시 SQL이 꼭 필요하다면 파라미터 바인딩 사용
from sqlalchemy import text

async def safe_raw_search(keyword: str, db):
    stmt = text("SELECT * FROM users WHERE username = :keyword")
    result = await db.execute(stmt, {"keyword": keyword})
    return result.fetchall()
```

---

## 5. Pydantic 커스텀 검증기(validator) 사용하기

`Field`만으로는 부족한 복잡한 규칙은 `@field_validator`로 직접 구현합니다.

```python
# models/post.py
from pydantic import BaseModel, Field, field_validator
import re

FORBIDDEN_WORDS = {"스팸", "광고", "클릭하세요"}

class PostCreateRequest(BaseModel):
    title: str = Field(min_length=2, max_length=100)
    content: str = Field(min_length=10, max_length=5000)
    tags: list[str] = Field(default_factory=list, max_length=5)

    @field_validator("title")
    @classmethod
    def title_must_not_be_spam(cls, value: str) -> str:
        for word in FORBIDDEN_WORDS:
            if word in value:
                raise ValueError(f"제목에 '{word}'를 사용할 수 없습니다.")
        return value

    @field_validator("tags", mode="before")
    @classmethod
    def tags_must_be_lowercase(cls, value: list) -> list:
        return [tag.strip().lower() for tag in value]
```

```python
# 검증 실패 예시
PostCreateRequest(
    title="광고 클릭하세요 대박 할인",
    content="이 상품을 지금 바로 구매하세요.",
    tags=["PYTHON", " FastAPI "],
)
# ValueError: 제목에 '광고'를 사용할 수 없습니다.
```

---

## 따라 하기 실습

### 실습 1 — 회원가입 API 검증 모델 만들기

`models/auth.py` 파일을 만들고 아래 규칙을 모두 적용한 `RegisterRequest` 모델을 작성하세요.

- `username`: 영문자, 숫자, 밑줄(`_`)만 허용, 3~20자
- `password`: 최소 8자, 대문자·숫자 각 1개 이상 포함
- `email`: 유효한 이메일 형식

```python
# models/auth.py
from pydantic import BaseModel, Field, EmailStr, field_validator
import re

class RegisterRequest(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    password: str = Field(min_length=8)
    email: EmailStr

    @field_validator("username")
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        if not re.fullmatch(r"[a-zA-Z0-9_]+", v):
            raise ValueError("username은 영문자, 숫자, _만 사용할 수 있습니다.")
        return v

    @field_validator("password")
    @classmethod
    def password_strength(cls, v: str) -> str:
        if not re.search(r"[A-Z]", v):
            raise ValueError("비밀번호에 대문자가 1개 이상 포함되어야 합니다.")
        if not re.search(r"\d", v):
            raise ValueError("비밀번호에 숫자가 1개 이상 포함되어야 합니다.")
        return v
```

### 실습 2 — FastAPI 라우터에 연결하기

`routers/auth.py`를 만들어 실습 1의 모델을 실제 엔드포인트에 연결하세요. 검증이 통과된 경우에만 성공 응답을 반환합니다.

```python
# routers/auth.py
from fastapi import APIRouter
from models.auth import RegisterRequest
from utils.sanitize import sanitize_text

router = APIRouter(prefix="/auth", tags=["auth"])

@router.post("/register")
async def register(body: RegisterRequest):
    # username은 추가로 HTML 정제까지 적용
    safe_username = sanitize_text(body.username)
    return {
        "ok": True,
        "message": f"{safe_username}님, 가입을 환영합니다!",
    }
```

터미널에서 테스트해 보세요.

```bash
# 성공 케이스
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"kihyuk_dev","password":"Secret123","email":"k@example.com"}'

# 실패 케이스 (비밀번호 규칙 위반)
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"kihyuk","password":"weakpass","email":"k@example.com"}'
```

### 실습 3 — 검증 오류 응답 형식 통일하기

`main.py`에 실습 2에서 확인한 422 오류를 커스텀 핸들러로 처리해, 아래와 같은 JSON 형식으로 응답하게 만드세요.

```json
{
  "ok": false,
  "errors": [
    { "field": "body → password", "message": "비밀번호에 대문자가 1개 이상 포함되어야 합니다." }
  ]
}
```

앞서 배운 `validation_exception_handler` 코드를 `main.py`에 붙여 넣고, 실습 2의 실패 케이스를 다시 실행해 응답이 바뀌는 것을 확인하세요.

---

## 자주 하는 실수

| 실수 | 발생하는 오류 / 증상 | 해결 방법 |
|------|---------------------|-----------|
| `Field`에 `min_length` 대신 `min`을 씀 | `pydantic_core.SchemaError: 'min' is not permitted` | 문자열엔 `min_length`, 숫자엔 `ge` 사용 |
| `EmailStr` 사용 시 `email-validator` 미설치 | `ImportError: email-validator is not installed` | `pip install "pydantic[email]"` 실행 |
| `@field_validator`에서 `@classmethod` 누락 | `PydanticUserError: validator functions should be class methods` | 반드시 `@classmethod`를 함께 붙이기 |
| f-string으로 SQL 직접 조합 | 데이터 전체 노출, 데이터 삭제 등 심각한 보안 사고 | SQLAlchemy ORM 또는 `:param` 바인딩 사용 |
| 검증만 하고 정제를 하지 않음 | XSS 공격 성공, 렌더링 시 스크립트 실행 | `html.escape()` 또는 전용 라이브러리 사용 |
| `Optional` 필드에 `None` 허용 여부를 명시 안 함 | `ValidationError: Input should be a valid string, not None` | `Optional[str] = None`으로 명시 |

---

## 확인 체크리스트

- [ ] `BaseModel`을 상속한 요청 모델을 별도 파일(`models/`)에 작성했다
- [ ] 문자열 필드에 `min_length`와 `max_length`를 모두 지정했다
- [ ] 숫자 필드에 `ge` 또는 `le`로 범위를 제한했다
- [ ] 이메일 필드에 `EmailStr`을 사용하고 `pydantic[email]`을 설치했다
- [ ] `@field_validator`를 작성할 때 `@classmethod`를 함께 붙였다
- [ ] `html.escape()`를 사용해 사용자 입력의 HTML 특수문자를 이스케이프했다
- [ ] SQL 쿼리에서 f-string 대신 ORM 또는 파라미터 바인딩을 사용했다
- [ ] 422 오류 응답이 일관된 JSON 형식으로 반환되도록 핸들러를 등록했다
- [ ] `curl` 또는 Swagger UI(`/docs`)로 성공·실패 케이스를 모두 직접 테스트했다

---

## 한 번 더 생각해 보기

1. **검증과 정제의 차이는 무엇인가요?** 검증만 통과한 값이 여전히 위험할 수 있는 상황을 하나 떠올려 보세요. 어떤 추가 조치가 필요할까요?

2. **`@field_validator`는 언제 쓰고, `Field()`의 파라미터는 언제 쓰면 좋을까요?** 두 방법을 모두 사용해야 할 때와 하나만으로 충분할 때를 구분해 보세요.

3. **프런트엔드에서도 똑같은 검증을 하는데, 백엔드에서 다시 해야 하나요?** 프런트엔드 검증을 우회하는 방법을 생각해 보고, 백엔드 검증이 반드시 필요한 이유를 정리해 보세요.

---

## 다음 장

다음 장에서는 인증(Authentication)과 JWT 토큰을 사용해 로그인한 사용자만 API를 호출할 수 있도록 보호하는 방법을 배웁니다.