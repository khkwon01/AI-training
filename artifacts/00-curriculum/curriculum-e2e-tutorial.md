# E2E 통합 튜토리얼: Python 서비스를 만들고 AWS에 배포하기

## 이 튜토리얼에서 만드는 것

이 튜토리얼에서는 메모를 추가하고, 조회하고, 삭제할 수 있는 REST API를 처음부터 끝까지 직접 만든다.

- 로컬에서 Python 코드를 작성한다
- GitHub에 올리고 AI에게 코드리뷰를 받는다
- AWS Lambda에 배포해서 실제 URL로 접속할 수 있도록 만든다

튜토리얼을 마치면 다음과 같은 URL을 직접 쓸 수 있다.

```
GET  https://xxxx.lambda-url.ap-northeast-1.on.aws/memos       → 메모 전체 조회
POST https://xxxx.lambda-url.ap-northeast-1.on.aws/memos?text=안녕  → 메모 추가
DELETE https://xxxx.lambda-url.ap-northeast-1.on.aws/memos?id=1     → 메모 삭제
```

---

## 전체 흐름도

이 튜토리얼의 큰 그림이다. 각 단계가 어떻게 연결되는지 먼저 이해하고 시작하면 헷갈리지 않는다.

```
로컬 Python 코딩
      |
      v
  git commit
      |
      v
  GitHub push
      |
      v
  Pull Request 생성
      |
      v
  AI 코드리뷰 (GitHub Copilot / Claude)
      |
      v
  리뷰 피드백 반영 → merge
      |
      v
  AWS Lambda 코드 업로드
      |
      v
  Lambda Function URL 활성화
      |
      v
  curl 또는 브라우저로 URL 테스트 완료
```

**필요한 사전 준비**

| 항목 | 확인 방법 |
|------|----------|
| Python 3.9 이상 | 터미널에서 `python --version` |
| Git 설치 | 터미널에서 `git --version` |
| GitHub 계정 | github.com에 로그인 가능 |
| AWS 계정 | aws.amazon.com에 로그인 가능 |
| VS Code | 실행 가능 |

---

## Part 1: 로컬에서 서비스 만들기 (Python)

### 1-1. 서비스 구조 설계

코드를 바로 작성하기 전에 "무엇을 만들 것인지"를 먼저 정리한다.
이 단계에서 AI에게 도움을 받으면 빠르게 구조를 잡을 수 있다.

**AI에게 물어보는 방법 (Claude 또는 ChatGPT)**

아래 프롬프트를 그대로 붙여넣어 보자.

```
메모를 추가, 조회, 삭제할 수 있는 Python REST API를 만들려고 해.
AWS Lambda에서 실행할 거야.
lambda_handler 함수 하나로 GET/POST/DELETE를 처리하는 구조를 설명해줘.
코드는 아직 필요 없고, 폴더 구조와 각 함수의 역할만 설명해줘.
```

AI가 제안하는 구조를 참고해서 아래처럼 폴더를 만든다.

**폴더 구조 만들기**

터미널을 열고 아래 명령어를 순서대로 실행한다.

```bash
# 프로젝트 폴더 만들기
mkdir memo-api
cd memo-api

# 파일 만들기
touch lambda_function.py
touch requirements.txt
touch test_lambda.py
```

VS Code에서 이 폴더를 열면 왼쪽 탐색기에 세 파일이 보인다.

### 1-2. 코드 작성

`lambda_function.py` 파일을 열고 아래 코드를 그대로 붙여넣는다.

