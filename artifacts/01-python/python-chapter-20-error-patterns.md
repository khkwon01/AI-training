## 이 장에서 배우는 것

- `try`, `except`, `finally` 블록의 역할과 사용법
- 파일 읽기, API 호출, 사용자 입력 등 실제 상황에서 발생하는 오류 처리
- 오류 종류를 구분해서 다르게 처리하는 방법
- 오류 메시지를 기록(로깅)하고 사용자에게 친절하게 안내하는 방법
- `raise`로 직접 오류를 발생시키는 패턴

---

## 먼저 쉬운 설명

프로그램을 만들다 보면 생각지 못한 일이 일어납니다. 파일이 없을 수도 있고, 인터넷이 끊길 수도 있고, 사용자가 숫자 대신 글자를 입력할 수도 있습니다.

오류 처리(Error Handling)는 이런 예상치 못한 상황이 생겼을 때 프로그램이 그냥 멈추는 대신, 친절하게 대응하도록 만드는 기술입니다.

마치 편의점 직원이 손님이 카드를 잘못 긁었을 때 "카드 단말기 오류입니다, 다시 시도해 주세요"라고 안내하는 것처럼, 프로그램도 문제가 생겼을 때 사용자에게 무슨 일이 일어났는지 알려줄 수 있어야 합니다.

---

## 1. try / except 기본 구조

오류가 발생할 수 있는 코드는 `try` 블록 안에 넣습니다. 오류가 실제로 발생했을 때 실행할 코드는 `except` 블록에 씁니다.

```python
# 파일: score_reader.py

def 점수_읽기(파일경로):
    try:
        with open(파일경로, "r", encoding="utf-8") as f:
            내용 = f.read()
            점수 = int(내용.strip())
            return 점수
    except FileNotFoundError:
        print(f"오류: '{파일경로}' 파일을 찾을 수 없습니다.")
        return None
    except ValueError:
        print("오류: 파일 안의 내용이 숫자가 아닙니다.")
        return None

결과 = 점수_읽기("학생점수.txt")
if 결과 is not None:
    print(f"점수: {결과}점")
```

**오류가 없을 때:**
```
점수: 95점
```

**파일이 없을 때:**
```
오류: '학생점수.txt' 파일을 찾을 수 없습니다.
```

**파일 안에 숫자가 아닌 내용이 있을 때:**
```
오류: 파일 안의 내용이 숫자가 아닙니다.
```

> **핵심 포인트:** `except` 뒤에 오류 종류를 적으면 해당 오류만 잡습니다. 오류 종류를 여러 개 쓸 수 있습니다.

---

## 2. 오류 메시지 자세히 보기

`except Exception as e` 패턴을 쓰면 실제 오류 메시지를 변수 `e`에 담아서 확인할 수 있습니다.

```python
# 파일: user_input.py

def 나이_입력받기():
    while True:
        try:
            입력값 = input("나이를 입력하세요: ")
            나이 = int(입력값)
            if 나이 < 0 or 나이 > 150:
                raise ValueError(f"{나이}는 올바른 나이 범위가 아닙니다.")
            return 나이
        except ValueError as e:
            print(f"잘못된 입력입니다: {e}")
            print("숫자로 된 나이를 다시 입력해 주세요.\n")

나이 = 나이_입력받기()
print(f"입력된 나이: {나이}세")
```

**실행 예시:**
```
나이를 입력하세요: 안녕
잘못된 입력입니다: invalid literal for int() with base 10: '안녕'
숫자로 된 나이를 다시 입력해 주세요.

나이를 입력하세요: -5
잘못된 입력입니다: -5는 올바른 나이 범위가 아닙니다.
숫자로 된 나이를 다시 입력해 주세요.

나이를 입력하세요: 28
입력된 나이: 28세
```

> **핵심 포인트:** `raise ValueError("메시지")`를 쓰면 직접 오류를 만들어 발생시킬 수 있습니다. 잘못된 값을 직접 걸러낼 때 유용합니다.

---

## 3. finally — 무조건 실행되는 블록

`finally` 블록은 오류가 났든 안 났든 **항상** 실행됩니다. 파일을 닫거나, 데이터베이스 연결을 끊거나, 임시 자원을 정리할 때 씁니다.

