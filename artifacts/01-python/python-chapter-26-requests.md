## 이 장에서 배우는 것

- `requests` 라이브러리를 설치하고 사용하는 방법
- HTTP GET, POST 요청을 보내는 방법
- 응답(response) 데이터를 읽고 JSON으로 파싱하는 방법
- 요청 헤더(header)와 쿼리 파라미터를 설정하는 방법
- 오류 상황을 감지하고 처리하는 방법
- `Session`을 사용해 요청을 효율적으로 관리하는 방법

---

## 먼저 쉬운 설명

인터넷에서 날씨 정보를 가져오거나, 번역 API를 호출하거나, 로그인이 필요한 서비스에 자동으로 데이터를 보내야 할 때가 있습니다. 이런 일을 하려면 **HTTP 요청**을 코드에서 직접 보내야 합니다.

브라우저가 웹사이트를 열 때 하는 일이 바로 HTTP 요청입니다. Python의 `requests` 라이브러리는 이 과정을 아주 간단하게 만들어 줍니다. 마치 브라우저 대신 코드가 직접 인터넷에 "이 페이지 좀 주세요" 하고 말하는 것과 같습니다.

실무에서는 외부 API 연동, 데이터 수집, 자동화 스크립트 작성 등에 매일 쓰이는 기술이니 꼭 익혀 두세요.

---

## 1. requests 설치와 기본 GET 요청

### 설치

```bash
pip install requests
```

### 가장 간단한 GET 요청

```python
# weather_check.py
import requests

response = requests.get("https://httpbin.org/get")

print("상태 코드:", response.status_code)   # 200이면 성공
print("응답 내용:", response.text[:200])    # 응답 본문 (문자열)
```

**실행 결과 예시:**
```
상태 코드: 200
응답 내용: {
  "args": {},
  "headers": {
    "Accept": "*/*",
    ...
  }
}
```

> `status_code`가 `200`이면 성공입니다. `404`는 주소를 못 찾은 것, `500`은 서버 오류입니다.

---

## 2. JSON 응답 파싱하기

대부분의 API는 JSON 형식으로 데이터를 돌려줍니다. `.json()` 메서드를 쓰면 Python 딕셔너리로 바로 변환됩니다.

```python
# exchange_rate.py
import requests

# 환율 정보를 제공하는 공개 API (예시)
url = "https://httpbin.org/json"
response = requests.get(url)

# JSON 파싱
data = response.json()
print(type(data))    # <class 'dict'>
print(data)
```

### 실제 API 응답에서 원하는 값 꺼내기

```python
# user_info.py
import requests

response = requests.get("https://jsonplaceholder.typicode.com/users/1")
user = response.json()

# 딕셔너리처럼 접근
print("이름:", user["name"])
print("이메일:", user["email"])
print("도시:", user["address"]["city"])
```

**실행 결과:**
```
이름: Leanne Graham
이메일: Sincere@april.biz
도시: Gwenborough
```

---

## 3. 쿼리 파라미터 전달하기

URL 뒤에 `?key=value` 형태로 붙이는 값을 **쿼리 파라미터**라고 합니다. 검색어나 필터 조건을 전달할 때 씁니다.

```python
# search_posts.py
import requests

# 방법 1: URL에 직접 쓰기 (비추천 — 한글, 특수문자 처리 번거로움)
# response = requests.get("https://jsonplaceholder.typicode.com/posts?userId=1")

# 방법 2: params 딕셔너리로 전달 (추천)
params = {
    "userId": 1,
    "_limit": 3,   # 결과를 3개만 가져오기
}

response = requests.get(
    "https://jsonplaceholder.typicode.com/posts",
    params=params,
)

posts = response.json()
print(f"가져온 게시글 수: {len(posts)}")
for post in posts:
    print(f"  - [{post['id']}] {post['title'][:30]}...")
```

**실행 결과:**
```
가져온 게시글 수: 3
  - [1] sunt aut facere repellat provi...
  - [2] qui est esse...
  - [3] ea molestias quasi exercitatio...
```

---

## 4. POST 요청으로 데이터 보내기

서버에 새 데이터를 **생성**하거나 **로그인 정보**를 전송할 때는 POST 요청을 씁니다.

### JSON 데이터 전송 (API에서 가장 흔한 방식)

```python
# create_post.py
import requests

new_post = {
    "title": "Python requests 공부 중",
    "body": "오늘 HTTP 클라이언트를 배웠습니다.",
    "userId": 1,
}

response = requests.post(
    "https://jsonplaceholder.typicode.com/posts",
    json=new_post,   # json= 을 쓰면 Content-Type 헤더도 자동 설정됨
)

print("상태 코드:", response.status_code)   # 201 Created
created = response.json()
print("생성된 ID:", created["id"])
print("제목:", created["title"])
```

