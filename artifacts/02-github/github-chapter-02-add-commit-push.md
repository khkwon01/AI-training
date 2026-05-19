# Chapter 02: add, commit, push

## 이 장에서 배우는 것

- `add`, `commit`, `push`가 각각 어떤 단계인지
- 파일이 `untracked`, `staged`, `committed` 상태로 바뀌는 흐름
- 초보자가 가장 자주 쓰는 `git status`, `git log`, `git diff`
- 작은 변경을 안전하게 GitHub에 올리는 실제 따라 하기 예제

---

## 먼저 쉬운 설명

파일을 수정했다고 해서 바로 GitHub에 올라가는 것은 아니다.
중간에 두 번 더 확인하는 단계가 있다.

| 단계 | 명령 | 쉬운 뜻 |
|------|------|---------|
| 1단계 | `git add` | "이 변경을 이번 기록에 넣을게" |
| 2단계 | `git commit` | "지금까지 고른 변경을 하나의 기록으로 저장할게" |
| 3단계 | `git push` | "내 컴퓨터의 기록을 GitHub에도 보낼게" |

택배에 비유하면:

- `add` = 상자에 넣을 물건 고르기
- `commit` = 상자를 닫고 라벨 붙이기
- `push` = 택배 보내기

즉, Git은 "무조건 바로 올리는 방식"이 아니라
"무엇을 올릴지 확인하고 기록한 뒤 보내는 방식"이다.

---

## 오늘 실습 시나리오

이번 장에서는 아주 작은 두 가지 변경만 한다.

1. `hello.py` 파일 만들기
2. `README.md`에 한 줄 추가하기

이렇게 하면 초보자가 아래 세 가지를 같이 배울 수 있다.

- 새 파일 추가
- 기존 파일 수정
- 어떤 파일을 commit에 넣을지 직접 선택

---

## 1. `git status`: 지금 상태 먼저 보기

clone한 `python-study` 폴더를 연 뒤, 아래처럼 `hello.py`를 만든다.

```python
print("Hello, GitHub")
```

그리고 `README.md` 맨 아래에 한 줄을 추가한다.

```text
이 저장소는 Git과 GitHub를 연습하기 위한 공간입니다.
```

그다음 터미널에서 실행한다.

```bash
git status
```

예상되는 화면:

```text
On branch main
Changes not staged for commit:
  modified:   README.md

Untracked files:
  hello.py
```

여기서 중요한 뜻은 두 가지다.

- `Untracked files`: 새 파일이라서 아직 Git이 추적하지 않는 상태
- `Changes not staged for commit`: 이미 알고 있는 파일이지만, 아직 이번 commit에 넣지 않은 상태

초보자는 작업할 때마다 `git status`를 먼저 보는 습관을 들이는 것이 좋다.

---

## 2. `git add`: 어떤 변경을 기록할지 고르기

두 파일을 모두 이번 기록에 넣고 싶다면:

```bash
git add hello.py README.md
```

다시 `git status`를 보면:

```text
On branch main
Changes to be committed:
  new file:   hello.py
  modified:   README.md
```

`Changes to be committed`는 "이제 commit 준비가 됐다"는 뜻이다.

### 왜 `git add`를 따로 할까

파일을 10개 고쳤더라도 이번 기록에 2개만 넣고 싶을 수 있다.
그래서 Git은 "변경"과 "기록 대상 선택"을 분리해 둔다.

### 초보자 팁

처음에는 `git add .`를 자주 쓰고 싶어진다.
하지만 학습 단계에서는 아래 방식이 더 좋다.

```bash
git add hello.py README.md
```

이유:

- 어떤 파일이 기록되는지 직접 눈으로 확인할 수 있다
- 실수로 관계없는 파일을 같이 올릴 가능성이 줄어든다

---

## 3. `git diff`: 무엇이 바뀌었는지 보기

commit 전에 "정말 이 내용이 맞는지" 보고 싶다면 `git diff`를 쓴다.

`README.md`를 add 하기 전에 보면:

```bash
git diff README.md
```

이미 add 한 뒤에는 staged 변경을 본다.

```bash
git diff --staged
```