```python
# 파일: db_connector.py

import sqlite3

def 사용자_조회(사용자_id):
    연결 = None
    try:
        연결 = sqlite3.connect("사용자DB.sqlite")
        커서 = 연결.cursor()
        커서.execute("SELECT 이름, 이메일 FROM 사용자 WHERE id = ?", (사용자_id,))
        결과 = 커서.fetchone()
        if 결과 is None:
            raise ValueError(f"ID {사용자_id}에 해당하는 사용자가 없습니다.")
        return {"이름": 결과[0], "이메일": 결과[1]}
    except sqlite3.DatabaseError as e:
        print(f"데이터베이스 오류: {e}")
        return None
    except ValueError as e:
        print(f"조회 오류: {e}")
        return None
    finally:
        if 연결:
            연결.close()
            print("데이터베이스 연결을 닫았습니다.")  # 항상 실행됨

사용자 = 사용자_조회(42)
if 사용자:
    print(f"이름: {사용자['이름']}, 이메일: {사용자['이메일']}")
```

> **핵심 포인트:** `finally`는 오류가 났을 때도, 안 났을 때도 실행됩니다. 열었으면 반드시 닫아야 하는 자원에 사용하세요.

---

## 4. 여러 오류를 단계별로 처리하기

실제 프로젝트에서는 오류의 종류에 따라 다르게 대응해야 합니다. 구체적인 오류를 위에, 범용적인 오류를 아래에 배치하세요.

```python
# 파일: api_client.py

import json
import urllib.request
import urllib.error

def 환율_가져오기(통화코드):
    url = f"https://api.example.com/rates/{통화코드}"
    try:
        with urllib.request.urlopen(url, timeout=5) as 응답:
            데이터 = json.loads(응답.read().decode("utf-8"))
            return 데이터["rate"]

    except urllib.error.HTTPError as e:
        # HTTP 오류 (404, 500 등)
        if e.code == 404:
            print(f"오류: '{통화코드}' 통화 코드를 찾을 수 없습니다.")
        else:
            print(f"서버 오류 (HTTP {e.code}): {e.reason}")
        return None

    except urllib.error.URLError as e:
        # 네트워크 연결 오류
        print(f"네트워크 오류: 인터넷 연결을 확인해 주세요. ({e.reason})")
        return None

    except json.JSONDecodeError:
        # 응답이 JSON 형식이 아닐 때
        print("오류: 서버 응답을 읽을 수 없습니다.")
        return None

    except Exception as e:
        # 예상 못한 모든 오류 (마지막에 배치)
        print(f"알 수 없는 오류가 발생했습니다: {type(e).__name__}: {e}")
        return None

환율 = 환율_가져오기("USD")
if 환율:
    print(f"1 USD = {환율} KRW")
```

> **핵심 포인트:** `except` 블록은 위에서 아래로 순서대로 검사합니다. 범용 오류인 `Exception`은 반드시 가장 아래에 씁니다.

---

## 5. 커스텀 예외 클래스 만들기

프로젝트가 커지면 내 프로그램만의 오류 종류를 직접 만드는 것이 좋습니다. 코드를 읽는 사람이 오류의 의미를 바로 알 수 있습니다.

```python
# 파일: order_service.py

class 주문오류(Exception):
    """주문 처리 중 발생하는 기본 오류"""
    pass

class 재고부족오류(주문오류):
    def __init__(self, 상품명, 요청수량, 재고수량):
        self.상품명 = 상품명
        self.요청수량 = 요청수량
        self.재고수량 = 재고수량
        super().__init__(
            f"'{상품명}' 재고 부족: 요청 {요청수량}개, 현재 재고 {재고수량}개"
        )

class 결제오류(주문오류):
    pass


재고 = {"사과": 3, "바나나": 10}

def 주문처리(상품명, 수량):
    try:
        if 상품명 not in 재고:
            raise 주문오류(f"'{상품명}' 상품이 존재하지 않습니다.")
        if 재고[상품명] < 수량:
            raise 재고부족오류(상품명, 수량, 재고[상품명])
        재고[상품명] -= 수량
        print(f"주문 완료: {상품명} {수량}개")

    except 재고부족오류 as e:
        print(f"[재고 부족] {e}")
        print("수량을 줄이거나 다른 상품을 선택해 주세요.")

    except 주문오류 as e:
        print(f"[주문 오류] {e}")

주문처리("사과", 5)
주문처리("사과", 2)
주문처리("수박", 1)
```

**실행 결과:**
```
[재고 부족] '사과' 재고 부족: 요청 5개, 현재 재고 3개
수량을 줄이거나 다른 상품을 선택해 주세요.
주문 완료: 사과 2개
[주문 오류] '수박' 상품이 존재하지 않습니다.
```

---

## 따라 하기 실습

### 실습 1 — 성적 파일 읽기 프로그램

`성적.txt` 파일을 만들고 아래 코드를 `grade_reader.py`로 저장해 실행해 보세요.

**성적.txt 내용:**
```
홍길동,95
김철수,88
이영희,abc
박민준,72
```

```python
# 파일: grade_reader.py

def 성적_파일_읽기(파일경로):
    성적_목록 = []
    try:
        with open(파일경로, "r", encoding="utf-8") as f:
            for 줄번호, 줄 in enumerate(f, start=1):
                줄 = 줄.strip()
                if not 줄:
                    continue
                try:
                    이름, 점수_문자열 = 줄.split(",")
                    점수 = int(점수_문자열)
                    성적_목록.append({"이름": 이름, "점수": 점수})
                except ValueError:
                    print(f"  경고: {줄번호}번째 줄 형식 오류 — '{줄}' (건너뜀)")
    except FileNotFoundError:
        print(f"오류: '{파일경로}' 파일이 없습니다.")
        return []

    return 성적_목록

성적들 = 성적_파일_읽기("성적.txt")
print(f"\n총 {len(성적들)}명 읽기 성공")
for 학생 in 성적들:
    print(f"  {학생['이름']}: {학생['점수']}점")
```

**예상 출력:**
```
  경고: 3번째 줄 형식 오류 — '이영희,abc' (건너뜀)

총 3명 읽기 성공
  홍길동: 95점
  김철수: 88점
  박민준: 72점
```

---

### 실습 2 — 실습 1에 평균 계산과 오류 처리 추가

`grade_stats.py`를 새로 만들고 실습 1의 `성적_파일_읽기` 함수를 가져와서 평균을 계산하되, 성적이 0명일 때도 안전하게 처리하세요.

```python
# 파일: grade_stats.py

from grade_reader import 성적_파일_읽기

def 평균_계산(성적_목록):
    try:
        if not 성적_목록:
            raise ValueError("성적 데이터가 없어 평균을 계산할 수 없습니다.")
        총점 = sum(학생["점수"] for 학생 in 성적_목록)
        평균 = 총점 / len(성적_목록)
        return round(평균, 1)
    except ZeroDivisionError:
        # 이 경우는 위의 if로 먼저 걸러지지만, 방어적으로 작성
        return 0.0

def 보고서_출력(파일경로):
    성적들 = 성적_파일_읽기(파일경로)
    try:
        평균 = 평균_계산(성적들)
        최고점 = max(학생["점수"] for 학생 in 성적들)
        최저점 = min(학생["점수"] for 학생 in 성적들)
        print(f"\n--- 성적 보고서 ---")
        print(f"평균: {평균}점")
        print(f"최고점: {최고점}점")
        print(f"최저점: {최저점}점")
    except ValueError as e:
        print(f"보고서 생성 실패: {e}")

보고서_출력("성적.txt")
보고서_출력("없는파일.txt")  # 빈 목록으로 평균 계산 시도
```

---

### 실습 3 — 오류 로그 파일에 기록하기

실습 2를 확장해 오류가 발생할 때마다 `error.log` 파일에 날짜와 함께 기록하는 기능을 추가하세요.

