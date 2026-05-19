## 이 장에서 배우는 것

- Git 커밋 메시지가 무엇이고 왜 중요한지 이해한다
- 좋은 커밋 메시지와 나쁜 커밋 메시지의 차이를 구분한다
- Conventional Commits 형식을 익혀 실제 프로젝트에 적용한다
- 커밋 메시지를 보고 변경 이력을 빠르게 파악하는 방법을 배운다
- 팀 협업에서 통용되는 커밋 메시지 규칙을 작성할 수 있다

---

## 먼저 쉬운 설명

택배 상자에 라벨이 없으면 어떻게 될까요? 상자를 일일이 열어봐야 무엇이 들었는지 알 수 있습니다. 커밋 메시지도 마찬가지입니다.

코드를 수정할 때마다 Git은 그 변경 내용을 저장합니다. 이때 남기는 메모가 바로 **커밋 메시지**입니다. 나중에 "왜 이 코드를 바꿨지?", "언제 이 버그가 생겼지?"라고 궁금할 때, 커밋 메시지가 잘 써 있으면 몇 초 안에 답을 찾을 수 있습니다.

혼자 작업할 때도 유용하지만, **팀으로 협업할 때는 필수**입니다. 좋은 커밋 메시지 하나가 동료에게 보내는 친절한 편지와 같습니다.

---

## 1. 커밋 메시지의 기본 구조

커밋 메시지는 세 부분으로 구성됩니다.

```
<타입>(<범위>): <제목>

<본문>

<꼬리말>
```

가장 중요한 것은 첫 번째 줄인 **제목**입니다. 나머지는 필요할 때만 씁니다.

### 좋은 예 vs 나쁜 예

```bash
# 나쁜 커밋 메시지 - 무엇을 했는지 전혀 모름
git commit -m "수정"
git commit -m "asdf"
git commit -m "고침"
git commit -m "작업중"

# 좋은 커밋 메시지 - 무엇을, 왜 했는지 명확함
git commit -m "fix: 로그인 버튼 클릭 시 페이지 새로고침 오류 수정"
git commit -m "feat: 회원가입 이메일 중복 확인 기능 추가"
git commit -m "docs: README에 로컬 실행 방법 추가"
```

---

## 2. Conventional Commits — 업계 표준 형식

많은 팀이 사용하는 **Conventional Commits** 규칙입니다. 타입을 정해두면 커밋 목록만 봐도 어떤 작업인지 바로 알 수 있습니다.

### 자주 쓰는 타입

| 타입 | 의미 | 예시 상황 |
|------|------|-----------|
| `feat` | 새 기능 추가 | 검색 기능 만들기 |
| `fix` | 버그 수정 | 오류 고치기 |
| `docs` | 문서 수정 | README 업데이트 |
| `style` | 코드 스타일 (기능 변경 없음) | 들여쓰기 정리 |
| `refactor` | 리팩터링 | 코드 구조 개선 |
| `test` | 테스트 추가/수정 | 단위 테스트 작성 |
| `chore` | 빌드, 설정 변경 | 패키지 업데이트 |

### 실제 커밋 메시지 작성

```bash
# feat: 새로운 기능
git commit -m "feat: 상품 목록 페이지에 무한 스크롤 추가"

# fix: 버그 수정 (어디서 무슨 버그인지 명시)
git commit -m "fix(auth): 비밀번호 재설정 링크 만료 후 오류 페이지 수정"

# docs: 문서
git commit -m "docs: API 사용 예제 코드 추가"

# refactor: 기능은 그대로, 코드만 개선
git commit -m "refactor: 사용자 조회 함수 중복 로직 제거"

# chore: 기능과 무관한 잡다한 작업
git commit -m "chore: eslint 버전 8.0으로 업그레이드"
```

---

## 3. 제목 줄 작성 규칙

제목 줄 하나가 커밋 메시지의 핵심입니다. 다음 규칙을 지키면 됩니다.