```python
# lambda_function.py
import json

# 메모리에 저장 (Lambda가 재시작되면 초기화됨)
# 실제 서비스에서는 DynamoDB 같은 DB를 사용한다
memos = []
next_id = 1


def get_memos():
    """전체 메모 목록 반환"""
    return {"memos": memos, "count": len(memos)}, 200


def add_memo(text):
    """새 메모 추가"""
    global next_id

    if not text or not text.strip():
        return {"error": "메모 내용을 입력해주세요"}, 400

    memo = {
        "id": next_id,
        "text": text.strip()
    }
    memos.append(memo)
    next_id += 1

    return {"message": "메모가 추가됐습니다", "memo": memo}, 201


def delete_memo(memo_id):
    """ID로 메모 삭제"""
    try:
        target_id = int(memo_id)
    except (ValueError, TypeError):
        return {"error": "ID는 숫자여야 합니다"}, 400

    for i, memo in enumerate(memos):
        if memo["id"] == target_id:
            removed = memos.pop(i)
            return {"message": "삭제됐습니다", "memo": removed}, 200

    return {"error": f"ID {target_id}인 메모를 찾을 수 없습니다"}, 404


def lambda_handler(event, context):
    """Lambda 진입점 - 모든 HTTP 요청이 이 함수로 들어온다"""

    # 어떤 요청이 들어왔는지 로그로 남긴다
    print(f"[요청] {json.dumps(event)}")

    # HTTP 메서드와 경로, 쿼리 파라미터 읽기
    http = event.get("requestContext", {}).get("http", {})
    method = http.get("method", "GET").upper()
    params = event.get("queryStringParameters") or {}

    # 요청 종류에 따라 처리
    if method == "GET":
        body, status = get_memos()

    elif method == "POST":
        text = params.get("text", "")
        body, status = add_memo(text)

    elif method == "DELETE":
        memo_id = params.get("id", "")
        body, status = delete_memo(memo_id)

    else:
        body = {"error": f"지원하지 않는 메서드: {method}"}
        status = 405

    # Lambda는 반드시 이 형태로 응답해야 한다
    return {
        "statusCode": status,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(body, ensure_ascii=False)
    }
```

이 코드에서 핵심은 `lambda_handler` 함수다.
Lambda로 HTTP 요청이 들어오면 항상 이 함수가 먼저 실행된다.
메서드(GET/POST/DELETE)에 따라 각기 다른 함수를 호출하는 구조다.

**requirements.txt 작성**

이 서비스는 Python 표준 라이브러리만 사용하므로 외부 패키지가 없다.

```
# requirements.txt
# 이 서비스는 json, 표준 라이브러리만 사용합니다
```

### 1-3. 로컬 테스트

배포 전에 로컬에서 먼저 동작을 확인한다.
`test_lambda.py` 파일을 열고 아래 코드를 붙여넣는다.

```python
# test_lambda.py
import json
from lambda_function import lambda_handler


def make_event(method, params=None):
    """테스트용 가짜 Lambda 이벤트 만들기"""
    return {
        "requestContext": {
            "http": {"method": method}
        },
        "queryStringParameters": params or {}
    }


def test_add_memo():
    event = make_event("POST", {"text": "첫 번째 메모"})
    result = lambda_handler(event, {})
    body = json.loads(result["body"])

    assert result["statusCode"] == 201, "추가 성공은 201이어야 한다"
    assert body["memo"]["text"] == "첫 번째 메모"
    print("메모 추가 테스트: 통과")


def test_get_memos():
    event = make_event("GET")
    result = lambda_handler(event, {})
    body = json.loads(result["body"])

    assert result["statusCode"] == 200
    assert body["count"] >= 1
    print("메모 조회 테스트: 통과")


def test_delete_memo():
    # 먼저 메모 하나 추가
    lambda_handler(make_event("POST", {"text": "삭제할 메모"}), {})

    # 조회해서 마지막 ID 확인
    result = lambda_handler(make_event("GET"), {})
    memos = json.loads(result["body"])["memos"]
    last_id = memos[-1]["id"]

    # 삭제
    event = make_event("DELETE", {"id": str(last_id)})
    result = lambda_handler(event, {})

    assert result["statusCode"] == 200
    print("메모 삭제 테스트: 통과")


def test_empty_memo():
    event = make_event("POST", {"text": ""})
    result = lambda_handler(event, {})

    assert result["statusCode"] == 400, "빈 메모는 400이어야 한다"
    print("빈 메모 거부 테스트: 통과")


if __name__ == "__main__":
    test_add_memo()
    test_get_memos()
    test_delete_memo()
    test_empty_memo()
    print("\n모든 테스트 통과!")
```

