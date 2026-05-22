# Chapter 21: 웹훅과 Lambda 자동화

## 이 장에서 배우는 것

- 웹훅(Webhook)이 무엇인지, 왜 사용하는지 이해한다
- AWS Lambda 함수를 만들고 HTTP 엔드포인트로 연결하는 방법을 익힌다
- 외부 이벤트(GitHub, Slack, 폼 제출 등)가 Lambda를 자동으로 실행하도록 연결한다
- Claude API를 Lambda 안에서 호출해 AI가 자동으로 응답하게 만든다
- 전체 자동화 흐름을 테스트하고 디버깅하는 능력을 기른다

---

## 먼저 쉬운 설명

여러분이 카페를 운영한다고 상상해 보세요. 손님이 주문서를 넣으면 주방 벨이 울리고, 주방장이 자동으로 요리를 시작합니다. 직접 "지금 주문 들어왔어요!"라고 소리치지 않아도 되죠.

웹훅은 이 **벨**과 같습니다. 어떤 사건(이벤트)이 일어나면 미리 등록해 둔 주소로 자동으로 알림을 보냅니다. AWS Lambda는 **주방장** 역할입니다. 알림이 오면 잠에서 깨어나 코드를 실행하고, 다시 잠듭니다.

이 둘을 AI(Claude)와 연결하면? 누군가 GitHub에 이슈를 올리는 순간, AI가 자동으로 내용을 분석해서 답변을 달아줍니다. 24시간 내내, 여러분이 자는 동안에도요.

이 장에서는 그 자동화 파이프라인 전체를 직접 만들어 봅니다.

---

## 1. 웹훅 기초 — "누가 나를 불렀니?"

웹훅은 단순한 HTTP POST 요청입니다. 이벤트가 발생한 서비스가 여러분의 서버로 JSON 데이터를 전송하는 것이죠.

```
[GitHub 이슈 생성] ──POST──▶ [여러분의 Lambda URL] ──▶ [AI 분석] ──▶ [댓글 자동 등록]
```

### 웹훅 페이로드 예시 (GitHub가 보내는 데이터)

```json
{
  "action": "opened",
  "issue": {
    "number": 42,
    "title": "로그인 버튼이 클릭이 안 됩니다",
    "body": "크롬에서 로그인 버튼을 누르면 아무 반응이 없어요. 콘솔 에러는 없습니다.",
    "user": {
      "login": "hong-gildong"
    }
  },
  "repository": {
    "full_name": "myteam/my-project"
  }
}
```

Lambda는 이 JSON을 받아서 원하는 작업을 수행합니다. 구조는 항상 이렇습니다:

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    # event: API Gateway가 전달한 HTTP 요청 전체
    # context: Lambda 실행 환경 정보 (보통 잘 안 씀)

    # 1. 요청 본문(body) 꺼내기
    body = json.loads(event["body"])

    # 2. 필요한 데이터 추출
    issue_title = body["issue"]["title"]
    issue_body = body["issue"]["body"]

    print(f"새 이슈 감지: {issue_title}")

    # 3. 응답 반환 (HTTP 200 OK)
    return {
        "statusCode": 200,
        "body": json.dumps({"message": "웹훅 수신 완료"})
    }
```

> **핵심 규칙**: Lambda는 반드시 `statusCode`가 포함된 딕셔너리를 반환해야 합니다. 이걸 빼먹으면 웹훅 발신 서비스가 "실패"로 판단합니다.

---

## 2. AWS Lambda 함수 만들기

### 2-1. Lambda 함수 생성 (AWS 콘솔)

1. AWS 콘솔 → Lambda → **함수 생성** 클릭
2. **처음부터 작성** 선택
3. 함수 이름: `ai-webhook-handler`
4. 런타임: **Python 3.12**
5. **함수 생성** 클릭

### 2-2. 기본 코드 구조

```python
# lambda_function.py — AI 웹훅 핸들러 기본 뼈대
import json
import os
import urllib.request