**실행 결과:**
```
상태 코드: 201
생성된 ID: 101
제목: Python requests 공부 중
```

> `json=` 파라미터를 쓰면 딕셔너리를 자동으로 JSON 문자열로 변환하고, `Content-Type: application/json` 헤더도 자동으로 붙여 줍니다.

---

## 5. 요청 헤더 설정하기

API 키나 인증 토큰을 전달하거나, 요청의 형식을 명시할 때 헤더를 사용합니다.

```python
# authenticated_request.py
import requests

API_TOKEN = "my-secret-token-1234"   # 실제로는 환경변수로 관리

headers = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Accept": "application/json",
    "User-Agent": "MyApp/1.0",
}

response = requests.get(
    "https://httpbin.org/headers",
    headers=headers,
)

data = response.json()
# 서버가 받은 헤더 확인
print(data["headers"]["Authorization"])
print(data["headers"]["User-Agent"])
```

---

## 6. 오류 처리하기

네트워크 문제나 잘못된 요청은 반드시 처리해야 합니다.

```python
# safe_request.py
import requests
from requests.exceptions import ConnectionError, Timeout, HTTPError

def 게시글_가져오기(post_id: int) -> dict | None:
    url = f"https://jsonplaceholder.typicode.com/posts/{post_id}"

    try:
        response = requests.get(url, timeout=5)   # 5초 안에 응답 없으면 포기
        response.raise_for_status()               # 4xx, 5xx면 예외 발생
        return response.json()

    except Timeout:
        print("오류: 서버 응답이 너무 느립니다.")
    except ConnectionError:
        print("오류: 인터넷 연결을 확인해 주세요.")
    except HTTPError as e:
        print(f"오류: HTTP {e.response.status_code} — {e}")

    return None


post = 게시글_가져오기(1)
if post:
    print("제목:", post["title"])

없는_게시글 = 게시글_가져오기(99999)   # 404 발생
```

**실행 결과:**
```
제목: sunt aut facere repellat provident occaecati excepturi optio reprehenderit
오류: HTTP 404 — 404 Client Error: Not Found for url: ...
```

---

## 7. Session으로 효율적으로 요청하기

같은 서버에 여러 번 요청할 때는 `Session`을 쓰면 연결을 재사용하고 헤더나 쿠키를 공통으로 설정할 수 있습니다.

```python
# session_example.py
import requests

# Session 없이: 매 요청마다 새 연결
# Session 있이: 연결 재사용 + 공통 설정 적용

with requests.Session() as session:
    # 모든 요청에 공통 헤더 적용
    session.headers.update({
        "Authorization": "Bearer my-token",
        "Accept": "application/json",
    })

    # 여러 요청을 보내도 헤더는 한 번만 설정
    user = session.get("https://jsonplaceholder.typicode.com/users/1").json()
    posts = session.get(
        "https://jsonplaceholder.typicode.com/posts",
        params={"userId": 1, "_limit": 2},
    ).json()

    print(f"사용자: {user['name']}")
    print(f"게시글 수: {len(posts)}")
```

---

## 따라 하기 실습

### 실습 1 — 공개 API에서 데이터 가져오기

`todo_viewer.py` 파일을 만들어 다음을 구현하세요.

```python
# todo_viewer.py
import requests

def 할일_목록_가져오기(user_id: int) -> list[dict]:
    response = requests.get(
        "https://jsonplaceholder.typicode.com/todos",
        params={"userId": user_id, "_limit": 5},
        timeout=5,
    )
    response.raise_for_status()
    return response.json()

todos = 할일_목록_가져오기(1)
for todo in todos:
    완료_여부 = "✔" if todo["completed"] else "✘"
    print(f"[{완료_여부}] {todo['title']}")
```

**기대 출력:**
```
[✔] delectus aut autem
[✘] quis ut nam facilis et officia qui
[✔] fugiat veniam minus
[✔] et porro tempora
[✘] laboriosam mollitia et enim quasi ...
```

---

### 실습 2 — POST로 댓글 작성하기

실습 1 파일에 이어서 `comment_writer.py`를 만드세요.

```python
# comment_writer.py
import requests

def 댓글_작성(post_id: int, 작성자: str, 내용: str) -> dict:
    payload = {
        "postId": post_id,
        "name": 작성자,
        "body": 내용,
        "email": "test@example.com",
    }
    response = requests.post(
        "https://jsonplaceholder.typicode.com/comments",
        json=payload,
        timeout=5,
    )
    response.raise_for_status()
    return response.json()

결과 = 댓글_작성(
    post_id=1,
    작성자="김파이썬",
    내용="requests 라이브러리 공부 중입니다!",
)
print(f"댓글 ID: {결과['id']}")
print(f"작성자: {결과['name']}")
```

