## 이 장에서 배우는 것

- AI 도구만 사용해서 메모 관리 웹 API를 처음부터 끝까지 만드는 전체 흐름
- 기획 → 설계 → 코딩 → 테스트 → GitHub 업로드 → AWS Lambda 배포 → 접속 확인의 7단계 사이클
- 각 단계에서 AI에게 어떻게 질문해야 원하는 결과를 얻을 수 있는지
- 막히는 지점에서 AI와 대화로 돌파하는 방법
- 완성된 서비스를 외부 URL로 공개 배포하는 경험

---

## 먼저 쉬운 설명

코딩을 배우다 보면 이런 생각이 드는 순간이 있습니다.

> "예제는 따라 했는데, 진짜 서비스는 어떻게 만드는 거지?"

튜토리얼은 각 부품을 설명해 줍니다. 하지만 부품을 조립해서 하나의 완성품을 만드는 경험은 따로 해야 합니다.

이 장은 그 경험입니다.

메모를 저장하고, 불러오고, 삭제하는 웹 API를 AI와 함께 만들어 볼 것입니다. 코드를 직접 다 외울 필요가 없습니다. AI에게 무엇을 만들지 설명하고, 나온 결과를 이해하고, 배포까지 밀어붙이는 것이 목표입니다.

이 장을 끝내면 URL이 생깁니다. 그 URL을 친구에게 보내면 친구도 접속할 수 있습니다. 그게 진짜 서비스입니다.

---

## 1. 전체 흐름 한눈에 보기

7단계가 있습니다. 각 단계는 독립적으로 보이지만 모두 연결되어 있습니다.

```
1단계 기획   → "어떤 서비스를 만들까?"
2단계 설계   → "어떻게 구조를 잡을까?"
3단계 코딩   → "AI가 코드를 써 준다"
4단계 테스트 → "로컬에서 잘 도는지 확인"
5단계 GitHub → "코드를 인터넷에 올린다"
6단계 배포   → "AWS Lambda로 서비스 오픈"
7단계 확인   → "외부 URL로 접속 성공"
```

각 단계 끝에 **체크포인트**가 있습니다. 체크포인트를 통과해야 다음으로 넘어갑니다.

---

## 2. 1단계 — 기획: AI와 함께 아이디어 구체화하기

막연한 아이디어를 AI와 대화하면서 구체적인 명세로 만드는 단계입니다.

### AI에게 보낼 프롬프트 (기획용)

```
나는 메모를 저장하는 간단한 웹 API를 만들고 싶어.
초보자가 만들기에 적당한 수준으로 기능을 제안해 줘.
기능은 3~5개로 제한하고, 각 기능에 필요한 HTTP 메서드와 URL 경로도 같이 알려줘.
```

AI가 돌려줄 내용은 보통 이런 형태입니다.

```
기능 목록:
1. 메모 전체 조회   GET  /memos
2. 메모 1개 조회   GET  /memos/{id}
3. 메모 추가       POST /memos
4. 메모 삭제       DELETE /memos/{id}
```

이 결과를 노트에 저장해 두세요. 이것이 앞으로 AI에게 질문할 때의 기준이 됩니다.

### 체크포인트 1

- [ ] 만들 기능 목록이 4개 이상 명확하게 정해졌다
- [ ] 각 기능에 URL 경로와 HTTP 메서드가 정해졌다
- [ ] 이 목록을 텍스트 파일에 저장했다

---

## 3. 2단계 — 설계: 구조 잡기

기획이 끝나면 "어떻게 만들지"를 설계합니다. 여기서는 어떤 도구를 쓸지, 파일 구조를 어떻게 잡을지 결정합니다.

### AI에게 보낼 프롬프트 (설계용)

```
Python으로 다음 API를 만들려고 해.

기능:
1. GET /memos — 전체 메모 조회
2. GET /memos/{id} — 단일 메모 조회
3. POST /memos — 메모 추가
4. DELETE /memos/{id} — 메모 삭제

AWS Lambda에 배포할 예정이야. Flask와 Mangum을 사용하는 구조로
프로젝트 파일 구조를 제안해 줘.
디렉터리 트리 형태로 보여줘.
```

AI가 제안하는 파일 구조 예시입니다.

```
memo-api/
├── app.py          ← Flask 앱 본체
├── requirements.txt ← 패키지 목록
└── tests/
    └── test_app.py  ← 테스트 코드
```

### 체크포인트 2

- [ ] 프로젝트 폴더 이름이 정해졌다 (예: `memo-api`)
- [ ] 사용할 라이브러리가 정해졌다 (Flask, Mangum)
- [ ] 파일 구조가 결정됐다

---

