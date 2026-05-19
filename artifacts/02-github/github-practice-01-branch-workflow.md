## 이 장에서 배우는 것

- 브랜치(branch)가 무엇인지, 왜 필요한지 이해한다
- `git branch`, `git checkout`, `git switch` 명령어를 사용할 수 있다
- 새 브랜치를 만들고, 작업하고, main 브랜치에 합치는(merge) 전체 흐름을 익힌다
- 브랜치 작업 중 자주 발생하는 오류를 스스로 해결할 수 있다

---

## 먼저 쉬운 설명

여러분이 중요한 문서를 수정해야 한다고 생각해 보세요. 원본을 바로 고치면 실수했을 때 되돌리기 어렵죠. 그래서 보통 "사본"을 만들어 작업하고, 완성되면 원본에 반영합니다.

Git의 **브랜치**가 바로 그 사본입니다.

- `main` 브랜치 = 완성된 안전한 코드
- 새 브랜치 = 내가 실험하고 작업하는 공간

브랜치 덕분에 팀원들이 서로 방해 없이 동시에 다른 기능을 만들 수 있고, 실수해도 원본(main)은 안전합니다. 이 흐름을 **브랜치 워크플로우(branch workflow)** 라고 부릅니다.

---

## 1. 브랜치 확인하고 만들기

현재 어떤 브랜치가 있는지 확인하고, 새 브랜치를 만드는 방법부터 시작합니다.

```bash
# 현재 브랜치 목록 확인
git branch

# 출력 예시:
# * main       ← 앞에 * 가 붙은 것이 현재 내 브랜치
#   develop

# 새 브랜치 만들기 (feature/login 이라는 이름)
git branch feature/login

# 다시 목록 확인
git branch

# 출력 예시:
#   feature/login
# * main
```

> **브랜치 이름 규칙 (팁)**
> - `feature/기능이름` — 새 기능 작업
> - `fix/버그이름` — 버그 수정
> - `docs/내용` — 문서 작업
> - 공백 대신 `-` 또는 `/` 사용

---

## 2. 브랜치 이동하기

브랜치를 만들었다고 해서 자동으로 이동하지 않습니다. 직접 이동해야 합니다.

```bash
# 방법 1: git switch (최신 권장 방식)
git switch feature/login

# 방법 2: git checkout (예전 방식, 지금도 자주 씁니다)
git checkout feature/login

# 브랜치 만들면서 동시에 이동 (가장 자주 쓰는 패턴)
git switch -c feature/signup
# 또는
git checkout -b feature/signup

# 이동 후 확인
git branch
# 출력:
#   feature/login
# * feature/signup    ← * 위치가 바뀌었습니다
#   main
```

```bash
# 현재 상태를 한눈에 보고 싶을 때
git status

# 출력 예시:
# On branch feature/signup
# nothing to commit, working tree clean
```

---

## 3. 브랜치에서 작업하고 커밋하기

브랜치를 이동한 뒤에는 평소처럼 파일을 수정하고 커밋합니다. 이 커밋은 현재 브랜치에만 쌓입니다.

```bash
# feature/login 브랜치로 이동
git switch feature/login

# 파일 생성 (예시)
echo "로그인 기능 작업 중" > login.py

# 상태 확인
git status
# On branch feature/login
# Untracked files:
#   login.py

# 스테이징 + 커밋
git add login.py
git commit -m "feat: 로그인 페이지 기본 구조 추가"

# 커밋 내역 확인
git log --oneline
# a1b2c3d (HEAD -> feature/login) feat: 로그인 페이지 기본 구조 추가
# e4f5g6h (main) 초기 프로젝트 설정
```

main 브랜치로 돌아가면 `login.py`가 보이지 않습니다. 브랜치가 분리되어 있기 때문입니다.

```bash
git switch main
ls
# login.py 가 없습니다 — 정상입니다!
```

---

## 4. 브랜치 합치기 (merge)

기능 개발이 완료되면 main 브랜치에 합칩니다.

```bash
# 1단계: main 브랜치로 이동
git switch main

# 2단계: feature/login 브랜치를 main에 합치기
git merge feature/login

# 출력 예시:
# Updating e4f5g6h..a1b2c3d
# Fast-forward
#  login.py | 1 +
#  1 file changed, 1 insertion(+)
#  create mode 100644 login.py

# 이제 main에서 login.py 확인 가능
ls
# login.py  README.md  ...
```

**Fast-forward merge**: 중간에 main이 변경되지 않았을 때 발생하는 깔끔한 합치기입니다. 브랜치의 커밋들이 그대로 main에 이어붙습니다.

---

## 5. 다 쓴 브랜치 정리하기

merge가 끝난 브랜치는 삭제해 브랜치 목록을 깔끔하게 유지합니다.

```bash
# 브랜치 삭제
git branch -d feature/login

# 출력:
# Deleted branch feature/login (was a1b2c3d).

# 삭제 확인
git branch
# * main
```

> merge하지 않은 브랜치를 강제 삭제할 때는 `-D`(대문자)를 씁니다.
> 작업 내용이 사라질 수 있으니 신중하게 사용하세요.

