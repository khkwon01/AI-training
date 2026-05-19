## 이 장에서 배우는 것

- `with` 문이 무엇인지, 왜 사용하는지 이해한다
- 파일, 네트워크 연결 같은 자원을 안전하게 열고 닫는 방법을 배운다
- `__enter__`와 `__exit__`를 구현해서 직접 컨텍스트 매니저를 만든다
- `contextlib.contextmanager` 데코레이터로 더 간단하게 작성하는 법을 익힌다
- 예외가 발생해도 자원이 반드시 정리되는 패턴을 이해한다

---

## 먼저 쉬운 설명

파일을 열면 반드시 닫아야 합니다. 데이터베이스에 연결하면 반드시 끊어야 합니다. 문제는, 코드 중간에 오류가 나면 `close()`를 깜빡하기 쉽다는 점입니다.

도서관에서 책을 빌린다고 상상해 보세요. 읽다가 갑자기 급한 일이 생겨서 집에 그냥 두고 나왔습니다. 책은 반납이 안 된 채로 남아 있고, 다른 사람은 그 책을 빌릴 수 없게 됩니다.

프로그램에서도 똑같은 일이 생깁니다. 파일을 열어 두면 다른 프로그램이 그 파일에 접근하지 못할 수 있고, 메모리가 낭비됩니다. 이것을 **자원 누수(resource leak)** 라고 합니다.

파이썬의 `with` 문은 "이 블록을 나가는 순간, 무슨 일이 있어도 자원을 정리해 줘"라고 약속하는 장치입니다. 예외가 발생해도, 정상 종료가 되어도, 반드시 정리가 실행됩니다.

---

## 1. `with` 문 기본 사용법

가장 흔한 예시는 파일 처리입니다. `with` 없이 파일을 다루면 이런 문제가 생깁니다.

```python
# 나쁜 예시 — 오류가 나면 파일이 닫히지 않는다
파일 = open("학생명단.txt", "w")
파일.write("김철수\n")
1 / 0  # 여기서 ZeroDivisionError 발생
파일.close()  # 이 줄은 실행되지 않는다!
```

`with`를 사용하면 오류가 나도 파일이 자동으로 닫힙니다.

```python
# 좋은 예시 — 예외가 발생해도 파일이 자동으로 닫힌다
with open("학생명단.txt", "w", encoding="utf-8") as 파일:
    파일.write("김철수\n")
    파일.write("이영희\n")
    파일.write("박민준\n")

# 이 시점에서 파일은 이미 닫혀 있다
print(파일.closed)  # True
```

`as 파일` 부분은 열린 파일 객체를 `파일`이라는 변수에 담는 것입니다. `with` 블록이 끝나면 파이썬이 자동으로 `파일.close()`를 호출합니다.

여러 파일을 동시에 열 수도 있습니다.

```python
# 학생 성적을 읽어서 결과 파일에 쓰기
with open("성적원본.txt", "r", encoding="utf-8") as 입력, \
     open("성적결과.txt", "w", encoding="utf-8") as 출력:
    for 줄 in 입력:
        이름, 점수 = 줄.strip().split(",")
        등급 = "합격" if int(점수) >= 60 else "불합격"
        출력.write(f"{이름}: {등급}\n")
```

---

## 2. 컨텍스트 매니저가 내부에서 하는 일

`with` 문이 마법처럼 느껴질 수 있지만, 실제로는 두 개의 메서드를 호출합니다.

| 시점 | 호출되는 메서드 | 하는 일 |
|------|----------------|---------|
| `with` 블록 진입 시 | `__enter__()` | 자원을 준비하고 `as` 변수에 넘겨 준다 |
| `with` 블록 종료 시 | `__exit__()` | 자원을 정리한다 (예외 여부 관계없이) |

이것을 직접 확인해 보겠습니다.

```python
# 컨텍스트 매니저의 동작 순서를 눈으로 확인하기
class 작업로그:
    def __init__(self, 작업명):
        self.작업명 = 작업명

    def __enter__(self):
        print(f"[시작] {self.작업명} 작업을 시작합니다.")
        return self  # as 변수에 전달되는 객체

    def __exit__(self, 예외종류, 예외값, 트레이스백):
        if 예외종류 is None:
            print(f"[완료] {self.작업명} 작업이 정상 완료됐습니다.")
        else:
            print(f"[오류] {self.작업명} 도중 오류 발생: {예외값}")
        return False  # False = 예외를 그대로 전파한다

with 작업로그("데이터 저장") as 로그:
    print("  데이터를 저장하는 중...")
    # 여기서 작업을 수행한다
```

실행 결과:
```
[시작] 데이터 저장 작업을 시작합니다.
  데이터를 저장하는 중...
[완료] 데이터 저장 작업이 정상 완료됐습니다.
```

`__exit__`의 반환값이 중요합니다. `True`를 반환하면 예외를 삼켜서 없애고, `False`를 반환하면 예외를 그대로 밖으로 전달합니다. 특별한 이유가 없으면 `False`를 반환하는 것이 안전합니다.

---

## 3. 직접 컨텍스트 매니저 클래스 만들기

실제 업무에서 쓸 수 있는 예시를 만들어 봅니다. 임시 파일을 사용하고 작업이 끝나면 자동으로 삭제하는 컨텍스트 매니저입니다.

```python
import os

class 임시파일:
    """작업 중에만 존재하고 블록이 끝나면 자동으로 삭제되는 파일"""

    def __init__(self, 파일명):
        self.파일명 = 파일명

    def __enter__(self):
        self.파일핸들 = open(self.파일명, "w", encoding="utf-8")
        print(f"임시 파일 생성: {self.파일명}")
        return self.파일핸들

    def __exit__(self, 예외종류, 예외값, 트레이스백):
        self.파일핸들.close()
        if os.path.exists(self.파일명):
            os.remove(self.파일명)
            print(f"임시 파일 삭제: {self.파일명}")
        return False


# 사용 예시
with 임시파일("temp_보고서.txt") as f:
    f.write("이 내용은 임시로만 존재합니다.\n")
    f.write("블록이 끝나면 파일이 사라집니다.\n")

# 이 시점에서 temp_보고서.txt 파일은 존재하지 않는다
print(os.path.exists("temp_보고서.txt"))  # False
```

---

## 4. `contextlib.contextmanager`로 더 간단하게 만들기

클래스를 만드는 것이 번거롭다면 `contextlib` 모듈의 `contextmanager` 데코레이터를 사용하면 일반 함수로 컨텍스트 매니저를 만들 수 있습니다.

핵심 규칙은 하나입니다: **`yield` 앞은 `__enter__`, `yield` 뒤는 `__exit__`입니다.**

```python
from contextlib import contextmanager
import time

@contextmanager
def 실행시간측정(작업명):
    """코드 블록의 실행 시간을 측정한다"""
    시작 = time.time()
    print(f"'{작업명}' 시작...")
    try:
        yield  # 이 시점에 with 블록의 내용이 실행된다
    finally:
        종료 = time.time()
        경과 = 종료 - 시작
        print(f"'{작업명}' 완료 — {경과:.3f}초 소요")


# 사용 예시
with 실행시간측정("대용량 파일 처리"):
    # 시간이 걸리는 작업 시뮬레이션
    total = sum(range(1_000_000))
    print(f"  합계: {total}")
```

실행 결과:
```
'대용량 파일 처리' 시작...
  합계: 499999500000
'대용량 파일 처리' 완료 — 0.042초 소요
```

`try/finally` 구조를 쓰는 이유는, `yield` 이후 코드가 예외 상황에서도 반드시 실행되도록 보장하기 위해서입니다. `finally` 블록은 예외 여부와 관계없이 항상 실행됩니다.

값을 `as` 변수로 전달하고 싶다면 `yield 값`으로 씁니다.

