# Chapter 04: VS Code와 GitHub 연결하기

## 이 장에서 배우는 것

- VS Code에서 GitHub 계정을 연결하는 단계별 방법 (Sign in 버튼 위치, 브라우저 인증 흐름)
- Source Control 패널의 구성 요소와 각 아이콘의 의미
- 터미널 없이 VS Code UI만으로 add → commit → push 하는 방법
- GitHub 저장소를 clone해서 VS Code에서 여는 방법
- 자주 발생하는 인증 문제 (인증 만료, SSH vs HTTPS) 해결법
- GitHub Pull Request 확장으로 PR을 VS Code 안에서 관리하는 방법

---

## 왜 VS Code와 GitHub를 연결해야 할까

터미널에서 `git add`, `git commit`, `git push` 명령을 입력하는 것은 정확하지만, 익숙해지기 전까지는 어렵고 실수하기 쉽다.

VS Code는 이 모든 과정을 **마우스 클릭** 몇 번으로 처리할 수 있는 Source Control 패널을 내장하고 있다. 더 중요한 것은, 파일 옆에 빨간색과 초록색으로 변경된 내용을 시각적으로 보여주기 때문에 "내가 무엇을 바꿨는지" 한눈에 파악할 수 있다는 점이다.

GitHub 계정을 VS Code에 연결하면:
- `push`할 때 매번 아이디와 비밀번호를 입력하지 않아도 된다
- VS Code 안에서 PR을 만들고 리뷰할 수 있다
- GitHub Copilot 같은 AI 도구와도 자동으로 연동된다
- 인증 토큰을 직접 관리하지 않아도 VS Code가 알아서 처리해준다

터미널 방식과 UI 방식은 대립하는 게 아니다. 처음에는 터미널로 각 명령의 의미를 익히고, 이후에는 VS Code UI를 함께 활용하면 훨씬 빠르게 작업할 수 있다.

---

## 1. VS Code에서 GitHub 계정 연결하기

### 1-1. Sign in 버튼 찾기

VS Code를 열면 왼쪽에 세로로 아이콘이 나열된 **Activity Bar**가 있다. 맨 아래쪽에 사람 모양 아이콘이 있다. 이것이 **계정 아이콘**이다.

```
Activity Bar (왼쪽 세로 막대)
┌─────────────────────────────┐
│  □  Explorer               │
│  🔍 Search                 │
│  ⑂  Source Control         │
│  ▷  Run and Debug          │
│  ⬡  Extensions             │
│                             │
│  👤 계정 아이콘 ← 여기       │
│  ⚙  설정                   │
└─────────────────────────────┘
```

계정 아이콘을 클릭하면 메뉴가 나온다. 여기서 **Sign in with GitHub**를 클릭한다.

> 만약 "Sign in with GitHub"가 보이지 않으면 이미 연결된 것일 수도 있다. 계정 아이콘 위에 마우스를 올려서 GitHub 사용자 이름이 표시되는지 확인한다.

### 1-2. 브라우저 인증 흐름

"Sign in with GitHub"를 클릭하면 VS Code가 브라우저를 자동으로 연다.

브라우저에서 일어나는 일의 순서:

1. GitHub 로그인 페이지가 열린다 (이미 로그인되어 있으면 이 단계를 건너뛴다)
2. **Authorize Visual-Studio-Code** 버튼이 있는 페이지가 나온다
3. 이 버튼을 클릭한다
4. "VS Code를 열겠습니까?" 라는 팝업이 브라우저에 나타난다 — **열기** 클릭
5. VS Code로 돌아온다

VS Code로 돌아온 뒤 계정 아이콘 위에 마우스를 올리면 GitHub 사용자 이름이 보인다. 이것이 연결 확인 방법이다.

> 브라우저에서 인증 후 VS Code로 돌아오지 않는 경우: VS Code 하단의 팝업에서 "Open" 버튼이 있는지 확인한다. 없다면 VS Code를 다시 열어 같은 과정을 반복한다.

### 1-3. 연결이 제대로 됐는지 확인

연결이 성공했으면 두 가지 방법으로 확인할 수 있다.

**방법 1**: 계정 아이콘에 마우스를 올렸을 때 GitHub 사용자 이름이 표시된다.

**방법 2**: 터미널에서 아래 명령을 실행한다.
```bash
git config --global user.email
```
이 이메일이 GitHub 계정 이메일과 같으면 연결이 정상이다.

---

## 2. Source Control 패널 이해하기

