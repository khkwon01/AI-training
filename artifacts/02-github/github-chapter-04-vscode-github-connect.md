# Chapter 04: VS Code와 GitHub 연결하기

## 이 장에서 배우는 것

- VS Code에서 GitHub 계정을 연결하는 방법
- Source Control 패널에서 변경 내용을 확인하고 commit하는 방법
- 터미널 없이 VS Code UI만으로 add → commit → push 하는 방법
- GitHub Pull Request 확장으로 PR을 VS Code 안에서 관리하는 방법
- 터미널 방식과 UI 방식을 언제 각각 쓰면 좋은지

---

## 먼저 쉬운 설명

앞 장에서는 터미널에서 `git add`, `git commit`, `git push` 명령을 직접 입력했다.

VS Code에는 이 과정을 마우스 클릭으로 할 수 있는 **Source Control** 패널이 내장되어 있다.

처음에는 터미널 방식으로 흐름을 익히고, 이후에는 VS Code UI를 함께 활용하면 더 빠르게 작업할 수 있다.

---

## 1. VS Code에서 GitHub 계정 연결

### 연결 방법

1. VS Code 왼쪽 Activity Bar 맨 아래 **계정 아이콘** (사람 모양) 클릭
2. **Sign in with GitHub** 선택
3. 브라우저가 열리면 GitHub 계정으로 로그인
4. VS Code 접근 허용 → **Authorize Visual-Studio-Code** 클릭
5. VS Code로 돌아오면 계정 이름이 표시됨

### 연결 확인

연결이 됐으면 계정 아이콘 위에 마우스를 올렸을 때 GitHub 사용자 이름이 보인다.

### 왜 계정을 연결해야 할까

- `push` 할 때 매번 로그인하지 않아도 된다
- GitHub Pull Request 확장을 사용할 수 있다
- VS Code의 Copilot 등 AI 도구와도 연동된다

---

## 2. Source Control 패널 이해하기

VS Code 왼쪽 Activity Bar에서 **가지 모양 아이콘** (Source Control)을 클릭하면 Git 상태를 볼 수 있다.

### 패널 구성

```
SOURCE CONTROL
├── Changes          ← git add 전 상태 (Untracked / Modified)
├── Staged Changes   ← git add 후 상태 (commit 대기)
└── 입력창           ← commit 메시지 입력
```

각 파일 옆의 아이콘:
- `U` (Untracked) — 새로 만든 파일, Git이 아직 모름
- `M` (Modified) — 기존 파일을 수정함
- `D` (Deleted) — 파일을 삭제함

---

## 3. UI로 add → commit → push 하기

### 1단계. 파일 변경사항 확인

파일을 수정하면 Source Control 아이콘에 숫자 배지가 생긴다.

Source Control 패널을 열면 변경된 파일 목록이 보인다.

### 2단계. Stage (add)

파일 이름 오른쪽의 **+** 버튼을 클릭하면 Staged Changes로 이동한다.

전체 파일을 한 번에 stage하려면 **Changes** 줄 오른쪽의 **+** 버튼을 클릭한다.

### 3단계. Commit 메시지 입력

패널 상단 입력창에 commit 메시지를 입력한다.

```
hello.py 추가
```

### 4단계. Commit

입력창 위 **✓ Commit** 버튼 클릭

또는 `Cmd/Ctrl + Enter`

### 5단계. Push

Commit 후 하단 상태 표시줄에 아래처럼 표시된다.

```
↑1  ←  GitHub보다 1개 앞서 있음
```

상태 표시줄의 **↑1** 클릭하거나, Source Control 패널 상단 **...** → **Push** 선택

---

## 4. GitHub Pull Request 확장 설치

PR을 VS Code 안에서 만들고 검토할 수 있는 확장이다.

1. Extensions에서 `GitHub Pull Requests` 검색
2. **GitHub Pull Requests** (제작자: GitHub) 선택 → **Install**
3. 설치 후 왼쪽 Activity Bar에 GitHub 아이콘 추가됨

