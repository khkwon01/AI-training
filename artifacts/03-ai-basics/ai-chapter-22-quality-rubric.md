## 이 장에서 배우는 것

- AI가 생성한 코드를 어떤 기준으로 평가해야 하는지 이해한다
- 품질 채점 루브릭(rubric)이 무엇인지, 왜 필요한지 설명할 수 있다
- 기능 정확성, 가독성, 보안, 오류 처리, 테스트 용이성 등 5가지 품질 축을 적용할 수 있다
- Python 코드로 간단한 자동 채점기를 직접 만들어 볼 수 있다
- AI 코드를 받아들이기 전에 스스로 체크리스트를 활용할 수 있다

---

## 먼저 쉬운 설명

GPT나 Claude 같은 AI에게 "로그인 함수 만들어 줘"라고 부탁하면 코드를 즉시 내줍니다. 그런데 그 코드가 **정말 좋은 코드인지** 어떻게 알 수 있을까요?

선생님이 학생의 글쓰기를 채점할 때 "내용 40점, 문법 30점, 창의성 30점"처럼 기준표를 씁니다. 코드도 마찬가지입니다. **채점 루브릭(rubric)** 이란 바로 이 기준표입니다.

AI가 생성한 코드를 무조건 믿고 붙여 넣으면 보안 취약점이 숨어 있거나, 팀원이 읽기 어려운 코드가 서비스에 들어갈 수 있습니다. 루브릭을 갖추면 "이 코드는 기능은 되지만 보안이 0점이라 못 쓴다"처럼 **근거 있는 판단**을 내릴 수 있습니다.

---

## 1. 품질 채점 루브릭이란 무엇인가

루브릭은 **여러 평가 항목에 점수 범위와 기준을 명시한 표**입니다.

아래는 AI 코드 평가에 쓸 5가지 축(axis)과 각 점수 구간의 의미입니다.

| 평가 축 | 0점 | 1점 | 2점 | 3점 |
|---|---|---|---|---|
| 기능 정확성 | 실행 불가 | 일부 케이스만 통과 | 대부분 통과 | 엣지 케이스 포함 전부 통과 |
| 가독성 | 이름·들여쓰기 엉망 | 변수명만 이상함 | 읽을 수 있음 | 변수·함수명이 의도를 설명함 |
| 보안 | SQL 인젝션 등 치명적 취약점 | 검증 누락 | 기본 검증 있음 | OWASP 기준 충족 |
| 오류 처리 | try/except 없음 | 모든 예외를 묵살 | 예외를 잡아 로그 출력 | 예외 종류별 처리 + 사용자 메시지 |
| 테스트 용이성 | 전역 상태 의존 | 하드코딩된 값 | 일부 분리 가능 | 순수 함수, 의존성 주입 가능 |

```python
# rubric_definition.py

# 각 축의 이름과 최대 점수를 딕셔너리로 정의합니다.
RUBRIC_AXES = {
    "기능_정확성": 3,
    "가독성": 3,
    "보안": 3,
    "오류_처리": 3,
    "테스트_용이성": 3,
}

TOTAL_MAX = sum(RUBRIC_AXES.values())  # 15점 만점


def 총점_계산(점수_딕셔너리: dict[str, int]) -> float:
    """0~100 사이의 정규화된 점수를 반환합니다."""
    raw = sum(점수_딕셔너리.values())
    return round(raw / TOTAL_MAX * 100, 1)


if __name__ == "__main__":
    예시_점수 = {
        "기능_정확성": 3,
        "가독성": 2,
        "보안": 1,
        "오류_처리": 2,
        "테스트_용이성": 1,
    }
    print(f"총점: {총점_계산(예시_점수)} / 100")
    # 출력: 총점: 60.0 / 100
```

---

## 2. 기능 정확성 — 코드가 실제로 작동하는가

AI 코드가 **모든 입력 케이스**에서 올바른 결과를 내야 비로소 3점입니다. 흔히 "행복 경로(happy path)"만 작동하고 엣지 케이스에서 터집니다.