초보자는 이 명령을 외울 필요는 없지만,
`status`만으로 불안할 때 "무슨 문장이 바뀌었는지" 직접 볼 수 있다는 점은 알아두는 편이 좋다.

---

## 4. `git commit`: 하나의 기록 만들기

이제 commit을 만든다.

```bash
git commit -m "hello.py 추가와 README 설명 보강"
```

예상되는 화면:

```text
[main 3a1b2c4] hello.py 추가와 README 설명 보강
 2 files changed, 2 insertions(+)
 create mode 100644 hello.py
```

다시 `git status`를 보면:

```text
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
nothing to commit, working tree clean
```

이 문장은 아래처럼 읽으면 된다.

- `ahead of 'origin/main' by 1 commit`
  GitHub보다 내 컴퓨터 기록이 1개 앞서 있다
- `working tree clean`
  아직 저장 안 한 변경이 남아 있지 않다

### 좋은 commit 메시지 예

| 나쁜 예 | 더 좋은 예 |
|--------|------------|
| `update` | `hello.py 파일 추가` |
| `fix` | `README 실행 설명 수정` |
| `asdf` | `hello.py 추가와 README 설명 보강` |

좋은 commit 메시지는 "무엇을 했는지"가 바로 보이는 문장이다.

---

## 5. `git push`: GitHub에 보내기

이제 내 컴퓨터의 commit을 GitHub로 보낸다.

```bash
git push origin main
```

처음 push할 때는 인증 창이 뜰 수 있다.
브라우저가 열리면 GitHub 계정으로 로그인하고 권한을 허용한다.

정상이라면 아래와 비슷한 화면이 나온다.

```text
To https://github.com/username/python-study.git
   1a2b3c4..3a1b2c4  main -> main
```

이제 GitHub 저장소 페이지를 새로고침하면 `hello.py`와 README 변경이 보인다.

### 왜 `push`를 마지막에 할까

이전 단계에서 commit까지 만들어 두면:

- 잘못된 변경을 한 번 더 확인할 수 있고
- commit 기록을 정리한 뒤 올릴 수 있고
- 인터넷이 잠깐 불안정해도 로컬 기록은 남는다

---

## 6. 언제 `main`에 바로 push하고, 언제 branch와 PR을 쓸까

처음 공부할 때는 둘 다 경험하는 편이 좋다.

### `main`에 바로 push해도 되는 경우

- 혼자 연습하는 아주 작은 저장소
- 실습 목적의 샘플 저장소
- Git 명령 흐름 자체를 익히는 첫 단계

### branch와 PR을 쓰는 것이 좋은 경우

- 검토를 받고 싶은 경우
- 작업 이유를 issue와 연결하고 싶은 경우
- 실수로 `main`을 크게 건드리고 싶지 않은 경우
- 실제 협업 흐름을 연습하는 경우

초보자 기준으로 아주 단순하게 정리하면:

```text
혼자 연습하는 첫 실습 = main 직접 push 가능
검토와 협업을 배우는 실습 = branch + pull request 사용
```

다음 GitHub browser walkthrough 장에서는
issue -> branch -> pull request 흐름을 이어서 연습한다.

---

## 7. `git log`: 기록 확인하기

최근 commit 목록을 짧게 보려면:

```bash
git log --oneline
```

예:

```text
3a1b2c4 hello.py 추가와 README 설명 보강
1a2b3c4 Initial commit
```

이 목록을 보면 "지금까지 어떤 단위로 작업했는지"를 빠르게 확인할 수 있다.

---

## 8. 전체 흐름 한눈에 보기

```text
파일 수정
  ↓
git status
  ↓
git add hello.py README.md
  ↓
git status
  ↓
git commit -m "hello.py 추가와 README 설명 보강"
  ↓
git push origin main
  ↓
GitHub 웹 화면에서 확인
```

---

## 9. 따라 하기 실습

### 실습 1. 새 파일과 기존 파일을 같이 올리기

1. `hello.py` 파일 만들기

```python
print("Hello, GitHub")
```

2. `README.md`에 한 줄 추가하기
3. 현재 상태 확인하기

```bash
git status
```

4. 두 파일만 add 하기

```bash
git add hello.py README.md
```

5. 다시 상태 확인하기

