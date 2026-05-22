# Chapter 08: CLAUDE.md와 Agentic Coding 안전하게 쓰기

## 이 장에서 배우는 것

- `CLAUDE.md`가 무엇인지 (프로젝트 설명서 역할)
- 실제 CLAUDE.md 예시 파일 전체 내용
- Agentic coding이란 무엇인지 (AI가 여러 파일을 자율 수정)
- Agentic coding 위험 시나리오 2개 (파일 덮어쓰기, 의존성 변경)
- `git diff`로 AI 변경 내용 검토하는 3단계
- 변경 승인 전 체크리스트
- 실습 3개: CLAUDE.md 직접 만들기, diff 검토하기, 위험한 변경 되돌리기

---

## 왜 CLAUDE.md와 안전 습관이 필요한가?

Claude Code에게 "이 프로젝트 전체를 리팩토링해줘"라고 말하면 어떻게 될까?

AI는 기쁘게 수십 개의 파일을 동시에 수정하기 시작한다.

편하긴 하다. 그런데 다음과 같은 일이 생길 수 있다.

- 잘 돌아가던 기능이 조용히 망가진다
- 중요한 설정 파일이 내가 모르는 사이에 바뀐다
- 배포 환경에서만 사용하던 변수가 개발 환경에서 덮어써진다
- 보안상 건드리면 안 되는 파일이 수정된다

이런 상황을 막기 위해 두 가지를 배운다.

1. `CLAUDE.md` — "AI야, 이 프로젝트에서는 이렇게 행동해줘"라고 미리 약속해 두는 파일
2. 변경 사항 검토 습관 — AI가 뭘 바꿨는지 눈으로 확인하고 승인하는 루틴

---

## 1. CLAUDE.md란?

`CLAUDE.md`는 프로젝트 루트에 두는 **AI 행동 지침서**다.

Claude Code는 작업을 시작할 때 이 파일을 자동으로 읽고, 거기 적힌 규칙을 따른다.

### 왜 필요한가?

AI는 매 대화마다 기억을 초기화한다. 지난번에 "테스트는 pytest로 해줘"라고 말했어도, 새 대화를 시작하면 AI는 이 사실을 모른다.

`CLAUDE.md`에 써두면:
- 매번 같은 규칙을 설명하지 않아도 된다
- 팀원이 같은 AI 설정을 공유할 수 있다
- 프로젝트에 맞는 코딩 스타일이 자동으로 반영된다
- 절대 건드리면 안 되는 파일을 AI가 보호한다

### CLAUDE.md 위치

반드시 프로젝트 **루트 폴더**에 있어야 한다.

```
my-project/          ← 프로젝트 루트
  CLAUDE.md          ← 여기에 있어야 함 (올바름)
  src/
    CLAUDE.md        ← 여기는 안 됨 (하위 폴더)
  app.py
  README.md
```

---

## 2. 실제 CLAUDE.md 예시

아래는 Python 웹 앱 프로젝트용 `CLAUDE.md` 예시 전체다. 직접 복사해서 자신의 프로젝트에 맞게 수정해도 좋다.

```markdown
# 프로젝트 개요

이 프로젝트는 개인 가계부 웹앱입니다.
수입/지출을 기록하고 월별 통계를 보여줍니다.

## 기술 스택

- Python 3.11
- Flask 3.0 (웹 프레임워크)
- SQLite (데이터베이스)
- pytest (테스트)
- Jinja2 (HTML 템플릿)

---

# 폴더 구조

```
my-budget/
  app/
    routes/       ← Flask 라우트 파일 (URL 처리)
    models/       ← 데이터베이스 모델
    templates/    ← HTML 템플릿
    static/       ← CSS, JS, 이미지
  tests/          ← pytest 테스트 파일
  config/         ← 환경 설정 (주의!)
  database.db     ← SQLite 데이터베이스 파일