```python
# score_correctness.py
# AI가 생성한 나누기 함수를 평가하는 예시입니다.


# ── AI가 생성한 코드 (채점 대상) ──────────────────────
def ai_divide(a: float, b: float) -> float:
    return a / b          # 보안·오류 처리 없이 단순 나눗셈


# ── 테스트 케이스로 기능 정확성 채점 ─────────────────
def 기능_정확성_채점(함수) -> int:
    케이스들 = [
        ((10, 2), 5.0),       # 일반 케이스
        ((0, 5), 0.0),        # 분자가 0
        ((-9, 3), -3.0),      # 음수
        ((7, 0), None),       # ← 분모가 0: ZeroDivisionError 기대
    ]

    통과 = 0
    for (a, b), 기대값 in 케이스들:
        try:
            결과 = 함수(a, b)
            if 기대값 is None:
                print(f"  FAIL ({a}/{b}): 예외가 발생해야 하는데 {결과} 반환")
            elif abs(결과 - 기대값) < 1e-9:
                통과 += 1
            else:
                print(f"  FAIL ({a}/{b}): {결과} ≠ {기대값}")
        except ZeroDivisionError:
            if 기대값 is None:
                통과 += 1      # 예외가 발생하길 기대했으므로 통과
            else:
                print(f"  FAIL ({a}/{b}): 예기치 않은 ZeroDivisionError")

    # 0 통과 → 0점, 1~2 → 1점, 3 → 2점, 4 → 3점
    if 통과 == len(케이스들):
        return 3
    elif 통과 >= len(케이스들) * 0.75:
        return 2
    elif 통과 >= 1:
        return 1
    return 0


점수 = 기능_정확성_채점(ai_divide)
print(f"기능 정확성: {점수}/3")
# 출력: FAIL (7/0): 예외가 발생해야 하는데 ...
#       기능 정확성: 2/3
```

> **왜 2/3인가?** `ai_divide`는 0으로 나눌 때 자체적으로 처리하지 않고 Python이 자동으로 `ZeroDivisionError`를 던집니다. 우리 테스트는 이 예외를 기대값으로 허용했으므로 4개 중 3개가 통과합니다.

---

## 3. 가독성 — 팀원이 내일도 이해할 수 있는가

가독성 채점은 **이름, 길이, 주석, 들여쓰기** 네 가지를 점검합니다. `ast` 모듈로 코드를 분석하면 자동화할 수 있습니다.

```python
# score_readability.py
import ast


# ── AI가 생성한 코드 문자열 ────────────────────────────
AI_코드 = """
def f(x,y,z):
    r=x*y
    if r>z: return r-z
    else: return 0
"""

# ── 좋은 코드 예시 ─────────────────────────────────────
좋은_코드 = """
def 할인_후_가격(정가: int, 수량: int, 할인_한도: int) -> int:
    합계 = 정가 * 수량
    if 합계 > 할인_한도:
        return 합계 - 할인_한도
    return 0
"""


def 가독성_채점(소스코드: str) -> int:
    tree = ast.parse(소스코드)
    짧은_이름_수 = 0
    총_이름_수 = 0

    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.arg, ast.Name)):
            이름 = getattr(node, "name", None) or getattr(node, "arg", None) or getattr(node, "id", None)
            if 이름 and not 이름.startswith("_"):
                총_이름_수 += 1
                if len(이름) <= 2:          # 한 글자·두 글자 이름은 의미 불명
                    짧은_이름_수 += 1

    if 총_이름_수 == 0:
        return 0

    짧은_비율 = 짧은_이름_수 / 총_이름_수
    if 짧은_비율 > 0.5:
        return 0
    elif 짧은_비율 > 0.25:
        return 1
    elif 짧은_비율 > 0.0:
        return 2
    return 3


print(f"AI 코드 가독성: {가독성_채점(AI_코드)}/3")      # 출력: 0/3
print(f"좋은 코드 가독성: {가독성_채점(좋은_코드)}/3")  # 출력: 3/3
```

---

## 4. 보안 — 악의적인 입력이 들어오면 어떻게 되는가

AI는 종종 **SQL 인젝션, 하드코딩된 비밀번호, 입력 검증 누락** 같은 취약점을 무의식적으로 만듭니다. 패턴 매칭으로 간단히 탐지할 수 있습니다.