```bash
# 규칙 1: 50자 이내로 짧게
# 나쁨 (너무 길다)
git commit -m "feat: 사용자가 마이페이지에서 프로필 사진을 업로드할 수 있는 기능과 사진 크기 제한 및 포맷 검증 로직 추가"

# 좋음 (핵심만)
git commit -m "feat: 마이페이지 프로필 사진 업로드 기능 추가"

# 규칙 2: 마침표 없이 끝내기
# 나쁨
git commit -m "fix: 결제 오류 수정."

# 좋음
git commit -m "fix: 결제 오류 수정"

# 규칙 3: 명령형으로 쓰기 (무엇을 "했다" 가 아니라 무엇을 "한다")
# 나쁨 - 과거형
git commit -m "feat: 다크모드를 추가했습니다"

# 좋음 - 명령형/현재형
git commit -m "feat: 다크모드 추가"
```

---

## 4. 본문과 꼬리말 작성하기

복잡한 변경이라면 본문에 이유와 맥락을 남깁니다. 꼬리말은 이슈 번호 연결에 씁니다.

```bash
# 본문이 필요한 경우: 에디터로 작성
git commit

# 에디터가 열리면 아래처럼 작성합니다:
```

```
fix(payment): 카드 결제 실패 시 금액 이중 차감 오류 수정

기존 코드에서 결제 API 타임아웃 발생 시 재시도 로직이
중복으로 실행되어 금액이 두 번 차감되는 문제가 있었음.

재시도 전에 이전 요청의 성공 여부를 먼저 확인하도록
로직을 수정함.

Fixes #234
```

```bash
# 한 줄 명령어로도 본문 작성 가능 (\n 사용)
git commit -m "fix: 장바구니 수량 음수 방지

수량 입력값이 0 미만일 경우 자동으로 1로 보정.

Closes #89"
```

---

## 5. .gitmessage 템플릿 설정하기

매번 형식을 기억하기 어렵다면 **커밋 메시지 템플릿**을 만들어두면 편리합니다.

```bash
# 1. 홈 디렉터리에 템플릿 파일 만들기
cat > ~/.gitmessage << 'EOF'
# <타입>(<범위>): <제목> — 50자 이내
# 타입: feat | fix | docs | style | refactor | test | chore


# 본문: 무엇을, 왜 변경했는지 설명 (선택사항)


# 꼬리말: Closes #이슈번호 (선택사항)

EOF

# 2. Git 전역 설정에 템플릿 등록
git config --global commit.template ~/.gitmessage

# 3. 이제 git commit 입력 시 템플릿이 자동으로 열림
git commit
```

---

## 따라 하기 실습

### 실습 1 — 기존 커밋 메시지 개선해보기

다음 나쁜 커밋 메시지를 좋은 메시지로 바꿔보는 연습입니다.

```bash
# 실습용 저장소 준비
mkdir commit-practice && cd commit-practice
git init

# 파일 만들기
echo "# 쇼핑몰 프로젝트" > README.md
echo "def get_user(): pass" > user.py
echo "def process_payment(): pass" > payment.py

# 나쁜 커밋 (연습용으로 일부러 나쁘게)
git add README.md
git commit -m "추가"

git add user.py
git commit -m "유저"

# 위 커밋들을 아래처럼 잘 쓰면 어떻게 달라지는지 비교해보세요
# git commit -m "docs: 프로젝트 소개 README 초안 작성"
# git commit -m "feat: 사용자 조회 함수 스켈레톤 추가"
```

### 실습 2 — Conventional Commits 형식으로 실제 커밋하기

```bash
# 이전 실습 저장소에 이어서 진행

# 기능 추가 커밋
echo "def get_user(user_id): return {'id': user_id, 'name': '홍길동'}" > user.py
git add user.py
git commit -m "feat(user): 사용자 ID로 사용자 정보 조회 기능 구현"

# 버그 수정 커밋
echo "def process_payment(amount):
    if amount <= 0:
        raise ValueError('금액은 0보다 커야 합니다')
    return True" > payment.py
git add payment.py
git commit -m "fix(payment): 결제 금액 0 이하 입력 허용 오류 수정"

# 커밋 이력 확인
git log --oneline
```

예상 출력:
```
a3f1c2e fix(payment): 결제 금액 0 이하 입력 허용 오류 수정
b7d9e1a feat(user): 사용자 ID로 사용자 정보 조회 기능 구현
c2a4f8b 유저
d1e5b9c 추가
```

