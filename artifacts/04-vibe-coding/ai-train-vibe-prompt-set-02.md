## 이 장에서 배우는 것

- AI(Claude, ChatGPT 등)에게 코드 설명을 요청하는 prompt를 작성하는 방법
- GitHub에 코드를 올리고 리뷰를 요청하는 prompt를 작성하는 방법
- AWS에 배포한 뒤 제대로 동작하는지 확인하는 prompt를 작성하는 방법
- 작은 서비스를 구현한 이후의 흐름(설명 → 리뷰 → 배포 검증)을 작은 prompt로 이어가는 방법

---

## 먼저 쉬운 설명

코드를 처음 작성했을 때 가장 두려운 순간은 "이게 맞는지 모르겠어"라는 느낌이 드는 순간입니다.

그런데 좋은 소식이 있습니다. AI에게 "이 코드 설명해 줘", "GitHub에 올리려면 어떻게 해?", "AWS에 배포했는데 잘 된 건지 확인해 줘" 같은 질문을 던지면, 마치 옆에 선배 개발자가 앉아 있는 것처럼 도움을 받을 수 있습니다.

이 장에서는 구현을 마친 뒤 이어지는 세 단계 — **설명 요청 → GitHub 리뷰 → AWS 검증** — 각각에 맞는 prompt를 배웁니다. 복사해서 바로 쓸 수 있도록 실제로 사용하는 형태로 적어 두었으니, 그대로 붙여넣고 자신의 코드에 맞게 한두 단어만 바꿔 보세요.

---

## 1. 코드 설명 요청 prompt

방금 작성한 코드가 어떻게 동작하는지 AI에게 물어보는 단계입니다. 코드를 이해해야 리뷰도 받을 수 있고, 배포 후 문제가 생겼을 때 스스로 고칠 수 있습니다.

### 기본 패턴

```
아래 코드를 초보자도 이해할 수 있게 한 줄씩 설명해 줘.
전문 용어가 나오면 쉬운 말로 바꿔서 설명해 줘.

[코드를 여기에 붙여넣기]
```

### 실전 예시 — Flask 할 일 목록 API

```python
# todo_app.py
from flask import Flask, request, jsonify

app = Flask(__name__)
todos = []

@app.route("/todos", methods=["GET"])
def get_todos():
    return jsonify(todos)

@app.route("/todos", methods=["POST"])
def add_todo():
    data = request.get_json()
    todos.append(data["title"])
    return jsonify({"message": "추가됨"}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

```
위의 todo_app.py 코드를 초보자도 이해할 수 있게 한 줄씩 설명해 줘.
@app.route 가 무엇인지, GET 과 POST 가 무엇인지도 함께 설명해 줘.
전문 용어가 나오면 쉬운 말로 바꿔서 설명해 줘.
```

### 흐름 이해 요청 prompt

```
이 코드에서 사용자가 /todos 에 POST 요청을 보내면
내부적으로 어떤 순서로 실행되는지 단계별로 설명해 줘.
```

---

## 2. GitHub 업로드 및 리뷰 요청 prompt

코드를 GitHub에 올리기 전에 AI에게 먼저 점검을 요청하고, 올린 뒤에는 리뷰 관점을 물어보는 단계입니다.

### GitHub 업로드 전 점검 prompt

```
아래 코드를 GitHub에 올리기 전에 확인해야 할 것들을 알려줘.
비밀번호나 API 키가 노출되어 있는지, 불필요한 파일이 포함되어 있는지 확인해 줘.

[코드 또는 파일 목록을 여기에 붙여넣기]
```

### .gitignore 생성 요청 prompt

```
Python Flask 프로젝트를 GitHub에 올릴 때 필요한 .gitignore 파일을 만들어 줘.
가상환경 폴더(venv), .env 파일, __pycache__ 폴더는 반드시 제외해 줘.
```

AI가 만들어 준 `.gitignore` 예시:

```gitignore
# .gitignore
venv/
.env
__pycache__/
*.pyc
.DS_Store
```

### 커밋 메시지 작성 요청 prompt

```
아래 변경 사항을 GitHub에 커밋할 때 쓸 메시지를 한국어로 만들어 줘.
변경 내용: 할 일 추가 API(/todos POST)와 조회 API(/todos GET)를 구현했다.
```

### PR(Pull Request) 설명 작성 요청 prompt

```
아래 코드 변경 사항을 바탕으로 GitHub Pull Request 설명을 작성해 줘.
- 무엇을 만들었는지
- 어떻게 테스트했는지
- 리뷰어가 특별히 봐줬으면 하는 부분