```bash
git status
```

6. commit 만들기

```bash
git commit -m "hello.py 추가와 README 설명 보강"
```

7. GitHub로 보내기

```bash
git push origin main
```

8. 브라우저에서 저장소를 새로고침하고 두 변경이 보이는지 확인하기

### 실습 2. 한 줄만 수정해서 다시 올리기

`hello.py`를 아래처럼 수정한다.

```python
print("Hello, GitHub")
print("두 번째 실행입니다.")
```

그다음 아래 순서로 다시 진행한다.

```bash
git status
git add hello.py
git commit -m "hello.py 출력 한 줄 추가"
git push origin main
```

---

## 10. 자주 막히는 지점

| 상황 | 보이는 메시지 | 먼저 할 일 |
|------|---------------|------------|
| add를 안 한 상태 | `nothing added to commit` | `git status`로 파일 상태 확인 후 `git add` |
| 잘못된 폴더에서 작업 | `not a git repository` | `pwd`와 `ls`로 현재 위치 확인 후 저장소 폴더로 이동 |
| push 인증 실패 | `Authentication failed` 또는 브라우저 인증 반복 | GitHub 로그인 상태와 VS Code/Git 인증 창 다시 확인 |
| 올릴 것이 없음 | `working tree clean` | 이미 commit했는지, 저장하지 않은 파일은 없는지 확인 |
| 다른 브랜치에 있음 | `On branch feature/...` | 지금 `main`에 올릴지, 브랜치를 유지할지 먼저 확인 |
| 인증 창이 안 뜸 | `403`, `rejected`, 권한 관련 메시지 | 브라우저 GitHub 로그인 상태와 저장소 권한부터 확인 |

---

## 11. 스스로 점검하기

아래 질문에 답할 수 있으면 이 장의 핵심을 이해한 것이다.

1. `git add`와 `git commit`은 무엇이 다른가?
2. 왜 작업 중간에 `git status`를 자주 보는가?
3. `working tree clean`은 어떤 상태를 뜻하는가?
4. `push`까지 끝난 뒤 어디에서 결과를 확인하는가?

```python
print("Hello, GitHub")
print("첫 번째 수정!")
```

저장 후 다시 전체 흐름을 실행한다.

```bash
git status
git add hello.py
git commit -m "hello.py에 두 번째 출력 추가"
git push
```

GitHub에서 파일 내용이 업데이트됐는지 확인한다.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| `add` 없이 `commit` 시도 | `nothing to commit` | 먼저 `git add <파일>` 실행 |
| commit 메시지 빈칸 | `Aborting commit due to empty commit message` | `-m "설명"` 으로 메시지 작성 |
| push 시 로그인 요청 | 팝업 또는 터미널에서 인증 요청 | GitHub 계정으로 로그인 |
| push 시 `rejected` 오류 | `Updates were rejected because the remote contains work...` | 먼저 `git pull` 실행 후 다시 push |
| 어떤 파일이 add 됐는지 모름 | - | `git status`로 `Changes to be committed` 확인 |

---

## 확인 체크리스트

- [ ] `git add`, `git commit`, `git push` 순서를 설명할 수 있는가
- [ ] `git status`로 각 단계를 확인할 수 있는가
- [ ] commit 메시지를 이해하기 쉽게 쓸 수 있는가
- [ ] `git push` 후 GitHub 저장소에서 변경 내용을 확인할 수 있는가
- [ ] `git log --oneline`으로 commit 기록을 볼 수 있는가

---

## 한 번 더 생각해 보기

1. `git add`와 `git commit`이 왜 두 단계로 나뉘어 있을까? 하나로 합치면 안 될까?
2. commit 메시지를 대충 쓰면 나중에 어떤 불편이 생길까?
3. `git push` 없이 `git commit`만 해두면 어떻게 될까?

---

## 다음 장

다음 장에서는 **branch**를 배운다. branch는 같은 저장소에서 독립적인 작업 공간을 만드는 방법이다. 기능을 추가하거나 수정할 때 원본 코드를 건드리지 않고 작업할 수 있다.

---

## 참고 자료

- Git 기본 명령어 — https://git-scm.com/docs
- GitHub Docs: Committing changes — https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository
