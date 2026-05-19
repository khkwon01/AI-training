## 이 장에서 배우는 것

- 풀 스택 서비스가 무엇인지, 어떤 부분들로 이루어져 있는지 이해한다
- 프론트엔드, 백엔드, 데이터베이스가 어떻게 연결되는지 그림으로 이해한다
- 실제 서비스가 요청을 받고 응답을 돌려주는 흐름을 코드로 따라간다
- 각 레이어가 왜 분리되어 있는지, 그 이유를 설명할 수 있다

---

## 먼저 쉬운 설명

카페를 생각해 보세요.

손님이 "아메리카노 주세요"라고 말하면, 직원이 주문을 받고, 바리스타가 커피를 만들고, 재료는 창고에서 꺼냅니다. 손님은 커피가 어떻게 만들어지는지 몰라도 됩니다. 그냥 주문하면 커피가 나옵니다.

웹 서비스도 똑같습니다.

- **손님** → 브라우저(프론트엔드)
- **직원** → 서버(백엔드)
- **창고** → 데이터베이스

이 세 가지가 함께 움직이는 것을 **풀 스택 서비스**라고 합니다. 이 장에서는 이 흐름을 코드로 직접 확인합니다.

---

## 1. 전체 구조 한눈에 보기

풀 스택 서비스는 세 개의 레이어로 나뉩니다.

```
[브라우저 / 앱]          ← 프론트엔드 (사용자가 보는 화면)
       ↕  HTTP 요청/응답
[API 서버]               ← 백엔드 (비즈니스 로직)
       ↕  SQL / NoSQL 쿼리
[데이터베이스]            ← 데이터 저장소
```

각 레이어는 **자기 역할만** 합니다.

- 프론트엔드는 화면을 그리고 사용자 입력을 받는다
- 백엔드는 요청을 처리하고 규칙을 적용한다
- 데이터베이스는 데이터를 안전하게 보관한다

왜 굳이 나눌까요? 한 레이어만 수정해도 나머지는 영향이 없기 때문입니다. 데이터베이스를 바꿔도 프론트엔드 코드는 그대로입니다.

---

## 2. 백엔드: API 서버 만들기

백엔드는 프론트엔드의 요청을 받아서 처리하고 결과를 돌려줍니다. Python의 FastAPI로 간단한 API를 만들어 봅니다.

```python
# app/main.py
from fastapi import FastAPI

app = FastAPI()

# 가상의 데이터 (나중에 실제 DB로 교체)
users_db = [
    {"id": 1, "name": "김철수", "email": "chulsoo@example.com"},
    {"id": 2, "name": "이영희", "email": "younghee@example.com"},
]

@app.get("/")
def health_check():
    """서버가 살아있는지 확인하는 엔드포인트"""
    return {"status": "ok", "message": "서버가 실행 중입니다"}

@app.get("/users")
def get_users():
    """전체 사용자 목록 반환"""
    return {"users": users_db}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    """특정 사용자 한 명 반환"""
    for user in users_db:
        if user["id"] == user_id:
            return {"user": user}
    return {"error": "사용자를 찾을 수 없습니다"}, 404
```

서버를 실행하면:

```bash
$ uvicorn app.main:app --reload

INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete.
```

브라우저에서 `http://127.0.0.1:8000/users`를 열면 JSON 응답이 보입니다.

---

## 3. 데이터베이스: 데이터를 영구적으로 저장하기

메모리에만 있는 데이터는 서버를 재시작하면 사라집니다. 데이터베이스를 연결해야 합니다.

```python
# app/database.py
import sqlite3

DB_PATH = "service.db"

def get_connection():
    """데이터베이스 연결 반환"""
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row  # 결과를 딕셔너리처럼 사용
    return conn

def init_db():
    """테이블이 없으면 생성"""
    conn = get_connection()
    conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id    INTEGER PRIMARY KEY AUTOINCREMENT,
            name  TEXT    NOT NULL,
            email TEXT    NOT NULL UNIQUE
        )
    """)
    conn.commit()
    conn.close()
```

```python
# app/main.py (데이터베이스 연결 버전)
from fastapi import FastAPI
from app.database import get_connection, init_db

app = FastAPI()

@app.on_event("startup")
def startup():
    """앱 시작 시 DB 초기화"""
    init_db()

@app.get("/users")
def get_users():
    conn = get_connection()
    rows = conn.execute("SELECT * FROM users").fetchall()
    conn.close()
    return {"users": [dict(row) for row in rows]}

@app.post("/users")
def create_user(name: str, email: str):
    conn = get_connection()
    try:
        conn.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            (name, email)
        )
        conn.commit()
        return {"message": f"{name} 님이 등록되었습니다"}
    except Exception as e:
        return {"error": str(e)}
    finally:
        conn.close()
```

---

## 4. 프론트엔드: 사용자 화면 연결하기

프론트엔드는 API 서버에 HTTP 요청을 보내고 결과를 화면에 표시합니다.