VS Code 왼쪽 Activity Bar에서 **가지 모양 아이콘** (세 개의 점과 선이 연결된 모양)을 클릭하면 Source Control 패널이 열린다. 단축키는 `Cmd+Shift+G` (Mac) 또는 `Ctrl+Shift+G` (Windows)이다.

### 2-1. 패널 구성

```
SOURCE CONTROL
├──────────────────────────────────────────
│  [  commit 메시지 입력창...            ]  ← 여기에 메시지 입력
│  [✓ Commit ▼]                            ← 클릭하면 commit
├──────────────────────────────────────────
│  STAGED CHANGES (2)                      ← git add 완료된 파일
│    memo.py                          M +  │
│    utils.py                         U +  │
├──────────────────────────────────────────
│  CHANGES (3)                            ← git add 전 파일
│    README.md                        M + │
│    test.py                          U + │
│    old_file.txt                     D + │
└──────────────────────────────────────────
```

### 2-2. 파일 옆 아이콘 의미

각 파일 이름 오른쪽에 알파벳 한 글자가 표시된다.

| 아이콘 | 영어 이름 | 의미 |
|--------|----------|------|
| `U` | Untracked | 새로 만든 파일. Git이 아직 한 번도 추적한 적 없음 |
| `M` | Modified | 기존에 Git이 알던 파일인데 내용을 수정함 |
| `D` | Deleted | 파일을 삭제함 |
| `A` | Added | Staged Changes 상태에서 새로 추가된 파일 |
| `R` | Renamed | 파일 이름을 바꿈 |
| `C` | Conflicted | merge conflict 상태 (빨간색으로 표시됨) |

### 2-3. Source Control 배지

Source Control 아이콘 위에 숫자 동그라미(배지)가 생기면 변경된 파일이 있다는 뜻이다. 예를 들어 배지에 `3`이 표시되면 3개 파일이 변경됐다는 뜻이다.

파일을 저장하지 않으면 배지가 생기지 않는다. 파일을 수정한 후 `Cmd+S`(Mac) 또는 `Ctrl+S`(Windows)로 저장해야 변경이 감지된다.

---

## 3. UI로 add → commit → push 하기

터미널 없이 VS Code 화면만으로 Git 작업을 하는 전체 흐름이다.

### 3단계 흐름 요약

```
파일 수정 → Stage (+버튼) → 메시지 입력 → Commit → Push
```

### 1단계. 파일 수정 후 Source Control 열기

파일을 수정하고 저장하면 Source Control 아이콘에 숫자 배지가 생긴다. `Cmd+Shift+G`로 Source Control 패널을 연다.

**Changes** 섹션에 수정한 파일이 보인다.

### 2단계. Stage (add에 해당)

파일을 stage한다는 것은 "이 파일을 다음 commit에 포함시키겠다"고 표시하는 것이다. 터미널의 `git add`에 해당한다.

**파일 하나만 stage하기**: 파일 이름 오른쪽 끝에 마우스를 올리면 `+` 버튼이 나타난다. 이 `+` 버튼을 클릭한다.

**모든 파일 한꺼번에 stage하기**: "CHANGES" 글자 오른쪽에 `+` 버튼이 있다. 이것을 클릭하면 모든 변경 파일이 한 번에 stage된다.

stage된 파일은 **Staged Changes** 섹션으로 이동한다.

> stage를 취소하고 싶으면: Staged Changes의 파일 이름 옆에 마우스를 올리면 `-` 버튼(Unstage)이 나타난다. 클릭하면 다시 Changes로 내려온다.

### 3단계. Commit 메시지 입력

패널 상단의 입력창에 commit 메시지를 입력한다.

좋은 commit 메시지 예:
```
메모 검색 기능 추가
feat: 키워드로 메모를 검색하는 search_memo() 함수 추가
fix: 빈 메모 추가 시 발생하는 오류 수정
```

나쁜 commit 메시지 예:
```
수정
업데이트
asdf
```

### 4단계. Commit

입력창 바로 아래의 **✓ Commit** 버튼을 클릭한다. 또는 입력창에 메시지를 입력한 상태에서 `Cmd+Enter`(Mac) / `Ctrl+Enter`(Windows)를 누른다.

commit이 완료되면 Staged Changes 섹션이 비워진다.

> "There are no staged changes to commit" 메시지가 나온다면: Staged Changes가 비어있다는 뜻이다. 파일 옆 `+` 버튼을 먼저 눌러 stage해야 한다.

### 5단계. Push

commit 후 상태 표시줄(화면 맨 아래 파란 막대)을 보면 아래처럼 표시된다.