### PR 만들기

1. branch를 만들고 작업 후 push
2. Activity Bar의 **GitHub 아이콘** 클릭
3. **Create Pull Request** 클릭
4. 제목과 설명 입력 → **Create**

브라우저를 열지 않고 VS Code 안에서 PR을 만들 수 있다.

---

## 5. 터미널 방식 vs UI 방식 비교

| 상황 | 추천 방식 |
|------|----------|
| Git을 처음 배울 때 | 터미널 — 명령어와 흐름을 직접 익힘 |
| 어떤 파일이 바뀌었는지 시각적으로 확인 | UI — diff 뷰가 훨씬 읽기 편함 |
| 빠르게 commit하고 push | UI — 클릭 몇 번으로 완료 |
| 복잡한 Git 작업 (rebase, cherry-pick 등) | 터미널 — UI보다 유연함 |
| 오류 메시지를 자세히 보고 싶을 때 | 터미널 — 전체 출력 확인 가능 |

---

## 6. 따라 하기 실습

### 실습 1. VS Code에서 GitHub 계정 연결

1. 계정 아이콘 → **Sign in with GitHub**
2. 브라우저에서 인증 완료
3. VS Code로 돌아와서 계정 이름 확인

### 실습 2. UI로 파일 commit하고 push하기

1. `python-study` 폴더에서 `hello.py` 내용 수정
2. Source Control 패널 열기
3. 파일 옆 **+** 클릭 (Stage)
4. 메시지 입력: `hello.py 수정`
5. **✓ Commit** 클릭
6. 상태 표시줄 **↑1** 클릭하여 Push
7. GitHub 웹에서 변경사항 확인

### 실습 3. diff 뷰로 변경 내용 확인하기

Source Control 패널에서 파일 이름을 클릭하면 좌우로 나뉜 diff 뷰가 열린다.

- 왼쪽: 변경 전
- 오른쪽: 변경 후 (초록: 추가, 빨강: 삭제)

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| GitHub 계정 미연결 | push 시 매번 인증 요청 | 계정 아이콘 → Sign in with GitHub |
| Stage 없이 Commit 시도 | "There are no staged changes" | 파일 옆 **+** 클릭 후 Commit |
| Push 버튼을 못 찾음 | Commit 후 GitHub에 반영 안 됨 | 상태 표시줄 **↑숫자** 클릭 또는 `...` → Push |
| Source Control 패널에 파일이 없음 | 수정했는데 변경 없다고 표시 | 파일을 저장했는지 확인 (`Cmd/Ctrl + S`) |

---

## 확인 체크리스트

- [ ] VS Code에 GitHub 계정이 연결되어 있는가
- [ ] Source Control 패널에서 변경된 파일을 볼 수 있는가
- [ ] UI로 Stage → Commit → Push를 할 수 있는가
- [ ] diff 뷰에서 변경 내용을 확인할 수 있는가
- [ ] GitHub 웹에서 push된 내용을 확인할 수 있는가

---

## 한 번 더 생각해 보기

1. Source Control 패널의 `U`와 `M` 아이콘은 각각 무슨 뜻인가?
2. Commit 후 Push를 바로 하지 않으면 어떤 상태가 되는가?
3. diff 뷰를 보면 어떤 정보를 얻을 수 있는가?

---

## 다음 장

다음 장에서는 branch를 만들고 Pull Request를 여는 흐름을 배운다. branch를 쓰면 메인 코드를 건드리지 않고 새 기능을 안전하게 작업할 수 있다.

---

## 참고 자료

- VS Code Source Control — https://code.visualstudio.com/docs/sourcecontrol/overview
- GitHub Pull Requests 확장 — https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github
- VS Code GitHub 인증 — https://code.visualstudio.com/docs/sourcecontrol/github