```python
# score_security.py
import re

# 보안 위험 패턴 목록 (패턴, 위험도, 설명)
위험_패턴 = [
    (r'password\s*=\s*["\'][^"\']+["\']', "치명적", "비밀번호 하드코딩"),
    (r'execute\([^?]',                    "치명적", "SQL 인젝션 위험 (파라미터화 미사용)"),
    (r'eval\(',                           "높음",   "eval() 사용"),
    (r'shell=True',                       "높음",   "shell=True 서브프로세스"),
    (r'#\s*TODO.*보안',                    "낮음",   "보안 관련 미완성 TODO"),
]


def 보안_채점(소스코드: str) -> tuple[int, list[str]]:
    발견된_문제: list[str] = []
    치명적_수 = 0

    for 패턴, 위험도, 설명 in 위험_패턴:
        if re.search(패턴, 소스코드, re.IGNORECASE):
            발견된_문제.append(f"[{위험도}] {설명}")
            if 위험도 == "치명적":
                치명적_수 += 1

    if 치명적_수 >= 1:
        점수 = 0
    elif len(발견된_문제) >= 2:
        점수 = 1
    elif len(발견된_문제) == 1:
        점수 = 2
    else:
        점수 = 3

    return 점수, 발견된_문제


# ── 테스트 ─────────────────────────────────────────────
취약한_코드 = """
import sqlite3

def 사용자_검색(이름):
    password = "admin1234"          # 하드코딩
    conn = sqlite3.connect("db.sqlite3")
    conn.execute("SELECT * FROM users WHERE name = '" + 이름 + "'")
"""

점수, 문제들 = 보안_채점(취약한_코드)
for 문제 in 문제들:
    print(f"  ⚠️  {문제}")
print(f"보안 점수: {점수}/3")
# 출력:
#   ⚠️  [치명적] 비밀번호 하드코딩
#   ⚠️  [치명적] SQL 인젝션 위험 (파라미터화 미사용)
#   보안 점수: 0/3
```

---

## 5. 오류 처리와 테스트 용이성 — 마지막 두 축

**오류 처리**: `except Exception: pass` 처럼 모든 예외를 조용히 삼키는 코드는 1점, 각 예외 종류를 구분하고 사용자 메시지까지 있으면 3점입니다.

**테스트 용이성**: 함수가 전역 변수나 파일 시스템에 직접 의존하면 단위 테스트를 짜기 어렵습니다. 의존성을 파라미터로 받으면 3점입니다.

```python
# score_error_testability.py
import ast


def 오류_처리_채점(소스코드: str) -> int:
    """except 절의 품질을 분석합니다."""
    tree = ast.parse(소스코드)
    except_절들 = [
        node for node in ast.walk(tree)
        if isinstance(node, ast.ExceptHandler)
    ]

    if not except_절들:
        return 0    # try/except 자체가 없음

    묵살_수 = sum(
        1 for h in except_절들
        if not h.body or (
            len(h.body) == 1
            and isinstance(h.body[0], ast.Pass)
        )
    )
    if 묵살_수 == len(except_절들):
        return 1    # 전부 pass로 묵살
    if 묵살_수 > 0:
        return 2    # 일부 묵살
    return 3        # 모두 처리


# ── 나쁜 예: except pass ──────────────────────────────
나쁜_예 = """
def 파일_읽기(경로):
    try:
        with open(경로) as f:
            return f.read()
    except Exception:
        pass
"""

# ── 좋은 예: 예외 종류별 처리 ────────────────────────
좋은_예 = """
def 파일_읽기(경로: str) -> str:
    try:
        with open(경로, encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError:
        raise FileNotFoundError(f"파일을 찾을 수 없습니다: {경로}")
    except PermissionError:
        raise PermissionError(f"읽기 권한이 없습니다: {경로}")
"""

print(f"나쁜 오류 처리: {오류_처리_채점(나쁜_예)}/3")    # 출력: 1/3
print(f"좋은 오류 처리: {오류_처리_채점(좋은_예)}/3")    # 출력: 3/3
```

---

## 6. 채점기 통합 — 모든 축을 하나로 묶기

앞서 만든 함수들을 하나의 `채점기`로 합칩니다. 이 모듈이 이 장의 최종 결과물입니다.