```
↑1  main  ← GitHub보다 1개 commit 앞서 있음
```

여기서 `↑1`을 클릭하면 push가 실행된다.

또는 Source Control 패널 상단의 `...` (점 세 개) 버튼 → **Push**를 선택해도 된다.

push가 성공하면 상태 표시줄에서 `↑` 표시가 사라진다.

---

## 4. GitHub 저장소를 clone해서 VS Code에서 열기

다른 사람의 저장소를 가져오거나, GitHub에서 처음 만든 저장소를 로컬에 가져올 때 clone을 사용한다.

### 4-1. 터미널에서 clone하고 VS Code로 열기

```bash
# GitHub에서 clone URL 복사 후 아래 명령 실행
git clone https://github.com/사용자이름/저장소이름.git

# 폴더로 이동
cd 저장소이름

# VS Code로 열기
code .
```

`code .` 명령이 실행되지 않는다면: VS Code에서 `Cmd+Shift+P` → "Shell Command: Install 'code' command in PATH" 를 실행한 후 터미널을 재시작한다.

### 4-2. VS Code 메뉴에서 clone하기

VS Code를 열었을 때 프로젝트가 없는 상태라면 시작 화면에 **Clone Git Repository** 링크가 있다. 이것을 클릭하면 URL을 입력할 수 있다.

또는 `Cmd+Shift+P` → "Git: Clone" → GitHub URL 입력 → 저장할 폴더 선택.

### 4-3. GitHub에서 clone URL 복사하는 방법

1. GitHub에서 저장소 페이지를 연다
2. 초록색 **Code** 버튼을 클릭한다
3. **HTTPS** 탭이 선택된 상태에서 URL 오른쪽의 복사 아이콘을 클릭한다

```
https://github.com/사용자이름/저장소이름.git
```

이 URL을 clone 명령이나 VS Code의 입력창에 붙여 넣는다.

---

## 5. 자주 겪는 인증 문제와 해결법

### 5-1. push할 때마다 아이디/비밀번호를 물어보는 경우

증상: `git push`를 실행할 때마다 GitHub 계정 정보를 다시 입력해야 한다.

원인: HTTPS 방식으로 clone했는데 자격증명 캐시가 없거나 만료됐다.

해결법 1 (VS Code 계정 연결): 위의 1번 섹션에서 설명한 대로 VS Code에 GitHub 계정을 연결한다. 한 번 연결하면 이후에는 자동으로 인증된다.

해결법 2 (자격증명 저장):
```bash
git config --global credential.helper store
```
이후 한 번만 입력하면 이후에는 자동으로 사용한다. 단, 비밀번호가 파일에 평문으로 저장되므로 개인 컴퓨터에서만 사용한다.

### 5-2. 인증이 만료된 경우

증상: 예전에는 잘 됐는데 어느 날부터 `push`할 때 "Authentication failed" 오류가 난다.

원인: GitHub 개인 액세스 토큰(Personal Access Token)이 만료됐다.

해결법:
1. VS Code 계정 아이콘 → 계정 이름 클릭 → **Sign Out**
2. 다시 **Sign in with GitHub** 클릭
3. 브라우저에서 다시 인증

또는 Mac의 키체인 앱에서 `github.com` 항목을 삭제한 후 다시 push를 시도하면 새로 인증 창이 뜬다.

### 5-3. HTTPS vs SSH — 무엇을 써야 할까

초보자에게는 **HTTPS** 방식이 훨씬 간단하다.

| | HTTPS | SSH |
|--|-------|-----|
| 설정 | VS Code에서 버튼 한 번 | SSH 키 생성 + GitHub에 등록 필요 |
| 사용 | 이후에는 자동 | 설정 후에는 비밀번호 입력 없음 |
| 추천 대상 | 처음 배우는 사람 | 터미널에 익숙한 사람 |

clone URL을 복사할 때 "HTTPS" 탭을 선택하면 HTTPS 방식이 된다. "SSH" 탭의 URL(`git@github.com:...`)은 SSH 방식이다. 아직 SSH 키를 설정하지 않았다면 HTTPS를 사용한다.

### 5-4. "Permission denied" 오류가 나는 경우

증상: `git push` 시 "Permission denied (publickey)" 오류

원인: SSH 방식으로 clone했는데 SSH 키가 설정되지 않았다.

빠른 해결법: 저장소를 HTTPS URL로 다시 연결한다.
```bash
# 현재 remote URL 확인
git remote -v

# HTTPS URL로 변경 (SSH → HTTPS)
git remote set-url origin https://github.com/사용자이름/저장소이름.git
```

