## 이 장에서 배우는 것

- GitHub 웹 화면에서 Issue를 직접 만드는 방법
- 브랜치를 선택하고 Pull Request(PR)를 여는 순서
- Issue와 PR이 서로 어떻게 연결되는지
- 화면을 보지 않아도 각 단계를 머릿속으로 그릴 수 있는 흐름

---

## 먼저 쉬운 설명

코드를 혼자 짤 때는 그냥 저장하고 끝이지만, 팀에서 함께 일할 때는 **"이 작업을 왜 했는지"**와 **"코드가 어떻게 바뀌었는지"**를 모두에게 알려야 합니다.

GitHub는 그 두 가지를 위한 도구를 제공합니다.

- **Issue** — "이런 문제가 있어요" 또는 "이 기능을 만들고 싶어요"라고 제안하는 공간
- **Pull Request(PR)** — "이렇게 코드를 바꿨어요, 확인해 주세요"라고 요청하는 공간

이 두 가지를 웹 화면에서 직접 만들어 보면, 나중에 터미널에서 git 명령어를 사용할 때도 전체 흐름이 훨씬 잘 이해됩니다.

---

## 1. Issue란 무엇인가

Issue는 **할 일 메모**입니다. 버그 신고, 기능 요청, 질문 등 프로젝트와 관련된 모든 대화가 여기서 시작됩니다.

### 실제 예시: 버그 신고 Issue

```
제목: 로그인 버튼 클릭 시 오류 페이지로 이동함

본문:
## 문제 설명
메인 화면에서 "로그인" 버튼을 누르면 404 오류 페이지로 이동합니다.

## 재현 방법
1. https://example.com 접속
2. 우측 상단 "로그인" 버튼 클릭
3. 404 페이지 표시됨

## 예상 동작
로그인 폼 페이지로 이동해야 합니다.

## 환경
- 브라우저: Chrome 124
- OS: macOS 14
```

Issue를 이렇게 구체적으로 작성하면, 다른 사람이 문제를 빠르게 이해하고 해결할 수 있습니다.

### Issue 번호

Issue를 만들면 자동으로 `#1`, `#2`처럼 번호가 붙습니다. 나중에 PR을 만들 때 이 번호를 활용합니다.

```
PR 본문에 "Closes #3" 라고 쓰면,
PR이 병합될 때 Issue #3이 자동으로 닫힙니다.
```

---

## 2. Issue 만들기: 화면 흐름 따라가기

웹 브라우저에서 GitHub 저장소를 열었다고 상상하면서 읽어 보세요.

```
[저장소 페이지 상단 탭]
Code | Issues | Pull requests | Actions | Projects | Wiki | Security | Insights | Settings
              ↑ 여기를 클릭
```

**단계별 흐름:**

```
1. 저장소 메인 페이지 접속
   예: github.com/내아이디/my-project

2. 상단 탭에서 "Issues" 클릭
   → Issue 목록 페이지로 이동

3. 우측의 초록색 "New issue" 버튼 클릭
   → Issue 작성 폼이 열림

4. 제목(Title) 입력
   예: "README 파일에 설치 방법이 빠져 있음"

5. 본문(Leave a comment) 입력
   마크다운 형식으로 작성 가능

6. 우측 사이드바 설정 (선택)
   - Assignees: 담당자 지정
   - Labels: 분류 태그 (bug, enhancement 등)
   - Milestone: 마일스톤 연결

7. "Submit new issue" 버튼 클릭
   → Issue 생성 완료, #번호 자동 부여
```

---

## 3. 브랜치란 무엇인가 (PR을 이해하기 위한 준비)

Pull Request를 만들려면 브랜치 개념을 먼저 알아야 합니다.

```
main 브랜치 (배포용, 안전한 코드만)
│
├── feature/login-fix 브랜치 (새 기능 개발 중)
│
└── hotfix/typo-readme 브랜치 (오타 수정 중)
```

브랜치는 **원본을 건드리지 않고 작업하는 복사본**이라고 생각하면 됩니다.

터미널에서 브랜치를 만들고 GitHub에 올리는 명령어는 이렇습니다:

```bash
# 새 브랜치 만들기
git checkout -b feature/add-readme-install

# 파일 수정 후 저장
# README.md 에 설치 방법 추가

# 변경사항 스테이징 및 커밋
git add README.md
git commit -m "docs: README에 설치 방법 섹션 추가"

# GitHub에 브랜치 올리기
git push origin feature/add-readme-install
```

이 명령어를 실행하면 GitHub 웹 화면에 새 브랜치가 생기고, PR을 만들 수 있는 상태가 됩니다.

---

## 4. Pull Request 만들기: 화면 흐름 따라가기

브랜치를 push한 직후 GitHub 저장소 페이지에 가면 노란색 배너가 뜹니다:

```
┌─────────────────────────────────────────────────────────────┐
│  feature/add-readme-install had recent pushes 1 minute ago  │
│                          [Compare & pull request]           │
└─────────────────────────────────────────────────────────────┘
```

**"Compare & pull request" 버튼을 클릭합니다.**

배너가 사라졌다면 직접 열 수도 있습니다:

```
1. 상단 탭 "Pull requests" 클릭
2. 우측 초록색 "New pull request" 버튼 클릭
3. "compare:" 드롭다운에서 내 브랜치 선택
   예: feature/add-readme-install
4. "base:" 는 main으로 유지
5. "Create pull request" 버튼 클릭
```

**PR 작성 폼:**

