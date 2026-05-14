## 이 장에서 배우는 것

- Workbook과 tiny service code 01–05가 어떤 순서로 이어지는지 파악한다.
- 각 단계에서 무엇을 만들고, 무엇이 추가되는지 한눈에 비교한다.
- 다음 확장 단계(06 이후)로 나아갈 준비를 한다.

---

## 먼저 쉬운 설명

처음 서비스 코드를 배울 때 "workbook이랑 예시 코드가 왜 따로 있지?" 하고 헷갈리는 경우가 많다.

간단하게 생각하면 이렇다.

> **Workbook = 문제지**, **Tiny service code = 완성된 답안지**

workbook에서 빈칸을 채우며 연습하고, 막히면 tiny service code를 열어서 확인한다. 두 파일은 항상 같은 번호끼리 짝이다.

---

## 1. Workbook과 Tiny Code의 짝 구조

아래 표처럼 각 번호가 1:1로 대응된다.

| Workbook 파일 | Tiny Service Code | 핵심 내용 |
|---|---|---|
| `workbook_01.py` | `tiny_service_01.py` | 서비스 클래스 뼈대 만들기 |
| `workbook_02.py` | `tiny_service_02.py` | 메서드 추가하기 |
| `workbook_03.py` | `tiny_service_03.py` | 입력값 검증 추가 |
| `workbook_04.py` | `tiny_service_04.py` | 예외 처리 추가 |
| `workbook_05.py` | `tiny_service_05.py` | 간단한 테스트 작성 |

각 단계는 앞 단계 코드를 **그대로 가져와서** 기능을 하나씩 추가하는 구조다.

```python
# tiny_service_01.py — 뼈대만 있는 서비스 클래스
class GreetingService:
    def greet(self, name: str) -> str:
        return f"안녕하세요, {name}님!"
```

---

## 2. 단계별 코드가 어떻게 자라는가

01에서 05까지 코드가 어떻게 확장되는지 핵심만 비교한다.

```python
# tiny_service_02.py — 메서드 하나 추가
class GreetingService:
    def greet(self, name: str) -> str:
        return f"안녕하세요, {name}님!"

    def farewell(self, name: str) -> str:
        return f"안녕히 가세요, {name}님!"
```

```python
# tiny_service_03.py — 입력값 검증 추가
class GreetingService:
    def greet(self, name: str) -> str:
        if not name or not name.strip():
            return "이름을 입력해 주세요."
        return f"안녕하세요, {name.strip()}님!"

    def farewell(self, name: str) -> str:
        if not name or not name.strip():
            return "이름을 입력해 주세요."
        return f"안녕히 가세요, {name.strip()}님!"
```

```python
# tiny_service_04.py — 예외 처리 추가
class GreetingService:
    def greet(self, name: str) -> str:
        if not isinstance(name, str):
            raise TypeError("name은 문자열이어야 합니다.")
        if not name.strip():
            raise ValueError("name이 비어 있습니다.")
        return f"안녕하세요, {name.strip()}님!"
```

```python
# tiny_service_05.py + test_tiny_service_05.py — 테스트 추가
import pytest
from tiny_service_05 import GreetingService

def test_greet_정상():
    svc = GreetingService()
    assert svc.greet("철수") == "안녕하세요, 철수님!"

def test_greet_빈값():
    svc = GreetingService()
    with pytest.raises(ValueError):
        svc.greet("   ")
```

---

## 3. 다음 확장 단계 미리 보기 (06 이후)

05까지 끝냈다면 다음 단계에서는 아래 기능이 하나씩 추가된다.

```python
# tiny_service_06.py 예고 — 외부 의존성 주입(Dependency Injection)
class GreetingService:
    def __init__(self, formatter):   # 외부에서 주입
        self.formatter = formatter

    def greet(self, name: str) -> str:
        name = name.strip()
        if not name:
            raise ValueError("name이 비어 있습니다.")
        return self.formatter.format(name)
```

