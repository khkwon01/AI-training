# ai-train GitHub 학습 지도

## 목표

Git과 GitHub를 이용해 코드를 버전 관리하고, AI와 협업하는 실무 워크플로우를 익힌다.

---

## 권장 학습 순서

### 0단계. 준비 확인

이 파트를 시작하기 전에 필요한 것:
- VS Code가 설치되어 있는가
- 터미널에서 기본 명령어(`ls`, `cd`, `pwd`)를 쓸 수 있는가
- GitHub 계정이 없다면 https://github.com 에서 가입

---

### 1단계. Git 기초 설치와 첫 저장소

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 1 | [Chapter 01: Git과 GitHub 시작](../02-github/ai-train-github-chapter-01-git-and-github.md) | Git 설치, `--global` 설정, GitHub 계정, clone |
| 2 | [Chapter 02: add, commit, push](../02-github/ai-train-github-chapter-02-add-commit-push.md) | 변경 기록하기, commit 메시지, push 흐름 |

**이 단계 마치면**: 파일을 수정하고 GitHub에 올릴 수 있다.

---

### 2단계. VS Code와 GitHub 연결

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 3 | [Chapter 04: VS Code와 GitHub 연결](../02-github/ai-train-github-chapter-04-vscode-github-connect.md) | Source Control UI, 계정 연결, 클릭으로 commit/push |

**이 단계 마치면**: 터미널 없이 VS Code에서 Git을 사용할 수 있다.

---

### 3단계. Branch와 협업 흐름

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 4 | [Chapter 05: branch와 Pull Request](../02-github/ai-train-github-chapter-05-branch-and-pr.md) | branch 생성, PR 만들기, merge |
| 5 | [Chapter 06: merge conflict 해결](../02-github/ai-train-github-chapter-06-merge-conflict.md) | conflict 표시 읽기, VS Code로 해결, 예방 습관 |
| 6 | [Chapter 07: issue 트래킹](../02-github/ai-train-github-chapter-07-issue-tracking.md) | issue 생성, branch 연결, `Closes #번호` 자동 close |

**이 단계 마치면**: 기능별로 branch를 나눠 작업하고 PR을 통해 합칠 수 있다.

---

### 4단계. AI 코드 리뷰 워크플로우

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 7 | [Chapter 03: AI review 워크플로우](../02-github/ai-train-github-chapter-03-ai-review-workflow.md) | PR, AI review, 협업 흐름 |
| 8 | [Chapter 08: PR 리뷰 가이드](../02-github/ai-train-github-chapter-08-pr-review-guide.md) | Files changed 읽기, 줄 코멘트, AI 코드 PR 체크리스트 |

**이 단계 마치면**: AI가 만든 코드를 PR로 올리고 리뷰받는 전체 흐름을 실습할 수 있다.

---

### 실습 자료

| 파일 | 내용 |
|------|------|
| [브라우저 따라하기](../02-github/ai-train-github-walkthrough-01-browser.md) | Issue → branch → PR 브라우저 실습 |
| [워크북](../02-github/ai-train-github-workbook-01.md) | GitHub 전체 실습 워크북 |
| [AI review 랩](../02-github/ai-train-github-lab-01-ai-review.md) | AI review 실습 랩 |
| [화면 참조 노트](../02-github/ai-train-github-cue-note-01-screen.md) | 주요 버튼 위치 빠른 참조 |

---

## 학습 팁

- **1~2단계는 꼭 순서대로**: commit/push를 모르면 branch도 의미가 없다
- **3단계부터는 반복이 핵심**: 작은 기능을 만들 때마다 issue → branch → PR 흐름을 반복해서 몸에 익힌다
- **혼자 연습할 때**: 저장소를 하나 만들고 branch를 여러 개 만들어서 merge 연습을 반복한다
- **conflict는 무섭지 않다**: 충돌은 두 사람이 같은 줄을 고쳤다는 뜻이고, VS Code가 시각적으로 도와준다

## 자주 막히는 지점

| 단계 | 막히는 부분 | 확인할 것 |
|------|-----------|----------|
| clone | `not found` 오류 | 터미널 위치가 올바른 폴더인지 `pwd` 확인 |
| push | 인증 요청 반복 | VS Code GitHub 계정 연결 상태 확인 |
| conflict | 어떤 코드를 남길지 모름 | 두 버전을 모두 읽고 의도에 맞는 쪽 선택 |
| PR merge | `Merge pull request` 버튼이 없음 | branch 보호 규칙 또는 리뷰 미완료 확인 |