def lambda_handler(event, context):
    """
    모든 웹훅 요청의 진입점.
    event["body"]에 웹훅 JSON 페이로드가 들어옵니다.
    """
    try:
        # 1. body가 문자열이면 파싱, 딕셔너리면 그대로 사용
        if isinstance(event.get("body"), str):
            body = json.loads(event["body"])
        else:
            body = event.get("body", {})

        # 2. 이벤트 타입 확인 (GitHub 예시)
        action = body.get("action", "")
        if action != "opened":
            # 이슈 열림 이벤트가 아니면 조용히 종료
            return _ok("무시된 이벤트")

        # 3. AI 분석 실행
        issue_title = body["issue"]["title"]
        issue_body  = body["issue"]["body"]
        ai_reply    = ask_claude(issue_title, issue_body)

        return _ok(ai_reply)

    except KeyError as e:
        # 예상한 키가 없을 때 — 웹훅 페이로드 구조가 다를 수 있음
        print(f"키 오류: {e}")
        return _error(400, f"필요한 필드가 없습니다: {e}")

    except Exception as e:
        print(f"예상치 못한 오류: {e}")
        return _error(500, "서버 내부 오류")


def _ok(message):
    return {"statusCode": 200, "body": json.dumps({"result": message}, ensure_ascii=False)}

def _error(code, message):
    return {"statusCode": code, "body": json.dumps({"error": message}, ensure_ascii=False)}
```

---

## 3. Claude API를 Lambda에 연결하기

Lambda는 인터넷에 접근할 수 있으므로 Claude API를 직접 호출할 수 있습니다. `anthropic` 패키지를 Lambda 레이어(Layer)로 추가하거나 패키지를 ZIP에 포함시킵니다.

### 3-1. 의존성 패키지 준비

```bash
# 로컬에서 실행 — Lambda에 올릴 패키지 폴더 만들기
mkdir ai_webhook_package
cd ai_webhook_package

pip install anthropic -t .   # 현재 폴더에 패키지 설치
cp ../lambda_function.py .   # 코드 복사
zip -r ../ai_webhook.zip .   # ZIP으로 압축
```

### 3-2. Claude 호출 함수

```python
# lambda_function.py에 추가
import anthropic

def ask_claude(issue_title: str, issue_body: str) -> str:
    """GitHub 이슈 내용을 받아 AI 답변을 반환합니다."""

    # API 키는 Lambda 환경 변수에서 가져옴 (절대 코드에 직접 쓰지 않는다!)
    api_key = os.environ["ANTHROPIC_API_KEY"]
    client  = anthropic.Anthropic(api_key=api_key)

    prompt = f"""당신은 친절한 오픈소스 프로젝트 메인테이너입니다.
아래 GitHub 이슈를 한국어로 분석하고, 초보자도 이해할 수 있는 답변을 작성해 주세요.

이슈 제목: {issue_title}
이슈 내용: {issue_body}

답변 형식:
1. 문제 요약 (1~2문장)
2. 예상 원인 (번호 목록)
3. 해결 방법 제안 (단계별)
"""

    message = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )

    return message.content[0].text
```

### 3-3. 환경 변수 설정 (AWS 콘솔)

Lambda 함수 → **구성** 탭 → **환경 변수** → **편집**

| 키 | 값 |
|---|---|
| `ANTHROPIC_API_KEY` | `sk-ant-api03-...` (실제 키 입력) |

> **보안 원칙**: API 키를 코드에 직접 쓰면 GitHub에 올렸을 때 노출됩니다. 반드시 환경 변수나 AWS Secrets Manager를 사용하세요.

---

## 4. API Gateway로 웹훅 URL 만들기

Lambda 함수는 URL이 없습니다. 외부에서 접근하려면 **API Gateway** 또는 **Lambda 함수 URL**을 연결해야 합니다.

### 4-1. Lambda 함수 URL 사용 (가장 간단한 방법)

```
Lambda 함수 → 구성 → 함수 URL → 함수 URL 생성
인증 유형: NONE (테스트용, 프로덕션에서는 인증 추가 필요)
```

생성되면 이런 URL이 나옵니다:
```
https://abcd1234efgh5678.lambda-url.ap-northeast-2.on.aws/
```

이 URL을 GitHub 웹훅에 등록하면 끝입니다.

### 4-2. GitHub 웹훅 등록

```
GitHub 저장소 → Settings → Webhooks → Add webhook

Payload URL : https://abcd1234efgh5678.lambda-url.ap-northeast-2.on.aws/
Content type: application/json
Events      : Let me select individual events → Issues 체크
Active      : ✅
```

### 4-3. 웹훅 서명 검증 (보안 필수)

GitHub는 요청이 진짜 GitHub에서 왔는지 증명하는 서명을 헤더에 포함합니다. 검증하지 않으면 누구나 여러분의 Lambda를 호출할 수 있습니다.

```python
# lambda_function.py에 추가
import hmac
import hashlib

WEBHOOK_SECRET = os.environ["GITHUB_WEBHOOK_SECRET"]

def verify_github_signature(payload_body: str, signature_header: str) -> bool:
    """GitHub 웹훅 서명을 검증합니다."""
    if not signature_header or not signature_header.startswith("sha256="):
        return False

    expected = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload_body.encode(),
        hashlib.sha256
    ).hexdigest()

    received = signature_header[len("sha256="):]

    # hmac.compare_digest: 타이밍 공격 방지를 위해 일반 == 대신 사용
    return hmac.compare_digest(expected, received)


def lambda_handler(event, context):
    # 서명 검증을 가장 먼저 수행
    signature = event.get("headers", {}).get("x-hub-signature-256", "")
    raw_body  = event.get("body", "")

    if not verify_github_signature(raw_body, signature):
        return _error(403, "서명 검증 실패")

    # 이후 기존 로직 계속...
```

---

## 따라 하기 실습

### 실습 1 — 로컬에서 Lambda 핸들러 테스트하기

실제 AWS 배포 전에 로컬에서 함수를 검증합니다.

```python
# test_local.py — 로컬 테스트용 스크립트
import json
from lambda_function import lambda_handler

# GitHub 웹훅이 보내는 것과 똑같은 형태의 가짜 이벤트 만들기
fake_event = {
    "body": json.dumps({
        "action": "opened",
        "issue": {
            "number": 1,
            "title": "다크모드에서 텍스트 색이 안 보입니다",
            "body": "배경이 검은데 글씨도 검어서 아무것도 안 보여요.",
            "user": {"login": "test-user"}
        },
        "repository": {"full_name": "myteam/my-project"}
    }),
    "headers": {
        "x-hub-signature-256": "sha256=테스트용_임시값"
    }
}

# 환경 변수 임시 설정 (실제로는 .env 파일 또는 export 사용)
import os
os.environ["ANTHROPIC_API_KEY"]      = "sk-ant-..."   # 실제 키 입력
os.environ["GITHUB_WEBHOOK_SECRET"]  = "my-secret"

result = lambda_handler(fake_event, None)
print(f"상태 코드: {result['statusCode']}")
print(f"응답 내용:\n{json.loads(result['body'])['result']}")
```

```bash
# 실행
python test_local.py
```

---

### 실습 2 — Lambda에 코드 배포하고 테스트 이벤트 실행하기

```bash
# 1. 패키지 빌드
mkdir -p dist
pip install anthropic -t dist/
cp lambda_function.py dist/
cd dist && zip -r ../ai_webhook.zip . && cd ..

# 2. AWS CLI로 업로드 (AWS CLI 설치 및 설정이 되어 있어야 함)
aws lambda update-function-code \
    --function-name ai-webhook-handler \
    --zip-file fileb://ai_webhook.zip \
    --region ap-northeast-2
```

AWS 콘솔에서 테스트 이벤트를 만들어 실행합니다:

```json
{
  "body": "{\"action\":\"opened\",\"issue\":{\"number\":99,\"title\":\"버튼 클릭이 안 됩니다\",\"body\":\"클릭해도 반응이 없어요\",\"user\":{\"login\":\"tester\"}},\"repository\":{\"full_name\":\"test/repo\"}}",
  "headers": {
    "x-hub-signature-256": "sha256=dummy"
  }
}
```

Lambda → **테스트** 탭 → 이벤트 이름 `github-issue-test` → **테스트** 클릭

---

### 실습 3 — 실제 GitHub 저장소에 이슈를 열고 자동 응답 확인하기

1. GitHub 저장소 Settings → Webhooks에 Lambda URL 등록
2. 이슈 탭 → **New issue** 클릭 → 실제 버그나 질문 작성 후 제출
3. AWS 콘솔 → Lambda → **모니터링** 탭 → CloudWatch 로그에서 실행 확인

```
# CloudWatch 로그에서 보일 내용 예시
START RequestId: a1b2c3d4-...
새 이슈 감지: 버튼 클릭이 안 됩니다
Claude 응답 생성 완료 (892 토큰)
END RequestId: a1b2c3d4-...
REPORT Duration: 3241.52 ms  Billed Duration: 3300 ms
```

> 실습 3까지 완료하면 진짜 동작하는 AI 자동화 파이프라인이 완성됩니다.

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 원인 | 해결 방법 |
|------|----------------|------|----------|
| `statusCode` 누락 | `Malformed Lambda proxy response` | Lambda 응답에 `statusCode` 없음 | 반환 딕셔너리에 `"statusCode": 200` 추가 |
| body를 파싱 안 함 | `TypeError: string indices must be integers` | `event["body"]`는 문자열 → `json.loads()` 필요 | `body = json.loads(event["body"])` 추가 |
| 패키지 미포함 | `Unable to import module 'lambda_function': No module named 'anthropic'` | ZIP에 `anthropic` 패키지 없음 | `pip install anthropic -t .` 후 재패키징 |
| 환경 변수 미설정 | `KeyError: 'ANTHROPIC_API_KEY'` | Lambda 환경 변수에 키 미등록 | Lambda 콘솔 → 구성 → 환경 변수 등록 |
| 타임아웃 | `Task timed out after 3.00 seconds` | 기본 타임아웃 3초, AI 응답은 더 걸림 | Lambda 구성 → 일반 → 제한 시간 30초로 변경 |
| HTTPS 아닌 URL 등록 | `Webhook delivery failed` | GitHub는 HTTPS만 허용 | Lambda 함수 URL은 기본적으로 HTTPS 제공, `http://` 주소 사용 금지 |
| 서명 검증 실패 | `서명 검증 실패 (403)` | `GITHUB_WEBHOOK_SECRET`이 GitHub 설정값과 다름 | GitHub Webhook 설정의 Secret과 Lambda 환경 변수값 일치 확인 |

---

## 확인 체크리스트

- [ ] `lambda_handler(event, context)` 함수 시그니처를 정확히 쓸 수 있다
- [ ] Lambda 응답에 `statusCode`가 반드시 포함되어야 함을 안다
- [ ] `event["body"]`가 문자열이라 `json.loads()`로 파싱해야 함을 안다
- [ ] API 키를 코드에 직접 쓰지 않고 환경 변수로 관리한다
- [ ] Lambda 타임아웃 기본값(3초)이 AI 응답에 부족하다는 것을 알고 늘릴 수 있다
- [ ] Lambda 함수 URL 또는 API Gateway로 외부 접근 가능한 엔드포인트를 만들 수 있다
- [ ] GitHub 웹훅 서명 검증 코드의 목적(위조 요청 차단)을 설명할 수 있다
- [ ] CloudWatch 로그에서 Lambda 실행 기록과 오류를 확인할 수 있다
- [ ] `pip install <패키지> -t .` 명령으로 Lambda 배포용 패키지를 준비할 수 있다

---

## 한 번 더 생각해 보기

1. **웹훅 vs 폴링**: 지금 만든 방식(웹훅)은 이벤트가 발생할 때만 Lambda가 실행됩니다. 반대로, Lambda가 5분마다 GitHub API를 호출해서 새 이슈를 확인하는 방식(폴링)과 비교하면 어떤 장단점이 있을까요? 비용, 속도, 복잡도 측면에서 생각해 보세요.

2. **실패 처리**: 지금 코드는 Claude API 호출이 실패하면 500 에러를 반환합니다. 하지만 GitHub는 실패한 웹훅을 자동으로 재시도합니다. 같은 이슈에 AI 답변이 여러 번 달리는 것을 막으려면 어떻게 설계해야 할까요?

3. **확장성**: 지금은 GitHub 이슈 하나만 처리합니다. Slack 메시지, 이메일 수신, Stripe 결제 완료 등 여러 종류의 웹훅을 하나의 Lambda에서 처리하려면 코드 구조를 어떻게 바꿀 수 있을까요?

---

## 다음 장

다음 장에서는 이번에 만든 Lambda를 확장해 **AI가 GitHub에 직접 댓글을 작성하고 레이블을 자동으로 분류**하는 완전한 이슈 트리아지(Triage) 봇을 완성합니다.