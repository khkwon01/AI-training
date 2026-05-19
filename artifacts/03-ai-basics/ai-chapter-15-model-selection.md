## 이 장에서 배우는 것

- AI 모델(GPT, Claude, Gemini 등)의 종류와 특징 차이를 이해한다
- 코딩 작업 유형에 따라 어떤 모델을 선택하면 좋은지 판단할 수 있다
- 모델 선택이 결과물의 품질과 비용에 미치는 영향을 안다
- 실제 코딩 프로젝트에서 모델을 고르는 기준을 스스로 적용할 수 있다

---

## 먼저 쉬운 설명

요리할 때 칼을 고르듯, 코딩 작업에도 "맞는 도구"가 있습니다.

채소를 썰 때는 작은 칼이 편하고, 큰 고기를 다룰 때는 큰 칼이 필요합니다. AI 모델도 마찬가지입니다. 간단한 코드 자동완성에 최고 성능 모델을 쓰면 돈만 낭비하고, 반대로 복잡한 아키텍처 설계에 작은 모델을 쓰면 엉성한 결과가 나옵니다.

이 장을 다 읽고 나면, "이 작업엔 이 모델"이라는 감각이 자연스럽게 생깁니다.

---

## 1. AI 모델의 크기와 능력 차이

모델은 크게 **소형(Small)**, **중형(Medium)**, **대형(Large)** 으로 나뉩니다.

| 크기 | 예시 모델 | 속도 | 비용 | 복잡도 처리 |
|------|-----------|------|------|-------------|
| 소형 | Claude Haiku, GPT-4o mini | 빠름 | 저렴 | 단순 작업 |
| 중형 | Claude Sonnet, GPT-4o | 보통 | 중간 | 대부분 작업 |
| 대형 | Claude Opus, o1, o3 | 느림 | 비쌈 | 고난도 추론 |

```python
# model_selector.py
# 작업 유형에 따라 모델을 자동으로 고르는 간단한 예시

def 모델_선택(작업_유형: str) -> str:
    단순_작업 = ["코드_자동완성", "오타_수정", "주석_추가"]
    중간_작업 = ["함수_작성", "버그_수정", "리팩토링"]
    복잡_작업 = ["아키텍처_설계", "알고리즘_최적화", "보안_감사"]

    if 작업_유형 in 단순_작업:
        return "claude-haiku-4-5"          # 빠르고 저렴
    elif 작업_유형 in 중간_작업:
        return "claude-sonnet-4-6"         # 균형 잡힌 선택
    elif 작업_유형 in 복잡_작업:
        return "claude-opus-4-7"           # 최고 품질
    else:
        return "claude-sonnet-4-6"         # 기본값

print(모델_선택("버그_수정"))        # claude-sonnet-4-6
print(모델_선택("아키텍처_설계"))    # claude-opus-4-7
print(모델_선택("주석_추가"))        # claude-haiku-4-5
```

---

## 2. 코딩 작업 유형별 모델 선택 기준

작업을 네 가지 카테고리로 나눠서 생각하면 쉽습니다.

**카테고리 A — 반복·단순 작업** (소형 모델 추천)
- 변수명 바꾸기, 코드 포맷팅, 간단한 주석 생성

**카테고리 B — 일반 개발 작업** (중형 모델 추천)
- 함수 작성, 단위 테스트 생성, REST API 엔드포인트 구현

**카테고리 C — 복잡한 추론 작업** (대형 모델 추천)
- 시스템 설계, 복잡한 버그 원인 분석, 성능 병목 찾기

**카테고리 D — 긴 문서·대용량 컨텍스트** (1M 컨텍스트 모델 추천)
- 수천 줄짜리 레거시 코드 분석, 대형 코드베이스 리뷰