## 4. 3단계 — 코딩: AI가 코드를 써 준다

설계가 끝나면 본격적으로 코드를 생성합니다. 이 단계에서는 AI가 대부분의 코드를 써 줍니다. 여러분이 할 일은 AI가 쓴 코드를 읽고 이해하는 것입니다.

### AI에게 보낼 프롬프트 (코딩용)

```
Python Flask와 Mangum으로 메모 API를 만들어 줘.

요구 사항:
- GET /memos → 전체 메모 반환
- GET /memos/<id> → id에 해당하는 메모 반환, 없으면 404
- POST /memos → JSON body에서 title과 content를 받아 저장
- DELETE /memos/<id> → id에 해당하는 메모 삭제, 없으면 404
- 데이터는 메모리(딕셔너리)에 저장 (DB 없이)
- AWS Lambda 핸들러도 포함해 줘
- 코드에 한국어 주석 달아 줘
```

AI가 생성해 주는 코드 예시입니다. 실제로 이 코드를 `app.py`에 저장하세요.

```python
# app.py
from flask import Flask, request, jsonify
from mangum import Mangum

app = Flask(__name__)

# 메모를 메모리에 저장하는 딕셔너리
memos = {}
next_id = 1


@app.route("/memos", methods=["GET"])
def get_all_memos():
    # 모든 메모를 리스트로 반환
    return jsonify(list(memos.values()))


@app.route("/memos/<int:memo_id>", methods=["GET"])
def get_memo(memo_id):
    # id에 해당하는 메모가 없으면 404 반환
    memo = memos.get(memo_id)
    if memo is None:
        return jsonify({"error": "메모를 찾을 수 없습니다"}), 404
    return jsonify(memo)


@app.route("/memos", methods=["POST"])
def create_memo():
    global next_id
    data = request.get_json()

    # 필수 필드가 없으면 400 반환
    if not data or "title" not in data or "content" not in data:
        return jsonify({"error": "title과 content가 필요합니다"}), 400

    memo = {
        "id": next_id,
        "title": data["title"],
        "content": data["content"],
    }
    memos[next_id] = memo
    next_id += 1
    return jsonify(memo), 201


@app.route("/memos/<int:memo_id>", methods=["DELETE"])
def delete_memo(memo_id):
    if memo_id not in memos:
        return jsonify({"error": "메모를 찾을 수 없습니다"}), 404
    del memos[memo_id]
    return jsonify({"message": "삭제되었습니다"})


# Lambda 핸들러 — AWS에서 이 함수가 호출됨
handler = Mangum(app)

if __name__ == "__main__":
    app.run(debug=True)
```

그리고 `requirements.txt`를 만드세요.

```
flask==3.0.3
mangum==0.17.0
```

### 체크포인트 3

- [ ] `app.py` 파일이 만들어졌다
- [ ] `requirements.txt`가 만들어졌다
- [ ] 코드를 위에서 아래로 한 번 읽었다

---

## 5. 4단계 — 테스트: 로컬에서 확인하기

배포하기 전에 내 컴퓨터에서 먼저 실행해 봅니다. 오류를 일찍 잡을수록 좋습니다.

### 가상 환경 만들고 실행하기

```bash
# 프로젝트 폴더로 이동
cd memo-api

# 가상 환경 생성
python -m venv venv

# 가상 환경 활성화 (Mac/Linux)
source venv/bin/activate

# 패키지 설치
pip install -r requirements.txt

# 서버 실행
python app.py
```

서버가 성공적으로 시작되면 터미널에 이런 메시지가 나옵니다.

```
 * Running on http://127.0.0.1:5000
 * Debug mode: on
```

### curl로 동작 확인하기

새 터미널 창을 열고 아래 명령어를 순서대로 실행해 보세요.

```bash
# 메모 추가
curl -X POST http://127.0.0.1:5000/memos \
  -H "Content-Type: application/json" \
  -d '{"title": "첫 메모", "content": "AI와 함께 만들었다"}'

# 전체 조회
curl http://127.0.0.1:5000/memos

# 단일 조회
curl http://127.0.0.1:5000/memos/1

# 삭제
curl -X DELETE http://127.0.0.1:5000/memos/1
```

각 명령어를 실행할 때 JSON 응답이 돌아오면 성공입니다.

### AI에게 보낼 프롬프트 (오류 해결용)

로컬 테스트 중 오류가 나면 오류 메시지를 그대로 복사해서 AI에게 붙여넣으세요.

```
Flask 앱을 실행했는데 아래 오류가 났어. 원인과 해결 방법을 알려줘.

[오류 메시지를 여기에 붙여넣기]
```

### 체크포인트 4