---

## 6. diff 뷰로 변경 내용 확인하기

Source Control 패널에서 파일 이름을 클릭하면 **diff 뷰**가 열린다. 이것은 "내가 무엇을 바꿨는지"를 한눈에 보여주는 화면이다.

```
변경 전 (왼쪽)                    변경 후 (오른쪽)
─────────────────────────────────────────────────────
  def add_memo(text):              def add_memo(text):
    memos.append(text)               if not text.strip():
                                 +     return False
                                 +   memos.append(text.strip())
                                 +   return True
```

- **초록색 줄**: 새로 추가된 내용
- **빨간색 줄**: 삭제된 내용
- **흰색/회색 줄**: 변경 없는 맥락 줄

diff를 보면서 "내가 의도한 변경이 맞는가"를 확인하는 습관을 들이면 실수로 엉뚱한 코드를 commit하는 일을 예방할 수 있다.

---

## 7. GitHub Pull Request 확장 설치

VS Code에서 PR을 만들고 리뷰하려면 별도 확장이 필요하다.

### 설치 방법

1. Activity Bar의 **확장 아이콘** (⬡ 모양) 클릭
2. 검색창에 `GitHub Pull Requests` 입력
3. **GitHub Pull Requests** (제작자: GitHub) 선택 → **Install** 클릭
4. 설치 후 Activity Bar에 GitHub 아이콘(고양이 모양)이 추가된다

### PR 만들기

1. feature branch에서 작업하고 push
2. Activity Bar의 GitHub 아이콘 클릭
3. 상단에 "Create Pull Request" 버튼 클릭 (또는 초록색 배너가 뜨면 거기서 클릭)
4. 제목, 설명, base branch 확인 후 **Create** 클릭

브라우저를 열지 않고 VS Code 안에서 PR을 만들 수 있다.

---

## 8. 터미널 방식 vs UI 방식 비교

두 방식은 경쟁 관계가 아니다. 상황에 따라 편한 것을 고르면 된다.

| 상황 | 추천 방식 | 이유 |
|------|----------|------|
| Git을 처음 배울 때 | 터미널 | 명령어와 흐름을 직접 익혀야 이해가 생김 |
| 어떤 파일이 바뀌었는지 확인 | UI (diff 뷰) | 초록/빨간 색으로 직관적으로 표시됨 |
| 빠르게 commit하고 push | UI | 클릭 몇 번으로 완료 |
| 복잡한 Git 작업 (rebase, cherry-pick) | 터미널 | UI보다 세밀한 제어 가능 |
| 오류 메시지를 자세히 보고 싶을 때 | 터미널 | 전체 출력 내용 확인 가능 |
| 여러 파일을 선택적으로 stage | UI | 파일별로 +버튼 클릭으로 쉽게 선택 |

---

## 실습

### 실습 1. 따라 하기: VS Code에서 GitHub 계정 연결

목표: VS Code와 GitHub를 연결하고 연결 상태를 확인한다.

1. VS Code를 연다
2. Activity Bar 맨 아래 계정 아이콘(사람 모양)을 클릭한다
3. **Sign in with GitHub**를 클릭한다
4. 브라우저가 자동으로 열린다
5. GitHub에 로그인되어 있지 않으면 로그인한다
6. **Authorize Visual-Studio-Code** 버튼을 클릭한다
7. "VS Code를 열겠습니까?" 팝업에서 **열기**를 클릭한다
8. VS Code로 돌아와서 계정 아이콘 위에 마우스를 올린다
9. GitHub 사용자 이름이 표시되면 연결 성공이다

문제가 생기면: 브라우저가 열리지 않는 경우 VS Code를 완전히 종료했다가 다시 열어 시도한다.

---

### 실습 2. 따라 하기: 저장소 clone → VS Code에서 열기

목표: GitHub 저장소를 clone해서 VS Code에서 연다.

1. GitHub에서 자신의 `python-study` 저장소를 연다
2. 초록색 **Code** 버튼 클릭 → **HTTPS** 탭에서 URL 복사
3. VS Code에서 터미널을 열어 아래를 실행한다 (터미널 열기: `Ctrl+`` ` 또는 상단 메뉴 Terminal → New Terminal)
   ```bash
   cd ~/Desktop
   git clone 복사한URL
   cd python-study
   code .
   ```
