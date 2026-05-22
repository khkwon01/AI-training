# Chapter 18: Claude Code와 함께하는 PR 워크플로우

## 이 장에서 배우는 것

- Claude Code 터미널에서 branch를 만드는 방법
- Claude Code에 기능 추가를 요청하는 효과적인 프롬프트 작성법
- `git diff`로 Claude Code가 변경한 내용을 꼼꼼히 검토하는 방법
- Conventional Commits 형식으로 commit 메시지 작성하기
- push 후 GitHub에서 PR 만들기 (터미널 `gh` CLI 포함)
- 실습: 메모 서비스에 검색 기능 추가 → Claude Code 요청 → diff 검토 → PR 생성

---

## 왜 AI와 Git 워크플로우를 함께 배워야 할까

Claude Code 같은 AI 도구는 코드를 대신 작성해준다. 하지만 AI가 파일을 수정했다고 해서 그냥 commit하고 push하면 안 된다.

왜냐하면:

1. **AI는 의도를 정확히 파악하지 못할 수 있다.** 요청한 것 외에 다른 부분도 바꾸거나, 미묘하게 다른 방식으로 구현할 수 있다.
2. **AI는 테스트를 직접 실행하지 않는다.** 문법적으로 올바른 코드라도 실행해보면 오류가 날 수 있다.
3. **최종 책임은 사람에게 있다.** 버그가 발생하면 "AI가 만들었어요"는 이유가 되지 않는다.

Git 워크플로우(branch → diff 검토 → commit → PR)는 AI가 만든 코드를 안전하게 검토하고 관리하는 구조를 제공한다. 이 흐름을 지키면 실수를 사전에 잡고, 언제든 이전 상태로 돌아갈 수 있다.

---

## 1. Claude Code 터미널에서 시작하기

### 1-1. Claude Code 설치 확인

터미널을 열고 아래 명령을 실행한다.

```bash
claude --version
```

버전 번호가 출력되면 설치된 것이다. 설치되지 않았다면:

```bash
npm install -g @anthropic-ai/claude-code
```

### 1-2. 프로젝트 폴더에서 시작

Claude Code는 **반드시 프로젝트 폴더 안에서 실행**해야 한다. 그래야 해당 프로젝트의 파일을 수정할 수 있다.

```bash
# 프로젝트 폴더로 이동
cd ~/Desktop/memo-service

# Claude Code 시작
claude
```

실행하면 아래처럼 프롬프트가 나타난다:

```
Claude Code v1.x.x  (memo-service 폴더)
> 
```

`>` 뒤에 커서가 깜빡이면 입력할 준비가 된 것이다.

### 1-3. Claude Code 종료

작업을 마치면 `/exit` 또는 `Ctrl+C`로 종료한다.

---

## 2. 작업 전: 새 branch 만들기

기능을 추가하기 전에 **반드시 새 branch를 먼저 만든다.** main branch에서 직접 작업하면:
- 실수로 main이 망가질 수 있다
- PR 없이 바로 main에 올라간다
- 협업 시 다른 사람의 작업과 충돌할 가능성이 높아진다

### 2-1. 터미널에서 branch 만들기

Claude Code를 시작하기 전, 또는 Claude Code 안에서 bash 명령을 실행해서 branch를 만든다.

```bash
# Claude Code 밖에서 (일반 터미널에서)
git checkout -b feature/add-search-memo
```

또는 Claude Code 안에서:

```
> git checkout -b feature/add-search-memo 명령을 실행해줘
```

Claude Code가 이 명령을 실행하고 결과를 알려준다:

```
Switched to a new branch 'feature/add-search-memo'
```

### 2-2. 좋은 branch 이름 규칙

| 패턴 | 예시 | 언제 사용 |
|------|------|----------|
| `feature/설명` | `feature/add-search-memo` | 새 기능 추가 |
| `fix/설명` | `fix/empty-string-error` | 버그 수정 |
| `docs/설명` | `docs/update-readme` | 문서 수정 |
| `refactor/설명` | `refactor/split-memo-module` | 코드 구조 개선 |
| `feature/이슈번호-설명` | `feature/5-add-search-memo` | issue와 연결 시 |

나쁜 이름: `branch1`, `test`, `my-work`, `수정`, `abc`

branch 이름만 봐도 무슨 작업인지 알 수 있어야 한다.

### 2-3. 현재 branch 확인