터미널에서 아래 명령어로 실행한다.

```bash
cd memo-api
python test_lambda.py
```

아래처럼 출력되면 성공이다.

```
메모 추가 테스트: 통과
메모 조회 테스트: 통과
메모 삭제 테스트: 통과
빈 메모 거부 테스트: 통과

모든 테스트 통과!
```

---

### Part 1 완료 체크포인트

아래 항목을 모두 확인하고 넘어간다.

- [ ] `memo-api/` 폴더가 생성됐다
- [ ] `lambda_function.py`에 코드가 들어가 있다
- [ ] `test_lambda.py`를 실행했을 때 "모든 테스트 통과!"가 출력된다
- [ ] `requirements.txt` 파일이 있다

---

## Part 2: GitHub에 올리기

### 2-1. GitHub 저장소 만들기

1. 브라우저에서 [github.com](https://github.com) 으로 이동한다
2. 오른쪽 위 `+` 버튼을 클릭한다
3. `New repository` 를 선택한다
4. Repository name에 `memo-api` 를 입력한다
5. `Public` 또는 `Private` 중 하나를 선택한다 (어느 것이든 괜찮다)
6. `Add a README file` 체크박스는 체크하지 않는다 (이미 로컬에 파일이 있으므로)
7. `Create repository` 버튼을 클릭한다

저장소가 만들어지면 화면에 아래와 비슷한 URL이 보인다.

```
https://github.com/내계정명/memo-api.git
```

이 URL을 복사해둔다.

### 2-2. 로컬과 GitHub 연결하기

터미널에서 `memo-api` 폴더 안에 있는지 확인하고 아래 명령어를 실행한다.

```bash
# Git 저장소 초기화
git init

# 내 이름과 이메일 설정 (처음 한 번만)
git config --global user.name "내 이름"
git config --global user.email "내이메일@example.com"

# 파일 전체를 스테이징 영역에 추가
git add lambda_function.py requirements.txt test_lambda.py

# 첫 번째 커밋
git commit -m "feat: 메모 API Lambda 초기 구현"

# GitHub 저장소 연결 (위에서 복사한 URL을 사용)
git remote add origin https://github.com/내계정명/memo-api.git

# main 브랜치로 push
git branch -M main
git push -u origin main
```

GitHub 페이지를 새로고침하면 파일이 올라간 것을 볼 수 있다.

### 2-3. 작업 브랜치 만들기

실제 개발에서는 `main` 브랜치에 바로 작업하지 않는다.
별도 브랜치를 만들어서 작업하고, Pull Request를 통해 합치는 것이 기본 규칙이다.

```bash
# feature 브랜치 생성 및 이동
git checkout -b feature/memo-api-v1
```

이제부터 이 브랜치에서 작업한다.

`lambda_function.py`를 열고 상단에 버전 정보를 한 줄 추가한다.

```python
# lambda_function.py
# version: 1.0.0
import json
```

저장 후 커밋하고 push한다.

```bash
git add lambda_function.py
git commit -m "feat: 버전 정보 추가"
git push -u origin feature/memo-api-v1
```

### 2-4. Pull Request 만들기

1. 브라우저에서 GitHub 저장소 페이지로 이동한다
2. 노란색 알림 배너에 `Compare & pull request` 버튼이 보인다 — 클릭한다
3. PR 제목을 입력한다: `feat: 메모 API Lambda 초기 구현`
4. 설명란에 아래 내용을 입력한다

```
## 변경 사항
- 메모 추가 (POST /memos?text=내용)
- 메모 전체 조회 (GET /memos)
- 메모 삭제 (DELETE /memos?id=숫자)

## 테스트
- test_lambda.py 모든 케이스 통과
```

5. `Create pull request` 버튼을 클릭한다

---

### Part 2 완료 체크포인트

- [ ] GitHub에 `memo-api` 저장소가 생성됐다
- [ ] `feature/memo-api-v1` 브랜치가 push됐다
- [ ] Pull Request가 열려 있다
- [ ] PR 제목과 설명이 채워져 있다

---

## Part 3: AI 코드리뷰

### 3-1. GitHub Copilot으로 PR 리뷰 받기

GitHub Copilot이 활성화된 계정이라면 PR에서 직접 AI 리뷰를 받을 수 있다.

1. GitHub PR 페이지에서 `Files changed` 탭을 클릭한다
2. 오른쪽 위에 `Copilot` 아이콘이 있으면 클릭한다
3. `Review changes` 를 선택한다
4. Copilot이 코드를 분석하고 코멘트를 남긴다

Copilot이 없는 경우에는 Claude를 사용한다.

### 3-2. Claude로 PR 리뷰 받기

브라우저에서 [claude.ai](https://claude.ai) 로 이동한 뒤, 아래 프롬프트와 함께 `lambda_function.py` 코드를 붙여넣는다.

```
아래는 AWS Lambda용 메모 API Python 코드야.
초보자가 작성한 코드인데, 아래 항목을 점검해줘:

1. 보안 문제 (입력값 검증, 오류 처리)
2. 코드 품질 (가독성, 함수 분리)
3. Lambda 환경에서 주의할 점
4. 개선하면 좋을 부분

코드:
(lambda_function.py 내용 붙여넣기)
```

### 3-3. 리뷰 피드백 적용하기

AI 리뷰 결과에서 중요한 항목을 골라 코드에 반영한다.

예를 들어 Claude가 "메모 텍스트 길이 제한이 없다"고 지적했다면,
`add_memo` 함수에 길이 체크를 추가한다.

```python
def add_memo(text):
    """새 메모 추가"""
    global next_id

    if not text or not text.strip():
        return {"error": "메모 내용을 입력해주세요"}, 400

    # AI 리뷰 반영: 텍스트 길이 제한 추가
    if len(text.strip()) > 500:
        return {"error": "메모는 500자를 초과할 수 없습니다"}, 400

    memo = {
        "id": next_id,
        "text": text.strip()
    }
    memos.append(memo)
    next_id += 1

    return {"message": "메모가 추가됐습니다", "memo": memo}, 201
```

수정 후 테스트를 다시 실행해서 기존 테스트가 여전히 통과하는지 확인한다.

```bash
python test_lambda.py
```

확인 후 커밋하고 push한다.

```bash
git add lambda_function.py
git commit -m "fix: AI 리뷰 반영 - 텍스트 길이 제한 추가"
git push
```

### 3-4. PR Merge하기

GitHub PR 페이지로 돌아가면 방금 push한 커밋이 반영된 것을 볼 수 있다.

1. 페이지 하단 `Merge pull request` 버튼을 클릭한다
2. `Confirm merge` 를 클릭한다
3. 초록색 `Pull request successfully merged` 메시지가 나오면 성공이다

---

### Part 3 완료 체크포인트

- [ ] AI(Copilot 또는 Claude)에게 코드리뷰를 받았다
- [ ] 리뷰 피드백 중 최소 1개를 코드에 반영했다
- [ ] 수정 후 테스트를 다시 돌렸을 때 통과한다
- [ ] PR이 `main` 브랜치에 merge됐다

---

## Part 4: AWS Lambda 배포

### 4-1. AWS 콘솔 접속

1. 브라우저에서 [aws.amazon.com](https://aws.amazon.com) 으로 이동한다
2. 오른쪽 위 `로그인` 버튼을 클릭한다
3. 루트 계정 이메일로 로그인한다

로그인 후 오른쪽 위 지역(Region)을 확인한다.
이 튜토리얼에서는 **도쿄(ap-northeast-1)** 를 기준으로 설명한다.
다른 지역을 사용해도 되지만, 일관성을 위해 처음부터 끝까지 같은 지역을 유지한다.

### 4-2. Lambda 함수 생성

1. 검색창에 `Lambda` 를 입력하고 클릭한다
2. 왼쪽 메뉴에서 `함수` 를 클릭한다
3. 오른쪽 위 `함수 생성` 버튼을 클릭한다
4. 아래와 같이 설정한다

| 항목 | 값 |
|------|-----|
| 생성 방법 | 새로 작성 |
| 함수 이름 | `memo-api` |
| 런타임 | `Python 3.12` |
| 아키텍처 | `x86_64` |

5. 하단 `함수 생성` 버튼을 클릭한다

### 4-3. 코드 업로드 — 방법 A: 콘솔 직접 입력

Lambda 함수 생성 후 바로 코드 편집기가 보인다.

1. `lambda_function.py` 탭을 클릭한다
2. 기존 코드를 전부 지운다
3. 로컬의 `lambda_function.py` 내용을 전체 복사해서 붙여넣는다
4. 오른쪽 위 `Deploy` 버튼을 클릭한다
5. "함수가 업데이트됨" 메시지가 나오면 성공이다

### 4-4. 코드 업로드 — 방법 B: zip 파일 업로드 (선택)

외부 패키지가 있을 때는 zip으로 업로드해야 한다.
이 서비스는 표준 라이브러리만 사용하므로 방법 A로 충분하지만,
zip 방법도 알아두면 나중에 유용하다.

터미널에서 실행한다.

```bash
cd memo-api

# zip 파일 만들기 (lambda_function.py만 포함)
zip function.zip lambda_function.py
```

Lambda 콘솔에서:

1. 코드 탭 → `업로드 위치` → `.zip 파일` 선택
2. `업로드` 버튼 클릭
3. `function.zip` 파일 선택
4. `저장` 클릭

### 4-5. Lambda Function URL 활성화

URL이 있어야 외부에서 HTTP 요청을 보낼 수 있다.

1. Lambda 함수 페이지에서 `구성` 탭을 클릭한다
2. 왼쪽 메뉴 `함수 URL` 을 클릭한다
3. `함수 URL 생성` 버튼을 클릭한다
4. 아래와 같이 설정한다

| 항목 | 값 |
|------|-----|
| 인증 유형 | `NONE` (누구나 접근 가능) |

> 참고: 실제 서비스에서는 `AWS_IAM` 인증을 사용한다. 이 튜토리얼에서는 테스트 편의를 위해 NONE을 선택한다.

5. `저장` 버튼을 클릭한다
6. 화면에 `https://xxxx.lambda-url.ap-northeast-1.on.aws/` 형태의 URL이 생긴다 — 복사해둔다

### 4-6. API Gateway로 URL 만들기 (선택)

Function URL 대신 API Gateway를 쓰면 경로(path) 기반 라우팅, 인증, 스로틀링을 더 세밀하게 제어할 수 있다.
이 튜토리얼에서는 Function URL을 기본으로 사용하고, API Gateway는 참고 사항으로 설명한다.

API Gateway를 쓰고 싶다면:

1. 검색창에 `API Gateway` 입력 → 클릭
2. `API 생성` → `HTTP API` 선택
3. `통합 추가` → `Lambda` 선택 → `memo-api` 함수 선택
4. `다음` 을 계속 클릭하고 `생성` 버튼 클릭
5. 스테이지 URL(`https://xxxx.execute-api.ap-northeast-1.amazonaws.com/`) 복사

### 4-7. 배포 후 curl 테스트

터미널을 열고 아래 명령어를 실행한다.
`YOUR_URL` 부분을 4-5 또는 4-6에서 복사한 실제 URL로 교체한다.

```bash
# 메모 전체 조회 (처음엔 비어 있음)
curl https://YOUR_URL

# 메모 추가
curl -X POST "https://YOUR_URL?text=첫번째메모"

# 메모 추가 (두 번째)
curl -X POST "https://YOUR_URL?text=두번째메모"

# 메모 전체 조회 (이제 2개가 보여야 함)
curl https://YOUR_URL

# ID 1번 메모 삭제
curl -X DELETE "https://YOUR_URL?id=1"

# 삭제 후 조회 (1개만 남아야 함)
curl https://YOUR_URL
```

아래와 비슷한 JSON 응답이 나오면 성공이다.

```json
{"memos": [{"id": 2, "text": "두번째메모"}], "count": 1}
```

Windows에서 curl 명령어가 없다면 [Postman](https://www.postman.com) 을 설치해서 대신 사용할 수 있다.
또는 브라우저 주소창에 URL을 입력하면 GET 요청을 테스트할 수 있다.

---

### Part 4 완료 체크포인트

- [ ] AWS Lambda에 `memo-api` 함수가 생성됐다
- [ ] 코드가 Lambda에 업로드되고 `Deploy`(배포) 됐다
- [ ] Function URL이 활성화됐고 URL을 복사했다
- [ ] `curl` 또는 브라우저로 GET 요청을 보냈을 때 JSON 응답이 왔다
- [ ] POST로 메모를 추가하고, GET으로 조회했을 때 추가한 메모가 보인다

---

## Part 5: 완성 확인

### 5-1. 전체 흐름 최종 체크리스트

아래 항목을 순서대로 확인한다.

**로컬 개발**
- [ ] `lambda_function.py` 에 GET/POST/DELETE 기능이 모두 구현됐다
- [ ] `test_lambda.py` 를 실행하면 모든 테스트가 통과한다
- [ ] `requirements.txt` 파일이 있다

**GitHub**
- [ ] GitHub에 `memo-api` 저장소가 있다
- [ ] `main` 브랜치에 최신 코드가 있다
- [ ] PR을 만들고 merge한 이력이 있다

**AI 코드리뷰**
- [ ] AI(Copilot 또는 Claude)에게 리뷰를 요청했다
- [ ] 받은 피드백을 코드에 반영했다

**AWS 배포**
- [ ] Lambda 함수가 생성됐고 코드가 배포됐다
- [ ] Function URL이 활성화됐다
- [ ] curl 또는 브라우저로 API 동작을 확인했다

### 5-2. 자주 겪는 문제와 해결 방법

| 문제 | 원인 | 해결 방법 |
|------|------|----------|
| curl 요청에 응답이 없다 | URL 오타 | Function URL을 다시 복사해서 확인 |
| `{"message": "Internal Server Error"}` | Lambda 코드 오류 | Lambda 콘솔 → `모니터링` → `CloudWatch 로그 보기` 에서 오류 확인 |
| 메모를 추가했는데 조회하면 사라진다 | Lambda 재시작 | 정상 동작. 메모리 저장이라 Lambda가 새로 뜨면 초기화된다 |
| `git push` 가 거부된다 | 인증 문제 | `git remote -v` 로 URL 확인, GitHub 토큰 재발급 |
| zip 업로드 후 handler를 못 찾는다 | 핸들러 이름 불일치 | Lambda 구성 → 런타임 설정에서 핸들러가 `lambda_function.lambda_handler` 인지 확인 |

### 5-3. 다음 단계 안내

이 튜토리얼에서 만든 서비스는 메모리 저장이라 Lambda가 재시작되면 데이터가 사라진다.
실제 서비스처럼 데이터를 영구 저장하려면 DynamoDB를 연결해야 한다.

아래 학습 경로로 이어서 공부할 수 있다.

| 다음 주제 | 위치 |
|-----------|------|
| DynamoDB로 메모 영구 저장 | `05-aws-deploy/aws-chapter-07-dynamodb.md` |
| GitHub Actions로 자동 배포 | `06-demo-services/service-chapter-04-github-actions.md` |
| 완성형 서비스 통합 실습 | `06-demo-services/service-chapter-05-full-integration.md` |
| Issue부터 배포까지 전체 실전 | `06-demo-services/service-chapter-06-issue-to-deploy.md` |

---

## 참고: 이 튜토리얼에서 사용한 핵심 명령어 모음

```bash
# 프로젝트 초기화
mkdir memo-api && cd memo-api

# 테스트 실행
python test_lambda.py

# Git 기본 흐름
git add 파일명
git commit -m "커밋 메시지"
git push

# 브랜치 만들기
git checkout -b feature/이름

# zip 만들기
zip function.zip lambda_function.py

# curl 테스트
curl https://YOUR_URL
curl -X POST "https://YOUR_URL?text=메모내용"
curl -X DELETE "https://YOUR_URL?id=1"
```