```

---

# 코딩 규칙

## 이름 규칙
- 함수 이름: snake_case (예: get_expense_list, calculate_monthly_total)
- 클래스 이름: PascalCase (예: ExpenseManager, UserAccount)
- 상수: 대문자 snake_case (예: MAX_RECORDS, DEFAULT_CURRENCY)
- 변수명은 한국어 주석으로 의미를 설명한다

## 함수 규칙
- 함수 하나는 한 가지 일만 한다
- 함수 길이는 30줄 이하로 유지한다
- 모든 함수에 docstring을 추가한다 (한국어 가능)

## 오류 처리
- 사용자 입력은 반드시 검증한다
- 데이터베이스 오류는 try/except로 잡는다
- 오류 메시지는 사용자가 이해할 수 있게 한국어로 작성한다

---

# 테스트

테스트 실행 방법:
```bash
pytest tests/
pytest tests/ -v        # 상세 출력
pytest tests/test_expense.py  # 특정 파일만
```

테스트 파일은 반드시 `test_`로 시작한다.
새 기능을 추가하면 해당 기능의 테스트도 같이 추가한다.

---

# 절대 하지 말 것

## 수정 금지 파일
- `config/secrets.py` — API 키, 비밀번호 포함
- `config/production.py` — 배포 환경 설정
- `database.db` — 직접 수정 금지 (SQLAlchemy를 통해서만)
- `.env` — 환경변수 파일

## 변경 금지 사항
- 기존 API 엔드포인트의 URL 경로를 바꾸지 않는다
  (예: `/api/expenses` → `/api/expense`로 바꾸면 기존 클라이언트가 깨짐)
- `models/` 폴더의 데이터베이스 스키마를 함부로 변경하지 않는다
  (마이그레이션 스크립트 필요)
- `requirements.txt`에서 패키지 버전을 임의로 변경하지 않는다

## 보안
- 사용자 입력을 SQL 쿼리에 직접 넣지 않는다 (SQL 인젝션 방지)
- 비밀번호를 평문으로 저장하지 않는다
- `print()`로 비밀번호나 API 키를 출력하지 않는다

---

# 현재 진행 상황

완성된 기능:
- [x] 지출 추가/조회/삭제
- [x] 사용자 로그인/로그아웃
- [ ] 월별 통계 차트 (작업 중)
- [ ] CSV 내보내기 (미시작)
```

이렇게 써두면 Claude는 작업할 때 자동으로 이 규칙을 따른다.

---

## 3. Agentic Coding이란?

**Agentic coding**이란 AI가 사람의 확인 없이 여러 파일을 읽고, 쓰고, 명령어를 실행하는 방식이다.

일반적인 AI 사용:
```
사용자: 코드 보여줘 → AI: 코드 제안 → 사용자: 복사/붙여넣기
```

Agentic coding:
```
사용자: "로그인 기능 추가해줘" → AI: 파일 읽기 → 파일 수정 → 파일 저장 → 테스트 실행
(사람이 확인하는 단계 없이 진행)
```

Claude Code는 Agentic coding이 가능한 도구다. 한 번의 요청으로 여러 파일을 자동으로 수정하고 명령어까지 실행할 수 있다.

**장점:** 반복적인 작업을 빠르게 처리할 수 있다.

**위험:** 사람이 예상하지 못한 파일도 수정될 수 있다.

---

## 4. Agentic Coding 위험 시나리오

### 위험 시나리오 1: 조용한 파일 덮어쓰기

```
사용자: "사용하지 않는 함수들 정리해줘"

AI 행동:
  - utils.py 분석 → format_date() 함수가 다른 곳에서 호출되지 않는 것처럼 보임
  - format_date() 삭제
  - helpers.py 분석 → parse_amount() 함수가 직접 호출되지 않는 것처럼 보임
  - parse_amount() 삭제
  - 총 12개 함수 제거

실제 결과:
  format_date()는 templates/report.html 에서 Jinja2 필터로 쓰이고 있었음
  AI는 HTML 파일까지 분석하지 못했음
  → 사이트 접속 시 500 Internal Server Error 발생
  → 함수가 삭제되어 로그 리포트 페이지 전체가 먹통
```

**왜 위험한가:** 함수 삭제는 취소하기 어렵다. `git`이 없으면 영영 사라진다. `git`이 있어도 어느 커밋에서 삭제됐는지 찾아야 한다.

**어떻게 막는가:** CLAUDE.md에 "어떤 함수도 삭제하기 전에 먼저 확인을 요청해줘"라고 명시한다. 그리고 AI 작업 전에 반드시 `git commit`으로 스냅샷을 만든다.

---

### 위험 시나리오 2: 의존성과 설정 파일 오염