- [ ] `python app.py` 실행 시 서버가 정상 시작된다
- [ ] curl로 메모 추가, 조회, 삭제가 모두 성공한다
- [ ] 존재하지 않는 id로 조회할 때 404가 돌아온다

---

## 6. 5단계 — GitHub 업로드: 코드를 인터넷에 올리기

로컬 테스트를 통과했으면 코드를 GitHub에 올립니다. Lambda 배포에도 필요하고, 코드 백업도 됩니다.

### `.gitignore` 만들기

`venv` 폴더와 캐시 파일은 올리지 않습니다.

```
# .gitignore
venv/
__pycache__/
*.pyc
.env
```

### Git 명령어 순서

```bash
# Git 초기화
git init

# 파일 추가
git add app.py requirements.txt .gitignore

# 첫 커밋
git commit -m "feat: 메모 API 초기 구현"
```

이후 GitHub에서 새 리포지터리를 만들고 아래 명령어로 연결합니다.

```bash
git remote add origin https://github.com/사용자명/memo-api.git
git branch -M main
git push -u origin main
```

### 체크포인트 5

- [ ] GitHub 리포지터리가 만들어졌다
- [ ] `app.py`, `requirements.txt`, `.gitignore`가 리포지터리에 올라갔다
- [ ] `venv/` 폴더는 올라가지 않았다

---

## 7. 6단계 — Lambda 배포: AWS에서 서비스 오픈하기

이제 AWS Lambda에 올려서 누구나 접속할 수 있게 합니다.

### 배포 패키지 만들기

Lambda는 코드와 패키지를 zip 파일로 받습니다.

```bash
# 패키지를 현재 폴더에 설치 (Lambda용)
pip install -r requirements.txt -t ./package

# package 폴더에 app.py 복사
cp app.py ./package/

# zip 파일 생성
cd package
zip -r ../memo-api.zip .
cd ..
```

### AI에게 보낼 프롬프트 (배포 설정용)

```
AWS Lambda에 Flask API를 배포하려고 해.
런타임은 Python 3.11이고 핸들러 이름은 app.handler야.
Lambda 함수 설정과 API Gateway 연결 방법을 단계별로 알려줘.
AWS 콘솔 기준으로 설명해 줘.
```

### AWS 콘솔에서 Lambda 함수 만들기

1. AWS 콘솔 → Lambda → 함수 생성 클릭
2. 설정값 입력:
   - 함수 이름: `memo-api`
   - 런타임: Python 3.11
   - 아키텍처: x86_64
3. 함수 생성 후 → 코드 탭 → `.zip 파일 업로드` 선택
4. `memo-api.zip` 업로드
5. 런타임 설정 → 핸들러를 `app.handler`로 변경 후 저장

### API Gateway 연결하기

```
Lambda 함수 페이지 → 트리거 추가 → API Gateway
→ HTTP API 선택 → 보안: 열기 → 추가
```

추가가 완료되면 API 엔드포인트 URL이 생성됩니다.

```
https://xxxxxxxx.execute-api.ap-northeast-2.amazonaws.com/
```

### 체크포인트 6

- [ ] Lambda 함수가 만들어졌다
- [ ] zip 업로드가 완료됐다
- [ ] 핸들러가 `app.handler`로 설정됐다
- [ ] API Gateway URL이 생성됐다

---

## 8. 7단계 — 확인: 외부 URL로 접속 성공하기

배포된 URL로 실제 API 호출을 해봅니다.

### 외부 URL로 테스트하기

앞서 생성된 URL을 사용해서 curl을 다시 실행합니다.

```bash
# 아래 URL을 본인의 API Gateway URL로 교체하세요
BASE_URL="https://xxxxxxxx.execute-api.ap-northeast-2.amazonaws.com"

# 메모 추가
curl -X POST $BASE_URL/memos \
  -H "Content-Type: application/json" \
  -d '{"title": "배포 성공", "content": "드디어 외부에서 접속된다!"}'

# 전체 조회
curl $BASE_URL/memos
```

아래처럼 JSON이 돌아오면 배포 완료입니다.

```json
[
  {
    "id": 1,
    "title": "배포 성공",
    "content": "드디어 외부에서 접속된다!"
  }
]
```

### 오류가 났을 때

Lambda 실행 중 오류가 나면 CloudWatch 로그에서 원인을 찾습니다.

```
AWS 콘솔 → Lambda → 함수 선택 → 모니터링 탭 → CloudWatch 로그 보기
```

오류 메시지를 복사해서 AI에게 붙여넣으세요.

```
Lambda 함수를 호출했더니 아래 오류가 났어.
CloudWatch 로그에서 찾은 내용이야. 원인과 해결 방법 알려줘.

[오류 로그를 여기에 붙여넣기]
```