4. VS Code가 새 창으로 열리면서 저장소 파일이 Explorer에 표시되는 것을 확인한다
5. Source Control 패널을 열어 변경사항이 없는(깨끗한) 상태인지 확인한다

---

### 실습 3. 직접 해보기: 파일 수정 → Source Control에서 커밋까지

목표: VS Code UI만으로 파일 수정부터 commit, push까지 전 과정을 완료한다.

1. VS Code에서 `python-study` 폴더를 연다
2. Explorer에서 `hello.py`를 클릭해서 연다
3. 파일 아무 곳에 한 줄 추가한다 (예: `# VS Code로 수정함` 주석 추가)
4. `Cmd+S` (Mac) 또는 `Ctrl+S` (Windows)로 저장한다
5. Source Control 아이콘에 배지 숫자(1)가 생겼는지 확인한다
6. Source Control 패널을 열어 `hello.py`가 Changes에 `M`으로 표시되는 것을 확인한다
7. `hello.py` 이름을 클릭해서 diff 뷰를 열어 변경 내용을 확인한다
8. `hello.py` 옆의 `+` 버튼을 클릭한다 → Staged Changes로 이동한다
9. 입력창에 커밋 메시지를 입력한다: `주석 추가: VS Code 수정 테스트`
10. **✓ Commit** 버튼을 클릭한다
11. 상태 표시줄에서 `↑1`을 클릭해서 push한다
12. GitHub 웹 브라우저에서 저장소를 새로고침해서 변경사항이 반영됐는지 확인한다

---

## 자주 하는 실수

| 상황 | 어떤 일이 생기나 | 해결 방법 |
|------|----------------|----------|
| GitHub 계정 미연결 | push 시 매번 아이디/비밀번호 입력창이 뜸 | 계정 아이콘 → Sign in with GitHub |
| 파일 저장 안 함 | Source Control에 변경사항이 표시 안 됨 | `Cmd/Ctrl + S`로 파일 저장 후 확인 |
| Stage(+버튼) 클릭 안 함 | "There are no staged changes to commit" 메시지 | 파일 옆 `+` 버튼 먼저 클릭 후 Commit |
| Commit 후 Push 안 함 | GitHub에 반영 안 됨, 상태 표시줄에 `↑1` 표시 | 상태 표시줄 `↑숫자` 클릭 또는 `...` → Push |
| HTTPS 대신 SSH URL 사용 | push 시 "Permission denied (publickey)" 오류 | `git remote set-url origin https://...` 로 변경 |
| Source Control에 파일 없음 | 저장소 폴더가 올바르게 열리지 않은 경우 | VS Code에서 해당 폴더를 Open Folder로 다시 열기 |

---

## 확인 체크리스트

- [ ] VS Code에 GitHub 계정이 연결되어 있고 계정 이름이 표시되는가
- [ ] GitHub 저장소를 clone해서 VS Code에서 열 수 있는가
- [ ] Source Control 패널에서 U, M, D 아이콘의 의미를 설명할 수 있는가
- [ ] `+` 버튼으로 파일을 stage할 수 있는가
- [ ] 커밋 메시지를 입력하고 Commit 버튼을 눌러 commit할 수 있는가
- [ ] 상태 표시줄의 `↑숫자`를 눌러 push할 수 있는가
- [ ] diff 뷰에서 초록색(추가)과 빨간색(삭제) 줄을 읽을 수 있는가
- [ ] GitHub 웹에서 push된 내용이 반영됐는지 확인할 수 있는가

---

## 한 번 더 생각해 보기

1. Source Control 패널의 `U`와 `M` 아이콘은 각각 무슨 뜻인가? 왜 구분이 필요할까?
2. Commit 후 Push를 바로 하지 않으면 어떤 상태가 되는가? 이 상태가 유용한 경우는 언제일까?
3. diff 뷰를 보면 어떤 정보를 얻을 수 있는가? 터미널에서 `git diff`를 실행한 것과 어떻게 다른가?
4. HTTPS와 SSH 중 어떤 방식을 선택했는가? 그 이유는 무엇인가?

---

## 다음 장

다음 장에서는 branch를 만들고 Pull Request를 여는 흐름을 배운다. branch를 쓰면 메인 코드를 건드리지 않고 새 기능을 안전하게 작업할 수 있다.

---

## 참고 자료

- VS Code Source Control — https://code.visualstudio.com/docs/sourcecontrol/overview
- GitHub Pull Requests 확장 — https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github
- VS Code GitHub 인증 — https://code.visualstudio.com/docs/sourcecontrol/github
- GitHub Personal Access Token 만들기 — https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