```html
<!-- frontend/index.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>사용자 목록</title>
</head>
<body>
  <h1>사용자 목록</h1>
  <button id="load-btn">목록 불러오기</button>
  <ul id="user-list"></ul>

  <script>
    document.getElementById("load-btn").addEventListener("click", async () => {
      // 백엔드 API에 요청
      const response = await fetch("http://127.0.0.1:8000/users");
      const data = await response.json();

      // 화면에 표시
      const list = document.getElementById("user-list");
      list.innerHTML = "";
      data.users.forEach(user => {
        const item = document.createElement("li");
        item.textContent = `${user.name} (${user.email})`;
        list.appendChild(item);
      });
    });
  </script>
</body>
</html>
```

이제 세 레이어가 모두 연결되었습니다.

```
index.html  →  fetch("/users")  →  main.py  →  service.db
            ←  JSON 응답        ←           ←
```

---

## 5. 프로젝트 폴더 구조

실제 서비스에서는 파일을 레이어별로 정리합니다.

```
my-service/
├── app/
│   ├── main.py        ← API 라우터 (진입점)
│   ├── database.py    ← DB 연결과 초기화
│   └── models.py      ← 데이터 형태 정의
├── frontend/
│   └── index.html     ← 사용자 화면
├── requirements.txt   ← Python 패키지 목록
└── service.db         ← SQLite 데이터베이스 파일
```

이렇게 나누면 팀원이 프론트엔드만 수정할 때, 백엔드 코드를 건드릴 필요가 없습니다.

---

## 따라 하기 실습

### 실습 1 — 서버 실행하고 API 확인하기

프로젝트 폴더를 만들고 FastAPI를 설치합니다.

```bash
mkdir my-service
cd my-service
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install fastapi uvicorn
```

`app/main.py`를 위의 **섹션 2** 코드로 작성한 뒤 실행합니다.

```bash
uvicorn app.main:app --reload
```

브라우저에서 `http://127.0.0.1:8000/users`를 열어 JSON이 보이면 성공입니다.

---

### 실습 2 — 데이터베이스 연결하고 사용자 추가하기

`app/database.py`를 **섹션 3** 코드로 작성하고, `app/main.py`를 데이터베이스 연결 버전으로 교체합니다.

터미널에서 사용자를 추가해 봅니다.

```bash
# curl 또는 브라우저 주소창에 입력
curl -X POST "http://127.0.0.1:8000/users?name=김철수&email=chulsoo@example.com"
```

다시 `GET /users`를 호출해서 방금 추가한 사람이 보이면 성공입니다.

---

### 실습 3 — 프론트엔드 연결하기

`frontend/index.html`을 **섹션 4** 코드로 작성합니다.

파일을 브라우저로 직접 열면 CORS 오류가 납니다. 간단한 해결책으로 FastAPI에 CORS 설정을 추가합니다.

```python
# app/main.py 상단에 추가
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

서버를 재시작하고 `index.html`을 열어 "목록 불러오기" 버튼을 클릭합니다. 사용자 목록이 화면에 표시되면 풀 스택 연결 완성입니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|----------|
| `app` 디렉터리에 `__init__.py`가 없음 | `ModuleNotFoundError: No module named 'app'` | `app/` 폴더 안에 빈 `__init__.py` 파일 생성 |
| 서버가 꺼져 있는데 fetch 실행 | `TypeError: Failed to fetch` | `uvicorn app.main:app --reload` 로 서버 먼저 실행 |
| CORS 설정 없이 HTML 파일에서 fetch | `CORS policy: No 'Access-Control-Allow-Origin'` | FastAPI에 `CORSMiddleware` 추가 |
| DB 경로 오류 | `sqlite3.OperationalError: unable to open database file` | 실행 경로가 `my-service/` 인지 확인, `DB_PATH` 절대경로로 변경 |
| POST 요청 후 GET 해도 빈 목록 | (에러 없음, 빈 배열 반환) | `conn.commit()` 누락 확인 |
| `uvicorn` 명령어를 찾을 수 없음 | `command not found: uvicorn` | 가상환경이 활성화되어 있는지 확인 (`source venv/bin/activate`) |

---

## 확인 체크리스트

- [ ] 풀 스택의 세 레이어(프론트엔드, 백엔드, 데이터베이스)를 말로 설명할 수 있다
- [ ] `uvicorn`으로 FastAPI 서버를 실행할 수 있다
- [ ] 브라우저에서 API 응답 JSON을 직접 확인했다
- [ ] SQLite 데이터베이스에 사용자를 추가하고 조회했다
- [ ] `frontend/index.html`에서 백엔드 API를 fetch로 호출했다
- [ ] CORS 에러가 왜 발생하는지, 어떻게 해결하는지 설명할 수 있다
- [ ] 프로젝트 폴더 구조를 레이어별로 나눌 수 있다

---

## 한 번 더 생각해 보기

1. 프론트엔드와 백엔드를 같은 파일에 모두 작성하면 어떤 문제가 생길까요? 팀 규모가 커질수록 왜 더 어려워질까요?

2. 지금은 SQLite를 사용했는데, 사용자가 수천 명이 된다면 어떤 데이터베이스로 바꿔야 할까요? 무엇이 달라질까요?

3. `GET /users`는 모든 사람이 호출할 수 있습니다. 로그인한 사람만 볼 수 있게 하려면 어느 레이어에서 처리해야 할까요?

---

## 다음 장

다음 장에서는 이 서비스를 AWS에 배포해서 인터넷에서 실제로 접근할 수 있게 만드는 방법을 배웁니다.