```python
# task_categorizer.py
import anthropic

def 코드_작업_실행(작업_설명: str, 코드: str) -> str:
    """작업 복잡도를 분석해서 알맞은 모델로 요청을 보냅니다."""

    # 키워드 기반 간단한 복잡도 판단
    복잡_키워드 = ["설계", "아키텍처", "최적화", "분석", "보안"]
    단순_키워드 = ["주석", "포맷", "이름변경", "정렬"]

    if any(k in 작업_설명 for k in 복잡_키워드):
        모델 = "claude-opus-4-7"
    elif any(k in 작업_설명 for k in 단순_키워드):
        모델 = "claude-haiku-4-5-20251001"
    else:
        모델 = "claude-sonnet-4-6"

    client = anthropic.Anthropic()

    메시지 = client.messages.create(
        model=모델,
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": f"작업: {작업_설명}\n\n코드:\n```python\n{코드}\n```"
            }
        ]
    )

    print(f"[사용된 모델: {모델}]")
    return 메시지.content[0].text


# 사용 예시
샘플_코드 = """
def 합계(numbers):
    total = 0
    for n in numbers:
        total = total + n
    return total
"""

결과 = 코드_작업_실행("이 함수에 주석을 추가해줘", 샘플_코드)
print(결과)
```

---

## 3. 비용과 속도 트레이드오프 이해하기

모델이 클수록 느리고 비쌉니다. 실제 프로젝트에서는 **비용 예산** 을 기준으로 모델을 섞어 씁니다.

```python
# cost_aware_selector.py
# 월 예산을 정해두고 작업 우선순위에 따라 모델을 배분하는 패턴

class 예산_인식_모델_선택기:
    # 토큰 1M 당 대략적인 비용 (달러, 2025년 기준 참고용)
    모델_비용표 = {
        "claude-haiku-4-5-20251001":  {"입력": 0.80,  "출력": 4.00},
        "claude-sonnet-4-6":          {"입력": 3.00,  "출력": 15.00},
        "claude-opus-4-7":            {"입력": 15.00, "출력": 75.00},
    }

    def __init__(self, 월_예산_달러: float):
        self.남은_예산 = 월_예산_달러
        self.총_사용액 = 0.0

    def 모델_추천(self, 작업_중요도: str) -> str:
        """
        작업_중요도: "높음" | "보통" | "낮음"
        예산이 부족하면 자동으로 저렴한 모델로 다운그레이드
        """
        if self.남은_예산 < 1.0:
            print("⚠️  예산 부족 — 가장 저렴한 모델로 전환합니다")
            return "claude-haiku-4-5-20251001"

        if 작업_중요도 == "높음":
            return "claude-opus-4-7"
        elif 작업_중요도 == "보통":
            return "claude-sonnet-4-6"
        else:
            return "claude-haiku-4-5-20251001"

    def 비용_차감(self, 모델명: str, 입력_토큰: int, 출력_토큰: int):
        비용 = self.모델_비용표[모델명]
        사용액 = (입력_토큰 / 1_000_000 * 비용["입력"] +
                  출력_토큰 / 1_000_000 * 비용["출력"])
        self.남은_예산 -= 사용액
        self.총_사용액 += 사용액
        print(f"이번 요청 비용: ${사용액:.4f} | 남은 예산: ${self.남은_예산:.2f}")


# 사용 예시
선택기 = 예산_인식_모델_선택기(월_예산_달러=10.0)

모델 = 선택기.모델_추천("높음")
print(f"선택된 모델: {모델}")

선택기.비용_차감(모델, 입력_토큰=500, 출력_토큰=300)
```

---

## 4. 실무에서 자주 쓰는 모델 선택 패턴

**패턴 1 — 폭포수(Waterfall):** 먼저 소형 모델로 초안을 만들고, 대형 모델로 검토한다.

**패턴 2 — 라우팅(Routing):** 요청이 들어오면 분류기가 자동으로 모델을 골라 보낸다.

**패턴 3 — 앙상블(Ensemble):** 중요한 결정은 여러 모델에 동시에 물어보고 결과를 비교한다.

