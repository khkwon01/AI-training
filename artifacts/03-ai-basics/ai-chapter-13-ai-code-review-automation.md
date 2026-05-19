## 이 장에서 배우는 것

- GitHub Actions 워크플로우 파일이 무엇인지, 어디에 두는지 이해한다
- Pull Request가 열릴 때 자동으로 AI 코드 리뷰가 달리는 구조를 설명할 수 있다
- Claude API를 호출하는 Python 스크립트를 직접 작성한다
- GitHub Secrets에 API 키를 안전하게 저장하고 워크플로우에서 꺼내 쓸 수 있다
- 워크플로우가 실패했을 때 오류 로그를 읽고 원인을 찾을 수 있다

---

## 먼저 쉬운 설명

코드를 PR로 올릴 때마다 팀원이 "이 변수 이름 좀 바꾸면 어때요?", "여기 예외 처리 빠진 것 같아요" 같은 코멘트를 달아 줍니다. 그런데 팀원이 바빠서 리뷰가 늦어지거나, 혼자 개발하는 경우엔 이런 피드백을 받기 어렵죠.

GitHub Actions는 PR을 열 때, 커밋을 올릴 때처럼 **특정 이벤트가 발생하면 자동으로 스크립트를 실행**해 주는 도구입니다. 여기에 AI API를 연결하면, PR을 열자마자 수 초 안에 AI가 변경된 코드를 읽고 리뷰 코멘트를 자동으로 달아 줍니다.

이 장이 끝나면 혼자 개발해도 항상 AI 리뷰어가 옆에 있는 환경을 만들 수 있습니다.

---

## 1. GitHub Actions 워크플로우 기본 구조

워크플로우 파일은 항상 `.github/workflows/` 폴더 아래에 `.yml` 확장자로 저장합니다. 파일 이름은 자유롭게 정할 수 있습니다.

```
내-프로젝트/
├── .github/
│   └── workflows/
│       └── ai-code-review.yml   ← 여기에 작성
├── src/
│   └── main.py
└── README.md
```

가장 단순한 워크플로우 구조는 다음과 같습니다.

```yaml
# .github/workflows/ai-code-review.yml

name: AI 코드 리뷰          # Actions 탭에 표시될 이름

on:                          # 언제 실행할지
  pull_request:
    types: [opened, synchronize]   # PR 생성 또는 새 커밋 Push 때

jobs:
  review:                    # 잡(job) 이름 (자유롭게)
    runs-on: ubuntu-latest   # 실행 환경

    steps:
      - name: 코드 내려받기
        uses: actions/checkout@v4

      - name: Python 설치
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: 필요한 패키지 설치
        run: pip install anthropic PyGithub

      - name: AI 코드 리뷰 실행
        run: python scripts/ai_review.py
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
          REPO_NAME: ${{ github.repository }}
```

> **핵심 개념**
> - `on`: 트리거 조건. `pull_request`는 PR 관련 이벤트를 의미합니다.
> - `jobs`: 실제로 실행할 작업 묶음입니다.
> - `steps`: 잡 안에서 순서대로 실행하는 각 단계입니다.
> - `${{ secrets.이름 }}`: GitHub Secrets에 저장한 비밀값을 꺼내는 문법입니다.

---

## 2. GitHub Secrets에 API 키 저장하기

API 키를 코드에 직접 쓰면 GitHub에 올라가서 누구나 볼 수 있습니다. 반드시 Secrets에 저장해야 합니다.

**저장 경로:**
```
GitHub 리포지토리 → Settings → Secrets and variables → Actions → New repository secret
```

| 이름 | 값 |
|------|-----|
| `ANTHROPIC_API_KEY` | `sk-ant-api03-...` (Anthropic 콘솔에서 복사) |

`GITHUB_TOKEN`은 GitHub이 자동으로 제공하므로 따로 등록하지 않아도 됩니다.

워크플로우 파일에서는 아래처럼 환경변수로 꺼내 씁니다.

```yaml
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

Python 스크립트에서는 `os.environ`으로 읽습니다.

```python
import os

api_key = os.environ["ANTHROPIC_API_KEY"]   # 없으면 KeyError 발생
```

---

## 3. PR의 변경된 코드(diff) 가져오기

AI에게 리뷰를 맡기려면 먼저 "어떤 코드가 바뀌었는지"를 알아야 합니다. `PyGithub` 라이브러리로 PR의 diff를 가져옵니다.

```python
# scripts/ai_review.py
import os
from github import Github

def get_pr_diff() -> str:
    """PR에서 변경된 코드 전체를 문자열로 반환합니다."""
    token = os.environ["GITHUB_TOKEN"]
    repo_name = os.environ["REPO_NAME"]       # 예: "username/my-project"
    pr_number = int(os.environ["PR_NUMBER"])

    g = Github(token)
    repo = g.get_repo(repo_name)
    pr = repo.get_pull(pr_number)

    diff_text = ""
    for file in pr.get_files():
        if file.patch is None:          # 바이너리 파일 등은 건너뜀
            continue
        diff_text += f"\n### 파일: {file.filename}\n"
        diff_text += f"```diff\n{file.patch}\n```\n"

    return diff_text
```

`file.patch`는 아래처럼 생긴 diff 텍스트입니다.

```diff
@@ -10,6 +10,8 @@ def calculate_total(items):
     total = 0
     for item in items:
-        total += item
+        if item > 0:          # 음수 필터링 추가
+            total += item
     return total
```

`+`로 시작하는 줄이 추가된 코드, `-`로 시작하는 줄이 삭제된 코드입니다.

---

## 4. Claude API로 코드 리뷰 요청하기

diff 텍스트를 Claude에 넘기고 리뷰를 받아 오는 함수입니다.

```python
import anthropic

def request_ai_review(diff_text: str) -> str:
    """diff를 Claude에 보내고 리뷰 코멘트를 받아 반환합니다."""
    client = anthropic.Anthropic(
        api_key=os.environ["ANTHROPIC_API_KEY"]
    )

    system_prompt = (
        "당신은 친절한 시니어 소프트웨어 엔지니어입니다. "
        "제공된 코드 diff를 검토하고 다음 항목을 한국어로 피드백해 주세요:\n"
        "1. 잠재적 버그나 예외 처리 누락\n"
        "2. 가독성 또는 명명 개선 제안\n"
        "3. 잘 작성된 부분에 대한 칭찬\n"
        "답변은 GitHub PR 코멘트에 바로 붙여 넣을 수 있는 Markdown 형식으로 작성하세요."
    )

    message = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": f"다음 코드 변경사항을 리뷰해 주세요:\n\n{diff_text}"
            }
        ],
        system=system_prompt,
    )

    return message.content[0].text
```

---

## 5. AI 리뷰 결과를 PR 코멘트로 달기

리뷰 텍스트를 받아 왔으면 PR에 코멘트로 달아 줍니다.

```python
def post_review_comment(review_text: str) -> None:
    """PR에 리뷰 코멘트를 작성합니다."""
    token = os.environ["GITHUB_TOKEN"]
    repo_name = os.environ["REPO_NAME"]
    pr_number = int(os.environ["PR_NUMBER"])

    g = Github(token)
    repo = g.get_repo(repo_name)
    pr = repo.get_pull(pr_number)

    comment_body = (
        "## 🤖 AI 코드 리뷰\n\n"
        + review_text
        + "\n\n---\n*이 리뷰는 Claude AI가 자동으로 생성했습니다.*"
    )
    pr.create_issue_comment(comment_body)
    print("코멘트 작성 완료!")


# 세 함수를 순서대로 실행하는 진입점
if __name__ == "__main__":
    diff = get_pr_diff()

    if not diff.strip():
        print("변경된 코드가 없습니다. 종료합니다.")
        exit(0)

    review = request_ai_review(diff)
    post_review_comment(review)
```

---

## 따라 하기 실습

### 실습 1 — 프로젝트 구조 만들고 워크플로우 파일 추가하기

로컬에서 아래 명령어로 폴더와 파일을 만듭니다.

```bash
# 프로젝트 루트에서 실행
mkdir -p .github/workflows scripts

touch .github/workflows/ai-code-review.yml
touch scripts/ai_review.py
```

`.github/workflows/ai-code-review.yml`에 2절의 워크플로우 YAML을 그대로 붙여 넣은 뒤 커밋하고 Push합니다.

```bash
git add .github/workflows/ai-code-review.yml
git commit -m "chore: AI 코드 리뷰 워크플로우 추가"
git push origin main
```

GitHub 리포지토리의 **Actions 탭**을 열어 워크플로우가 목록에 나타나는지 확인합니다.

---

### 실습 2 — `scripts/ai_review.py` 완성하고 Secrets 등록하기

3·4·5절의 코드를 `scripts/ai_review.py` 한 파일에 합쳐서 저장합니다.

```python
# scripts/ai_review.py  (전체 코드)
import os
from github import Github
import anthropic


def get_pr_diff() -> str:
    # ... (3절 코드)


def request_ai_review(diff_text: str) -> str:
    # ... (4절 코드)


def post_review_comment(review_text: str) -> None:
    # ... (5절 코드)


if __name__ == "__main__":
    diff = get_pr_diff()
    if not diff.strip():
        print("변경된 코드가 없습니다. 종료합니다.")
        exit(0)
    review = request_ai_review(diff)
    post_review_comment(review)
```

그 다음 GitHub 리포지토리의 **Settings → Secrets and variables → Actions**에서 `ANTHROPIC_API_KEY`를 등록합니다.

```bash
git add scripts/ai_review.py
git commit -m "feat: AI 코드 리뷰 스크립트 추가"
git push origin main
```

---

### 실습 3 — 테스트용 PR 열어서 자동 리뷰 확인하기

새 브랜치를 만들어 간단한 Python 함수를 추가하고 PR을 엽니다.

```bash
git checkout -b feature/add-greeting
```

`src/greeting.py` 파일을 새로 만들고 저장합니다.

```python
# src/greeting.py
def greet(name):
    print("안녕하세요, " + name)
```

```bash
git add src/greeting.py
git commit -m "feat: 인사 함수 추가"
git push origin feature/add-greeting
```

GitHub에서 **Compare & pull request** 버튼을 눌러 PR을 생성합니다. 수십 초 안에 Actions가 실행되고, PR 코멘트에 AI 리뷰가 달립니다. Actions 탭에서 실행 로그도 확인해 보세요.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|----------------------|-----------|
| `ANTHROPIC_API_KEY` Secrets 등록을 빠뜨림 | `KeyError: 'ANTHROPIC_API_KEY'` | Settings → Secrets에서 키 등록 확인 |
| 워크플로우 파일을 `.github/workflows/` 가 아닌 다른 위치에 저장 | Actions 탭에 워크플로우가 아예 안 보임 | 폴더 경로 철자 확인 (`.github`는 점으로 시작) |
| YAML 들여쓰기를 탭(Tab)으로 사용 | `mapping values are not allowed here` | YAML은 반드시 스페이스 2칸 들여쓰기 사용 |
| `pull_request` 이벤트에서 `GITHUB_TOKEN` 권한 부족 | `403 Resource not accessible by integration` | Settings → Actions → General → Workflow permissions를 **Read and write** 로 변경 |
| diff가 너무 커서 토큰 초과 | `anthropic.BadRequestError: prompt is too long` | `max_tokens` 늘리거나 파일 개수·줄 수 제한 로직 추가 |
| `pip install` 없이 스크립트 실행 | `ModuleNotFoundError: No module named 'anthropic'` | 워크플로우에 `pip install anthropic PyGithub` 단계 추가 |

---

## 확인 체크리스트

- [ ] `.github/workflows/ai-code-review.yml` 파일이 존재한다
- [ ] `on: pull_request` 트리거가 올바르게 설정되어 있다
- [ ] `ANTHROPIC_API_KEY`가 GitHub Secrets에 등록되어 있다
- [ ] `scripts/ai_review.py`에 `get_pr_diff`, `request_ai_review`, `post_review_comment` 세 함수가 모두 있다
- [ ] 테스트 PR을 열었을 때 Actions 탭에서 워크플로우가 실행됐다
- [ ] PR 코멘트에 AI 리뷰가 자동으로 달렸다
- [ ] Actions 탭에서 실행 로그를 직접 열어 각 step이 녹색(성공)임을 확인했다

---

## 한 번 더 생각해 보기

1. 지금 구현은 PR에 코멘트가 열릴 때마다 새 코멘트를 추가합니다. 같은 PR에 커밋을 여러 번 올리면 코멘트가 여러 개 쌓이는데, 이를 방지하려면 어떤 방법을 쓸 수 있을까요?

2. 현재는 PR의 모든 파일 diff를 AI에 보냅니다. 파일이 수십 개이거나 diff가 수천 줄이면 어떤 문제가 생길까요? 어떻게 줄일 수 있을까요?

3. AI 리뷰의 품질은 system prompt에 크게 달려 있습니다. 팀의 코딩 컨벤션이나 자주 발생하는 버그 유형을 system prompt에 추가하면 어떤 효과가 있을지 생각해 보세요.

---

## 다음 장

다음 장에서는 이 자동 리뷰 시스템을 확장해, AI가 보안 취약점과 테스트 누락 여부까지 검사하고 PR 머지를 자동으로 차단하는 **Branch Protection Rule 연동**을 설정합니다.