```python
from contextlib import contextmanager

@contextmanager
def 데이터베이스연결(호스트):
    """데이터베이스 연결을 열고 자동으로 닫는다 (시뮬레이션)"""
    print(f"DB 연결 시작: {호스트}")
    연결정보 = {"호스트": 호스트, "상태": "연결됨"}  # 실제로는 DB 커넥션 객체
    try:
        yield 연결정보  # as 변수에 이 값이 들어간다
    except Exception as e:
        print(f"DB 오류 발생, 롤백 처리: {e}")
        raise
    finally:
        print(f"DB 연결 종료: {호스트}")


with 데이터베이스연결("localhost:5432") as db:
    print(f"  쿼리 실행 중 (연결 상태: {db['상태']})")
```

---

## 따라 하기 실습

### 실습 1 — 로그 파일 자동 관리

매일 날짜별로 로그 파일을 만들고, 작업이 끝나면 자동으로 닫히는 컨텍스트 매니저를 만들어 보세요.

`파일명: daily_logger.py`

```python
from contextlib import contextmanager
from datetime import date

@contextmanager
def 일별로그(접두사):
    오늘 = date.today().strftime("%Y%m%d")
    파일명 = f"{접두사}_{오늘}.log"
    파일 = open(파일명, "a", encoding="utf-8")
    print(f"로그 파일 열기: {파일명}")
    try:
        yield 파일
    finally:
        파일.close()
        print(f"로그 파일 닫기: {파일명}")


# 실행해 보세요
with 일별로그("서비스") as 로그:
    로그.write("서버 시작\n")
    로그.write("요청 처리 완료\n")
```

**확인**: 실행 후 오늘 날짜가 들어간 `.log` 파일이 생겼나요? 파일 내용을 열어서 두 줄이 들어 있는지 확인하세요.

---

### 실습 2 — 오류가 나도 정리되는지 확인하기

실습 1의 파일을 수정해서, 로그를 쓰다가 오류가 발생해도 파일이 제대로 닫히는지 확인합니다.

`파일명: daily_logger.py` (수정)

```python
# 실습 1의 코드 아래에 추가하세요

print("\n--- 오류 상황 테스트 ---")
try:
    with 일별로그("오류테스트") as 로그:
        로그.write("정상 줄\n")
        raise ValueError("의도적으로 발생시킨 오류")  # 예외 발생!
        로그.write("이 줄은 실행되지 않는다\n")
except ValueError as e:
    print(f"예외가 바깥에서 잡혔습니다: {e}")

print("프로그램은 계속 실행됩니다.")
```

**확인**: "로그 파일 닫기" 메시지가 예외 이후에도 출력되는지 보세요. 파일을 열어서 "정상 줄"만 들어 있는지 확인하세요.

---

### 실습 3 — 중첩 컨텍스트 매니저로 트랜잭션 흉내 내기

실습 1, 2를 바탕으로, 입력 파일을 읽어서 처리하고 결과를 출력 파일에 쓰는 파이프라인을 만듭니다.

`파일명: score_processor.py`

```python
from contextlib import contextmanager

@contextmanager
def 안전한파일읽기(파일명):
    try:
        f = open(파일명, "r", encoding="utf-8")
        yield f
    except FileNotFoundError:
        print(f"파일을 찾을 수 없습니다: {파일명}")
        yield None  # None을 넘겨서 호출 측에서 처리하게 한다
    finally:
        try:
            f.close()
        except UnboundLocalError:
            pass  # 파일이 열리지 않았으면 닫을 필요 없다


# 먼저 테스트용 입력 파일을 만든다
with open("학생점수.txt", "w", encoding="utf-8") as f:
    f.write("김철수,85\n이영희,92\n박민준,58\n최지수,74\n")

# 파이프라인 실행
with 안전한파일읽기("학생점수.txt") as 입력, \
     open("성적결과.txt", "w", encoding="utf-8") as 출력:
    if 입력:
        for 줄 in 입력:
            이름, 점수문자 = 줄.strip().split(",")
            점수 = int(점수문자)
            등급 = "A" if 점수 >= 90 else "B" if 점수 >= 80 else "C" if 점수 >= 70 else "F"
            출력.write(f"{이름}: {점수}점 ({등급})\n")
        print("처리 완료. 성적결과.txt를 확인하세요.")
```

