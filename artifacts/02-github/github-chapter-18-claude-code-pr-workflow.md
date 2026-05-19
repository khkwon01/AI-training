## 이 장에서 배우는 것

- Claude Code 터미널에서 새 브랜치를 만드는 방법
- Claude Code가 수정한 코드를 `git diff`로 직접 눈으로 확인하는 습관
- `commit → push → GitHub PR 생성` 전체 흐름을 Claude Code와 함께 완성하는 방법
- AI가 제안한 코드를 그냥 믿지 않고 검토하는 실전 태도

---

## 먼저 쉬운 설명

지금까지 Git 명령어를 배웠다면, 이번 장에서는 **AI 도구인 Claude Code를 터미널에서 함께 쓰면서** 실제 개발 흐름을 경험한다.

Claude Code는 터미널에서 실행하는 AI 코딩 도우미다. "이 파일에 함수를 추가해줘"라고 말하면 Claude Code가 직접 파일을 수정해준다. 하지만 **AI가 바꾼 내용을 반드시 내 눈으로 확인해야 한다.** `git diff`가 바로 그 도구다.

AI가 코드를 잘못 수정하거나, 내가 원하지 않은 것을 바꾸는 경우가 생각보다 많다. `git diff`로 확인하는 습관이 없으면 버그를 그대로 올려버릴 수 있다.

---

## 1. Claude Code 터미널에서 시작하기

Claude Code는 터미널에서 `claude` 명령어로 실행한다.

```bash
# 프로젝트 폴더로 이동
cd my-project

# Claude Code 시작
claude
```

실행하면 아래처럼 프롬프트가 나타난다.

```
Claude Code v1.x.x
> 
```

여기서 자연어로 작업을 요청할 수 있다.

```
> 이 프로젝트에 인사말을 출력하는 greet() 함수를 hello.py에 추가해줘
```

Claude Code가 파일을 수정한 후 어떤 파일을 바꿨는지 알려준다.

---

## 2. 새 브랜치 만들기

기능을 추가하기 전에 **항상 새 브랜치를 만든다.** `main` 브랜치에서 직접 작업하면 나중에 되돌리기 어렵다.

```bash
# Claude Code 안에서도 터미널 명령어를 직접 쓸 수 있다
# 또는 Claude Code 밖에서 먼저 브랜치를 만들어도 된다

git checkout -b feature/add-greet-function
```

브랜치 이름은 **작업 내용을 알 수 있게** 짓는 것이 좋다.

```
좋은 예: feature/add-greet-function
나쁜 예: branch1, test, abc
```

Claude Code에게 브랜치를 만들어 달라고 요청할 수도 있다.

```
> feature/add-greet-function 브랜치를 새로 만들어줘
```

---

## 3. AI가 바꾼 것을 반드시 diff로 확인한다

Claude Code가 파일을 수정하고 나면 **그냥 믿지 말고** 반드시 `git diff`로 확인한다.

```bash
git diff
```

출력 예시:

```diff
diff --git a/hello.py b/hello.py
index e69de29..b3c5e6a 100644
--- a/hello.py
+++ b/hello.py
@@ -0,0 +1,5 @@
+def greet(name):
+    """이름을 받아 인사말을 출력한다."""
+    print(f"안녕하세요, {name}님!")
+
+greet("세계")
```

`+` 기호가 붙은 줄이 **추가된 코드**다. `-` 기호가 붙은 줄은 **삭제된 코드**다.

확인할 포인트:
- 원하지 않는 파일이 수정되지 않았는가?
- 함수 이름, 변수 이름이 내가 원한 대로인가?
- 불필요한 코드(테스트 코드, 주석)가 들어가지 않았는가?

---

## 4. commit → push → PR 생성

확인이 끝났으면 커밋하고 GitHub에 올린다.

```bash
# 수정된 파일을 스테이징
git add hello.py

# 커밋 메시지 작성 (무엇을 왜 했는지 명확하게)
git commit -m "feat: greet() 함수 추가 - 이름을 받아 인사말 출력"

# 원격 저장소에 브랜치 올리기
git push origin feature/add-greet-function
```