```python
# ai_code_scorer.py
from rubric_definition import RUBRIC_AXES, 총점_계산
from score_correctness import 기능_정확성_채점, ai_divide
from score_readability import 가독성_채점
from score_security import 보안_채점
from score_error_testability import 오류_처리_채점


def 전체_채점(소스코드: str, 테스트_함수=None) -> dict:
    결과: dict[str, int] = {}

    # 1. 기능 정확성 (테스트 함수가 주어졌을 때만)
    if 테스트_함수:
        결과["기능_정확성"] = 기능_정확성_채점(테스트_함수)
    else:
        결과["기능_정확성"] = -1   # -1 = 측정 불가

    # 2. 가독성
    결과["가독성"] = 가독성_채점(소스코드)

    # 3. 보안
    결과["보안"], 보안_문제들 = 보안_채점(소스코드)

    # 4. 오류 처리
    결과["오류_처리"] = 오류_처리_채점(소스코드)

    # 5. 테스트 용이성 (이 예시에서는 간단히 고정 2점)
    결과["테스트_용이성"] = 2

    # 보안 문제 상세 출력
    if 보안_문제들:
        print("\n[보안 경고]")
        for 문제 in 보안_문제들:
            print(f"  - {문제}")

    # 최종 점수 (-1 항목 제외)
    측정된_점수 = {k: v for k, v in 결과.items() if v >= 0}
    결과["총점"] = 총점_계산(측정된_점수)
    return 결과


if __name__ == "__main__":
    import json
    샘플_코드 = open("sample_ai_code.py").read()
    리포트 = 전체_채점(샘플_코드, 테스트_함수=ai_divide)
    print(json.dumps(리포트, ensure_ascii=False, indent=2))
```

---

## 따라 하기 실습

### 실습 1 — 루브릭 정의 파일 만들기

`rubric_definition.py` 파일을 새로 만들고 아래 내용을 그대로 입력한 뒤 실행하세요.

```python
# rubric_definition.py

RUBRIC_AXES = {
    "기능_정확성": 3,
    "가독성": 3,
    "보안": 3,
    "오류_처리": 3,
    "테스트_용이성": 3,
}

TOTAL_MAX = sum(RUBRIC_AXES.values())


def 총점_계산(점수_딕셔너리: dict[str, int]) -> float:
    raw = sum(점수_딕셔너리.values())
    return round(raw / TOTAL_MAX * 100, 1)


if __name__ == "__main__":
    테스트 = {"기능_정확성": 3, "가독성": 2, "보안": 1, "오류_처리": 2, "테스트_용이성": 1}
    print(총점_계산(테스트))   # 기대 출력: 60.0
```

**확인**: `python rubric_definition.py` 를 실행하면 `60.0`이 출력되어야 합니다.

---

### 실습 2 — AI에게 코드를 받아 채점해 보기

1. ChatGPT나 Claude에게 `"Python으로 두 수를 더하는 함수를 만들어 줘"`라고 요청합니다.
2. 받은 코드를 `sample_ai_code.py`에 저장합니다.
3. 아래 채점 스크립트를 `quick_score.py`로 저장하고 실행합니다.

```python
# quick_score.py
from score_readability import 가독성_채점
from score_security import 보안_채점
from score_error_testability import 오류_처리_채점

소스 = open("sample_ai_code.py", encoding="utf-8").read()

가독성 = 가독성_채점(소스)
보안, 경고 = 보안_채점(소스)
오류 = 오류_처리_채점(소스)

print(f"가독성   : {가독성}/3")
print(f"보안     : {보안}/3  경고={경고}")
print(f"오류 처리: {오류}/3")
```

**예상 결과**: 간단한 덧셈 함수라면 보안·오류 처리 모두 낮게 나오는 경우가 많습니다. 점수가 낮다고 나쁜 것은 아닙니다. **이유를 이해하는 것**이 목적입니다.

---

### 실습 3 — 점수를 높이도록 코드를 수정하기

실습 2에서 받은 AI 코드를 직접 수정해 오류 처리 점수를 1점에서 3점으로 올려 보세요.

```python
# sample_ai_code_improved.py

# ── 원래 AI 코드 (오류 처리 없음) ──
# def 더하기(a, b):
#     return a + b

# ── 수정된 코드 ──────────────────────
def 더하기(a: float, b: float) -> float:
    """두 수를 더합니다. 숫자가 아닌 입력은 TypeError를 발생시킵니다."""
    if not isinstance(a, (int, float)):
        raise TypeError(f"a는 숫자여야 합니다. 받은 값: {type(a).__name__}")
    if not isinstance(b, (int, float)):
        raise TypeError(f"b는 숫자여야 합니다. 받은 값: {type(b).__name__}")
    return a + b
```