```bash
git branch
```

출력에서 `*` 가 붙은 줄이 현재 branch다.

```
* feature/add-search-memo
  main
```

---

## 3. Claude Code에 기능 추가 요청하기

### 3-1. 좋은 프롬프트의 구성 요소

AI에게 요청할 때 막연하게 "기능 추가해줘"라고 하면 AI가 잘못 이해하거나 과도하게 수정할 수 있다.

좋은 프롬프트는 다음 요소를 포함한다:

1. **어떤 파일**을 수정하는지
2. **무슨 함수/기능**을 추가하는지
3. **어떻게 동작**해야 하는지 (입력, 출력, 예외 처리)
4. **하지 말아야 할 것** (기존 코드를 건드리지 말것 등)

### 3-2. 나쁜 프롬프트 vs 좋은 프롬프트

나쁜 프롬프트:
```
> 검색 기능 추가해줘
```

문제: AI가 어떤 파일에, 어떤 방식으로, 어떤 예외 처리를 해야 하는지 모른다.

좋은 프롬프트:
```
> memo.py 파일에 search_memo(keyword) 함수를 추가해줘.
> 기능:
> - memos 리스트에서 keyword를 포함하는 항목만 반환
> - keyword가 빈 문자열이면 전체 memos 반환
> - keyword가 None이면 [] 반환
> 기존 함수(add_memo, delete_memo, list_memos)는 수정하지 마.
```

### 3-3. 실제 요청 예시

Claude Code 프롬프트에 아래처럼 입력한다:

```
> memo.py에 search_memo(keyword) 함수를 추가해줘.
  키워드가 포함된 메모만 반환하고,
  빈 문자열이면 전체 메모를 반환해야 해.
  기존 함수는 건드리지 마.
```

Claude Code가 파일을 수정하고 어떤 내용을 추가했는지 설명한다.

```
memo.py에 search_memo() 함수를 추가했습니다.

추가된 내용:
- def search_memo(keyword): 함수 추가 (line 25)
  - keyword가 빈 문자열이면 memos 전체 반환
  - 그 외에는 keyword 포함된 항목만 필터링해서 반환
```

### 3-4. 단계적으로 요청하기

한 번에 너무 많은 것을 요청하면 AI가 실수할 가능성이 높아진다.

한 번에 하나씩 요청하는 것이 좋다.

```
# 1단계: 함수 추가
> search_memo(keyword) 함수를 memo.py에 추가해줘

# (diff 확인 후)

# 2단계: 예외 처리 추가
> search_memo에 None 입력 처리를 추가해줘

# (diff 확인 후)

# 3단계: 필요하면 더 요청
```

---

## 4. git diff로 변경 내용 검토하기

Claude Code가 파일을 수정하면 **반드시 `git diff`로 확인**한다. 이것이 가장 중요한 습관이다.

### 4-1. git diff 실행

```bash
git diff
```

### 4-2. diff 출력 읽기

```diff
diff --git a/memo.py b/memo.py
index 3a7f1c2..b9e4d5a 100644
--- a/memo.py
+++ b/memo.py
@@ -18,6 +18,16 @@ def list_memos():
     for i, memo in enumerate(memos, 1):
         print(f"{i}. {memo}")

+def search_memo(keyword):
+    """키워드가 포함된 메모를 반환한다."""
+    if keyword is None:
+        return []
+    if not keyword.strip():
+        return memos[:]
+    result = [m for m in memos if keyword in m]
+    return result
+
 def delete_memo(number):
```

| 요소 | 의미 |
|------|------|
| `diff --git a/memo.py b/memo.py` | memo.py 파일이 변경됨 |
| `--- a/memo.py` | 변경 전 파일 |
| `+++ b/memo.py` | 변경 후 파일 |
| `@@ -18,6 +18,16 @@` | 변경 위치 (18번 줄 근처) |
| `-` 로 시작하는 줄 | 삭제된 줄 |
| `+` 로 시작하는 줄 | 추가된 줄 |

### 4-3. diff를 보면서 확인할 체크리스트

```
□ 내가 요청한 파일만 수정됐는가? (요청하지 않은 파일이 바뀌었으면 주의)
□ 요청한 함수/내용이 올바르게 추가됐는가?
□ 기존 함수가 의도치 않게 수정되지 않았는가?
□ 함수 이름, 매개변수 이름이 내가 원한 대로인가?
□ 예외 처리가 포함돼 있는가?
□ 불필요한 주석이나 테스트 코드가 섞이지 않았는가?
```

### 4-4. 수정된 파일 목록만 확인

변경된 파일이 무엇인지만 먼저 보고 싶을 때:

```bash
git status
```

```
On branch feature/add-search-memo
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   memo.py
```

`modified:` 옆에 파일명이 표시된다. 요청하지 않은 파일이 없는지 확인한다.

### 4-5. 특정 파일만 diff 보기

```bash
git diff memo.py
```

파일이 여러 개 수정됐을 때 하나씩 확인할 수 있다.

### 4-6. VS Code에서 diff 보기

터미널 diff가 읽기 어렵다면 VS Code의 Source Control 패널을 사용한다.

1. VS Code에서 `Cmd+Shift+G` (Mac) / `Ctrl+Shift+G` (Windows)
2. Changes 목록에서 파일 이름 클릭
3. 좌우로 나뉜 diff 뷰에서 색으로 변경 내용 확인

---

## 5. Conventional Commits 형식으로 commit 메시지 작성

### 5-1. Conventional Commits란

Conventional Commits는 commit 메시지를 일관된 형식으로 작성하는 규칙이다. 팀원이 commit 목록만 봐도 무슨 작업을 했는지 바로 파악할 수 있다.

기본 형식:

```
<타입>: <요약>

<본문 (선택사항)>

<푸터 (선택사항)>
```

### 5-2. 타입 종류

| 타입 | 의미 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | `feat: 메모 검색 기능 추가` |
| `fix` | 버그 수정 | `fix: 빈 문자열 검색 시 오류 수정` |
| `docs` | 문서 변경 | `docs: README에 사용법 추가` |
| `refactor` | 기능 변경 없이 코드 구조 개선 | `refactor: memo 함수 모듈 분리` |
| `test` | 테스트 코드 추가/수정 | `test: search_memo 단위 테스트 추가` |
| `chore` | 빌드, 설정 등 기타 변경 | `chore: requirements.txt 업데이트` |
| `style` | 코드 스타일만 변경 (기능 변화 없음) | `style: 들여쓰기 통일` |

### 5-3. 좋은 commit 메시지 예시

짧은 버전:
```
feat: 메모 검색 기능 추가
```

상세 버전 (큰 변경사항일 때):
```
feat: 메모 검색 기능 추가

keyword로 메모를 검색하는 search_memo() 함수를 추가했다.
- keyword가 포함된 메모만 반환
- 빈 문자열 입력 시 전체 메모 반환
- None 입력 시 빈 리스트 반환

Closes #5
```

### 5-4. 나쁜 commit 메시지 예시

```
수정
업데이트
fix
wip
asdf
Claude Code로 변경
```

왜 나쁜가: commit 목록에서 "무엇을 왜 했는지" 알 수 없다.

### 5-5. commit 실행

```bash
# 파일 stage
git add memo.py

# commit (짧은 메시지)
git commit -m "feat: 메모 검색 기능 추가"

# commit (여러 줄 메시지)
git commit -m "feat: 메모 검색 기능 추가

keyword로 메모를 검색하는 search_memo() 함수 추가.
빈 문자열이면 전체 메모 반환.

Closes #5"
```

### 5-6. AI가 만든 코드임을 명시

AI 도구를 사용했다면 commit 메시지나 PR 설명에 명시하는 것이 좋다.

```
feat: 메모 검색 기능 추가 (Claude Code)
```

또는 PR 설명에:

```
이 PR의 코드는 Claude Code의 도움으로 작성되었습니다.
로컬에서 실행 테스트 완료.
```

---

## 6. push 후 GitHub에서 PR 만들기

### 6-1. push

```bash
git push origin feature/add-search-memo
```

처음 push할 때 `-u` 옵션을 추가하면 이후 `git push`만 입력해도 된다:

```bash
git push -u origin feature/add-search-memo
```

push 성공 시 출력:

```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
...
To https://github.com/내이름/memo-service.git
 * [new branch]      feature/add-search-memo -> feature/add-search-memo
```

그리고 바로 아래에 PR 링크가 나온다:

```
remote: Create a pull request for 'feature/add-search-memo' on GitHub by visiting:
remote:   https://github.com/내이름/memo-service/pull/new/feature/add-search-memo
```

이 링크를 브라우저에서 열면 PR 만들기 화면으로 바로 이동한다.