### 실습 3 — 본문 있는 커밋 작성하기

```bash
# 복잡한 변경사항에 본문 추가
cat > order.py << 'EOF'
def create_order(user_id, items):
    if not items:
        raise ValueError("주문 항목이 비어있습니다")
    total = sum(item['price'] * item['qty'] for item in items)
    return {'user_id': user_id, 'items': items, 'total': total}
EOF

git add order.py

# 본문까지 포함한 커밋 작성
git commit -m "feat(order): 주문 생성 함수 구현

빈 주문 항목 방지 검증과 총액 자동 계산 로직 포함.
추후 쿠폰 및 할인 적용은 별도 함수로 분리 예정.

Refs #15"

# 결과 확인
git log --oneline
git show HEAD
```

---

## 자주 하는 실수

| 실수 | 증상 / 오류 메시지 | 해결 방법 |
|------|-------------------|-----------|
| 커밋 메시지를 빈 칸으로 저장 | `Aborting commit due to empty commit message.` | 에디터에서 `#`으로 시작하지 않는 텍스트를 최소 한 줄 작성 |
| 에디터가 열려서 당황 | vim이 열리고 아무것도 안 됨 | `i` 눌러 입력 모드 → 메시지 작성 → `ESC` → `:wq` 입력 후 엔터 |
| 타입 오타 (fett, fixs 등) | 팀 린터에서 오류 발생 | `feat`, `fix` 등 정확한 타입만 사용, commitlint 도구 활용 |
| 너무 많은 변경을 한 커밋에 | 커밋 메시지가 길어지고 설명 불가 | 기능 단위로 작게 나눠 여러 번 커밋 |
| 한글/영어 혼용 불통일 | 이력이 지저분하게 보임 | 팀에서 한 언어로 통일 (한국 팀은 한글 권장) |
| 이미 푸시한 커밋 메시지 수정 | `git push` 후 강제 푸시 필요 → 팀원 혼란 | 푸시 전에 `git commit --amend`로 수정, 푸시 후엔 수정 자제 |
| 범위(scope) 없이 모든 변경에 feat 사용 | 커밋 이력에서 어느 모듈인지 불분명 | `feat(모듈명):` 형식으로 범위 명시 |

---

## 확인 체크리스트

- [ ] 커밋 메시지 제목이 50자 이내인가?
- [ ] `feat`, `fix`, `docs` 등 올바른 타입을 사용했는가?
- [ ] 제목 끝에 마침표(`.`)를 붙이지 않았는가?
- [ ] "수정", "고침" 같은 모호한 단어 대신 구체적인 내용을 썼는가?
- [ ] 복잡한 변경이라면 본문에 이유를 설명했는가?
- [ ] 관련 이슈 번호가 있다면 꼬리말에 `Closes #번호`로 연결했는가?
- [ ] `git log --oneline`으로 커밋 이력을 확인하고 읽기 쉬운가?
- [ ] 한 커밋에 하나의 논리적 변경만 담겨 있는가?

---

## 한 번 더 생각해 보기

1. **6개월 후의 나**에게 지금 작성한 커밋 메시지가 충분한 정보를 줄 수 있을까요? 오늘 작성한 커밋 이력을 `git log --oneline`으로 확인하고, 메시지만 보고 무슨 작업인지 파악할 수 있는지 점검해보세요.

2. 팀 프로젝트에서 커밋 메시지 규칙이 없다면 어떤 문제가 생길까요? 규칙을 새로 만든다면 팀원들이 가장 쉽게 따를 수 있는 최소한의 규칙은 무엇일지 생각해보세요.

3. 커밋을 "작게 자주" 하는 것과 "크게 가끔" 하는 것 중 어느 쪽이 더 나을까요? 각각의 장단점을 생각해보고, 어떤 상황에서 전략을 달리해야 할지 이야기해보세요.

---

## 다음 장

다음 장에서는 GitHub에서 브랜치를 만들고 Pull Request를 통해 팀원과 코드를 리뷰하는 협업 워크플로우를 배웁니다.