```bash
# 강제 삭제 (주의!)
git branch -D feature/미완성기능
```

---

## 따라 하기 실습

### 실습 1 — 로그인 기능 브랜치 만들기

```bash
# 1. 실습용 프로젝트 폴더 초기화
mkdir my-project && cd my-project
git init
echo "# My Project" > README.md
git add README.md
git commit -m "초기 커밋"

# 2. feature/login 브랜치 만들고 이동
git switch -c feature/login

# 3. 로그인 파일 만들기
echo "def login(): pass" > login.py
git add login.py
git commit -m "feat: login 함수 뼈대 추가"

# 4. 브랜치 로그 확인
git log --oneline
```

완료 확인: `git log --oneline`에서 커밋이 2개 보이고, `HEAD -> feature/login`이 표시되면 성공입니다.

---

### 실습 2 — 또 다른 브랜치와 병행 작업

```bash
# main으로 돌아가기
git switch main

# 회원가입 브랜치 따로 만들기
git switch -c feature/signup

# 회원가입 파일 추가
echo "def signup(): pass" > signup.py
git add signup.py
git commit -m "feat: signup 함수 뼈대 추가"

# 두 브랜치 모두 확인
git branch
# * feature/signup
#   feature/login
#   main
```

완료 확인: `git branch`에서 세 개의 브랜치가 보이면 성공입니다.

---

### 실습 3 — main에 두 브랜치 차례로 합치고 정리하기

```bash
# main으로 이동
git switch main

# feature/login 먼저 합치기
git merge feature/login
ls   # login.py 확인

# feature/signup 합치기
git merge feature/signup
ls   # login.py, signup.py 모두 확인

# 완료된 브랜치 삭제
git branch -d feature/login
git branch -d feature/signup

# 최종 상태 확인
git branch         # main 만 남아있어야 함
git log --oneline  # 커밋 3개가 보여야 함
```

완료 확인: `git branch`에 `main`만 남고, `ls`에서 `login.py`와 `signup.py`가 모두 보이면 완벽합니다.

---

## 자주 하는 실수

| 상황 | 오류 메시지 / 증상 | 해결 방법 |
|------|------------------|-----------|
| 브랜치 이름을 잘못 입력 | `error: pathspec 'feature/logn' did not match any file(s) known to git` | `git branch`로 정확한 이름 확인 후 다시 시도 |
| main에 있는데 merge 대상 브랜치에서 실행 | 의도와 다른 방향으로 merge됨 | merge 전 항상 `git branch`로 현재 위치 확인 |
| 커밋 안 한 변경사항이 있을 때 브랜치 이동 | `error: Your local changes to the following files would be overwritten by checkout` | 먼저 `git add` + `git commit` 또는 `git stash`로 임시 저장 |
| merge 안 된 브랜치를 `-d`로 삭제 시도 | `error: The branch 'feature/xxx' is not fully merged.` | 정말 필요 없으면 `git branch -D` 사용, 아니면 먼저 merge |
| 새 브랜치를 만들었는데 이동을 안 함 | `git branch feature/x` 후 여전히 main에서 커밋됨 | `git switch feature/x`로 이동 확인, 또는 처음부터 `git switch -c feature/x` 사용 |
| 로컬에 없는 원격 브랜치로 이동 시도 | `fatal: invalid reference: feature/remote-branch` | `git fetch` 후 `git switch feature/remote-branch` |

---

## 확인 체크리스트

- [ ] `git branch` 명령어로 현재 브랜치 목록을 확인할 수 있다
- [ ] `git switch -c 브랜치이름` 으로 브랜치를 만들면서 동시에 이동할 수 있다
- [ ] 브랜치를 이동할 때 `*` 표시가 바뀌는 것을 확인했다
- [ ] 브랜치에서 커밋한 내용이 main에는 보이지 않는 것을 직접 확인했다
- [ ] `git merge` 명령어로 작업 브랜치를 main에 합칠 수 있다
- [ ] merge 전에 반드시 main 브랜치로 이동해야 한다는 것을 이해했다
- [ ] `git branch -d` 로 완료된 브랜치를 삭제할 수 있다
- [ ] 커밋하지 않은 변경사항이 있을 때 브랜치 이동이 막히는 이유를 설명할 수 있다

---

## 한 번 더 생각해 보기

1. **브랜치 없이 main에서만 작업하면 어떤 문제가 생길까요?** 두 명의 팀원이 동시에 같은 파일을 수정한다면 어떻게 될지 상상해 보세요.

2. **`git switch main` 후 `git merge feature/login` 대신, `feature/login` 브랜치에 있는 상태에서 `git merge main`을 실행하면 어떻게 될까요?** 방향이 반대로 바뀌면 결과가 어떻게 다를지 생각해 보세요.

3. **브랜치 이름을 `feature/login`처럼 슬래시(`/`)로 구분해서 짓는 이유는 무엇일까요?** 팀 전체가 같은 규칙으로 이름을 지으면 어떤 이점이 있을지 떠올려 보세요.

---

## 다음 장

다음 장에서는 두 브랜치가 같은 파일을 동시에 수정했을 때 발생하는 **충돌(conflict)** 을 해결하는 방법을 배웁니다.