### 6-2. GitHub 웹에서 PR 만들기

링크를 클릭하거나 GitHub 저장소 페이지를 열면 노란색 배너가 나타난다:

```
feature/add-search-memo had recent pushes less than a minute ago.
[Compare & pull request]
```

**Compare & pull request** 버튼을 클릭한다.

PR 만들기 화면에서:

1. **제목**: `feat: 메모 검색 기능 추가`
2. **설명** (아래 형식 권장):

```markdown
## 변경 내용
keyword로 메모를 검색하는 search_memo() 함수를 추가했습니다.

## 구현 내용
- keyword가 포함된 메모만 반환
- 빈 문자열 입력 시 전체 메모 반환
- None 입력 시 빈 리스트 반환

## 테스트
- search_memo("Python") → "Python"이 포함된 메모 반환 확인
- search_memo("") → 전체 메모 반환 확인
- search_memo(None) → [] 반환 확인

## 참고
이 코드는 Claude Code의 도움으로 작성 후 로컬 테스트 완료.

Closes #5
```

3. **Reviewers**: 리뷰어 지정 (협업 시)
4. **Create pull request** 클릭

### 6-3. GitHub CLI로 PR 만들기 (선택)

`gh` CLI가 설치됐다면 터미널에서 바로 만들 수 있다.

```bash
# gh CLI 설치 확인
gh --version

# PR 만들기
gh pr create \
  --title "feat: 메모 검색 기능 추가" \
  --body "search_memo() 함수 추가. 빈 문자열 처리 포함. Closes #5"
```

또는 대화형으로:

```bash
gh pr create
# 대화형으로 제목, 설명, base branch 등을 입력
```

---

## 실습

### 실습 1. 따라 하기: Claude Code로 검색 기능 추가

목표: Claude Code를 사용해서 memo.py에 search_memo() 함수를 추가한다.

**준비**:
- `memo-service` 저장소가 로컬에 있어야 한다
- `memo.py`에 기본 메모 함수(add_memo, list_memos, delete_memo)가 있어야 한다

**1단계: branch 만들기**

```bash
cd memo-service
git checkout -b feature/add-search-memo
git branch  # 현재 branch 확인
```

**2단계: Claude Code 시작**

```bash
claude
```

**3단계: 기능 요청**

```
> memo.py에 search_memo(keyword) 함수를 추가해줘.
  기능:
  1. keyword가 포함된 메모만 리스트로 반환
  2. keyword가 빈 문자열("")이면 전체 memos 반환
  3. keyword가 None이면 빈 리스트 반환
  기존 add_memo, list_memos, delete_memo 함수는 수정하지 마.
```

**4단계: Claude Code 종료**

```
/exit
```

---

### 실습 2. 따라 하기: diff 검토하고 commit

목표: Claude Code가 만든 코드를 꼼꼼히 검토하고 commit한다.

**1단계: 변경된 파일 확인**

```bash
git status
```

`memo.py`만 변경됐는지 확인한다. 다른 파일이 변경됐으면 의도한 것인지 확인한다.

**2단계: diff 확인**

```bash
git diff memo.py
```

아래 체크리스트를 따라 확인한다:

```
□ search_memo 함수가 추가됐는가 (+ 줄 확인)
□ 기존 add_memo, list_memos, delete_memo가 변경되지 않았는가 (- 줄 없어야 함)
□ 함수 이름이 search_memo인가 (오타 확인)
□ None 처리가 있는가
□ 빈 문자열 처리가 있는가
```

문제가 있으면 Claude Code에게 다시 요청한다:

```
> search_memo에서 None 처리가 빠진 것 같아. if keyword is None: return [] 추가해줘
```

**3단계: 실제로 실행해서 테스트**

```bash
python3 -c "
from memo import add_memo, search_memo, memos
add_memo('Python 배우기')
add_memo('Git 공부하기')
add_memo('Python 복습')
print(search_memo('Python'))   # ['Python 배우기', 'Python 복습']
print(search_memo(''))          # 전체 메모
print(search_memo(None))        # []
"
```

출력이 예상과 같은지 확인한다.

**4단계: commit**

```bash
git add memo.py
git commit -m "feat: 메모 검색 기능 추가

keyword로 메모를 검색하는 search_memo() 함수 추가.
- keyword 포함 메모만 반환
- 빈 문자열이면 전체 반환
- None이면 [] 반환

Closes #5"
```

