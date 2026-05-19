## 이 장에서 배우는 것

- 혼자 작업할 때와 팀으로 작업할 때 Git 워크플로우가 왜 다른지 이해한다
- 솔로 개발자에게 적합한 단순한 브랜치 전략을 익힌다
- 팀 협업에서 자주 쓰이는 Feature Branch 워크플로우를 실습한다
- Pull Request(PR) / Merge Request(MR) 의 목적과 흐름을 설명할 수 있다
- 충돌(conflict)이 생겼을 때 당황하지 않고 해결하는 절차를 배운다

---

## 먼저 쉬운 설명

혼자 음식을 만들 때는 재료를 바로 꺼내 쓰면 됩니다. 그런데 여러 명이 같은 주방에서 요리하면 어떻게 될까요? 누가 어떤 냄비를 쓰는지, 재료를 어디에 뒀는지 미리 약속해두지 않으면 금방 엉망이 됩니다.

Git도 마찬가지입니다. 혼자라면 `main` 브랜치 하나에 바로 커밋해도 크게 문제없지만, 팀에서 그렇게 하면 서로의 코드가 충돌하거나 미완성 코드가 배포되는 사고가 납니다.

이 장에서는 **"나 혼자 쓸 때"** 와 **"팀이 함께 쓸 때"** 의 Git 워크플로우를 나란히 비교하면서, 왜 규칙이 필요한지, 그 규칙이 실제로 어떻게 생겼는지 배웁니다.

---

## 1. 솔로 개발자 워크플로우 — 단순하게, 하지만 안전하게

혼자 개발할 때도 최소한의 브랜치 전략은 있으면 좋습니다. 가장 흔한 방법은 **main + 작업 브랜치** 패턴입니다.

```
main ──●──────────────────●── (배포 가능한 코드만 있음)
        \                /
         feature/login ●──●   (새 기능 개발)
```

### 1-1. 기본 흐름

```bash
# 1. 새 기능을 시작할 때 브랜치를 만든다
git switch -c feature/add-login

# 2. 파일을 수정하고 커밋한다
# (login.py 파일을 만들었다고 가정)
git add login.py
git commit -m "feat: 로그인 기능 초안 추가"

# 3. 기능이 완성되면 main에 합친다
git switch main
git merge feature/add-login

# 4. 다 쓴 브랜치는 정리한다
git branch -d feature/add-login
```

### 1-2. 솔로 워크플로우 규칙 (권장)

```
main      → 언제나 실행 가능한 코드만 유지
feature/* → 새 기능 하나당 브랜치 하나
fix/*     → 버그 수정 전용
```

> **포인트:** 혼자라도 `main`에 직접 커밋하는 습관은 나중에 팀으로 합류했을 때 가장 먼저 고쳐야 할 습관입니다.

---

## 2. 팀 협업 워크플로우 — Feature Branch + Pull Request

팀에서 가장 널리 쓰이는 방식은 **GitHub Flow** 입니다. 규칙이 단순하면서도 강력합니다.

```
main ──●────────────────────────●── (항상 배포 가능)
        \                      /
         feature/user-profile ●──●──● (PR 후 merge)
```

### 2-1. GitHub Flow 전체 순서

```bash
# ① 최신 main을 받아온다 (팀원 코드가 이미 반영돼 있을 수 있다)
git switch main
git pull origin main

# ② 내 작업 브랜치를 만든다
git switch -c feature/user-profile

# ③ 코드를 작성하고 커밋한다
# (user_profile.py 를 수정했다고 가정)
git add user_profile.py
git commit -m "feat: 사용자 프로필 조회 API 추가"

# ④ 원격 저장소에 내 브랜치를 올린다
git push origin feature/user-profile

# ⑤ GitHub에서 Pull Request를 열고 팀원의 리뷰를 요청한다
# (이 단계는 GitHub 웹에서 진행)

# ⑥ 리뷰 통과 후 main에 merge (보통 GitHub 버튼 클릭)

# ⑦ 로컬 정리
git switch main
git pull origin main
git branch -d feature/user-profile
```

### 2-2. 좋은 브랜치 이름 짓기

```bash
# 좋은 예시
feature/add-payment-api
fix/null-pointer-in-cart
docs/update-readme

# 나쁜 예시
my-branch          # 무슨 작업인지 모름
test123            # 의미 없음
feature/            # 빈 이름
```

