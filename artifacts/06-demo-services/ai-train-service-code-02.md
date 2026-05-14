## 이 장에서 배우는 것

- 이미 만든 기능 옆에 새 기능을 자연스럽게 추가하는 방법
- 저장된 데이터를 목록으로 돌려주는 엔드포인트 만들기
- 리스트(list)를 JSON 응답으로 반환하는 패턴
- 기능을 하나씩 쌓아가는 "작은 단계 개발" 감각

---

## 먼저 쉬운 설명

지난 장에서 우리는 할 일 하나를 **추가**하는 기능을 만들었습니다.  
그런데 아무리 열심히 추가해도 "지금 뭐가 들어있지?" 를 볼 방법이 없으면 쓸 수가 없겠죠?

실제 서비스도 똑같습니다.  
쇼핑몰을 예로 들면 "상품 등록" 다음에 자연스럽게 필요한 것은 "상품 목록 보기"입니다.

이 장에서는 딱 그 한 가지만 추가합니다.  
**"저장된 것들을 전부 보여줘"** — 목록 조회 기능입니다.

---

## 1. 지난 장 코드를 떠올려 보기

지난 장에서 만든 `todo_service.py`는 이렇게 생겼습니다.

```python
# todo_service.py  (지난 장 결과물)
from flask import Flask, request, jsonify

app = Flask(__name__)

todos = []   # 메모리에 할 일 목록 저장

@app.route("/todos", methods=["POST"])
def add_todo():
    data = request.get_json()
    title = data["title"]
    todos.append({"id": len(todos) + 1, "title": title})
    return jsonify({"message": "추가 완료"}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

지금은 `POST /todos` 하나뿐입니다.  
여기에 `GET /todos` 를 **한 블록만** 추가할 겁니다.

---

## 2. 목록 조회 기능 추가하기

새로 추가할 코드는 단 5줄입니다.  
기존 `add_todo` 함수 **바로 아래**에 붙여 넣으면 됩니다.

```python
# todo_service.py  (이번 장에서 추가하는 부분만 표시)

@app.route("/todos", methods=["GET"])   # ← GET 방식으로 같은 주소 사용
def list_todos():
    return jsonify(todos), 200          # ← 리스트를 JSON으로 그대로 반환
```

**완성된 파일 전체**:

```python
# todo_service.py
from flask import Flask, request, jsonify

app = Flask(__name__)

todos = []

@app.route("/todos", methods=["POST"])
def add_todo():
    data = request.get_json()
    title = data["title"]
    todos.append({"id": len(todos) + 1, "title": title})
    return jsonify({"message": "추가 완료"}), 201

@app.route("/todos", methods=["GET"])   # ★ 새로 추가
def list_todos():                       # ★ 새로 추가
    return jsonify(todos), 200          # ★ 새로 추가

if __name__ == "__main__":
    app.run(debug=True)
```

> **포인트**: 주소(`/todos`)는 같지만, HTTP 메서드(`POST` / `GET`)로 역할을 구분합니다.  
> 이 규칙을 **REST** 라고 부릅니다. 지금은 "같은 주소, 다른 동작" 정도로만 기억해도 충분합니다.

---

## 3. 동작 흐름 눈으로 따라가기

```
클라이언트                   서버 (todo_service.py)
   │                               │
   │  POST /todos                  │
   │  { "title": "우유 사기" }  ──► │  todos 리스트에 추가
   │                    ◄──────── │  {"message": "추가 완료"}
   │                               │
   │  POST /todos                  │
   │  { "title": "운동하기" }   ──► │  todos 리스트에 추가
   │                    ◄──────── │  {"message": "추가 완료"}
   │                               │
   │  GET /todos              ──►  │  todos 리스트 전체 조회
   │                    ◄──────── │  [{"id":1,"title":"우유 사기"},
   │                               │   {"id":2,"title":"운동하기"}]
