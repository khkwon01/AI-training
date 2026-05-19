## 이 장에서 배우는 것

- 역할(Role)을 AI에게 부여하는 프롬프트 패턴이 무엇인지 이해한다
- Planner(계획자), Implementer(구현자), Reviewer(검토자) 세 가지 역할 패턴을 구분한다
- 각 역할에 맞는 프롬프트를 직접 작성한다
- 역할 조합으로 코드 품질을 높이는 방법을 실습한다

---

## 먼저 쉬운 설명

AI에게 "코드 짜줘"라고 하면 어떤 일이 일어날까요?

AI는 최선을 다하지만, **역할이 없으면 방향이 없습니다.** 마치 회사에서 팀장도 없고 업무 분담도 없이 "그냥 일 해"라고 하는 것과 같습니다.

반면 이렇게 하면 어떨까요?

> "너는 지금 시니어 아키텍트야. 이 기능을 어떻게 설계할지 **계획**만 세워줘."

> "너는 지금 백엔드 개발자야. 위 계획대로 **코드를 구현**해줘."

> "너는 지금 코드 리뷰어야. 이 코드의 **문제점**을 찾아줘."

역할을 명확히 하면 AI의 응답이 훨씬 구체적이고 유용해집니다. 이게 바로 **역할 기반 프롬프트 패턴**입니다.

---

## 1. Planner(계획자) 역할 패턴

Planner는 실제 코드를 작성하지 않습니다. **무엇을**, **어떤 순서로**, **왜** 만들지를 설계합니다.

```
# ✅ Planner 프롬프트 예시

역할: 당신은 경험 많은 소프트웨어 아키텍트입니다.

임무: 사용자 로그인 기능을 설계해 주세요.

규칙:
- 코드를 작성하지 마세요
- 단계별 설계 계획만 작성하세요
- 각 단계에서 왜 그렇게 하는지 이유를 설명하세요
- 예상되는 위험 요소도 포함하세요

요청: JWT를 사용한 로그인 API를 설계해 주세요.
```

**AI가 반환하는 응답 예시:**

```
[설계 계획]

1단계: 사용자 입력 검증
   - 이유: 잘못된 데이터가 DB에 들어가는 것을 막기 위해
   - 위험: 이메일 형식 오류, 비밀번호 길이 미달

2단계: DB에서 사용자 조회
   - 이유: 등록된 사용자인지 확인
   - 위험: SQL 인젝션, 없는 이메일 처리

3단계: 비밀번호 해시 비교
   - 이유: 평문 비밀번호를 DB에 저장하면 안 됨
   - 위험: bcrypt 비교 실패 시 처리

4단계: JWT 토큰 발급
   - 이유: 이후 요청에서 인증 상태 유지
   - 위험: 토큰 만료 시간 설정 필요
```

---

## 2. Implementer(구현자) 역할 패턴

Implementer는 계획을 **실제 동작하는 코드**로 만듭니다. Planner가 만든 설계를 입력으로 받습니다.

```
# ✅ Implementer 프롬프트 예시

역할: 당신은 Python 백엔드 개발자입니다.

임무: 아래 설계 계획을 Python FastAPI 코드로 구현해 주세요.

규칙:
- 설계 계획의 순서를 반드시 따르세요
- 각 단계를 함수로 분리하세요
- 타입 힌트를 반드시 포함하세요
- 에러 처리를 빠뜨리지 마세요

설계 계획:
[위에서 Planner가 작성한 계획 붙여넣기]
```

**AI가 반환하는 코드 예시:**

```python
# login_service.py

from fastapi import HTTPException
from passlib.context import CryptContext
import jwt
from datetime import datetime, timedelta

pwd_context = CryptContext(schemes=["bcrypt"])
SECRET_KEY = "your-secret-key"

# 1단계: 사용자 입력 검증
def validate_login_input(email: str, password: str) -> None:
    if "@" not in email:
        raise HTTPException(status_code=400, detail="이메일 형식이 올바르지 않습니다")
    if len(password) < 8:
        raise HTTPException(status_code=400, detail="비밀번호는 8자 이상이어야 합니다")

# 2단계: DB에서 사용자 조회
def get_user_by_email(db, email: str):
    user = db.query(User).filter(User.email == email).first()
    if not user:
        raise HTTPException(status_code=404, detail="존재하지 않는 사용자입니다")
    return user

# 3단계: 비밀번호 해시 비교
def verify_password(plain_password: str, hashed_password: str) -> bool:
    if not pwd_context.verify(plain_password, hashed_password):
        raise HTTPException(status_code=401, detail="비밀번호가 일치하지 않습니다")
    return True

# 4단계: JWT 토큰 발급
def create_jwt_token(user_id: int) -> str:
    payload = {
        "user_id": user_id,
        "exp": datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm="HS256")
```