```
사용자: "코드 스타일을 PEP8에 맞게 전부 고쳐줘"

AI 행동:
  - 프로젝트 전체 .py 파일 스캔: 83개
  - 모든 파일 들여쓰기, 공백, 줄 길이 수정
  - config/database.py도 Python 파일이므로 포함되어 수정됨
  - DB 연결 문자열 주변 들여쓰기가 바뀌면서 파싱 오류 발생

에러 메시지:
  sqlalchemy.exc.ArgumentError: Could not parse rfc1738 URL from string
  
원인:
  DATABASE_URL = (
      "postgresql://user:password@"
      "localhost:5432/mydb"   # ← 들여쓰기 변경으로 문자열 연결이 깨짐
  )
```

**왜 위험한가:** AI는 "설정 파일도 Python 파일이니까 스타일 수정 대상"이라고 판단한다. 사람이 보기엔 당연히 건드리면 안 되는 파일이어도.

**다른 의존성 변경 사례:**

```
requirements.txt 변경 예시:
  - 변경 전: flask==2.3.0
  - 변경 후: flask>=2.3.0   (AI가 "더 유연한 버전 지정"으로 수정)
  
결과:
  다음 배포 시 flask 3.0.0이 설치됨
  3.0에서 제거된 API를 쓰는 코드가 전부 오류 발생
```

**어떻게 막는가:** CLAUDE.md에 수정 금지 파일 목록을 명시한다. 그리고 `git diff`로 변경 내용을 반드시 검토한다.

---

## 5. git diff로 AI 변경 내용 검토하기 (3단계)

AI가 작업을 마치면 **바로 실행하기 전에** 반드시 변경 내용을 확인하는 습관을 들인다.

### 1단계: 어떤 파일이 바뀌었는지 목록 확인

```bash
git diff --stat
```

출력 예시:
```
 app/routes/expense.py   | 15 ++++++++-------
 app/models/expense.py   |  8 ++---
 config/database.py      |  3 ++-
 requirements.txt        |  2 +-
 4 files changed, 15 insertions(+), 13 deletions(-)
```

여기서 확인할 것:
- 예상치 못한 파일이 포함되어 있는가? (`config/database.py`, `requirements.txt` 같은 것)
- 변경된 파일 수가 너무 많지 않은가?

### 2단계: 민감한 파일부터 상세 확인

목록에서 위험해 보이는 파일부터 자세히 본다.

```bash
git diff config/database.py
git diff requirements.txt
```

### 3단계: 전체 변경 내용 확인

```bash
git diff
```

**`git diff` 출력 읽는 법:**

```diff
--- a/app/routes/expense.py       ← 변경 전 파일
+++ b/app/routes/expense.py       ← 변경 후 파일

@@ -12,7 +12,6 @@               ← 변경 위치 (12번째 줄 근처)

 def get_expenses():
-    expenses = db.query(Expense).filter(active=True).all()  ← 삭제된 줄 (빨간색)
+    expenses = db.query(Expense).all()                       ← 추가된 줄 (초록색)
     return jsonify(expenses)
```

- `-`(빨간색): 삭제된 줄
- `+`(초록색): 추가된 줄
- 공백으로 시작하는 줄: 변경 없음 (맥락 표시용)

위 예시에서는 `active=True` 필터가 조용히 제거되었다. 비활성화된 지출 항목도 모두 조회되게 바뀐 것이다. 매우 위험한 변경이다.

---

## 6. 변경 승인 전 체크리스트

AI 작업 완료 후 다음을 확인하고 나서 코드를 커밋하거나 실행한다.

```
변경 승인 전 체크리스트:

파일 범위 확인
[ ] 예상한 파일만 변경되었는가?
[ ] 설정 파일(config/, .env, secrets.py)이 포함되어 있지 않은가?
[ ] requirements.txt나 package.json이 바뀌지 않았는가?

코드 내용 확인
[ ] 삭제된 코드(-로 표시된 줄)가 다른 곳에서 쓰이지 않는가?
[ ] 조건문이 제거되거나 변경되지 않았는가?
[ ] 보안 관련 코드(인증, 암호화)가 변경되지 않았는가?
[ ] API URL 경로가 바뀌지 않았는가?

동작 확인
[ ] 변경된 파일에 대한 테스트가 통과하는가?
[ ] 직접 실행해서 주요 기능이 동작하는가?
[ ] 오류 메시지나 경고가 새로 생기지 않았는가?
```

---

## 실습

### 실습 1: 내 프로젝트용 CLAUDE.md 만들기 (따라 하기)

터미널에서 프로젝트 폴더로 이동한 후 VS Code로 파일을 만든다.

```bash
cd ~/내-프로젝트-폴더
code CLAUDE.md
```