**확인**: `성적결과.txt`를 열어서 4명의 이름과 등급이 올바르게 들어 있는지 확인하세요. 그 다음, `학생점수.txt`를 존재하지 않는 파일명으로 바꿔서 오류 처리가 작동하는지도 테스트해 보세요.

---

## 자주 하는 실수

| 실수 | 발생하는 오류 메시지 | 원인 | 해결 방법 |
|------|---------------------|------|-----------|
| `with` 없이 파일을 열고 close를 빠뜨림 | (오류 없음, 데이터가 저장 안 됨) | 버퍼가 플러시되지 않아 파일에 내용이 기록되지 않을 수 있다 | `with open(...)` 패턴을 항상 사용한다 |
| `__exit__`에서 `return True`를 잘못 씀 | (오류 없음, 예외가 조용히 사라짐) | `True` 반환 시 예외가 삼켜진다 | 예외를 숨겨야 하는 명확한 이유가 없으면 `return False` 사용 |
| `@contextmanager`에서 `yield`를 두 번 씀 | `RuntimeError: generator didn't stop` | `contextmanager` 함수는 `yield`가 정확히 한 번이어야 한다 | `yield`가 한 곳에만 있는지 확인한다 |
| `@contextmanager`에서 `try/finally` 없이 `yield` 사용 | (예외 발생 시 정리 코드가 실행 안 됨) | `finally` 없으면 예외 발생 시 `yield` 이후 코드가 건너뛰어진다 | `try: yield ... finally: 정리코드` 구조를 항상 지킨다 |
| `with` 블록 밖에서 `as` 변수를 사용 | `ValueError: I/O operation on closed file` | 블록을 벗어나면 파일이 이미 닫혀 있다 | 파일 작업은 반드시 `with` 블록 안에서 한다 |
| `encoding` 파라미터를 빠뜨림 | `UnicodeDecodeError: 'cp949' codec can't decode...` | Windows에서 한글 파일을 열 때 기본 인코딩이 맞지 않는다 | `open(..., encoding="utf-8")`을 명시한다 |

---

## 확인 체크리스트

- [ ] `with open("파일명") as f:` 패턴을 외우지 않고 직접 타이핑할 수 있다
- [ ] `with` 블록을 나가면 파일이 자동으로 닫힌다는 것을 이해했다
- [ ] `__enter__`는 블록 진입 시, `__exit__`는 블록 종료 시 호출됨을 말로 설명할 수 있다
- [ ] `__exit__`의 세 번째 파라미터(예외종류, 예외값, 트레이스백)가 무엇을 의미하는지 안다
- [ ] `@contextmanager` 데코레이터를 쓸 때 `yield` 앞뒤로 `try/finally`를 넣어야 하는 이유를 설명할 수 있다
- [ ] 예외가 발생해도 `finally` 블록은 반드시 실행된다는 것을 실습으로 확인했다
- [ ] 두 개의 컨텍스트 매니저를 `with A as a, B as b:` 형태로 한 줄에 쓸 수 있다
- [ ] `return False`와 `return True`의 차이를 이해했다

---

## 한 번 더 생각해 보기

1. `with` 문 없이 `try/finally`로도 파일을 안전하게 닫을 수 있습니다. 그렇다면 왜 `with` 문을 사용하는 것이 더 좋을까요? 코드의 길이와 가독성 측면에서 생각해 보세요.

2. `__exit__`에서 `return True`를 반환하면 예외가 사라집니다. 이것이 유용한 경우는 언제일까요? 반대로 예외를 삼키면 안 되는 경우는 어떤 상황일까요?

3. 지금까지 여러분이 작성한 코드 중에서 `close()`를 직접 호출하거나, 정리 코드를 잊어버렸을 것 같은 부분이 있나요? 그 코드를 `with` 문으로 바꾼다면 어떻게 달라질까요?

---

## 다음 장

다음 장에서는 파이썬의 `logging` 모듈을 사용해서 컨텍스트 매니저와 함께 구조화된 로그를 남기는 방법을 배웁니다.