---

## 3. merge vs rebase — 팀에서 자주 헷갈리는 개념

### 3-1. merge: 두 흐름을 그대로 합치기

```bash
git switch main
git merge feature/user-profile
```

```
결과:
main ──●──────────────────●──[merge commit]
        \                /
         feature ●──●──●
```

히스토리에 **합쳐진 흔적(merge commit)** 이 남습니다. 팀에서 누가 언제 합쳤는지 추적하기 쉽습니다.

### 3-2. rebase: 브랜치 시작점을 최신으로 옮기기

```bash
git switch feature/user-profile
git rebase main
```

```
결과:
main ──●──●──●──[feature 커밋들이 이어 붙음]
```

히스토리가 **직선**으로 깔끔해집니다. 단, **이미 push한 브랜치에 rebase하면 팀원과 충돌**이 생기므로 주의가 필요합니다.

> **팀 초보자 권장:** 처음에는 `merge`만 써도 충분합니다. `rebase`는 팀 컨벤션을 먼저 확인하세요.

---

## 4. 충돌(Conflict) 해결하기

충돌은 두 사람이 같은 파일의 같은 줄을 동시에 수정했을 때 발생합니다. 무서워 보이지만 Git이 어디서 충돌이 났는지 정확히 알려줍니다.

### 4-1. 충돌이 생기면 파일 안이 이렇게 됩니다

```python
# config.py

<<<<<<< HEAD
DEBUG = False   # 내가 main에서 수정한 내용
=======
DEBUG = True    # feature 브랜치에서 수정한 내용
>>>>>>> feature/user-profile
```

| 표시 | 의미 |
|---|---|
| `<<<<<<< HEAD` | 현재 브랜치(main)의 내용 시작 |
| `=======` | 구분선 |
| `>>>>>>> feature/...` | 합치려는 브랜치의 내용 끝 |

### 4-2. 충돌 해결 순서

```bash
# 1. 충돌 난 파일을 직접 열어서 원하는 내용으로 고친다
#    (위의 <<<, ===, >>> 마커를 모두 삭제하고 최종 코드만 남김)

# config.py 최종 결과 예시:
# DEBUG = False

# 2. 수정한 파일을 스테이지에 올린다
git add config.py

# 3. 충돌 해결 커밋을 만든다
git commit -m "fix: DEBUG 설정 충돌 해결 (False 유지)"
```

---

## 따라 하기 실습

### 실습 1 — 솔로 워크플로우 체험

```bash
# 1. 실습용 디렉터리 준비
mkdir git-workflow-practice && cd git-workflow-practice
git init

# 2. 첫 커밋 (main 베이스)
echo "# My App" > README.md
git add README.md
git commit -m "chore: 프로젝트 초기화"

# 3. 새 기능 브랜치 생성
git switch -c feature/greeting

# 4. 기능 파일 추가
cat > greeting.py << 'EOF'
def say_hello(name: str) -> str:
    return f"안녕하세요, {name}님!"
EOF

git add greeting.py
git commit -m "feat: 인사말 함수 추가"

# 5. main에 합치기
git switch main
git merge feature/greeting

# 6. 브랜치 삭제
git branch -d feature/greeting

# 확인
git log --oneline --graph
```

**기대 출력:**
```
*   <hash> feat: 인사말 함수 추가
* <hash> chore: 프로젝트 초기화
```

---

### 실습 2 — 의도적으로 충돌 만들고 해결하기

```bash
# (실습 1 결과물에서 이어서 진행)

# 1. main에서 파일 수정
echo "version = '1.0.0'" > config.py
git add config.py
git commit -m "chore: 버전 파일 추가"

# 2. 새 브랜치에서 같은 파일 다르게 수정
git switch -c fix/version-update
echo "version = '1.1.0'" > config.py
git add config.py
git commit -m "fix: 버전 1.1.0으로 업데이트"

# 3. main으로 돌아가 또 다르게 수정
git switch main
echo "version = '2.0.0'" > config.py
git add config.py
git commit -m "chore: 버전 2.0.0으로 올림"

# 4. 충돌을 일으켜 보기
git merge fix/version-update
# → CONFLICT (content): Merge conflict in config.py 메시지 등장!

# 5. 파일을 열어 마커를 제거하고 '2.0.0'으로 정리
#    (텍스트 에디터로 config.py 수정)

# 6. 해결 완료
git add config.py
git commit -m "fix: 버전 충돌 해결 (2.0.0 유지)"
```

