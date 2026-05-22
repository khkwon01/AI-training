# E2E 실습 워크시트: Issue → AI 코딩 → PR → Lambda 배포

---

## 이 워크시트의 목적

이 워크시트는 실제 개발 팀이 일하는 방식을 처음부터 끝까지 직접 따라해보는 실습 가이드입니다.

**시나리오**: 메모 서비스에 "태그 추가" 기능을 개발합니다.

- GitHub Issue로 작업 항목을 만들고
- 브랜치를 분기해서 Claude Code로 기능을 구현하고
- Pull Request를 열고 AI 코드 리뷰를 받은 뒤
- merge 후 Lambda에 배포해서 실제 API로 확인합니다

**소요 시간**: 약 60~90분

**사전 조건**:
- GitHub 계정이 있다
- AWS 계정이 있고 SAM CLI가 설치되어 있다
- Claude Code가 설치되어 있다 (`npm install -g @anthropic-ai/claude-code`)
- Git이 설치되어 있다

---

## 전체 흐름 한눈에 보기

```
[1] GitHub Issue 만들기
        ↓
[2] 브랜치 만들기 + 로컬 체크아웃
        ↓
[3] Claude Code로 기능 구현
        ↓
[4] 변경 내용 검토 + commit
        ↓
[5] push + PR 만들기
        ↓
[6] AI 코드 리뷰 + merge
        ↓
[7] Lambda 배포 + 검증
```

---

## 단계 1: GitHub Issue 만들기

### 목표

개발할 기능을 GitHub Issue로 문서화합니다. Issue는 "무엇을 왜 만드는가"를 기록하는 공간입니다. 코드를 작성하기 전에 Issue를 만들면 나중에 이 변경이 왜 생겼는지 이력이 남습니다.

### 실행 방법

**1. 저장소 페이지 열기**

GitHub에서 실습에 사용할 저장소를 엽니다. 없다면 새로 만듭니다.

- `https://github.com` 접속
- 오른쪽 상단 `+` → `New repository`
- 이름: `memo-service`, Public 선택, `Add a README file` 체크 후 생성

**2. Issues 탭 클릭**

저장소 페이지 상단 탭에서 `Issues`를 클릭합니다.

**3. New issue 클릭**

오른쪽 초록색 `New issue` 버튼을 클릭합니다.

**4. 이슈 제목과 본문 작성**

아래 예시를 복사해서 붙여넣습니다.

**이슈 제목**:
```
feat: 메모에 태그 추가 기능 구현
```

**이슈 본문**:
```markdown
## 배경

현재 메모 서비스는 텍스트만 저장합니다. 사용자가 메모를 분류하기 어렵습니다.
태그 기능을 추가하면 메모를 주제별로 묶어서 관리할 수 있습니다.

## 요구사항

- 메모 생성 시 태그 목록을 함께 전달할 수 있다
- 태그는 문자열 배열 형태로 저장한다 (예: `["work", "urgent"]`)
- 태그 없이 메모를 만들면 빈 배열로 처리한다
- 특정 태그로 메모를 조회할 수 있다 (GET /memos?tag=work)

## API 예시

메모 생성 (POST /memos):
```json
{
  "content": "프로젝트 마감일 확인",
  "tags": ["work", "urgent"]
}
```

응답:
```json
{
  "id": "abc123",
  "content": "프로젝트 마감일 확인",
  "tags": ["work", "urgent"],
  "created_at": "2026-01-01T00:00:00Z"
}
```

## 완료 기준

- [ ] POST /memos에서 tags 필드를 받아서 저장한다
- [ ] GET /memos?tag=work 로 해당 태그의 메모만 조회할 수 있다
- [ ] tags 없이 메모를 만들면 빈 배열로 저장된다
```

**5. 라벨 설정**

오른쪽 사이드바 `Labels` 클릭 → `enhancement` 선택

라벨이 없다면 `Labels` 옆 `Edit` → 기본 라벨 중 `enhancement` 선택

**6. 담당자 설정**

오른쪽 사이드바 `Assignees` 클릭 → 자신의 계정 선택

**7. Submit new issue 클릭**

초록색 `Submit new issue` 버튼을 클릭합니다.

### 완료 체크포인트