```python
# routing_pattern.py
import anthropic
import re

client = anthropic.Anthropic()

def 라우팅_에이전트(사용자_요청: str) -> str:
    """
    패턴 2 — 라우팅
    먼저 가벼운 분류 모델로 작업 유형을 파악하고,
    그 결과에 따라 실제 모델을 선택합니다.
    """

    # 1단계: 소형 모델로 분류
    분류_결과 = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=50,
        messages=[{
            "role": "user",
            "content": (
                f"다음 코딩 요청의 복잡도를 한 단어로만 답하세요 "
                f"(단순/보통/복잡): '{사용자_요청}'"
            )
        }]
    )
    복잡도 = 분류_결과.content[0].text.strip()

    # 2단계: 분류 결과로 실제 모델 선택
    모델_맵 = {
        "단순": "claude-haiku-4-5-20251001",
        "보통": "claude-sonnet-4-6",
        "복잡": "claude-opus-4-7",
    }
    선택_모델 = 모델_맵.get(복잡도, "claude-sonnet-4-6")
    print(f"분류: {복잡도} → 모델: {선택_모델}")

    # 3단계: 선택된 모델로 실제 작업 수행
    결과 = client.messages.create(
        model=선택_모델,
        max_tokens=1024,
        messages=[{"role": "user", "content": 사용자_요청}]
    )
    return 결과.content[0].text


print(라우팅_에이전트("리스트를 오름차순으로 정렬하는 파이썬 코드 써줘"))
print(라우팅_에이전트("분산 시스템에서 일관성과 가용성의 트레이드오프를 설명하고 코드로 시뮬레이션해줘"))
```

---

## 따라 하기 실습

### 실습 1 — 모델 선택기 기본 버전 만들기

`model_guide/step1_basic_selector.py` 파일을 만들고 아래 코드를 작성합니다.

```python
# model_guide/step1_basic_selector.py
import anthropic

client = anthropic.Anthropic()

def 간단_질문(질문: str, 모델: str) -> str:
    응답 = client.messages.create(
        model=모델,
        max_tokens=256,
        messages=[{"role": "user", "content": 질문}]
    )
    return 응답.content[0].text

# 같은 질문을 두 모델에 보내서 결과를 비교해봅니다
질문 = "파이썬 리스트에서 중복을 제거하는 가장 간단한 방법은?"

print("=== Haiku (소형) ===")
print(간단_질문(질문, "claude-haiku-4-5-20251001"))

print("\n=== Sonnet (중형) ===")
print(간단_질문(질문, "claude-sonnet-4-6"))
```

실행 후 두 모델의 답변 길이와 설명 방식 차이를 비교해보세요.

---

### 실습 2 — 자동 라우팅 추가하기

실습 1 파일을 기반으로 `model_guide/step2_auto_routing.py` 를 만듭니다.

```python
# model_guide/step2_auto_routing.py
import anthropic

client = anthropic.Anthropic()

복잡도_모델_맵 = {
    "단순": "claude-haiku-4-5-20251001",
    "보통": "claude-sonnet-4-6",
    "복잡": "claude-opus-4-7",
}

def 자동_라우팅(요청: str) -> str:
    # 분류
    분류 = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=10,
        messages=[{
            "role": "user",
            "content": f"복잡도를 단순/보통/복잡 중 하나로만 답하세요: '{요청}'"
        }]
    ).content[0].text.strip()

    모델 = 복잡도_모델_맵.get(분류, "claude-sonnet-4-6")
    print(f"→ 분류: {분류}, 모델: {모델}")

    return client.messages.create(
        model=모델,
        max_tokens=512,
        messages=[{"role": "user", "content": 요청}]
    ).content[0].text

# 테스트
테스트_목록 = [
    "정수를 문자열로 바꾸는 파이썬 코드",
    "OAuth2 인증 흐름을 구현하는 방법",
    "마이크로서비스 간 데이터 일관성을 유지하는 아키텍처 설계",
]

for 요청 in 테스트_목록:
    print(f"\n요청: {요청}")
    print(자동_라우팅(요청))
    print("-" * 40)
```

---

### 실습 3 — 비용 추적 기능 붙이기

`model_guide/step3_cost_tracker.py` 를 만들어 실습 2에 비용 추적을 추가합니다.