### 체크포인트 7

- [ ] 외부 URL로 메모 추가 요청이 성공한다
- [ ] 외부 URL로 전체 조회 요청이 성공한다
- [ ] 친구에게 URL을 보내서 접속 확인했다

---

## 따라 하기 실습

### 실습 1 — 기획서 파일 만들기

`memo-api/` 폴더를 만들고 그 안에 `plan.txt` 파일을 만드세요. 아래 내용을 채웁니다.

```
서비스 이름: 메모 관리 API
기능 목록:
- GET /memos
- GET /memos/<id>
- POST /memos
- DELETE /memos/<id>
데이터 저장 방식: 메모리 딕셔너리
배포 대상: AWS Lambda + API Gateway
```

저장했으면 AI에게 이 파일을 보여주고 빠진 항목이 있는지 물어보세요.

### 실습 2 — 로컬에서 전체 흐름 테스트하기

`app.py`를 만들고 실행한 뒤 아래 4가지 시나리오를 curl로 직접 확인합니다.

1. 메모 2개 추가하기
2. 전체 조회해서 2개가 나오는지 확인하기
3. id=1 메모 삭제하기
4. 전체 조회해서 1개만 남았는지 확인하기

전부 성공하면 터미널 화면을 스크린샷으로 저장해 두세요.

### 실습 3 — 배포 후 동일한 시나리오 반복하기

실습 2에서 했던 4가지 시나리오를 배포된 외부 URL로 동일하게 실행해 보세요. 결과가 같으면 배포가 올바르게 완료된 것입니다.

막히면 CloudWatch 로그를 열고 AI에게 오류를 보여주세요.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|----------------------|-----------|
| `pip install` 없이 실행 | `ModuleNotFoundError: No module named 'flask'` | `pip install -r requirements.txt` 실행 |
| 가상 환경 비활성화 상태로 실행 | 다른 프로젝트의 패키지가 충돌 | `source venv/bin/activate` 먼저 실행 |
| Lambda 핸들러 이름 오타 | `Runtime.HandlerNotFound` | 핸들러를 정확히 `app.handler`로 설정 |
| zip에 `app.py`를 빠뜨림 | `Unable to import module 'app'` | package 폴더에 `cp app.py ./package/` 후 다시 zip |
| API Gateway URL에 경로 빠뜨림 | `{"message":"Not Found"}` | URL 끝에 `/memos`를 추가했는지 확인 |
| POST 요청에 Content-Type 없음 | `400 Bad Request`, data가 None | `-H "Content-Type: application/json"` 추가 |
| Lambda 메모리 초기화 문제 | 요청마다 메모가 사라짐 | 정상 동작 — 메모리 저장 방식의 한계, DynamoDB로 개선 필요 |

---

## 확인 체크리스트

- [ ] 기획 단계: 기능 4개 이상이 명확히 정해졌다
- [ ] 설계 단계: 파일 구조와 사용 라이브러리가 결정됐다
- [ ] 코딩 단계: `app.py`와 `requirements.txt`가 만들어졌다
- [ ] 테스트 단계: 로컬에서 4가지 API 기능이 모두 동작한다
- [ ] GitHub 단계: 리포지터리에 코드가 올라갔고 `venv/`는 빠져 있다
- [ ] 배포 단계: Lambda 함수가 생성됐고 API Gateway URL이 있다
- [ ] 확인 단계: 외부 URL로 메모 추가와 조회가 성공한다
- [ ] AI와 대화하면서 막힌 지점을 스스로 해결해 봤다

---

## 한 번 더 생각해 보기

1. Lambda는 요청이 없을 때는 꺼져 있습니다. 그래서 메모리에 저장한 데이터는 요청이 끝나면 사라집니다. 데이터를 영구적으로 저장하려면 어떤 AWS 서비스를 추가해야 할까요? AI에게 물어보세요.

2. 지금 API는 아무나 접근할 수 있습니다. 나만 사용할 수 있게 하려면 어떤 방법이 있을까요? "API 인증"이라는 키워드로 AI와 대화해 보세요.

3. 이 장에서 7단계를 모두 직접 해봤습니다. 어느 단계가 가장 어려웠나요? 그 단계를 AI가 어떻게 도와줬는지 짧게 정리해 보세요. 다음에 비슷한 서비스를 만들 때 그 정리가 시간을 절약해 줄 것입니다.

---

## 다음 장

다음 장에서는 Lambda 함수에 DynamoDB를 연결해서 메모가 요청 사이에도 사라지지 않도록 영구 저장소를 추가하는 방법을 배웁니다.