```
제목: docs: README에 설치 방법 섹션 추가

본문:
## 변경 내용
- README.md에 `## 설치 방법` 섹션 추가
- pip 설치 명령어 및 가상환경 설정 방법 기재

## 관련 Issue
Closes #2

## 테스트
- [ ] README 렌더링 확인
- [ ] 링크 동작 확인
```

작성 후 **"Create pull request"** 를 누르면 PR이 열립니다.

---

## 따라 하기 실습

### 실습 1: 첫 번째 Issue 만들기

다음 시나리오로 Issue를 직접 만들어 보세요.

**상황:** `my-python-project` 저장소의 `requirements.txt` 파일에 `requests` 라이브러리가 누락되어 있습니다.

```
[따라 할 단계]

1. github.com/내아이디/my-python-project 접속
2. "Issues" 탭 클릭
3. "New issue" 클릭
4. 제목 입력:
   requirements.txt에 requests 라이브러리 누락

5. 본문 입력:
   ## 문제
   `pip install -r requirements.txt` 실행 후
   `import requests`를 하면 ModuleNotFoundError 발생
   
   ## 해결 방법 제안
   requirements.txt에 `requests==2.31.0` 추가 필요

6. Labels에서 "bug" 선택
7. "Submit new issue" 클릭
```

Issue 번호를 기억해 두세요. 다음 실습에서 씁니다.

---

### 실습 2: 브랜치 만들고 GitHub에 올리기

실습 1에서 만든 Issue를 실제로 고쳐 보겠습니다.

```bash
# 1. 저장소 클론 (처음이라면)
git clone https://github.com/내아이디/my-python-project.git
cd my-python-project

# 2. 수정용 브랜치 만들기
git checkout -b fix/add-requests-dependency

# 3. requirements.txt 수정
# 파일을 열어서 아래 한 줄 추가:
# requests==2.31.0

# 4. 커밋
git add requirements.txt
git commit -m "fix: requirements.txt에 requests 라이브러리 추가"

# 5. GitHub에 올리기
git push origin fix/add-requests-dependency
```

---

### 실습 3: Pull Request 열기

실습 2에서 올린 브랜치로 PR을 만듭니다.

```
1. GitHub 저장소 페이지로 이동
2. 노란 배너의 "Compare & pull request" 클릭
   (또는 Pull requests → New pull request)

3. base: main  ←  compare: fix/add-requests-dependency  확인

4. 제목 입력:
   fix: requirements.txt에 requests 라이브러리 추가

5. 본문 입력:
   ## 변경 내용
   - requests==2.31.0 를 requirements.txt에 추가
   
   ## 관련 Issue
   Closes #1    ← 실습 1에서 만든 Issue 번호로 바꾸세요

6. "Create pull request" 클릭
```

PR이 열리면, Files changed 탭에서 `requirements.txt`의 변경 내용을 확인할 수 있습니다.

---

## 자주 하는 실수

| 실수 | 화면에 보이는 메시지 또는 증상 | 해결 방법 |
|------|-------------------------------|-----------|
| `git push` 없이 PR 만들려 함 | 브랜치가 드롭다운에 안 보임 | `git push origin 브랜치이름` 먼저 실행 |
| base와 compare 브랜치를 반대로 선택 | "There isn't anything to compare" 오류 | base=main, compare=내 작업 브랜치로 재설정 |
| 이미 병합된 브랜치로 PR 시도 | "This branch has no new commits" | 새 브랜치를 만들어서 다시 작업 |
| Issue 번호를 잘못 씀 | PR 본문에서 링크가 연결 안 됨 | Issues 탭에서 번호 확인 후 `Closes #번호` 형식으로 수정 |
| 제목 없이 "Submit new issue" 클릭 | "Title can't be blank" 오류 표시 | 제목 입력 후 다시 제출 |
| main 브랜치에서 직접 push | "Protected branch" 오류 (저장소 보호 설정 시) | 새 브랜치 만들고 PR 경로로 진행 |

---

## 확인 체크리스트

스스로 점검해 보세요. 모두 체크할 수 있으면 이 장을 완료한 것입니다.

- [ ] GitHub 저장소에서 "Issues" 탭을 찾아 클릭할 수 있다
- [ ] "New issue" 버튼을 눌러 제목과 본문을 작성하고 제출할 수 있다
- [ ] Issue에 자동으로 번호(`#1`, `#2` 등)가 붙는다는 것을 안다
- [ ] 브랜치를 만들고 `git push origin 브랜치이름`으로 GitHub에 올릴 수 있다
- [ ] "Compare & pull request" 버튼이 어디서 나타나는지 안다
- [ ] PR에서 base 브랜치와 compare 브랜치의 역할을 구분할 수 있다
- [ ] PR 본문에 `Closes #번호`를 써서 Issue와 연결할 수 있다
- [ ] PR의 "Files changed" 탭에서 변경 내용을 확인할 수 있다

---

## 한 번 더 생각해 보기

1. Issue를 먼저 만들지 않고 바로 PR을 열면 어떤 점이 불편할까요? 팀원 입장에서 생각해 보세요.

2. PR에서 base 브랜치를 `main` 대신 `develop`으로 설정하면 어떤 상황에서 유용할까요?

3. 같은 저장소에서 여러 사람이 동시에 다른 브랜치로 작업하고 있다면, PR을 병합하는 순서가 중요할까요? 왜 그럴까요?

---

## 다음 장

다음 장에서는 PR에서 코드 리뷰를 남기고, 충돌(conflict)이 생겼을 때 GitHub 웹 화면에서 직접 해결하는 방법을 배웁니다.