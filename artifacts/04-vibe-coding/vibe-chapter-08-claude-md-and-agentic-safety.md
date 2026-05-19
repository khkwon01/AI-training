## 이 장에서 배우는 것

- `CLAUDE.md`가 무엇인지, 왜 프로젝트마다 만들어야 하는지 이해한다
- 실제 `CLAUDE.md` 파일을 직접 작성할 수 있다
- AI가 여러 파일을 자율로 수정하는 "Agentic coding"의 편리함과 위험성을 안다
- `git diff`로 AI가 바꾼 내용을 검토하는 습관을 익힌다

---

## 먼저 쉬운 설명

Claude Code에 "이 프로젝트 전체를 리팩토링해줘"라고 말하면 어떻게 될까?  
AI는 기쁘게 **수십 개의 파일을 동시에 수정**하기 시작한다.  
편하긴 한데… 내가 모르는 사이에 중요한 설정 파일이 바뀌거나, 잘 돌아가던 기능이 조용히 망가질 수 있다.

이런 상황을 막기 위해 두 가지를 배운다.

1. **`CLAUDE.md`** — "AI야, 이 프로젝트에서는 이렇게 행동해줘"라고 미리 약속해 두는 파일
2. **변경 사항 검토 습관** — AI가 뭘 바꿨는지 눈으로 확인하고 승인하는 루틴

---

## 1. CLAUDE.md란?

`CLAUDE.md`는 프로젝트 루트에 두는 **AI 행동 지침서**다.  
Claude Code는 작업을 시작할 때 이 파일을 자동으로 읽고, 거기 적힌 규칙을 따른다.

### 왜 필요한가?

AI는 매 대화마다 기억을 초기화한다.  
"지난번에 테스트는 pytest로 한다고 했잖아"가 안 통한다는 뜻이다.  
`CLAUDE.md`에 써두면 매번 말하지 않아도 된다.

### 실제 CLAUDE.md 예시

```markdown
# 프로젝트 개요

이 프로젝트는 가계부 웹앱입니다.
Python 3.11 + Flask + SQLite를 사용합니다.

# 코딩 규칙

- 함수 이름은 snake_case로 작성한다 (예: get_expense_list)
- 변수명은 한국어 주석으로 설명한다
- 테스트는 pytest를 사용한다 (`pytest tests/` 로 실행)

# 절대 하지 말 것

- `database.db` 파일을 직접 수정하지 않는다
- `config/secrets.py` 파일을 변경하지 않는다
- 기존 API 엔드포인트의 URL 경로를 바꾸지 않는다

# 폴더 구조

app/
  routes/   ← Flask 라우트 파일
  models/   ← DB 모델
  templates/ ← HTML 템플릿
tests/      ← pytest 테스트
config/     ← 환경 설정 (수정 주의!)
```

이렇게 써두면 Claude는 작업할 때 자동으로 이 규칙을 따른다.

---

## 2. Agentic Coding의 위험성

**Agentic coding**이란 AI가 사람의 확인 없이 파일을 읽고, 쓰고, 명령어를 실행하는 방식이다.  
Claude Code의 강력한 기능이지만, 초보자에게 특히 위험한 시나리오가 있다.

### 위험 시나리오 1: 조용한 삭제

```
사용자: "사용하지 않는 함수들 정리해줘"

AI 행동:
  - utils.py 에서 format_date() 삭제
  - helpers.py 에서 parse_amount() 삭제
  - 총 12개 함수 제거

실제 결과:
  format_date()는 template/report.html 에서 여전히 쓰이고 있었음
  → 사이트 접속 시 500 Internal Server Error 발생
```

실수로 지워진 코드는 `git`이 없으면 **영영 사라진다**.

### 위험 시나리오 2: 설정 파일 오염

```
사용자: "코드 스타일을 PEP8에 맞게 전부 고쳐줘"

AI 행동:
  - 모든 .py 파일 수정 (83개 파일)
  - config/database.py 도 포함되어 수정됨
  - DB 연결 문자열의 들여쓰기가 바뀌면서 파싱 오류 발생

에러 메시지:
  sqlalchemy.exc.ArgumentError: Could not parse rfc1738 URL from string
```

AI는 "설정 파일도 코드니까 고쳐야지"라고 생각한다.  
사람이 보기엔 당연히 건드리면 안 되는 파일이어도.

---

## 3. git diff로 변경 사항 검토하기

AI가 작업을 마치면, **바로 실행하기 전에** 반드시 확인하는 습관을 들인다.

```bash
# AI가 수정한 전체 변경 사항 확인
git diff

# 어떤 파일이 바뀌었는지만 먼저 확인
git diff --stat

# 특정 파일만 확인
git diff app/routes/expense.py
```

**`git diff` 출력 읽는 법:**