아래 템플릿을 복사해서 자신의 프로젝트에 맞게 수정한다.

```markdown
# 프로젝트 개요

(이 프로젝트가 무엇을 하는 앱인지 한 문장으로)

## 기술 스택

- 언어: Python 3.11
- 프레임워크: (Flask / Django / 없음)
- 데이터베이스: (SQLite / 없음)
- 테스트: pytest

---

# 폴더 구조

```
(프로젝트 폴더 구조를 여기에)
```

---

# 코딩 규칙

- 함수 이름은 영어 snake_case (예: get_user_list)
- 주석은 한국어로 작성한다
- 함수 하나는 한 가지 일만 한다
- 새 기능 추가 시 테스트도 함께 추가한다

---

# 테스트 실행 방법

```bash
pytest tests/
```

---

# 절대 하지 말 것

- (예: config/secrets.py 수정 금지)
- (예: 기존 API URL 경로 변경 금지)
- (예: database.db 직접 수정 금지)
- 어떤 파일도 삭제하기 전에 먼저 확인을 요청한다
```

파일을 저장한 후 Claude Code에서 프로젝트를 열고 간단한 작업을 요청해본다.

```
"현재 프로젝트 구조를 설명해줘"
```

Claude가 CLAUDE.md를 읽었다는 것을 응답에서 확인할 수 있다.

**직접 해보기:** "절대 하지 말 것" 섹션에 자신의 프로젝트에서 실제로 건드리면 안 되는 파일이나 규칙을 추가해보자. 구체적인 파일명을 포함해야 더 효과적이다.

---

### 실습 2: AI 변경 후 git diff로 검토하기 (따라 하기)

실습을 위한 준비 단계:

```bash
# 1. 연습용 git 저장소 준비 (이미 있으면 생략)
cd ~/내-프로젝트-폴더
git init
git add .
git commit -m "AI 작업 전 초기 상태"
```

AI에게 작업을 요청한다. 예를 들어:

```
"add_memo 함수에 입력값 검증을 추가해줘.
text가 빈 문자열이면 '메모 내용을 입력해주세요'라고 출력하고 저장하지 말아줘."
```

AI가 파일을 수정하면:

```bash
# 2단계: 어떤 파일이 바뀌었는지 먼저 확인
git diff --stat
```

```bash
# 3단계: 변경 내용 상세 확인
git diff
```

diff를 읽으면서 다음을 체크한다:

```
[ ] 변경된 파일이 memo_service.py 하나뿐인가?
[ ] 추가된 줄(+)이 내가 요청한 검증 로직인가?
[ ] 삭제된 줄(-)이 있다면 왜 삭제됐는지 이해하는가?
[ ] 예상치 못한 파일이 변경되지 않았는가?
```

모두 확인이 됐으면:

```bash
# 4단계: 확인 후 커밋
git add memo_service.py
git commit -m "add_memo에 입력값 검증 추가"
```

**직접 해보기:** AI에게 더 큰 범위의 작업을 요청해보자. 예를 들어 "모든 함수에 docstring 추가해줘"라고 하고, `git diff --stat`으로 몇 개의 파일이 변경됐는지 확인한다. 예상보다 많은 파일이 변경됐다면 하나씩 확인한다.

---

### 실습 3: 위험한 변경 되돌리기 (따라 하기)

이 실습에서는 AI가 실수로 잘못된 변경을 했을 때 어떻게 되돌리는지 연습한다.

**준비 단계:**

```bash
# 현재 상태를 커밋으로 저장
git add .
git commit -m "실습 3 시작 전 상태"
```

**의도적으로 잘못된 변경 시뮬레이션:**

`memo_service.py`에서 `memos = []` 줄을 직접 삭제해보자. 이게 AI가 "사용하지 않는 전역 변수를 정리했다"라고 착각하는 상황이다.

```bash
# 변경 후 diff 확인
git diff
```

출력:
```diff
--- a/memo_service.py
+++ b/memo_service.py
@@ -1,4 +1,3 @@
-memos = []                      ← 이 줄이 삭제됨
 
 def add_memo(text):
     memos.append(text)
```

`memos = []`가 삭제되면 `add_memo`를 호출할 때 `NameError`가 난다는 것을 알 수 있다.

**방법 1: 특정 파일만 되돌리기**

```bash
git checkout -- memo_service.py
```

이 명령은 `memo_service.py`를 마지막 커밋 상태로 되돌린다.

