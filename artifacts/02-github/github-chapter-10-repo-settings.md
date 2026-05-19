## 이 장에서 배우는 것

- GitHub 저장소(repository)의 기본 설정 메뉴를 탐색할 수 있다
- 브랜치(branch)가 무엇인지, 왜 보호해야 하는지 설명할 수 있다
- `main` 브랜치에 직접 push를 막는 브랜치 보호 규칙을 설정할 수 있다
- Pull Request(PR)를 반드시 거치도록 워크플로를 구성할 수 있다
- 협업 중 실수로 중요한 코드가 덮어씌워지는 사고를 예방할 수 있다

---

## 먼저 쉬운 설명

공사 현장을 상상해 보세요. 완성된 건물(= `main` 브랜치)에는 아무나 들어가서 벽을 부수면 안 되겠죠? 그래서 "이 문은 허가받은 사람만 열 수 있어요"라는 규칙이 필요합니다.

GitHub에서도 마찬가지입니다. 팀원이 실수로 `main`에 바로 push하거나, 리뷰 없이 코드를 합치면 버그가 생기거나 동료의 작업이 사라질 수 있습니다. **브랜치 보호 규칙(Branch Protection Rule)**은 이런 사고를 막아 주는 안전장치입니다.

혼자 작업할 때는 크게 신경 쓰지 않아도 되지만, 팀 프로젝트나 취업 후 현업에서는 이 설정이 기본 중의 기본입니다. 지금 익혀 두면 나중에 큰 사고를 예방할 수 있습니다.

---

## 1. GitHub 저장소 설정 메뉴 구조

저장소 페이지 상단 탭에서 **Settings**를 클릭하면 왼쪽에 메뉴가 나타납니다.

```
[저장소 탭]
Code | Issues | Pull requests | Actions | Projects | Security | Insights | Settings
                                                                             ↑ 여기 클릭

[Settings 왼쪽 메뉴]
General          ← 저장소 이름, 설명, 기본 브랜치 변경
Collaborators    ← 팀원 초대
Branches         ← 브랜치 보호 규칙 (이 장의 핵심!)
Tags
Actions
Webhooks
...
```

> **팁:** Settings 탭이 보이지 않으면 해당 저장소의 관리자(owner)가 아닌 것입니다. 본인이 만든 저장소로 연습하세요.

---

## 2. 브랜치란 무엇인가

브랜치는 코드의 **분기점**입니다. 나무에서 줄기(trunk)가 뻗어 나가듯, `main`이라는 기준 코드에서 새 기능을 만들 때 가지(branch)를 따서 작업합니다.

```
main ──●──────────────────────●── (완성된 코드)
        \                    /
feature  ●──●──●──●──●──●──●     (기능 개발 중)
```

터미널에서 브랜치를 만들고 이동하는 명령어는 다음과 같습니다.

```bash
# 현재 브랜치 확인
git branch

# 새 브랜치 만들기
git branch feature/login-page

# 새 브랜치로 이동
git switch feature/login-page

# 만들면서 바로 이동 (위 두 줄을 한 번에)
git switch -c feature/login-page

# 현재 어느 브랜치인지 확인
git status
# On branch feature/login-page
```

---

## 3. 브랜치 보호 규칙 설정하기

**Settings → Branches → Add branch protection rule** 순서로 이동합니다.

### 3-1. 보호할 브랜치 이름 패턴 입력

```
Branch name pattern: main
```

`main`이라고 입력하면 `main` 브랜치에만 규칙이 적용됩니다.  
`release/*`처럼 와일드카드(`*`)를 쓰면 `release/1.0`, `release/2.0` 등 여러 브랜치에 한 번에 적용됩니다.

### 3-2. 핵심 옵션 설명

```
☑ Require a pull request before merging
    └ Required number of approvals: 1
      (merge 전에 PR을 반드시 거쳐야 함, 승인 1명 필요)

☑ Require status checks to pass before merging
    (CI 테스트가 통과해야만 merge 가능)

☑ Do not allow bypassing the above settings
    (관리자도 예외 없이 규칙 적용)

☑ Restrict who can push to matching branches
    (지정된 사람만 push 허용)
```

---

## 4. Pull Request 워크플로 이해하기

브랜치 보호를 설정하면 다음 순서로만 코드를 합칠 수 있습니다.

```
1. 새 브랜치 만들기
   git switch -c feature/회원가입-폼

2. 코드 수정 후 커밋
   git add src/signup.py
   git commit -m "feat: 회원가입 폼 유효성 검사 추가"

3. 원격 저장소에 push
   git push origin feature/회원가입-폼

4. GitHub에서 Pull Request 열기
   → "Compare & pull request" 버튼 클릭

5. 팀원이 코드 리뷰 후 승인(Approve)

6. main에 Merge
```

