# ai-train GitHub Basics Chapter 01: Git과 GitHub 시작하기

## 이 장에서 배우는 것

- Git과 GitHub가 각각 무엇인지, 왜 다른지
- Git 설치와 초기 설정 방법
- GitHub 계정을 만들고 저장소를 만드는 방법
- `clone`으로 저장소를 내 컴퓨터에 가져오는 방법
- 초보자가 자주 막히는 지점과 해결 방법

---

## 먼저 쉬운 설명

코드를 작성하다 보면 이런 상황이 생긴다.

- "어제 잘 되던 코드인데 오늘 고치고 나서 안 된다. 어제로 되돌리고 싶다."
- "같이 작업하는 사람이 어떤 부분을 바꿨는지 알고 싶다."
- "내 컴퓨터가 고장나도 코드가 사라지지 않았으면 좋겠다."

**Git**은 이 문제를 해결하는 도구다. 파일의 변경 내용을 단계별로 기록해 두고, 언제든지 이전 상태로 돌아갈 수 있게 해준다.

**GitHub**는 Git으로 기록한 내용을 인터넷에 저장하고 공유하는 공간이다.

정리하면:
- Git = 내 컴퓨터에서 변경 내용을 추적하는 도구
- GitHub = Git 기록을 올려두는 인터넷 저장소 (github.com)

---

## 1. Git 설치 확인

먼저 Git이 설치되어 있는지 확인한다.

VS Code 터미널을 열고 입력한다.

```bash
git --version
```

이렇게 나오면 설치된 것이다.

```
git version 2.39.3
```

아무것도 나오지 않거나 `command not found`가 나오면 설치가 필요하다.

### Mac에서 Git 설치

터미널에서 아래를 실행한다.

```bash
xcode-select --install
```

팝업 창이 뜨면 "Install" 버튼을 클릭한다. 설치가 끝나면 `git --version`으로 다시 확인한다.

### Windows에서 Git 설치

1. https://git-scm.com 에 접속한다
2. **Download for Windows** 버튼 클릭
3. 설치 파일 실행, 기본 옵션 그대로 Next → Install
4. 설치 완료 후 VS Code를 재시작하고 터미널에서 `git --version` 확인

---

## 2. Git 초기 설정

Git을 처음 쓸 때 이름과 이메일을 한 번 설정해야 한다. commit 기록에 누가 작업했는지 표시되기 때문이다.

`--global` 옵션은 "이 컴퓨터의 모든 Git 프로젝트에 적용"한다는 뜻이다. 한 번만 설정하면 모든 프로젝트에서 쓰인다.

```bash
git config --global user.name "내 이름"
git config --global user.email "내이메일@example.com"
```

예:
```bash
git config --global user.name "Mina Kim"
git config --global user.email "mina@example.com"
```

설정이 됐는지 확인:
```bash
git config --global user.name
git config --global user.email
```

입력한 이름과 이메일이 출력되면 정상이다.

---

## 3. GitHub 계정과 저장소 만들기

### 계정 만들기

1. https://github.com 에 접속한다
2. 오른쪽 위 **Sign up** 클릭
3. 이메일, 비밀번호, 사용자 이름(username) 입력
4. 이메일 인증 완료

username은 나중에 저장소 주소에 들어간다. 예: `github.com/mina-kim/my-project`

### 저장소 만들기

GitHub에 로그인한 상태에서:

1. 오른쪽 위 **+** 버튼 클릭 → **New repository** 선택
2. **Repository name** 입력: 예) `python-study`
3. **Public** 또는 **Private** 선택
   - Public: 누구나 볼 수 있음
   - Private: 나만 볼 수 있음 (처음에는 Private 권장)
4. **Add a README file** 체크박스 체크
5. **Create repository** 클릭

저장소가 만들어지면 다음과 같은 URL이 생긴다.

```
https://github.com/username/python-study
```

---

## 4. 저장소 URL 복사하기

`clone`을 하려면 저장소 주소(URL)가 필요하다.

1. 저장소 페이지에서 초록색 **Code** 버튼 클릭
2. **HTTPS** 탭이 선택된 상태에서 URL 복사 (복사 아이콘 클릭)

주소 형태:
```
https://github.com/username/python-study.git
```

---

## 5. `clone`: 저장소를 내 컴퓨터로 가져오기

`clone`은 GitHub에 있는 저장소를 내 컴퓨터의 폴더로 복사해 오는 명령이다.

VS Code 터미널에서 저장소를 만들 위치로 이동한 뒤:

```bash
git clone https://github.com/username/python-study.git
```

실행하면 현재 폴더 안에 `python-study` 폴더가 생긴다.

```
python-study/
└── README.md
```

### clone 후 폴더 열기