변경 내용: [변경 내용을 여기에 붙여넣기]
```

---

## 3. AWS 배포 검증 prompt

AWS에 배포한 뒤 "진짜로 잘 동작하는 건지" 확인하는 단계입니다. 배포 성공 기준을 AI에게 물어보고, 실제로 확인하는 방법도 함께 배웁니다.

### 배포 성공 기준 확인 prompt

```
Flask 앱을 AWS EC2에 배포했어.
배포가 성공했는지 확인하기 위한 체크리스트를 만들어 줘.
초보자도 따라할 수 있는 방법으로 설명해 줘.
```

### 배포 후 동작 확인 prompt

```
AWS EC2 퍼블릭 IP가 54.123.45.67 이야.
Flask 앱이 정상적으로 실행 중인지 확인하는 curl 명령어를 알려줘.
GET /todos 와 POST /todos 둘 다 테스트하고 싶어.
```

AI가 만들어 줄 명령어 예시:

```bash
# GET 요청 테스트
curl http://54.123.45.67:5000/todos

# POST 요청 테스트
curl -X POST http://54.123.45.67:5000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "장보기"}'
```

### 에러 발생 시 원인 파악 prompt

```
AWS EC2에서 Flask 앱을 실행했는데 아래 에러가 났어.
어떤 이유로 발생하는지, 어떻게 고치는지 초보자도 알 수 있게 설명해 줘.

에러 메시지:
[에러 메시지를 여기에 붙여넣기]
```

---

## 4. 전체 흐름을 이어주는 연결 prompt

설명 → GitHub → AWS 세 단계를 하나의 흐름으로 연결하는 prompt 세트입니다. 작은 서비스 하나를 처음부터 끝까지 배포할 때 이 순서대로 사용하면 됩니다.

```
[1단계 — 코드 이해]
이 코드가 어떻게 동작하는지 초보자도 알 수 있게 설명해 줘: [코드 붙여넣기]

[2단계 — GitHub 준비]
이 코드를 GitHub에 올리기 전에 확인해야 할 것들과 .gitignore 파일을 알려줘.

[3단계 — 배포 확인]
AWS EC2 IP [IP 주소] 에 Flask 앱을 배포했어.
정상 동작 여부를 확인하는 curl 명령어와 체크리스트를 만들어 줘.
```

---

## 따라 하기 실습

### 실습 1 — 코드 설명 요청해 보기

아래 파일을 그대로 만든 뒤, AI에게 설명을 요청해 봅니다.

**파일명:** `memo_app.py`

```python
# memo_app.py
from flask import Flask, request, jsonify

app = Flask(__name__)
memos = []

@app.route("/memos", methods=["GET"])
def get_memos():
    return jsonify(memos)

@app.route("/memos", methods=["POST"])
def add_memo():
    data = request.get_json()
    memos.append({"id": len(memos) + 1, "content": data["content"]})
    return jsonify({"message": "메모가 저장되었습니다."}), 201

if __name__ == "__main__":
    app.run(debug=True)
```

AI에게 보낼 prompt:

```
위의 memo_app.py 코드를 초보자도 이해할 수 있게 설명해 줘.
특히 jsonify 가 무엇인지, 201 숫자가 무슨 의미인지 알려줘.
```

---

### 실습 2 — GitHub 업로드 준비하기

실습 1에서 만든 `memo_app.py` 를 GitHub에 올리기 전에 점검합니다.

AI에게 보낼 prompt:

```
memo_app.py 를 GitHub에 올리려고 해.
1. .gitignore 파일에 어떤 내용을 넣어야 하는지 알려줘.
2. 커밋 메시지를 한국어로 만들어 줘. 변경 내용은 메모 추가/조회 API 구현이야.
```

AI가 만들어 준 내용을 바탕으로 아래 두 파일을 만들어 봅니다.

**파일명:** `.gitignore`

```gitignore
venv/
.env
__pycache__/
*.pyc
.DS_Store
```

**커밋 메시지 예시 (AI가 생성):**

```
feat: 메모 추가 및 조회 API 구현 (/memos GET, POST)
```

---

### 실습 3 — AWS 배포 검증 prompt 작성하기

실습 2까지 완료한 코드를 AWS EC2에 배포했다고 가정하고, 검증 prompt를 작성해 봅니다.

AI에게 보낼 prompt:

```
memo_app.py 를 AWS EC2 (IP: 13.124.0.1) 에 배포했어.
아래 두 가지를 확인하고 싶어:
1. 서버가 정상적으로 실행 중인지 확인하는 curl 명령어
2. 메모 추가가 잘 되는지 테스트하는 curl 명령어