| 단계 | 추가 개념 |
|---|---|
| 06 | 의존성 주입(DI) |
| 07 | 데이터 저장소 연결 |
| 08 | HTTP 엔드포인트 연결 |

---

## 따라 하기 실습

### 실습 1 — workbook_01 열고 뼈대 완성하기

1. `workbook_01.py`를 열면 아래처럼 빈칸이 있다.
2. `pass` 자리에 `greet` 메서드를 직접 작성한다.
3. `tiny_service_01.py`와 비교해서 맞는지 확인한다.

```python
# workbook_01.py
class GreetingService:
    def greet(self, name: str) -> str:
        pass  # ← 여기를 채워 보세요
```

### 실습 2 — workbook_03 검증 로직 채우기

1. `workbook_03.py`를 열어 `greet` 메서드에 빈 문자열 검증을 추가한다.
2. 터미널에서 `python workbook_03.py`를 실행해 결과를 확인한다.
3. `tiny_service_03.py`와 비교해 차이를 메모한다.

```bash
$ python workbook_03.py
# 기대 출력: 이름을 입력해 주세요.
```

### 실습 3 — test_tiny_service_05.py 실행하고 통과시키기

1. `tiny_service_05.py`에 `TypeError` 처리가 빠져 있다면 추가한다.
2. `pytest test_tiny_service_05.py`를 실행해 전체 테스트가 통과하는지 확인한다.
3. 실패한 테스트가 있으면 오류 메시지를 읽고 어느 줄인지 찾아 수정한다.

```bash
$ pytest test_tiny_service_05.py -v
# 기대 출력:
# test_greet_정상 PASSED
# test_greet_빈값 PASSED
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|---|---|---|
| 들여쓰기를 탭·스페이스 혼용 | `IndentationError: unexpected indent` | 파일 전체를 스페이스 4칸으로 통일한다 |
| `self` 매개변수를 빠뜨림 | `TypeError: greet() takes 1 positional argument but 2 were given` | 메서드 첫 번째 매개변수에 `self`를 추가한다 |
| `ValueError` 대신 `return`으로 처리 | 테스트가 예외를 기대하는데 통과하지 않음 | `with pytest.raises(ValueError):`가 잡을 수 있도록 `raise ValueError(...)`로 바꾼다 |
| 빈 문자열 `""` vs 공백 `"   "` 구분 안 함 | 테스트 `test_greet_빈값` 실패 | `name.strip()`으로 공백 제거 후 `not name` 체크 |
| workbook과 tiny code 번호를 혼동 | 코드가 예상과 다르게 동작 | 항상 같은 번호끼리(`workbook_03` ↔ `tiny_service_03`) 비교한다 |

---

## 확인 체크리스트

- [ ] workbook_01 ~ 05와 tiny_service_01 ~ 05가 1:1로 짝이라는 것을 설명할 수 있다.
- [ ] 01 → 05로 갈수록 어떤 기능이 추가되는지 순서대로 말할 수 있다.
- [ ] `pytest test_tiny_service_05.py`를 실행해서 모든 테스트가 통과했다.
- [ ] `ValueError`와 `TypeError`를 언제 사용하는지 구분할 수 있다.
- [ ] 06 이후 확장 단계에서 무엇이 추가되는지 한 가지 이상 설명할 수 있다.

---

## 한 번 더 생각해 보기

1. workbook과 tiny service code를 분리해 두는 이유가 무엇일까? 처음부터 답을 보면 어떤 문제가 생길까?
2. 04단계에서 예외 처리를 추가했는데, 예외를 발생시키는 것과 그냥 `return "오류 메시지"`로 돌려주는 것의 차이는 무엇일까?
3. 05단계 테스트 코드가 없다면, 내 코드에 버그가 생겼을 때 어떻게 발견할 수 있을까?

---

## 다음 장

다음 장에서는 `tiny_service_06.py`를 기반으로 **의존성 주입(Dependency Injection)** 패턴을 배우고, 서비스 코드가 외부 컴포넌트와 어떻게 협력하는지 살펴본다.