VS Code에서 방금 생긴 폴더를 연다.

1. **File > Open Folder**
2. `python-study` 폴더 선택 → **Open**

이제 이 폴더 안에서 작업하면 Git이 변경 내용을 추적한다.

---

### 처음 인증은 언제 필요한가

`clone`은 공개 저장소라면 로그인 없이 되는 경우도 있다.
하지만 아래 순간에는 GitHub 인증이 필요할 수 있다.

- private 저장소를 clone할 때
- `push`로 변경을 올릴 때
- VS Code가 GitHub 계정 권한을 확인할 때

초보자는 `clone`은 되는데 `push`에서 막히는 경우가 많다.
이것은 흔한 상황이다.

### 인증이 잘 된 상태는 어떻게 보일까

- 브라우저에서 GitHub 로그인 상태가 유지된다
- VS Code 또는 Git 인증 창에서 계정 선택이 보인다
- `git push` 후 저장소 웹 화면에 파일이 올라온다

### 자주 막히는 인증 문제

| 상황 | 보이는 증상 | 먼저 확인할 것 |
|------|-------------|----------------|
| 로그인 창이 안 뜸 | `push`가 바로 실패하거나 반복 요청 | 브라우저 로그인 상태, VS Code 재실행 |
| 계정을 잘못 고름 | 다른 저장소는 되는데 이 저장소만 실패 | 현재 로그인한 GitHub 계정 |
| 반복 인증 | 매번 비밀번호나 로그인 요청 | VS Code GitHub 로그인 상태 |
| 권한 없음 | `403` 또는 `permission denied` | 저장소 권한과 로그인 계정 |

---

## 6. 따라 하기 실습

### 실습 1. Git 설정 확인하기

터미널에서 순서대로 실행한다.

```bash
git --version
git config --global user.name
git config --global user.email
```

세 가지 모두 값이 나오면 준비 완료다.

### 실습 2. GitHub 저장소 만들고 clone하기

1. GitHub에서 `python-study` 저장소를 만든다 (README 포함)
2. 저장소의 HTTPS URL을 복사한다
3. VS Code 터미널에서 clone한다

```bash
git clone https://github.com/username/python-study.git
```

4. 생성된 폴더를 VS Code에서 연다
5. 폴더 안에 `README.md` 파일이 있는지 확인한다

### 실습 3. clone한 저장소 상태 보기

폴더 안에서 터미널을 열고:

```bash
git status
```

이렇게 나오면 정상이다.

```
On branch main
nothing to commit, working tree clean
```

---

## 자주 하는 실수

| 상황 | 증상 / 오류 메시지 | 해결 방법 |
|------|-----------------|----------|
| Git이 설치 안 됨 | `command not found: git` | Git 설치 후 VS Code 재시작 |
| user.name / user.email 미설정 | commit 시 `Please tell me who you are` 오류 | `git config --global user.name "이름"` 실행 |
| HTTPS URL 대신 SSH URL 사용 | `git@github.com:...` 형태 → 인증 오류 발생 가능 | Code 버튼 클릭 후 **HTTPS** 탭 선택 후 복사 |
| clone 후 다른 폴더에서 작업 | `git status` 실행 시 `not a git repository` | `cd python-study` 로 clone한 폴더 안으로 이동 |
| `git status`에서 `HEAD detached` | 특정 commit을 직접 체크아웃한 상태 | `git checkout main` 으로 main 브랜치로 돌아오기 |

---

## 확인 체크리스트

- [ ] `git --version` 명령이 정상 출력되는가
- [ ] `git config --global user.name` 에 내 이름이 설정되어 있는가
- [ ] GitHub에 계정과 저장소를 만들 수 있는가
- [ ] 저장소의 HTTPS URL을 복사할 수 있는가
- [ ] `git clone <URL>` 로 저장소를 내 컴퓨터에 가져올 수 있는가
- [ ] clone 후 `git status` 로 상태를 확인할 수 있는가

---

## 한 번 더 생각해 보기

1. Git과 GitHub는 어떻게 다른가? 둘 중 하나만 있어도 될까?
2. `clone`을 하면 원본 저장소(GitHub)의 내용이 삭제될까?
3. `git status`가 `nothing to commit, working tree clean`이라고 표시하는 것은 무슨 뜻인가?

---

## 다음 장

다음 장에서는 파일을 수정하고 그 변경 내용을 Git에 기록하는 방법을 배운다.  
`add`, `commit`, `push` 세 단계를 순서대로 실습한다.

---

## 참고 자료

- Git 공식 문서 — https://git-scm.com/doc
- GitHub Docs: Getting started — https://docs.github.com/en/get-started
- GitHub Docs: Create a repo — https://docs.github.com/en/get-started/quickstart/create-a-repo