```diff
--- a/app/routes/expense.py
+++ b/app/routes/expense.py
@@ -12,7 +12,6 @@
 def get_expenses():
-    expenses = db.query(Expense).filter(active=True).all()  # ← 삭제된 줄 (빨간색)
+    expenses = db.query(Expense).all()                       # ← 추가된 줄 (초록색)
     return jsonify(expenses)
```

`-`(빨간색)는 삭제, `+`(초록색)는 추가다.  
위 예시에서는 `active=True` 필터가 조용히 제거되었다. 아주 위험한 변경이다.

---

## 따라 하기 실습

### 실습 1: 내 프로젝트용 CLAUDE.md 만들기

터미널에서 프로젝트 폴더로 이동한 뒤:

```bash
# 프로젝트 루트로 이동
cd ~/my-project

# CLAUDE.md 파일 생성 (VS Code로 열기)
code CLAUDE.md
```

아래 템플릿을 복사해서 자신의 프로젝트에 맞게 수정한다:

```markdown
# 프로젝트 개요
(프로젝트가 뭘 하는 앱인지 한 줄로)

# 기술 스택
- 언어: Python 3.11
- 프레임워크: (Flask / Django / 없음)
- DB: (SQLite / 없음)

# 코딩 규칙
- (예: 함수명은 영어 snake_case)
- (예: 주석은 한국어로)

# 절대 건드리지 말 것
- (예: config/secrets.py)
- (예: 기존 DB 스키마)

# 테스트 실행 방법
(예: pytest tests/)
```

### 실습 2: AI 변경 후 git diff 확인하기

```bash
# 1. 현재 상태를 git에 저장 (스냅샷)
git add .
git commit -m "AI 작업 전 상태 저장"

# 2. Claude Code에게 작업 요청
# (예: "TODO 주석이 달린 함수들 구현해줘")

# 3. 작업 후 무엇이 바뀌었는지 확인
git diff --stat   # 파일 목록 먼저
git diff          # 상세 내용 확인

# 4. 마음에 들면 커밋, 아니면 되돌리기
git checkout .    # 모든 변경 취소 (주의: 되돌릴 수 없음!)
```

### 실습 3: CLAUDE.md에 금지 항목 테스트하기

`CLAUDE.md`에 아래 줄을 추가한다:

```markdown
# 절대 하지 말 것
- `test_data/` 폴더의 파일은 수정하지 않는다
```

그 다음 Claude Code에게 말한다:

```
"test_data 폴더에 있는 파일도 리팩토링해줘"
```

Claude가 거절하거나 경고를 보낸다면 `CLAUDE.md`가 제대로 작동하는 것이다.

---

## 자주 하는 실수

| 실수 | 발생하는 상황 | 해결 방법 |
|------|-------------|----------|
| `CLAUDE.md`를 프로젝트 루트가 아닌 하위 폴더에 만듦 | Claude가 파일을 인식 못 함 | 반드시 `ls CLAUDE.md`로 루트에 있는지 확인 |
| AI 작업 전에 `git commit`을 안 함 | 되돌릴 스냅샷이 없음 | 습관적으로 작업 전 커밋 먼저 |
| `git diff` 없이 바로 실행 | 숨은 버그를 나중에 발견 | 항상 diff → 확인 → 실행 순서 지키기 |
| "절대 하지 말 것"에 너무 모호하게 씀 | AI가 규칙을 잘못 해석 | 구체적인 파일명·폴더명으로 명시 |
| `git checkout .` 실수로 실행 | 내가 작성한 코드까지 삭제됨 | 커밋 전에는 이 명령어 금지 |

---

## 확인 체크리스트

- [ ] 내 프로젝트 루트에 `CLAUDE.md` 파일이 있다
- [ ] `CLAUDE.md`에 "절대 수정하면 안 되는 파일/폴더"를 명시했다
- [ ] AI에게 작업을 시키기 전에 `git commit`으로 현재 상태를 저장했다
- [ ] AI 작업 완료 후 `git diff --stat`으로 변경된 파일 목록을 확인했다
- [ ] `git diff`로 실제 코드 변경 내용을 한 번 이상 읽어봤다

---

## 한 번 더 생각해 보기

1. AI가 "효율을 높이려고" 내 코드를 바꿨는데, 그 변경이 왜 위험할 수 있을까? 내 프로젝트에서 절대 바뀌면 안 되는 파일이 있다면 무엇인가?

2. `CLAUDE.md`에 규칙을 너무 많이 쓰면 어떤 문제가 생길까? 반대로 너무 적게 쓰면?

3. `git diff`가 불편하게 느껴진다면, 어떤 도구나 방법을 사용하면 더 쉽게 변경 사항을 확인할 수 있을까?

---

## 다음 장

다음 장에서는 실제 미니 프로젝트(가계부 앱)를 처음부터 끝까지 AI와 함께 만들며, 지금까지 배운 모든 vibe coding 습관을 종합 실습한다.