---

## 3. Reviewer(검토자) 역할 패턴

Reviewer는 완성된 코드를 **비판적으로 검토**합니다. 칭찬이 아니라 문제를 찾는 게 임무입니다.

```
# ✅ Reviewer 프롬프트 예시

역할: 당신은 보안에 특화된 시니어 코드 리뷰어입니다.

임무: 아래 코드를 검토하고 문제점을 보고해 주세요.

규칙:
- 칭찬하지 마세요, 문제점만 나열하세요
- 심각도를 [높음/중간/낮음]으로 분류하세요
- 각 문제에 수정 코드를 제안하세요
- 보안, 성능, 가독성 세 가지 관점에서 검토하세요

검토할 코드:
[위에서 Implementer가 작성한 코드 붙여넣기]
```

**AI가 반환하는 리뷰 예시:**

```
[코드 리뷰 결과]

🔴 [높음] 보안 - SECRET_KEY 하드코딩
   위치: login_service.py, 7번 줄
   문제: SECRET_KEY = "your-secret-key" 가 소스코드에 노출됨
   수정:
   import os
   SECRET_KEY = os.environ.get("JWT_SECRET_KEY")
   if not SECRET_KEY:
       raise ValueError("JWT_SECRET_KEY 환경변수가 설정되지 않았습니다")

🟡 [중간] 보안 - 사용자 존재 여부 노출
   위치: get_user_by_email(), 14번 줄
   문제: "존재하지 않는 사용자" 메시지로 이메일 유효성을 공격자가 확인 가능
   수정: "이메일 또는 비밀번호가 올바르지 않습니다" 로 통일

🟡 [중간] 성능 - datetime.utcnow() deprecated
   위치: create_jwt_token(), 22번 줄
   문제: Python 3.12부터 utcnow()는 사용 중단 예정
   수정: datetime.now(timezone.utc) 사용
```

---

## 4. 세 역할을 조합하는 방법

Planner → Implementer → Reviewer 순서로 연결하면 혼자서도 팀처럼 작업할 수 있습니다.

```
# ✅ 역할 체인 프롬프트 예시

[1차 대화]
역할: Planner
"사용자 프로필 수정 API를 설계해 줘"
→ 설계 계획 받기

[2차 대화]
역할: Implementer
"위 계획으로 코드 구현해 줘" + [1차 결과 붙여넣기]
→ 코드 받기

[3차 대화]
역할: Reviewer
"이 코드 검토해 줘" + [2차 결과 붙여넣기]
→ 문제점 목록 받기

[4차 대화]
역할: Implementer
"리뷰 결과 반영해서 코드 수정해 줘" + [3차 결과 붙여넣기]
→ 최종 코드 받기
```

---

## 따라 하기 실습

### 실습 1: Planner 프롬프트 작성하기

`prompts/planner_prompt.txt` 파일을 만들고 아래 내용을 저장하세요.

```
# planner_prompt.txt

역할: 당신은 경험 많은 소프트웨어 아키텍트입니다.

임무: 할 일 목록(Todo List) 앱의 "할 일 추가" 기능을 설계해 주세요.

규칙:
- 코드를 작성하지 마세요
- 단계별 설계 계획만 작성하세요
- 각 단계에서 왜 그렇게 하는지 이유를 설명하세요
- 예상되는 위험 요소도 포함하세요

요청: POST /todos API 엔드포인트를 설계해 주세요.
```

이 프롬프트를 Claude에 입력하고 설계 계획을 받아 `outputs/plan_result.txt`에 저장하세요.

---

### 실습 2: Implementer 프롬프트 작성하기

실습 1의 결과를 활용하여 `prompts/implementer_prompt.txt` 파일을 만드세요.

```
# implementer_prompt.txt

역할: 당신은 Python FastAPI 백엔드 개발자입니다.

임무: 아래 설계 계획을 코드로 구현해 주세요.

규칙:
- 설계 계획의 순서를 반드시 따르세요
- 각 단계를 함수로 분리하세요
- 타입 힌트를 반드시 포함하세요
- 에러 처리를 빠뜨리지 마세요

설계 계획:
[실습 1에서 받은 plan_result.txt 내용 붙여넣기]
```