---

### 실습 3 — 오류 처리까지 통합하기

`api_client.py`를 만들어 실습 1, 2를 하나의 클래스로 묶고 오류 처리를 추가하세요.

```python
# api_client.py
import requests
from requests.exceptions import RequestException

class JSONPlaceholderClient:
    BASE_URL = "https://jsonplaceholder.typicode.com"

    def __init__(self):
        self.session = requests.Session()
        self.session.headers.update({"Accept": "application/json"})

    def 할일_가져오기(self, user_id: int) -> list[dict]:
        try:
            r = self.session.get(
                f"{self.BASE_URL}/todos",
                params={"userId": user_id, "_limit": 3},
                timeout=5,
            )
            r.raise_for_status()
            return r.json()
        except RequestException as e:
            print(f"요청 실패: {e}")
            return []

    def 댓글_쓰기(self, post_id: int, 내용: str) -> dict | None:
        try:
            r = self.session.post(
                f"{self.BASE_URL}/comments",
                json={"postId": post_id, "body": 내용, "name": "test", "email": "a@b.com"},
                timeout=5,
            )
            r.raise_for_status()
            return r.json()
        except RequestException as e:
            print(f"댓글 쓰기 실패: {e}")
            return None


client = JSONPlaceholderClient()

todos = client.할일_가져오기(1)
for t in todos:
    print(f"할일: {t['title']}")

댓글 = client.댓글_쓰기(1, "Session으로 요청하니 편하네요!")
if 댓글:
    print(f"댓글 생성 완료, ID: {댓글['id']}")
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|-----------|
| `requests` 미설치 상태로 import | `ModuleNotFoundError: No module named 'requests'` | `pip install requests` 실행 |
| `timeout` 미설정으로 무한 대기 | 프로그램이 멈추고 응답 없음 | `requests.get(url, timeout=5)` 처럼 항상 timeout 지정 |
| `.json()` 호출했는데 오류 발생 | `requests.exceptions.JSONDecodeError` | `response.text`로 원본 확인 후 API가 실제 JSON 반환하는지 확인 |
| 4xx/5xx 상태 코드를 오류로 처리 안 함 | 에러 없이 이상한 데이터 사용됨 | `response.raise_for_status()` 항상 호출 |
| `data=` 와 `json=` 혼동 | 서버가 데이터를 인식 못 함 | JSON API면 `json=딕셔너리`, 폼 전송이면 `data=딕셔너리` 사용 |
| HTTPS URL을 HTTP로 잘못 입력 | `ConnectionError` 또는 리다이렉트 루프 | URL이 `https://`로 시작하는지 확인 |
| 응답을 변수에 저장 안 하고 바로 `.json()` | `AttributeError: 'NoneType' object has no attribute 'json'` | `response = requests.get(...)` 으로 변수에 먼저 저장 |

---

## 확인 체크리스트

- [ ] `pip install requests` 로 라이브러리를 설치할 수 있다
- [ ] `requests.get()` 으로 GET 요청을 보내고 `response.status_code` 를 확인할 수 있다
- [ ] `response.json()` 으로 응답을 딕셔너리로 파싱할 수 있다
- [ ] `params=` 로 쿼리 파라미터를 전달할 수 있다
- [ ] `requests.post(url, json=데이터)` 로 POST 요청을 보낼 수 있다
- [ ] `headers=` 로 인증 토큰 등 요청 헤더를 설정할 수 있다
- [ ] `timeout=` 을 설정해 무한 대기를 방지할 수 있다
- [ ] `raise_for_status()` 와 `try/except` 로 오류를 처리할 수 있다
- [ ] `requests.Session()` 을 사용해 공통 헤더를 설정하고 연결을 재사용할 수 있다

---

## 한 번 더 생각해 보기

1. `requests.get()` 을 쓸 때 `timeout` 을 설정하지 않으면 어떤 문제가 생길 수 있을까요? 실제 서비스에서는 어떤 영향을 미칠까요?

2. API 키나 인증 토큰을 코드 안에 직접 쓰는 것(`API_TOKEN = "abc123"`)은 왜 위험할까요? 어떻게 하면 안전하게 관리할 수 있을까요?

3. `data=딕셔너리` 와 `json=딕셔너리` 는 어떤 차이가 있을까요? 어떤 상황에서 각각을 쓰는 것이 맞을까요?

---

## 다음 장

다음 장에서는 `requests`로 가져온 데이터를 파일로 저장하고, CSV·JSON 형식으로 구조화하는 방법을 배웁니다.