```python
# 파일: grade_logger.py

import datetime
from grade_reader import 성적_파일_읽기

def 오류_기록(메시지):
    지금 = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open("error.log", "a", encoding="utf-8") as 로그파일:
        로그파일.write(f"[{지금}] {메시지}\n")

def 안전한_보고서_출력(파일경로):
    try:
        성적들 = 성적_파일_읽기(파일경로)
        if not 성적들:
            raise ValueError(f"'{파일경로}'에서 유효한 성적을 읽지 못했습니다.")
        평균 = sum(s["점수"] for s in 성적들) / len(성적들)
        print(f"평균 점수: {평균:.1f}점")
    except ValueError as e:
        오류_기록(str(e))
        print(f"처리 중 문제가 발생했습니다. error.log를 확인하세요.")
    except Exception as e:
        오류_기록(f"예상치 못한 오류: {type(e).__name__}: {e}")
        print("알 수 없는 오류가 발생했습니다.")
    finally:
        print("보고서 처리 완료.")

안전한_보고서_출력("성적.txt")
안전한_보고서_출력("없는파일.txt")
```

---

## 자주 하는 실수

| 실수 | 실제 오류 메시지 | 올바른 해결법 |
|------|----------------|--------------|
| `except:` 만 단독으로 사용 | 오류 없음 (하지만 모든 오류를 삼켜버림) | `except Exception as e:` 로 쓰고 반드시 `e`를 출력하거나 기록 |
| 구체적 오류보다 `Exception`을 위에 배치 | 오류 없음 (구체적 오류가 절대 실행 안 됨) | 구체적인 오류(`ValueError`, `FileNotFoundError`)를 항상 위에 |
| `try` 블록이 너무 큼 | 어느 줄에서 오류가 났는지 파악 불가 | 오류 가능성이 있는 한두 줄만 `try` 안에 |
| 파일을 `open()` 후 닫지 않음 | `ResourceWarning: unclosed file` | `with open(...) as f:` 구문 사용 |
| `int("abc")` 처리 안 함 | `ValueError: invalid literal for int() with base 10: 'abc'` | `try/except ValueError` 로 감싸기 |
| 없는 파일 열기 시도 | `FileNotFoundError: [Errno 2] No such file or directory: '파일명'` | `except FileNotFoundError` 로 처리 |
| 딕셔너리에 없는 키 접근 | `KeyError: '키이름'` | `except KeyError` 또는 `.get("키이름")` 사용 |
| 0으로 나누기 | `ZeroDivisionError: division by zero` | 나누기 전에 `if 분모 == 0:` 체크 |

---

## 확인 체크리스트

- [ ] `try` 블록 안에 오류가 발생할 수 있는 코드를 넣을 수 있다.
- [ ] `FileNotFoundError`, `ValueError`, `KeyError`의 차이를 설명할 수 있다.
- [ ] `except 오류종류 as e:` 패턴으로 오류 메시지를 변수에 담을 수 있다.
- [ ] `finally` 블록이 언제 실행되는지 설명할 수 있다.
- [ ] `raise ValueError("메시지")`로 직접 오류를 발생시킬 수 있다.
- [ ] 여러 `except` 블록을 쓸 때 구체적인 오류를 위에, `Exception`을 아래에 배치할 수 있다.
- [ ] `Exception` 하나로 모든 오류를 처리하는 것의 단점을 설명할 수 있다.
- [ ] 커스텀 예외 클래스를 `Exception`을 상속해 만들 수 있다.

---

## 한 번 더 생각해 보기

1. `except:` 단독 사용이 위험한 이유는 무엇일까요? 예를 들어 `Ctrl+C`로 프로그램을 종료할 때 어떤 문제가 생길지 생각해 보세요.

2. `finally`를 쓰지 않고 `try` 블록 마지막 줄에 `연결.close()`를 쓰면 어떤 상황에서 문제가 생길까요?

3. 커스텀 예외 클래스(`재고부족오류`)를 만들면 `ValueError`를 그냥 쓰는 것보다 어떤 점이 좋을까요? 오류를 잡는 코드 입장에서 생각해 보세요.

---

## 다음 장

다음 장에서는 오류를 콘솔에 출력하는 대신 `logging` 모듈을 사용해 파일과 화면에 수준별(DEBUG, INFO, WARNING, ERROR)로 체계적으로 기록하는 방법을 배웁니다.