```

---

## 따라 하기 실습

### 실습 1 — 서버 실행하고 항목 추가하기

터미널 창 두 개를 엽니다.

**창 A (서버 실행)**:
```bash
python todo_service.py
```

**창 B (항목 추가)**:
```bash
curl -X POST http://127.0.0.1:5000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "물 마시기"}'

curl -X POST http://127.0.0.1:5000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "독서 30분"}'
```

예상 응답:
```json
{"message": "추가 완료"}
```

---

### 실습 2 — 목록 조회해 보기

```bash
curl http://127.0.0.1:5000/todos
```

예상 응답:
```json
[
  {"id": 1, "title": "물 마시기"},
  {"id": 2, "title": "독서 30분"}
]
```

빈 상태에서 조회하면 어떻게 될까요?  
서버를 재시작한 뒤 바로 아래 명령을 실행해 보세요.

```bash
curl http://127.0.0.1:5000/todos
```

예상 응답:
```json
[]
```

리스트가 비어 있을 때도 에러가 아니라 빈 배열 `[]` 이 돌아오는 것을 확인합니다.

---

### 실습 3 — 브라우저로도 확인하기

브라우저 주소창에 아래를 입력합니다:

```
http://127.0.0.1:5000/todos
```

`GET` 요청은 브라우저로도 바로 확인할 수 있습니다.  
항목을 몇 개 추가한 뒤 새로고침(F5)하면서 목록이 늘어나는 것을 눈으로 확인해 보세요.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `GET` 메서드를 적지 않고 `@app.route("/todos")` 만 씀 | POST 요청이 `list_todos`로 잘못 라우팅됨 | `methods=["GET"]` 을 반드시 명시 |
| `jsonify(todos)` 대신 `return todos` 로 씀 | `TypeError: The view function did not return a valid response` | Flask는 딕셔너리/리스트를 직접 반환하면 오류, `jsonify()` 로 감싸야 함 |
| 서버를 재시작하고 목록이 사라짐 | 응답이 `[]` | 정상 동작 — 메모리 저장이라 재시작하면 초기화됨. 영구 저장은 다음 장에서 배움 |
| `curl` 명령에서 포트 번호를 빠뜨림 | `curl: (7) Failed to connect` | `5000` 포트 포함 `http://127.0.0.1:5000/todos` |
| `Content-Type` 헤더 없이 POST 전송 | `400 Bad Request` 또는 `None` 반환 | `-H "Content-Type: application/json"` 추가 |

---

## 확인 체크리스트

- [ ] `todo_service.py` 에 `list_todos` 함수를 추가했다
- [ ] `@app.route("/todos", methods=["GET"])` 데코레이터를 정확히 작성했다
- [ ] `curl POST` 로 항목을 2개 이상 추가했다
- [ ] `curl GET` 으로 추가한 항목이 목록에 보이는 것을 확인했다
- [ ] 빈 목록일 때 `[]` 가 반환됨을 확인했다
- [ ] 브라우저에서 `http://127.0.0.1:5000/todos` 를 열어봤다
- [ ] POST와 GET이 같은 주소에서 다른 역할을 하는 이유를 한 문장으로 설명할 수 있다

---

## 한 번 더 생각해 보기

1. 지금은 서버를 재시작하면 목록이 사라집니다. 데이터를 영구적으로 유지하려면 무엇이 필요할까요?

2. 할 일이 100개라면 전부 돌려주는 것이 좋을까요, 아니면 10개씩 나눠서 돌려주는 것이 좋을까요? 그 이유는 무엇인가요?

3. `POST /todos` 와 `GET /todos` 가 같은 주소를 쓰는데 서버는 어떻게 두 요청을 구분할까요?

---

## 다음 장

다음 장에서는 **특정 할 일 하나를 ID로 조회하는** `GET /todos/<id>` 기능을 추가하면서, URL에 변수를 담는 방법을 배웁니다.