---

### 실습 3. 직접 해보기: push 후 PR 생성

목표: 완성된 branch를 push하고 GitHub에서 PR을 만들어 전체 흐름을 완료한다.

**1단계: push**

```bash
git push origin feature/add-search-memo
```

터미널 출력에서 PR 링크를 찾는다:

```
remote:   https://github.com/내이름/memo-service/pull/new/feature/add-search-memo
```

**2단계: PR 만들기**

브라우저에서 위 링크를 클릭하거나, GitHub 저장소 페이지에서 노란 배너의 **Compare & pull request**를 클릭한다.

아래 내용으로 PR을 작성한다:

- **제목**: `feat: 메모 검색 기능 추가`
- **설명**: 변경 내용, 테스트 방법, `Closes #번호` 포함
- **Labels**: `enhancement` 선택

**3단계: Files changed 확인**

PR 페이지에서 **Files changed** 탭을 클릭하고 diff를 한 번 더 확인한다.

**4단계: Create pull request**

모두 확인했으면 **Create pull request** 버튼을 클릭한다.

**5단계: 셀프 리뷰 또는 merge**

혼자 작업 중이라면 본인이 직접 **Merge pull request**를 클릭해서 main에 합친다.

merge 후 issue가 자동으로 Closed 상태가 됐는지 Issues 탭에서 확인한다.

---

## 자주 하는 실수

| 실수 | 어떤 일이 생기나 | 해결 방법 |
|------|----------------|----------|
| main branch에서 직접 작업 | PR 없이 main에 push됨 | 작업 전 반드시 `git checkout -b 브랜치이름` |
| diff 확인 안 함 | AI가 요청 외 파일도 수정하거나 버그 코드를 넣음 | `git diff` 또는 `git status`로 반드시 검토 |
| 코드 실행 테스트 안 함 | 문법은 맞지만 실행 시 오류 발생 | `python3 파일명.py`로 직접 실행 확인 |
| commit 메시지 불명확 | 나중에 "왜 이걸 바꿨지?" 모름 | Conventional Commits 형식 사용 |
| push 전 원격 저장소 설정 안 됨 | `error: remote origin does not exist` | `git remote add origin URL` 후 다시 push |
| `git push origin 브랜치명` 없이 push | `error: The current branch has no upstream branch` | 첫 push 시 `-u origin 브랜치명` 명시 |
| PR 설명 없이 만들기 | 리뷰어가 "이게 뭔가?" 모름 | 변경 내용, 테스트 방법, Closes #번호 포함 |

---

## 확인 체크리스트

- [ ] `claude` 명령으로 Claude Code를 실행할 수 있다
- [ ] 작업 전 `git checkout -b 브랜치이름`으로 새 branch를 만들었다
- [ ] Claude Code에게 구체적인 프롬프트로 기능을 요청했다
- [ ] Claude Code 수정 후 `git diff`로 변경 내용을 직접 눈으로 확인했다
- [ ] 코드를 실제로 실행해서 동작을 테스트했다
- [ ] `feat:`, `fix:` 형식의 Conventional Commits 메시지로 commit했다
- [ ] `git push origin 브랜치이름`으로 push했다
- [ ] GitHub에서 PR을 만들고 Files changed 탭을 확인했다
- [ ] PR 설명에 변경 내용과 `Closes #번호`를 포함했다

---

## 한 번 더 생각해 보기

1. Claude Code가 코드를 수정했을 때 `git diff`를 확인하지 않으면 어떤 위험이 있을까?
2. `main` branch에서 직접 작업하는 것과 새 branch에서 작업하는 것의 실질적인 차이는 무엇일까?
3. Conventional Commits 형식을 사용하면 어떤 실용적인 장점이 있을까?
4. AI가 만든 코드도 PR과 리뷰 과정을 거쳐야 하는 이유는 무엇일까?

---

## 다음 장

다음 장에서는 GitHub Actions를 사용해서 PR이 올라올 때 자동으로 테스트가 실행되는 CI 워크플로우를 설정하는 방법을 배운다.

---

## 참고 자료

- Conventional Commits 규격 — https://www.conventionalcommits.org/ko/v1.0.0/
- GitHub CLI (gh) 설치 — https://cli.github.com/
- GitHub Docs: Creating a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request
- git diff 사용법 — https://git-scm.com/docs/git-diff