응답으로 받은 코드를 `outputs/todo_api.py`에 저장하세요.

---

### 실습 3: Reviewer 프롬프트 작성하기

실습 2의 코드를 검토하는 `prompts/reviewer_prompt.txt` 파일을 만드세요.

```
# reviewer_prompt.txt

역할: 당신은 보안과 코드 품질에 특화된 시니어 개발자입니다.

임무: 아래 코드를 검토하고 문제점을 보고해 주세요.

규칙:
- 문제점만 나열하세요 (칭찬 금지)
- 심각도를 [높음/중간/낮음]으로 분류하세요
- 각 문제에 수정 코드를 제안하세요
- 보안, 성능, 가독성 세 관점에서 검토하세요

검토할 코드:
[실습 2에서 받은 todo_api.py 내용 붙여넣기]
```

리뷰 결과를 `outputs/review_result.txt`에 저장하고, 지적된 문제 중 하나를 직접 수정해 보세요.

---

## 자주 하는 실수

| 실수 | 발생하는 문제 | 해결 방법 |
|---|---|---|
| 역할 없이 "코드 짜줘"라고만 함 | AI가 너무 광범위하게 응답하거나 원하는 방향과 다름 | 역할, 임무, 규칙 세 가지를 항상 명시 |
| Planner에게 코드 작성 허용 | 계획 없이 바로 구현으로 넘어가 설계가 빠짐 | `"코드를 작성하지 마세요"` 규칙을 명시 |
| Reviewer에게 "좋은 점도 말해줘" | 칭찬이 많아지고 실제 문제점을 가볍게 넘어감 | `"칭찬하지 마세요, 문제점만"` 으로 명시 |
| 이전 대화 결과를 붙여넣지 않음 | AI가 맥락 없이 새로 만들어 연속성이 끊김 | 항상 이전 단계 결과를 프롬프트에 포함 |
| 역할을 너무 많이 한 번에 부여 | `"당신은 플래너이자 구현자이자 리뷰어입니다"` → 역할 혼용으로 품질 저하 | 한 번에 하나의 역할만 부여 |
| 규칙이 모호함 | `"잘 만들어 줘"` → AI가 기준을 모름 | `"타입 힌트 포함"`, `"함수 분리"` 처럼 구체적으로 작성 |

---

## 확인 체크리스트

- [ ] Planner 역할 프롬프트에 "코드를 작성하지 마세요" 규칙이 포함되어 있다
- [ ] Implementer 프롬프트에 이전 Planner 결과가 포함되어 있다
- [ ] Reviewer 프롬프트에 심각도 분류 규칙이 명시되어 있다
- [ ] 프롬프트마다 역할(Role), 임무(Task), 규칙(Rules) 세 가지 섹션이 있다
- [ ] 실습 파일 세 개(`planner_prompt.txt`, `implementer_prompt.txt`, `reviewer_prompt.txt`)를 모두 만들었다
- [ ] Reviewer가 지적한 문제 중 최소 하나를 직접 수정해 봤다
- [ ] 역할 체인(Planner → Implementer → Reviewer)의 순서를 이해하고 설명할 수 있다

---

## 한 번 더 생각해 보기

1. **Reviewer 역할을 먼저 적용하면 어떻게 될까요?** 코드가 없는 상태에서 리뷰어를 쓰면 무슨 일이 생길까요? 어떤 순서가 왜 중요한지 생각해 보세요.

2. **같은 코드를 보안 전문가 리뷰어와 성능 전문가 리뷰어에게 각각 검토시키면 어떤 차이가 날까요?** 역할 설명에서 전문 분야를 바꾸는 것만으로 어떤 효과를 얻을 수 있을지 생각해 보세요.

3. **실제 팀 프로젝트에서 역할 기반 프롬프트를 어떻게 활용할 수 있을까요?** 팀원 각자가 서로 다른 역할 프롬프트를 맡아 AI를 사용한다면 협업이 어떻게 달라질지 상상해 보세요.

---

## 다음 장

다음 장에서는 역할 기반 프롬프트를 더 발전시켜, **멀티 에이전트 워크플로우**—여러 AI 역할이 자동으로 연결되어 동작하는 파이프라인—를 구성하는 방법을 배웁니다.