`quick_score.py`에서 `sample_ai_code.py` 대신 `sample_ai_code_improved.py`를 읽어 점수가 오르는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| `from rubric_definition import 총점_계산` 했는데 `ModuleNotFoundError` | `ModuleNotFoundError: No module named 'rubric_definition'` | `rubric_definition.py`가 같은 폴더에 있는지 확인. `python -m` 없이 실행하면 경로가 달라집니다. |
| `ast.parse(소스코드)`가 `SyntaxError` 발생 | `SyntaxError: invalid syntax` | AI가 생성한 코드에 문법 오류가 있음. 채점 전에 `python -m py_compile sample_ai_code.py`로 문법 검사를 먼저 하세요. |
| 점수가 전부 0점 나옴 | 출력: `기능_정확성: 0/3 ...` | `RUBRIC_AXES`의 키 이름과 채점 딕셔너리의 키 이름이 다름. 띄어쓰기 대신 `_`를 일관되게 쓰세요. |
| `except Exception: pass`인데 오류처리 2점이 나옴 | 기대 1점인데 2점 출력 | `pass` 노드 탐지 로직에서 빈 함수 바디를 잘못 읽은 경우. `ast.Pass`가 아닌 `ast.Expr`로 저장된 `pass`가 있는지 확인하세요. |
| 한글 함수명 `ast.parse` 오류 | `SyntaxError: invalid character` | Python 3.0 이상에서는 한글 식별자를 지원하지만, 파일 첫 줄에 `# -*- coding: utf-8 -*-` 또는 파일 저장 인코딩이 UTF-8인지 확인하세요. |
| `open("sample_ai_code.py")` 가 `UnicodeDecodeError` | `UnicodeDecodeError: 'cp949' codec can't decode` | Windows에서 자주 발생. `open(..., encoding="utf-8")`을 명시하세요. |

---

## 확인 체크리스트

- [ ] `RUBRIC_AXES` 딕셔너리의 5가지 축 이름을 보지 않고 말할 수 있다
- [ ] `총점_계산` 함수에 점수 딕셔너리를 넣으면 0~100 점수가 나오는 이유를 설명할 수 있다
- [ ] AI 코드에서 SQL 인젝션 위험을 패턴으로 탐지하는 원리를 설명할 수 있다
- [ ] `except Exception: pass`가 왜 1점인지, 어떻게 고쳐야 3점이 되는지 코드로 보여줄 수 있다
- [ ] `ast.parse()`가 무슨 역할을 하는지 한 문장으로 설명할 수 있다
- [ ] `sample_ai_code.py`를 실습 2에서 직접 만들고 채점 스크립트를 실행했다
- [ ] 실습 3에서 오류 처리 점수를 1점 이상 올리는 코드 수정을 완료했다

---

## 한 번 더 생각해 보기

1. **루브릭의 가중치**: 이 장에서는 5가지 축에 동일한 가중치(각 3점)를 줬습니다. 하지만 금융 서비스라면 보안을 두 배로, 프로토타입이라면 가독성을 더 낮게 잡아야 할 수 있습니다. 여러분이 일하는 또는 배우고 싶은 분야에서는 어떤 축이 가장 중요할까요? 이유도 함께 생각해 보세요.

2. **자동화의 한계**: `가독성_채점` 함수는 이름 길이만 봤습니다. `x`라는 이름이지만 수학 공식을 표현하는 코드에서는 완전히 합리적입니다. 자동 채점기가 잘못 판단하는 사례를 두 가지 이상 떠올려 보고, 이를 개선하려면 어떤 규칙을 추가해야 할지 메모해 보세요.

3. **팀 기준 만들기**: 팀마다 코드 스타일이 다릅니다. 만약 여러분이 팀의 루브릭 담당자라면, 팀원들과 어떤 과정을 거쳐 기준을 합의할까요? "다수결"과 "전문가 결정" 중 어느 방식이 더 좋을지, 그 이유를 생각해 보세요.

---

## 다음 장

다음 장에서는 이 채점 루브릭을 GitHub Actions CI 파이프라인에 연결해, Pull Request가 올라올 때마다 AI 생성 코드를 자동으로 채점하고 리포트를 코멘트로 달아주는 워크플로를 만들어 봅니다.