확인:
```bash
git diff    # 아무것도 안 나오면 정상 (변경사항이 없어진 것)
```

**방법 2: 여러 파일이 잘못 변경됐을 때 전체 되돌리기**

```bash
git checkout .    # 주의: 커밋하지 않은 모든 변경이 사라짐
```

이 명령은 매우 강력하다. 커밋하지 않은 변경사항을 **전부** 되돌리므로, 내가 직접 작성한 코드도 사라질 수 있다. 사용 전에 반드시 `git diff`로 확인한다.

**중요한 교훈:**

AI에게 작업을 시키기 **전에** 항상 커밋을 먼저 한다. 커밋이 있으면 언제든 그 시점으로 돌아갈 수 있다. 커밋이 없으면 되돌릴 방법이 없다.

```bash
# AI 작업 전에 항상 이 루틴을 따른다
git add .
git commit -m "AI 작업 전 스냅샷: (요청 내용 한 줄 요약)"

# AI에게 작업 요청
# ...

# 작업 후 검토
git diff --stat
git diff

# 문제 없으면 커밋, 문제 있으면 되돌리기
```

---

## 자주 하는 실수 정리표

| 실수 | 발생하는 상황 | 해결 방법 |
|------|-------------|----------|
| `CLAUDE.md`를 하위 폴더에 만듦 | Claude가 파일을 인식 못 함 | `ls CLAUDE.md`로 루트에 있는지 확인 |
| AI 작업 전 `git commit` 안 함 | 되돌릴 스냅샷이 없음 | 작업 전 커밋을 습관화 |
| `git diff` 없이 바로 실행 | 숨은 버그를 나중에 발견 | diff → 확인 → 실행 순서 지키기 |
| "절대 하지 말 것"에 모호하게 씀 | AI가 규칙을 잘못 해석 | 구체적인 파일명, 폴더명으로 명시 |
| `git checkout .` 실수로 실행 | 내가 작성한 코드까지 삭제됨 | 커밋 전에는 이 명령어 금지 |
| diff를 읽지 않고 승인 | 위험한 변경 그대로 적용 | 삭제된 줄(-) 반드시 확인 |
| 한 번에 너무 큰 작업 요청 | 수십 개 파일 변경으로 검토 불가 | 작은 단위로 나눠서 요청 |

---

## 확인 체크리스트

- [ ] 내 프로젝트 루트에 `CLAUDE.md` 파일이 있다
- [ ] `CLAUDE.md`에 기술 스택과 코딩 규칙이 명시되어 있다
- [ ] `CLAUDE.md`에 "절대 수정하면 안 되는 파일/폴더"가 구체적으로 명시되어 있다
- [ ] AI에게 작업을 시키기 전에 `git commit`으로 현재 상태를 저장했다
- [ ] AI 작업 완료 후 `git diff --stat`으로 변경된 파일 목록을 확인했다
- [ ] `git diff`로 실제 코드 변경 내용(삭제된 줄 포함)을 확인했다
- [ ] 잘못된 변경을 `git checkout -- 파일명`으로 되돌릴 수 있다
- [ ] Agentic coding의 위험 시나리오 2가지를 설명할 수 있다

---

## 한 번 더 생각해 보기

1. AI가 "효율을 높이려고" 내 코드를 바꿨는데, 그 변경이 왜 위험할 수 있을까? 내 프로젝트에서 절대 바뀌면 안 되는 파일이 있다면 무엇인가?

2. `CLAUDE.md`에 규칙을 너무 많이 쓰면 어떤 문제가 생길까? 반대로 너무 적게 쓰면?

3. `git diff`가 불편하게 느껴진다면, VS Code의 Source Control 패널을 이용하면 같은 내용을 시각적으로 볼 수 있다. 두 방법 중 어느 것이 더 편한가?

4. "AI 작업 전 커밋"과 "AI 작업 후 커밋"을 구분하면 어떤 이점이 있을까? 나쁜 점은?

---

## 다음 장

다음 장에서는 실제 미니 프로젝트(가계부 앱)를 처음부터 끝까지 AI와 함께 만들며, 지금까지 배운 모든 vibe coding 습관을 종합 실습한다.

---

## 참고 자료

- Claude Code 공식 문서 — https://docs.anthropic.com/claude-code
- Git 기초 — https://git-scm.com/book/ko/v2
- git diff 사용법 — https://git-scm.com/docs/git-diff