---

### 실습 3 — 팀 협업 시뮬레이션 (원격 없이 로컬 2브랜치로 흉내내기)

```bash
# 동료가 먼저 push했다고 가정하고, 최신 main을 pull하는 상황을 연습한다

# 1. "동료의 작업"을 흉내내어 main에 커밋 추가
git switch main
echo "AUTHOR = 'Alice'" >> config.py
git add config.py
git commit -m "chore: 작성자 정보 추가 (Alice)"

# 2. 내 작업 브랜치는 그 이전 시점에서 분기됐다고 가정
git switch -c feature/my-feature HEAD~1
echo "TIMEOUT = 30" >> config.py
git add config.py
git commit -m "feat: 타임아웃 설정 추가"

# 3. main의 최신 내용을 내 브랜치에 반영 (merge 방식)
git merge main
# 충돌 없으면 자동 완료, 충돌 있으면 실습 2처럼 해결

# 4. 최종 상태 확인
git log --oneline --graph --all
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| `main`에 직접 push했을 때 팀 규칙 위반 | 별도 에러 없지만 팀원 코드 덮어쓸 수 있음 | 항상 feature 브랜치에서 작업 후 PR |
| push 전 pull을 안 해서 충돌 | `error: failed to push some refs` | `git pull --rebase origin main` 후 재시도 |
| 충돌 마커를 파일에 그대로 남기고 커밋 | 코드가 실행되지 않거나 파싱 오류 | `git diff` 로 `<<<<<<<` 마커 남아있는지 확인 |
| 브랜치를 삭제하지 않아 목록이 쌓임 | `git branch` 목록이 수십 개 | merge 완료 후 `git branch -d 브랜치명` |
| 잘못된 브랜치에서 작업 시작 | 원치 않는 브랜치에 커밋 쌓임 | `git log --oneline` 으로 현재 브랜치 먼저 확인 |
| rebase 후 force push로 공유 브랜치 덮어씀 | 팀원 로컬 히스토리와 원격이 달라짐 | 공유 브랜치에는 절대 `git push --force` 금지 |

---

## 확인 체크리스트

- [ ] `git switch -c 브랜치명` 으로 새 브랜치를 만들고 이동할 수 있다
- [ ] 솔로 워크플로우와 GitHub Flow의 차이를 한 문장으로 설명할 수 있다
- [ ] `git merge`와 `git rebase`의 결과가 히스토리에서 어떻게 다른지 안다
- [ ] 충돌 파일에서 `<<<<<<<`, `=======`, `>>>>>>>` 마커를 직접 찾아 제거할 수 있다
- [ ] PR(Pull Request)을 여는 목적(코드 리뷰, 논의, 품질 보증)을 설명할 수 있다
- [ ] merge 후 필요 없어진 브랜치를 `git branch -d` 로 정리할 수 있다
- [ ] `git log --oneline --graph` 로 브랜치 히스토리를 시각적으로 확인할 수 있다

---

## 한 번 더 생각해 보기

1. **혼자 작업할 때도 브랜치를 만드는 게 귀찮게 느껴진다면, 어떤 상황에서 그 귀찮음을 감수할 가치가 있을까요?** 예를 들어, 새 기능을 개발하는 도중에 긴급 버그가 생겼다면 브랜치가 있는 것과 없는 것이 어떻게 다를지 상상해 보세요.

2. **팀에서 코드 리뷰 없이 바로 main에 merge하면 어떤 문제가 생길 수 있을까요?** 코드 품질, 버그 발견, 지식 공유 측면에서 각각 생각해 보세요.

3. **`git rebase`는 히스토리를 깔끔하게 만들지만, 이미 공유된 브랜치에 쓰면 위험하다고 했습니다. 왜 그럴까요?** 두 사람이 같은 브랜치를 기준으로 작업하고 있는데 한 명이 rebase를 하면 다른 사람의 로컬 저장소에서 무슨 일이 생길지 그림으로 그려 보세요.

---

## 다음 장

다음 장에서는 GitHub Actions를 활용해 PR이 열릴 때 자동으로 테스트와 린트를 실행하는 CI 파이프라인을 구성하는 방법을 배웁니다.