push가 성공하면 터미널에 GitHub PR 링크가 바로 나타난다.

```
remote: Create a pull request for 'feature/add-greet-function' on GitHub by visiting:
remote:   https://github.com/내이름/my-project/pull/new/feature/add-greet-function
```

그 링크를 브라우저에서 열거나, `gh` CLI를 쓸 수도 있다.

```bash
# GitHub CLI로 PR 바로 생성
gh pr create --title "greet() 함수 추가" --body "이름을 받아 인사말을 출력하는 함수를 추가했습니다."
```

---

## 따라 하기 실습

### 실습 1: Claude Code로 파일 수정 후 diff 확인

1. 터미널에서 `claude`를 실행한다.
2. 다음과 같이 요청한다:
   ```
   > calculator.py 파일을 만들고 add(a, b) 함수를 추가해줘
   ```
3. Claude Code가 파일을 만든 후 확인한다:
   ```bash
   git diff
   ```
4. `+` 줄이 원하는 코드인지 눈으로 검토한다.

---

### 실습 2: 브랜치 만들고 커밋하기

1. 새 브랜치를 만든다:
   ```bash
   git checkout -b feature/add-calculator
   ```
2. Claude Code에게 `subtract(a, b)` 함수도 추가해 달라고 요청한다.
3. `git diff`로 확인한 후 커밋한다:
   ```bash
   git add calculator.py
   git commit -m "feat: 계산기 함수 add, subtract 추가"
   ```

---

### 실습 3: push하고 PR 만들기

1. GitHub에 브랜치를 올린다:
   ```bash
   git push origin feature/add-calculator
   ```
2. 터미널에 나타난 링크를 클릭하거나 `gh pr create`로 PR을 만든다.
3. GitHub에서 PR 페이지를 열어 **Files changed** 탭에서 diff를 다시 한 번 확인한다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 상황 | 해결 방법 |
|---|---|---|
| main 브랜치에서 직접 작업 | PR 없이 main에 push됨 | 작업 전 `git checkout -b 브랜치이름` 먼저 실행 |
| diff 확인 안 함 | AI가 원하지 않는 파일까지 수정 | `git diff` 또는 `git status`로 반드시 검토 |
| push 전 원격 저장소 설정 안 됨 | `error: remote origin does not exist` | `git remote add origin <URL>` 후 다시 push |
| 브랜치 이름 없이 push | `error: The current branch has no upstream branch` | `git push origin 브랜치이름`으로 명시 |
| 커밋 없이 push 시도 | `Everything up-to-date` | `git add` → `git commit` 후 push |

---

## 확인 체크리스트

- [ ] `claude` 명령어로 Claude Code를 터미널에서 실행할 수 있다
- [ ] 작업 전 `git checkout -b` 로 새 브랜치를 만들었다
- [ ] Claude Code가 파일을 수정한 후 `git diff`로 변경 내용을 직접 확인했다
- [ ] `git add` → `git commit` → `git push` 순서로 실행했다
- [ ] GitHub에서 PR 페이지의 **Files changed** 탭을 열어 최종 확인했다
- [ ] PR 제목과 설명에 무엇을 왜 했는지 적었다

---

## 한 번 더 생각해 보기

1. Claude Code가 코드를 수정했을 때 `git diff`를 확인하지 않으면 어떤 문제가 생길 수 있을까?
2. `main` 브랜치에서 직접 작업하는 것과 새 브랜치에서 작업하는 것의 차이는 무엇일까?
3. PR을 만들 때 팀원이 내 코드를 이해하려면 제목과 설명에 어떤 내용이 있어야 할까?

---

## 다음 장

다음 장에서는 GitHub Actions를 사용해서 PR이 올라올 때 자동으로 테스트가 실행되는 CI 워크플로우를 설정하는 방법을 배운다.