초보자도 바로 복사해서 쓸 수 있게 알려줘.
```

AI가 생성한 명령어를 터미널에 직접 실행해 보고, 결과를 확인합니다.

```bash
# 1. 서버 상태 확인
curl http://13.124.0.1:5000/memos

# 2. 메모 추가 테스트
curl -X POST http://13.124.0.1:5000/memos \
  -H "Content-Type: application/json" \
  -d '{"content": "오늘 할 일: Flask 배포 완료하기"}'

# 3. 메모가 실제로 저장됐는지 다시 조회
curl http://13.124.0.1:5000/memos
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| `.env` 파일을 GitHub에 올림 | 경고 없이 업로드됨, 이후 API 키 유출 | `.gitignore` 에 `.env` 추가 후 `git rm --cached .env` 실행 |
| AWS 보안 그룹에서 포트를 열지 않음 | `curl: (7) Failed to connect to ... port 5000` | AWS EC2 보안 그룹 인바운드 규칙에 포트 5000 추가 |
| Flask를 `0.0.0.0` 이 아닌 기본값으로 실행 | 로컬에서는 되는데 외부에서 접속 불가 | `app.run(host="0.0.0.0", port=5000)` 으로 변경 |
| `Content-Type` 헤더 누락 | `400 Bad Request` 또는 `None` 반환 | curl 명령에 `-H "Content-Type: application/json"` 추가 |
| venv 폴더를 통째로 커밋 | `git push` 가 매우 느림, 용량 초과 | `.gitignore` 에 `venv/` 추가, 이미 올렸다면 `git rm -r --cached venv/` 실행 |
| prompt에 코드를 붙여넣지 않음 | AI가 "코드를 보여주세요"라고 응답 | prompt 안에 코드를 직접 붙여넣거나 파일명을 명시 |

---

## 확인 체크리스트

- [ ] AI에게 코드 설명을 요청하는 prompt를 직접 작성해 봤다.
- [ ] `.gitignore` 에 `venv/`, `.env`, `__pycache__/` 를 추가했다.
- [ ] 커밋 메시지를 AI에게 요청해서 만들었다.
- [ ] AWS EC2 보안 그룹에서 필요한 포트를 열었다.
- [ ] `app.run(host="0.0.0.0")` 으로 실행했다.
- [ ] `curl` 명령어로 GET 과 POST 를 모두 테스트했다.
- [ ] AI에게 에러 메시지를 붙여넣고 해결 방법을 물어봤다.

---

## 한 번 더 생각해 보기

1. AI에게 코드 설명을 요청할 때 "그냥 설명해 줘" 보다 "초보자 기준으로, 한 줄씩, 전문 용어 없이" 처럼 조건을 추가하면 어떤 점이 달라질까요?

2. GitHub에 코드를 올리기 전에 AI에게 점검을 요청하는 것과, 올린 뒤에 리뷰를 요청하는 것은 어떤 차이가 있을까요? 두 방법을 언제 각각 사용하면 좋을까요?

3. AWS 배포 검증 prompt를 작성할 때 IP 주소나 포트 번호처럼 "나만의 정보"를 직접 넣는 것이 왜 중요할까요? AI에게 정보를 줄수록 어떤 점이 좋아지나요?

---

## 다음 장

다음 장에서는 여러 개의 API를 가진 서비스를 AI와 함께 단계별로 확장하는 방법과, 데이터베이스를 연결하는 prompt 패턴을 배웁니다.