```python
# model_guide/step3_cost_tracker.py
import anthropic
from dataclasses import dataclass, field

client = anthropic.Anthropic()

@dataclass
class 사용_기록:
    모델: str
    입력_토큰: int
    출력_토큰: int
    추정_비용_달러: float

내역: list[사용_기록] = []

# 토큰당 비용 (달러/1M 토큰, 참고용)
비용표 = {
    "claude-haiku-4-5-20251001": (0.80, 4.00),
    "claude-sonnet-4-6":         (3.00, 15.00),
    "claude-opus-4-7":           (15.00, 75.00),
}

def 요청_및_기록(모델: str, 내용: str) -> str:
    응답 = client.messages.create(
        model=모델,
        max_tokens=512,
        messages=[{"role": "user", "content": 내용}]
    )

    입력 = 응답.usage.input_tokens
    출력 = 응답.usage.output_tokens
    단가입력, 단가출력 = 비용표.get(모델, (3.00, 15.00))
    비용 = (입력 / 1_000_000 * 단가입력) + (출력 / 1_000_000 * 단가출력)

    내역.append(사용_기록(모델, 입력, 출력, 비용))
    return 응답.content[0].text

def 비용_리포트():
    print("\n===== 비용 리포트 =====")
    for i, r in enumerate(내역, 1):
        print(f"{i}. {r.모델} | 입력 {r.입력_토큰} | 출력 {r.출력_토큰} | ${r.추정_비용_달러:.5f}")
    총합 = sum(r.추정_비용_달러 for r in 내역)
    print(f"총 추정 비용: ${총합:.5f}")

# 여러 작업 실행
요청_및_기록("claude-haiku-4-5-20251001", "변수명을 snake_case로 바꾸는 예시")
요청_및_기록("claude-sonnet-4-6", "파이썬으로 간단한 REST API 엔드포인트 작성")
요청_및_기록("claude-opus-4-7", "이진 탐색 트리의 삽입 알고리즘을 최적화하는 방법")

비용_리포트()
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|----------------------|-----------|
| 없는 모델명을 입력 | `404 model_not_found` | 공식 문서에서 정확한 모델 ID 확인 (`claude-sonnet-4-6` 등) |
| 항상 가장 큰 모델만 사용 | 비용 폭탄, 응답 느림 | 작업 복잡도에 맞게 소형·중형 모델 혼용 |
| 항상 가장 작은 모델만 사용 | 답변이 피상적이거나 코드에 버그 포함 | 복잡한 작업엔 중형 이상 모델 사용 |
| `max_tokens` 를 너무 작게 설정 | 응답이 중간에 잘림 | 작업 특성에 맞게 충분한 값 설정 (최소 512) |
| API 키 없이 실행 | `AuthenticationError: No API key provided` | `ANTHROPIC_API_KEY` 환경변수 설정 확인 |
| 모델명에 오타 | `InvalidRequestError` | 모델명은 대소문자·하이픈 포함 정확히 입력 |
| 분류 모델 결과를 믿고 무조건 따름 | 엉뚱한 모델이 선택됨 | 분류 결과 로그를 출력해 수동으로 확인하는 단계 추가 |

---

## 확인 체크리스트

- [ ] 소형·중형·대형 모델의 특징 차이를 설명할 수 있다
- [ ] 코딩 작업 유형(단순/보통/복잡)에 따라 어떤 모델을 골라야 하는지 말할 수 있다
- [ ] `anthropic.Anthropic()` 클라이언트를 생성하고 `messages.create()` 를 호출할 수 있다
- [ ] 라우팅 패턴의 두 단계(분류 → 실행)를 직접 코드로 작성할 수 있다
- [ ] `usage.input_tokens` 와 `usage.output_tokens` 로 토큰 사용량을 확인할 수 있다
- [ ] 실습 3의 비용 리포트 출력 결과를 보고 총 비용을 계산할 수 있다
- [ ] `AuthenticationError` 가 발생했을 때 원인과 해결 방법을 안다

---

## 한 번 더 생각해 보기

1. 여러분이 만들고 싶은 프로젝트에서 가장 자주 쓸 것 같은 코딩 작업은 무엇인가요? 그 작업엔 어떤 크기의 모델이 적합할까요?

2. 라우팅 패턴에서 "분류"를 담당하는 소형 모델이 틀린 분류를 내놓으면 어떤 문제가 생길까요? 이를 방지하려면 어떤 안전장치를 추가할 수 있을까요?

3. 비용이 전혀 상관없다면 항상 가장 큰 모델을 쓰는 게 나을까요? 속도나 응답 일관성 측면에서 대형 모델이 오히려 불리한 상황이 있을까요?

---

## 다음 장

다음 장에서는 선택한 모델에 **프롬프트 캐싱(Prompt Caching)** 을 적용해 반복 요청의 비용을 최대 90%까지 줄이는 방법을 배웁니다.