만약 3단계에서 `main`에 직접 push하려 하면 아래 오류가 납니다.

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Required status check "ci/test" is expected.
To https://github.com/yourname/your-repo.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'https://github.com/yourname/your-repo.git'
```

이 오류가 보이면 정상입니다. 브랜치 보호 규칙이 작동하고 있다는 신호입니다.

---

## 따라 하기 실습

### 실습 1 — 새 저장소 만들고 파일 추가하기

1. GitHub에서 `my-branch-practice`라는 이름으로 새 공개 저장소를 만듭니다 (README 포함 초기화).
2. 터미널에서 클론합니다.

```bash
git clone https://github.com/본인아이디/my-branch-practice.git
cd my-branch-practice
```

3. `main` 브랜치에 파일을 하나 만들어 커밋합니다.

```bash
echo "# 브랜치 실습" > notes.md
git add notes.md
git commit -m "docs: 실습 노트 파일 추가"
git push origin main
```

---

### 실습 2 — 브랜치 보호 규칙 설정하기

1. GitHub 저장소 페이지 → **Settings** → **Branches** → **Add branch protection rule** 클릭
2. **Branch name pattern**에 `main` 입력
3. 아래 옵션만 체크합니다.
   - `Require a pull request before merging` (Required approvals: **1**)
   - `Do not allow bypassing the above settings`
4. **Create** 버튼 클릭

설정 후 `main`에 직접 push가 막히는지 확인합니다.

```bash
echo "직접 push 테스트" >> notes.md
git add notes.md
git commit -m "test: 보호 규칙 확인"
git push origin main
# → GH006 오류가 뜨면 성공!
```

---

### 실습 3 — PR을 통해 정상적으로 merge하기

1. 새 브랜치를 만들고 파일을 수정합니다.

```bash
git switch -c feature/실습-완료-표시
echo "- [x] 브랜치 보호 실습 완료" >> notes.md
git add notes.md
git commit -m "docs: 실습 완료 체크박스 추가"
git push origin feature/실습-완료-표시
```

2. GitHub 저장소 페이지로 이동하면 `Compare & pull request` 버튼이 보입니다. 클릭해서 PR을 엽니다.
3. 본인 저장소라면 스스로 Approve한 뒤 **Merge pull request** 버튼을 클릭합니다.
4. 로컬에서 변경 사항을 내려받습니다.

```bash
git switch main
git pull origin main
cat notes.md
# - [x] 브랜치 보호 실습 완료  ← 이 줄이 보이면 성공
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|----------------------|-----------|
| `main`에 직접 push | `remote: error: GH006: Protected branch update failed` | 새 브랜치를 만들고 PR로 merge하세요 |
| 브랜치 이름을 잘못 입력 | push는 되는데 GitHub에서 브랜치가 안 보임 | `git branch`로 현재 브랜치 이름을 확인하고, `git push origin 정확한-브랜치-이름` 입력 |
| 클론 후 브랜치를 안 만들고 작업 | 커밋은 됐는데 push 막힘 | `git switch -c 새브랜치`로 현재 커밋을 새 브랜치로 옮길 수 있음 |
| PR 승인자가 없어서 merge 불가 | `Review required` 배너 표시 | 팀원에게 리뷰 요청하거나, 연습 중이라면 보호 규칙에서 required approvals를 0으로 임시 변경 |
| 로컬 `main`이 오래됨 | merge 후 로컬에서 파일이 예전 버전 | `git switch main && git pull origin main` 실행 |
| Settings 탭이 없음 | 탭 자체가 화면에 안 보임 | 해당 저장소의 owner가 아닙니다. 본인 소유 저장소에서 연습하세요 |

---

## 확인 체크리스트

- [ ] GitHub 저장소 Settings 탭을 직접 열어 본 적 있다
- [ ] Branches 메뉴에서 브랜치 보호 규칙을 하나 이상 만들었다
- [ ] `main`에 직접 push했을 때 GH006 오류가 뜨는 것을 직접 확인했다
- [ ] 새 브랜치를 만들고, 커밋하고, GitHub에 push했다
- [ ] GitHub에서 PR을 열어 Merge까지 완료했다
- [ ] `git pull origin main`으로 로컬 `main`을 최신 상태로 유지하는 습관이 생겼다
- [ ] 브랜치 이름을 `feature/기능이름` 형식으로 의미 있게 짓고 있다

---

## 한 번 더 생각해 보기

1. 혼자 작업하는 개인 프로젝트에서도 브랜치 보호를 설정하는 게 의미가 있을까요? 어떤 상황에서 도움이 될지 생각해 보세요.

2. `Require a pull request before merging` 옵션에서 **Required approvals**를 1이 아니라 2로 설정하면 어떤 일이 생길까요? 팀 규모와 작업 속도에 어떤 영향을 줄 수 있을지 이야기해 보세요.

3. 실습 2에서 `Do not allow bypassing the above settings`를 체크하지 않으면 어떤 일이 생길까요? 관리자 권한을 가진 사람이 규칙을 우회하는 게 좋은 일일 때는 언제이고, 위험한 때는 언제일지 생각해 보세요.

---

## 다음 장

다음 장에서는 지금까지 배운 브랜치와 PR 워크플로를 바탕으로, 실제 팀 협업 시나리오를 따라 하면서 **충돌(Merge Conflict) 해결 방법**을 배웁니다.