- [ ] Issue가 생성되고 이슈 번호(#1, #2 등)가 부여됐다
- [ ] `enhancement` 라벨이 붙어 있다
- [ ] 자신이 Assignee로 지정됐다
- [ ] 이슈 URL을 복사해 두었다 (나중에 사용)

---

## 단계 2: 브랜치 만들기 + 로컬 체크아웃

### 목표

main 브랜치에 직접 코드를 커밋하지 않고, 기능별 브랜치를 만들어서 작업합니다. 이렇게 하면 여러 사람이 동시에 다른 기능을 개발할 수 있습니다.

### 실행 방법

**방법 A: GitHub UI에서 브랜치 만들기 (권장)**

1. GitHub 저장소 페이지에서 `Code` 탭 클릭
2. 왼쪽 상단 브랜치 선택 드롭다운 (`main` 표시된 부분) 클릭
3. 텍스트 입력창에 새 브랜치 이름 입력:

```
feat/add-tag-support
```

4. `Create branch: feat/add-tag-support from 'main'` 클릭

**방법 B: 터미널에서 브랜치 만들기**

```bash
# 저장소 클론 (처음인 경우)
git clone https://github.com/[계정명]/memo-service.git
cd memo-service

# main 브랜치 최신 상태로 업데이트
git checkout main
git pull origin main

# 새 브랜치 만들고 이동
git checkout -b feat/add-tag-support
```

**로컬에 체크아웃**

GitHub UI에서 브랜치를 만든 경우, 터미널에서 가져옵니다.

```bash
# 저장소를 아직 클론하지 않았다면
git clone https://github.com/[계정명]/memo-service.git
cd memo-service

# 원격 브랜치 정보 가져오기
git fetch origin

# 새 브랜치로 이동
git checkout feat/add-tag-support
```

현재 브랜치 확인:

```bash
git branch
# * feat/add-tag-support
#   main
```

`*`가 `feat/add-tag-support` 앞에 있으면 올바른 브랜치에 있는 것입니다.

### 브랜치 이름 규칙 (참고)

| 접두사 | 용도 | 예시 |
|--------|------|------|
| `feat/` | 새 기능 추가 | `feat/add-tag-support` |
| `fix/` | 버그 수정 | `fix/null-pointer-on-empty-memo` |
| `refactor/` | 코드 개선 (기능 변화 없음) | `refactor/extract-memo-validator` |
| `docs/` | 문서 수정 | `docs/update-api-readme` |

### 완료 체크포인트

- [ ] `feat/add-tag-support` 브랜치가 생성됐다
- [ ] 터미널에서 `git branch` 실행 시 `*` 가 `feat/add-tag-support` 앞에 있다
- [ ] `ls` 또는 `dir` 명령으로 저장소 파일이 보인다

---

## 단계 3: Claude Code로 기능 구현

### 목표

Claude Code를 사용해서 태그 기능을 구현합니다. AI에게 정확한 컨텍스트를 제공하는 법을 연습합니다.

### 실행 방법

**1. 기본 메모 서비스 파일 준비**

아직 `app.py`가 없다면 기본 코드를 만듭니다. 저장소 루트에 `app.py` 파일을 생성합니다.

```python
# app.py — 기본 메모 서비스 (태그 기능 없는 버전)
import json
import uuid
import datetime

# 인메모리 저장소 (실습용)
memos = {}


def lambda_handler(event, context):
    method = event.get("httpMethod", "GET")
    path = event.get("path", "/")

    if method == "POST" and path == "/memos":
        return create_memo(event)
    elif method == "GET" and path == "/memos":
        return list_memos(event)
    else:
        return {
            "statusCode": 404,
            "body": json.dumps({"error": "Not found"})
        }


def create_memo(event):
    body = json.loads(event.get("body") or "{}")
    content = body.get("content", "")

    if not content:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "content is required"})
        }

    memo_id = str(uuid.uuid4())[:8]
    memo = {
        "id": memo_id,
        "content": content,
        "created_at": datetime.datetime.utcnow().isoformat()
    }
    memos[memo_id] = memo

    return {
        "statusCode": 201,
        "body": json.dumps(memo)
    }


def list_memos(event):
    return {
        "statusCode": 200,
        "body": json.dumps(list(memos.values()))
    }
```

**2. Claude Code 실행**

터미널에서 저장소 폴더로 이동한 뒤 Claude Code를 실행합니다.

```bash
cd memo-service
claude
```

**3. 실제 프롬프트 예시**

Claude Code 프롬프트 창에 다음을 입력합니다.

```
app.py 파일에 메모 태그 기능을 추가해줘.

요구사항:
1. POST /memos 요청 body에 "tags" 필드를 받을 수 있어야 해 (문자열 배열)
2. tags가 없으면 빈 배열 []로 저장해
3. GET /memos?tag=work 처럼 쿼리 파라미터로 특정 태그의 메모만 필터링할 수 있어야 해
4. 응답의 memo 객체에 tags 필드가 포함되어야 해

현재 코드는 app.py에 있어. 기존 create_memo, list_memos 함수를 수정해줘.
```

**Claude Code가 수정할 파일 예시**

Claude Code는 다음과 같이 `app.py`를 수정할 것입니다.

```python
# app.py — 태그 기능 추가 버전
import json
import uuid
import datetime

memos = {}


def lambda_handler(event, context):
    method = event.get("httpMethod", "GET")
    path = event.get("path", "/")

    if method == "POST" and path == "/memos":
        return create_memo(event)
    elif method == "GET" and path == "/memos":
        return list_memos(event)
    else:
        return {
            "statusCode": 404,
            "body": json.dumps({"error": "Not found"})
        }


def create_memo(event):
    body = json.loads(event.get("body") or "{}")
    content = body.get("content", "")

    if not content:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "content is required"})
        }

    # 태그 처리: 없으면 빈 배열
    tags = body.get("tags", [])
    if not isinstance(tags, list):
        tags = []

    memo_id = str(uuid.uuid4())[:8]
    memo = {
        "id": memo_id,
        "content": content,
        "tags": tags,  # 태그 필드 추가
        "created_at": datetime.datetime.utcnow().isoformat()
    }
    memos[memo_id] = memo

    return {
        "statusCode": 201,
        "body": json.dumps(memo)
    }


def list_memos(event):
    query_params = event.get("queryStringParameters") or {}
    tag_filter = query_params.get("tag")

    all_memos = list(memos.values())

    # 태그 필터링
    if tag_filter:
        all_memos = [m for m in all_memos if tag_filter in m.get("tags", [])]

    return {
        "statusCode": 200,
        "body": json.dumps(all_memos)
    }
```

**4. Claude Code 결과 확인**

Claude Code가 파일을 수정한 뒤 `/exit` 또는 `Ctrl+C`로 종료합니다.

```bash
# 파일이 수정됐는지 확인
cat app.py
```

### 완료 체크포인트

- [ ] `app.py`에 `tags` 필드 처리 코드가 추가됐다
- [ ] `create_memo` 함수에서 `tags = body.get("tags", [])` 코드가 보인다
- [ ] `list_memos` 함수에서 `tag_filter` 관련 코드가 보인다

---

## 단계 4: 변경 내용 검토 + commit

### 목표

코드를 commit하기 전에 변경 내용을 직접 눈으로 확인합니다. Claude Code가 수정한 내용이 의도한 대로인지 검증하는 습관을 기릅니다.

### 실행 방법

**1. git diff로 변경 내용 확인**

```bash
git diff
```

출력 예시:

```diff
diff --git a/app.py b/app.py
index 3a2b1c4..9f8e2d1 100644
--- a/app.py
+++ b/app.py
@@ -28,6 +28,10 @@ def create_memo(event):
         }

+    # 태그 처리: 없으면 빈 배열
+    tags = body.get("tags", [])
+    if not isinstance(tags, list):
+        tags = []
+
     memo_id = str(uuid.uuid4())[:8]
     memo = {
         "id": memo_id,
         "content": content,
+        "tags": tags,
         "created_at": datetime.datetime.utcnow().isoformat()
     }
```

### git diff 읽는 법

| 기호 | 의미 |
|------|------|
| `---` | 변경 전 파일 |
| `+++` | 변경 후 파일 |
| `-` (빨간색) | 삭제된 줄 |
| `+` (초록색) | 추가된 줄 |
| `@@` 이후 숫자 | 변경이 일어난 줄 번호 |

**2. 변경 내용을 스테이징 영역에 추가**

```bash
git add app.py
```

**3. 스테이징 상태 확인**

```bash
git status
```

```
On branch feat/add-tag-support
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app.py
```

**4. commit 작성**

좋은 commit 메시지 형식:

```
feat: add tag support to memo service

- POST /memos now accepts optional 'tags' field (string array)
- GET /memos?tag=<name> filters memos by tag
- Missing tags default to empty array []

Closes #1
```

터미널에서 실행:

```bash
git commit -m "feat: add tag support to memo service

- POST /memos now accepts optional 'tags' field (string array)
- GET /memos?tag=<name> filters memos by tag
- Missing tags default to empty array []

Closes #1"
```

### 좋은 commit 메시지 규칙

**제목 줄 (첫 번째 줄)**:
- `feat:` 새 기능, `fix:` 버그 수정, `refactor:` 리팩터링, `docs:` 문서
- 50자 이내로 작성
- 명령형으로 작성 ("추가했다" 대신 "추가")

**본문 (선택사항)**:
- 무엇을 왜 변경했는지 설명
- 줄 당 72자 이내

**`Closes #이슈번호`의 의미**:
- PR이 merge될 때 연결된 Issue를 자동으로 닫아줍니다
- `Closes #1` — 이슈 #1이 PR merge 시 자동 close

### 완료 체크포인트

- [ ] `git diff`로 추가된 코드(`+` 줄)가 의도한 내용과 일치한다
- [ ] `git status`에서 `app.py`가 `Changes to be committed`에 있다
- [ ] commit 메시지에 `feat:` 접두사와 `Closes #이슈번호`가 포함됐다
- [ ] `git log --oneline`으로 새 commit이 목록에 보인다

---

## 단계 5: push + PR 만들기

### 목표

로컬의 commit을 GitHub에 올리고 Pull Request를 만듭니다. PR은 코드 리뷰와 merge를 위한 공식 채널입니다.

### 실행 방법

**1. 원격 저장소에 브랜치 push**

```bash
git push origin feat/add-tag-support
```

처음 push하는 브랜치라면 `-u` 옵션을 붙입니다.

```bash
git push -u origin feat/add-tag-support
```

출력:

```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 892 bytes | 892.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote:
remote: Create a pull request for 'feat/add-tag-support' on GitHub by visiting:
remote:      https://github.com/[계정명]/memo-service/pull/new/feat/add-tag-support
remote:
To https://github.com/[계정명]/memo-service.git
 * [new branch]      feat/add-tag-support -> feat/add-tag-support
Branch 'feat/add-tag-support' set up to track remote branch 'feat/add-tag-support' from 'origin'.
```

**2. GitHub에서 PR 만들기**

push 후 GitHub 저장소 페이지를 열면 노란 배너가 나타납니다.

```
feat/add-tag-support had recent pushes  [Compare & pull request]
```

`Compare & pull request` 버튼을 클릭합니다.

또는 `Pull requests` 탭 → `New pull request` → base: `main`, compare: `feat/add-tag-support` 선택

**3. PR 제목과 본문 작성**

**PR 제목**:
```
feat: 메모에 태그 추가 기능 구현 (#1)
```

**PR 본문**:
```markdown
## 변경 요약

메모 서비스에 태그 기능을 추가합니다.

## 변경 내용

- `POST /memos` 요청 body에 `tags` 필드 추가 (선택사항)
- `GET /memos?tag=<name>` 으로 태그별 메모 조회 기능 추가
- `tags` 없이 메모 생성 시 빈 배열 `[]` 로 기본값 처리

## 테스트 방법

```bash
# 태그 있는 메모 생성
curl -X POST https://[URL]/Prod/memos \
  -H "Content-Type: application/json" \
  -d '{"content": "업무 메모", "tags": ["work", "urgent"]}'

# 태그로 조회
curl "https://[URL]/Prod/memos?tag=work"

# 태그 없이 생성
curl -X POST https://[URL]/Prod/memos \
  -H "Content-Type: application/json" \
  -d '{"content": "일반 메모"}'
```

## 관련 이슈

Closes #1
```

**4. Create pull request 클릭**

초록색 `Create pull request` 버튼을 클릭합니다.

### 완료 체크포인트

- [ ] `git push` 후 GitHub에 브랜치가 올라갔다
- [ ] PR이 생성됐고 PR 번호(#2 등)가 부여됐다
- [ ] PR 제목에 이슈 번호가 참조되어 있다
- [ ] `Closes #1` 이 PR 본문에 포함됐다
- [ ] PR 페이지의 `Files changed` 탭에서 수정된 코드가 보인다

---

## 단계 6: AI 코드 리뷰 + merge

### 목표

AI에게 코드 리뷰를 요청하고, 피드백을 검토한 뒤 PR을 merge합니다.

### 실행 방법

**방법 A: Claude Code로 리뷰 요청**

터미널에서 저장소 폴더로 이동 후 Claude Code를 실행합니다.

```bash
claude
```

다음 프롬프트를 입력합니다.

```
app.py 파일에서 방금 추가한 태그 기능 코드를 리뷰해줘.

확인해줘야 할 사항:
1. 에러 처리가 충분한가? (태그가 문자열 배열이 아닌 경우 등)
2. 보안 취약점이 있는가? (인젝션 공격 등)
3. 성능 문제가 있는가? (메모가 많을 때 태그 필터링 효율성)
4. 코드 가독성과 유지보수성

개선이 필요한 부분이 있으면 구체적인 코드 예시와 함께 알려줘.
```

**방법 B: GitHub PR에서 GitHub Copilot 리뷰 (Copilot이 있는 경우)**

PR 페이지에서 오른쪽 사이드바 `Reviewers` → `Copilot` 선택 후 리뷰 요청

**예상 리뷰 피드백 예시**

AI 리뷰어는 이런 문제를 찾을 수 있습니다.

```
발견된 문제:

1. 태그 유효성 검사 부족
   - 현재: 태그가 리스트인지만 확인
   - 개선: 각 태그가 문자열인지, 빈 문자열이 아닌지 확인 필요

2. 태그 최대 개수 제한 없음
   - 악의적 사용자가 수천 개의 태그를 넣을 수 있음
   - 개선: 태그는 최대 10개로 제한 권장

3. 태그 대소문자 정규화 없음
   - "Work"와 "work"가 다른 태그로 취급됨
   - 개선: 소문자로 통일 권장
```

**리뷰 피드백 반영 예시**

Claude Code에 피드백을 전달하고 수정을 요청합니다.

```
리뷰 피드백을 반영해서 app.py의 create_memo 함수를 개선해줘:
1. 태그 개수를 최대 10개로 제한
2. 각 태그를 소문자로 정규화
3. 빈 문자열 태그 제거
```

Claude Code가 수정한 코드:

```python
def create_memo(event):
    body = json.loads(event.get("body") or "{}")
    content = body.get("content", "")

    if not content:
        return {
            "statusCode": 400,
            "body": json.dumps({"error": "content is required"})
        }

    # 태그 처리: 유효성 검사 및 정규화
    raw_tags = body.get("tags", [])
    if not isinstance(raw_tags, list):
        raw_tags = []

    # 소문자 변환, 빈 문자열 제거, 최대 10개 제한
    tags = [t.lower().strip() for t in raw_tags if isinstance(t, str) and t.strip()]
    tags = list(dict.fromkeys(tags))  # 중복 제거 (순서 유지)
    tags = tags[:10]  # 최대 10개

    memo_id = str(uuid.uuid4())[:8]
    memo = {
        "id": memo_id,
        "content": content,
        "tags": tags,
        "created_at": datetime.datetime.utcnow().isoformat()
    }
    memos[memo_id] = memo

    return {
        "statusCode": 201,
        "body": json.dumps(memo)
    }
```

**수정 후 추가 commit 및 push**

```bash
git add app.py
git commit -m "refactor: improve tag validation based on code review

- Normalize tags to lowercase
- Remove empty string tags
- Limit tags to maximum 10 items
- Remove duplicate tags while preserving order"

git push origin feat/add-tag-support
```

**PR merge**

GitHub PR 페이지에서:

1. PR 페이지 아래로 스크롤
2. 초록색 `Merge pull request` 버튼 클릭
3. `Confirm merge` 클릭

merge 완료 후:
- PR 상태가 `Merged`(보라색)로 바뀝니다
- Issue #1이 자동으로 `Closed` 상태가 됩니다 (`Closes #1` 덕분에)

**로컬 main 브랜치 업데이트**

```bash
git checkout main
git pull origin main
```

### 완료 체크포인트

- [ ] AI 코드 리뷰 피드백을 받았다
- [ ] 피드백 중 적어도 하나를 반영해서 코드를 수정했다
- [ ] 수정된 코드를 추가 commit으로 push했다
- [ ] PR이 `Merged` 상태가 됐다
- [ ] Issue #1이 자동으로 `Closed` 됐다
- [ ] 로컬 main 브랜치에 merge된 코드가 포함됐다

---

## 단계 7: Lambda 배포 + 검증

### 목표

merge된 코드를 Lambda에 배포하고, 새로운 태그 기능이 실제로 동작하는지 확인합니다.

### 실행 방법

**방법 A: SAM CLI로 재배포 (권장)**

이전에 SAM으로 배포한 적 있다면 그대로 재배포합니다.

```bash
# 저장소 루트에 template.yaml이 있어야 합니다
# 없다면 먼저 sam init 하거나 아래 template.yaml을 만듭니다

sam build && sam deploy
```

template.yaml이 없다면 아래 내용으로 생성합니다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Memo Service with Tag Support

Globals:
  Function:
    Timeout: 10

Resources:
  MemoServiceFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Handler: app.lambda_handler
      Runtime: python3.12
      Events:
        CreateMemo:
          Type: Api
          Properties:
            Path: /memos
            Method: post
        ListMemos:
          Type: Api
          Properties:
            Path: /memos
            Method: get

Outputs:
  MemoServiceApi:
    Description: "Memo Service API endpoint"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod"
```

빌드 및 배포:

```bash
sam build
sam deploy
# (처음이라면 sam deploy --guided)
```

**방법 B: AWS 콘솔에서 코드 업데이트**

SAM이 없거나 이미 콘솔로 배포한 경우:

1. AWS 콘솔 → Lambda → 함수 선택
2. `Code` 탭 클릭
3. `Code source` 섹션에서 `app.py` 파일 선택
4. 로컬의 `app.py` 내용을 전체 복사해서 붙여넣기
5. `Deploy` 버튼 클릭

### curl로 새 기능 테스트

배포 URL을 변수에 저장해두면 편합니다.

```bash
BASE_URL="https://[출력된URL]/Prod"
```

**테스트 1: 태그 있는 메모 생성**

```bash
curl -X POST "$BASE_URL/memos" \
  -H "Content-Type: application/json" \
  -d '{"content": "프로젝트 마감일 확인", "tags": ["work", "urgent"]}'
```

예상 응답:

```json
{
  "id": "abc12345",
  "content": "프로젝트 마감일 확인",
  "tags": ["work", "urgent"],
  "created_at": "2026-01-01T00:00:00"
}
```

**테스트 2: 태그 없이 메모 생성**

```bash
curl -X POST "$BASE_URL/memos" \
  -H "Content-Type: application/json" \
  -d '{"content": "오늘 할 일 정리"}'
```

예상 응답:

```json
{
  "id": "def67890",
  "content": "오늘 할 일 정리",
  "tags": [],
  "created_at": "2026-01-01T00:00:01"
}
```

**테스트 3: 대소문자 정규화 확인**

```bash
curl -X POST "$BASE_URL/memos" \
  -H "Content-Type: application/json" \
  -d '{"content": "대소문자 테스트", "tags": ["Work", "URGENT"]}'
```

예상 응답: `"tags": ["work", "urgent"]` (소문자로 정규화됨)

**테스트 4: 태그 필터링**

```bash
# work 태그가 있는 메모만 조회
curl "$BASE_URL/memos?tag=work"
```

`work` 태그가 있는 메모만 응답에 포함됩니다.

**테스트 5: 없는 태그로 조회**

```bash
curl "$BASE_URL/memos?tag=nonexistent"
```

예상 응답: `[]` (빈 배열)

### CloudWatch 로그 확인

Lambda가 실행될 때 로그가 자동으로 CloudWatch에 저장됩니다.

**AWS 콘솔에서 확인**

1. AWS 콘솔 → Lambda → 함수 선택
2. `Monitor` 탭 클릭
3. `View CloudWatch logs` 버튼 클릭
4. 최신 로그 스트림 클릭

**AWS CLI로 확인**

```bash
# 로그 그룹 이름은 /aws/lambda/[함수이름] 형식
aws logs tail /aws/lambda/MemoServiceFunction --follow
```

`--follow` 옵션은 실시간으로 새 로그를 스트리밍합니다. curl로 요청을 보내면 로그가 즉시 나타납니다.

**로그 예시**

```
START RequestId: abc-123 Version: $LATEST
END RequestId: abc-123
REPORT RequestId: abc-123  Duration: 2.45 ms  Billed Duration: 3 ms  Memory Size: 128 MB  Max Memory Used: 40 MB
```

Duration이 짧을수록 좋습니다. 처음에는 200~500ms가 나올 수 있는데 (Cold Start), 연속 요청 시 2~10ms로 줄어듭니다.

### 완료 체크포인트

- [ ] `sam deploy` 또는 콘솔 업데이트로 새 코드가 배포됐다
- [ ] 태그 있는 메모를 생성하면 응답에 `tags` 필드가 포함된다
- [ ] 태그 없이 메모를 생성하면 `"tags": []`가 반환된다
- [ ] `?tag=work` 쿼리로 특정 태그의 메모만 조회된다
- [ ] CloudWatch 로그에서 함수 실행 기록을 확인했다

---

## 마무리: 완성 체크리스트

전체 워크플로우를 완주했는지 확인합니다.

### 전체 단계 완료 체크

- [ ] **단계 1**: GitHub Issue #1이 생성됐고 `enhancement` 라벨이 붙어 있다
- [ ] **단계 2**: `feat/add-tag-support` 브랜치가 생성됐고 로컬에서 작업했다
- [ ] **단계 3**: Claude Code로 태그 기능이 `app.py`에 구현됐다
- [ ] **단계 4**: `feat: add tag support` commit이 `Closes #1` 포함하여 생성됐다
- [ ] **단계 5**: PR이 생성됐고 변경 파일이 올바르게 표시됐다
- [ ] **단계 6**: AI 코드 리뷰 피드백을 반영하여 PR이 merge됐다
- [ ] **단계 7**: 배포된 API에서 태그 기능이 curl로 확인됐다

### 이 실습에서 배운 것

| 개념 | 배운 내용 |
|------|----------|
| GitHub Issue | 코드를 작성하기 전에 "무엇을 왜 만드는가" 기록 |
| 브랜치 전략 | main에 직접 push하지 않고 기능별 브랜치 사용 |
| AI 코딩 어시스턴트 | 명확한 컨텍스트 + 구체적 요구사항 제공 |
| git diff | 커밋 전 변경 내용 직접 눈으로 검증 |
| commit 메시지 | `feat:` 접두사, 본문에 이유 설명, `Closes #이슈` |
| PR | 코드 리뷰와 merge의 공식 채널 |
| AI 코드 리뷰 | 보안, 에러 처리, 성능 관점의 자동 리뷰 |
| SAM CLI | 코드로 Lambda 배포 — 재현 가능하고 팀 공유 가능 |
| CloudWatch | Lambda 실행 로그로 디버깅 |

---

## 다음 단계 안내

이 워크시트를 완료했다면 다음 주제로 이어갈 수 있습니다.

### 난이도를 높이고 싶다면

1. **데이터베이스 연결**: 인메모리 `memos` dict 대신 DynamoDB에 저장하기
2. **인증 추가**: API Gateway에 API Key 또는 JWT 인증 추가하기
3. **CI/CD 파이프라인**: GitHub Actions로 push 시 자동 배포하기
4. **여러 Lambda 함수**: 생성/조회/삭제를 별도 함수로 분리하기

### 추가 실습 아이디어

- 태그로 메모를 정렬하는 기능 추가 (`?sort=tag`)
- 메모 수정 기능 추가 (`PUT /memos/{id}`)
- 메모 삭제 기능 추가 (`DELETE /memos/{id}`)
- 태그 목록 조회 API 추가 (`GET /tags`)

### 참고 자료

- artifacts/04-vibe-coding/vibe-chapter-07-full-service-with-ai.md
- artifacts/02-github/github-chapter-18-claude-code-pr-workflow.md
- artifacts/00-curriculum/curriculum-e2e-tutorial.md
- artifacts/05-aws-deploy/aws-chapter-19-sam-cli-deployment.md
