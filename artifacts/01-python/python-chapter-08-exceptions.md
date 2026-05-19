# Chapter 08: 예외 처리 (Exception Handling)

## 이 장에서 배우는 것

- 예외가 왜 필요하고 어떤 상황에서 발생하는지
- `try / except / else / finally` 4가지 구조의 역할과 차이
- 자주 만나는 예외 5가지와 코드 예시
- 예외를 잡아서 다시 던지는 패턴 (re-raise)
- 나만의 커스텀 예외 만들기 기초
- 실습 3개: 점수 입력 검증, 파일 안전하게 읽기, 복합 예외 처리

---

## 왜 예외 처리가 필요한가

프로그램은 항상 개발자가 예상한 대로만 실행되지 않는다.

예를 들어 이런 상황을 생각해 보자.

- 사용자가 숫자를 입력해야 하는 곳에 "안녕"이라고 입력했다
- 읽으려는 파일이 삭제되어 있다
- 서버에서 데이터를 받으려 했는데 인터넷이 끊겼다
- 딕셔너리에서 없는 키를 조회했다

이런 상황이 발생하면 Python은 프로그램 실행을 즉시 멈추고 오류 메시지를 출력한다. 이것을 **예외(Exception)**라고 한다.

예외 처리를 배우지 않으면 사용자가 조금만 이상하게 입력해도 프로그램이 뻗어버린다. 반대로 예외 처리를 잘 하면:

- 사용자에게 친절한 안내 메시지를 보여줄 수 있다
- 문제가 생겨도 프로그램이 계속 동작할 수 있다
- 어디서 어떤 문제가 생겼는지 기록할 수 있다

---

## 1. 예외 처리의 4가지 구조: try / except / else / finally

Python의 예외 처리는 4개의 키워드로 이루어진다. 각각의 역할이 다르다.

```
try:
    # 오류가 날 수도 있는 코드
except 예외종류:
    # 오류가 났을 때 실행되는 코드
else:
    # 오류가 없었을 때만 실행되는 코드
finally:
    # 오류 여부와 관계없이 항상 실행되는 코드
```

### try

오류가 날 수도 있는 코드를 이 블록 안에 넣는다. Python은 이 블록을 실행하다가 오류가 생기면 즉시 멈추고 `except` 블록으로 이동한다.

### except

`try` 블록에서 오류가 발생했을 때 실행된다. 어떤 종류의 오류를 잡을지 지정할 수 있다.

### else

`try` 블록이 오류 없이 완료됐을 때만 실행된다. "성공했을 때 처리"를 분리해서 코드를 깔끔하게 만든다.

### finally

오류가 났든 안 났든 **항상** 실행된다. 파일을 닫거나 데이터베이스 연결을 끊는 등 반드시 해야 하는 정리 작업에 쓴다.

### 전체 구조 예시

```python
try:
    number = int(input("숫자를 입력하세요: "))
    result = 100 / number
except ValueError:
    print("숫자 형식이 아닙니다.")
except ZeroDivisionError:
    print("0으로 나눌 수 없습니다.")
else:
    print(f"결과: {result}")
finally:
    print("입력 처리를 마쳤습니다.")
```

실행 흐름을 정리하면 이렇다:

| 상황 | try | except | else | finally |
|------|-----|--------|------|---------|
| "abc" 입력 | 오류 발생 | ValueError 실행 | 건너뜀 | 실행 |
| "0" 입력 | 오류 발생 | ZeroDivisionError 실행 | 건너뜀 | 실행 |
| "5" 입력 | 정상 완료 | 건너뜀 | 실행 | 실행 |

---

## 2. 자주 만나는 예외 5가지

### ValueError: 잘못된 값

값의 형식이 맞지 않을 때 발생한다. 가장 자주 만나는 예외다.

```python
# 오류 발생 상황
number = int("안녕")  # "안녕"을 숫자로 바꿀 수 없다

# 에러 메시지
# ValueError: invalid literal for int() with base 10: '안녕'

# 처리 방법
try:
    number = int(input("숫자: "))
    print(f"입력한 숫자: {number}")
except ValueError:
    print("숫자만 입력해 주세요.")
```

### TypeError: 잘못된 타입

서로 맞지 않는 타입으로 연산을 시도할 때 발생한다.

```python
# 오류 발생 상황
result = "나이: " + 25  # 문자열과 숫자를 + 로 연결할 수 없다

# 에러 메시지
# TypeError: can only concatenate str (not "int") to str

# 처리 방법
try:
    age = 25
    message = "나이: " + str(age)  # str()로 변환해야 한다
    print(message)
except TypeError:
    print("타입이 맞지 않습니다.")
```

### KeyError: 없는 딕셔너리 키

딕셔너리에 없는 키를 조회할 때 발생한다.

```python
# 오류 발생 상황
student = {"name": "김철수", "score": 85}
print(student["grade"])  # "grade" 키가 없다

# 에러 메시지
# KeyError: 'grade'

# 처리 방법
try:
    print(student["grade"])
except KeyError:
    print("해당 항목이 없습니다.")

# 더 간단한 방법: .get()은 KeyError 없이 None 반환
grade = student.get("grade", "없음")  # 키가 없으면 "없음" 반환
print(grade)
```

### FileNotFoundError: 파일 없음

존재하지 않는 파일을 열려고 할 때 발생한다.

```python
# 오류 발생 상황
with open("data.txt", "r") as f:
    content = f.read()

# 에러 메시지
# FileNotFoundError: [Errno 2] No such file or directory: 'data.txt'

# 처리 방법
try:
    with open("data.txt", "r", encoding="utf-8") as f:
        content = f.read()
    print(content)
except FileNotFoundError:
    print("파일을 찾을 수 없습니다. 파일 이름을 확인해 주세요.")
```

### ZeroDivisionError: 0으로 나누기

숫자를 0으로 나누려 할 때 발생한다.

```python
# 오류 발생 상황
result = 10 / 0

# 에러 메시지
# ZeroDivisionError: division by zero

# 처리 방법
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("0으로 나눌 수 없습니다.")
        return None

print(safe_divide(10, 2))  # 5.0
print(safe_divide(10, 0))  # None (오류 메시지 출력 후)
```

---

## 3. 여러 예외를 동시에 처리하기

하나의 `try` 블록에서 여러 종류의 예외를 각각 처리할 수 있다.

```python
def process_input(text):
    try:
        number = int(text)
        result = 100 / number
        return result
    except ValueError:
        print(f"'{text}'는 숫자로 변환할 수 없습니다.")
    except ZeroDivisionError:
        print("0으로 나눌 수 없습니다.")
    except Exception as e:
        # 예상하지 못한 다른 오류가 생기면 이쪽으로 온다
        print(f"예상치 못한 오류: {e}")
    return None

process_input("abc")   # ValueError
process_input("0")     # ZeroDivisionError
process_input("4")     # 정상: 25.0
```

`except Exception as e`는 마지막 안전망 역할을 한다. 여기서 `e`에는 실제 오류 메시지가 들어있다.

---

## 4. 예외 정보 활용하기

예외 객체를 `as e`로 받으면 오류 메시지를 직접 출력하거나 로그에 기록할 수 있다.

```python
try:
    number = int("hello")
except ValueError as e:
    print(f"오류 종류: {type(e).__name__}")
    print(f"오류 내용: {e}")

# 출력:
# 오류 종류: ValueError
# 오류 내용: invalid literal for int() with base 10: 'hello'
```

---

## 5. 예외를 잡아서 다시 던지기 (re-raise)

예외를 처리하되, 상위 코드에도 오류가 발생했음을 알려야 할 때 사용하는 패턴이다.

```python
def load_config(filename):
    try:
        with open(filename, "r", encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError as e:
        print(f"설정 파일을 찾을 수 없습니다: {filename}")
        raise  # 예외를 다시 던진다 (상위 코드로 전파)

try:
    config = load_config("config.txt")
except FileNotFoundError:
    print("프로그램을 시작할 수 없습니다. 설정 파일이 필요합니다.")
```

`raise`만 쓰면 같은 예외를 그대로 다시 던진다. 다른 예외로 바꿔서 던질 수도 있다.

```python
def parse_age(text):
    try:
        age = int(text)
        if age < 0 or age > 150:
            raise ValueError(f"나이 범위가 올바르지 않습니다: {age}")
        return age
    except ValueError as e:
        raise ValueError(f"나이 입력 오류: {e}") from e
```

---

## 6. 커스텀 예외 만들기

Python 내장 예외 외에도 프로젝트에 맞는 예외를 직접 만들 수 있다. `Exception` 클래스를 상속받으면 된다.

```python
# 커스텀 예외 정의
class ScoreOutOfRangeError(Exception):
    """점수가 0~100 범위를 벗어났을 때 발생하는 예외"""
    def __init__(self, score):
        self.score = score
        super().__init__(f"점수 {score}은 0~100 범위를 벗어났습니다.")

# 사용 예시
def validate_score(score):
    if not isinstance(score, int):
        raise TypeError("점수는 정수여야 합니다.")
    if score < 0 or score > 100:
        raise ScoreOutOfRangeError(score)
    return score

# 테스트
try:
    validate_score(150)
except ScoreOutOfRangeError as e:
    print(e)  # 점수 150은 0~100 범위를 벗어났습니다.

try:
    validate_score("A+")
except TypeError as e:
    print(e)  # 점수는 정수여야 합니다.
```

커스텀 예외는 직접 만든 함수나 클래스에서 특별한 오류 상황을 표현할 때 유용하다.

---

## 실습 1. 점수 입력 프로그램 (0~100 검증)

### 따라 하기

숫자가 아닌 값이나 범위를 벗어난 값이 입력되면 계속 다시 입력을 요청하는 프로그램을 만들어 보자.

```python
def get_valid_score():
    """0~100 사이의 유효한 점수를 입력받는 함수"""
    while True:
        user_input = input("점수를 입력하세요 (0~100): ")

        try:
            score = int(user_input)
        except ValueError:
            print(f"  오류: '{user_input}'는 숫자가 아닙니다. 숫자만 입력해 주세요.")
            continue  # 다시 입력 요청

        if score < 0 or score > 100:
            print(f"  오류: {score}은 올바른 점수가 아닙니다. 0에서 100 사이로 입력해 주세요.")
            continue  # 다시 입력 요청

        return score  # 유효한 값이면 반환

score = get_valid_score()
print(f"\n입력된 점수: {score}")

if score >= 90:
    print("등급: A")
elif score >= 80:
    print("등급: B")
elif score >= 70:
    print("등급: C")
else:
    print("등급: F")
```

실행 결과 예시:
```
점수를 입력하세요 (0~100): abc
  오류: 'abc'는 숫자가 아닙니다. 숫자만 입력해 주세요.
점수를 입력하세요 (0~100): 150
  오류: 150은 올바른 점수가 아닙니다. 0에서 100 사이로 입력해 주세요.
점수를 입력하세요 (0~100): 85

입력된 점수: 85
등급: B
```

### 직접 해보기

위 코드를 수정해서 다음 기능을 추가해 보자.

1. 점수를 3번 이상 잘못 입력하면 "입력 횟수를 초과했습니다."라고 출력하고 프로그램을 종료한다
2. 소수점 점수도 허용하도록 수정한다 (`int()` 대신 `float()` 사용)

---

## 실습 2. 파일 안전하게 읽기

### 따라 하기

파일을 읽을 때 다양한 오류 상황을 처리하는 함수를 만들어 보자.

```python
def read_file_safe(filename):
    """
    파일을 안전하게 읽는 함수.
    파일이 없거나 읽을 수 없으면 None을 반환한다.
    """
    try:
        with open(filename, "r", encoding="utf-8") as f:
            content = f.read()
        print(f"파일을 성공적으로 읽었습니다. ({len(content)}자)")
        return content
    except FileNotFoundError:
        print(f"오류: '{filename}' 파일을 찾을 수 없습니다.")
        print("파일 이름과 경로를 다시 확인해 주세요.")
        return None
    except PermissionError:
        print(f"오류: '{filename}' 파일을 읽을 권한이 없습니다.")
        return None
    except UnicodeDecodeError:
        print(f"오류: '{filename}' 파일의 인코딩이 맞지 않습니다.")
        print("UTF-8이 아닌 파일일 수 있습니다.")
        return None
    finally:
        print("파일 읽기 시도를 완료했습니다.")

# 테스트
result = read_file_safe("test.txt")

if result is not None:
    print("\n--- 파일 내용 ---")
    print(result)
else:
    print("\n파일을 읽지 못했습니다.")
```

먼저 "test.txt" 파일을 만들어서 테스트해 보자. 그 다음 파일 이름을 "없는파일.txt"로 바꿔서 오류 처리가 작동하는지 확인한다.

### 직접 해보기

다음 기능을 추가해 보자.

1. 파일을 읽는 데 성공하면 각 줄을 번호와 함께 출력한다 (`"1: 첫 번째 줄"` 형식)
2. 파일이 비어있으면 "파일이 비어있습니다."라고 알려준다

---

## 실습 3. 여러 학생 점수 처리하기

### 따라 하기

딕셔너리 리스트에서 점수를 계산하되, 데이터에 문제가 있어도 프로그램이 멈추지 않도록 만들어 보자.

```python
students = [
    {"name": "김철수", "score": 85},
    {"name": "이영희", "score": "없음"},   # 점수가 문자열
    {"name": "박민준"},                    # score 키가 없음
    {"name": "최수아", "score": 92},
    {"name": "정태양", "score": -10},      # 음수 점수
]

valid_scores = []
errors = []

for student in students:
    name = student.get("name", "이름없음")

    try:
        score = student["score"]           # KeyError 가능
        score = int(score)                 # ValueError 가능
        if score < 0 or score > 100:
            raise ValueError(f"범위 초과: {score}")
        valid_scores.append(score)
        print(f"  {name}: {score}점 (정상)")
    except KeyError:
        error_msg = f"{name}: 점수 데이터가 없습니다."
        errors.append(error_msg)
        print(f"  경고: {error_msg}")
    except ValueError as e:
        error_msg = f"{name}: {e}"
        errors.append(error_msg)
        print(f"  경고: {error_msg}")

print(f"\n--- 결과 ---")
if valid_scores:
    average = sum(valid_scores) / len(valid_scores)
    print(f"유효한 점수 수: {len(valid_scores)}")
    print(f"평균 점수: {average:.1f}")
else:
    print("유효한 점수가 없습니다.")

if errors:
    print(f"\n처리 실패 항목 ({len(errors)}개):")
    for error in errors:
        print(f"  - {error}")
```

### 직접 해보기

위 코드를 바탕으로 다음을 해보자.

1. 오류가 있는 학생 데이터를 `errors` 리스트에 수집해서 마지막에 한 번에 출력한다 (이미 구현되어 있다. 동작 방식을 이해해 보자)
2. 최고 점수와 최저 점수도 함께 출력한다

---

## 초보자가 자주 막히는 지점

### 막힘 1: `except:` 만 쓰는 것

```python
# 나쁜 예 - 모든 오류를 막연하게 처리
try:
    result = int(input("숫자: "))
except:
    print("오류!")
```

이렇게 하면 `Ctrl+C`로 프로그램을 종료하려 해도 잡아버리고, 어떤 오류가 왜 났는지 알 수 없다.

```python
# 좋은 예 - 어떤 오류인지 명시
try:
    result = int(input("숫자: "))
except ValueError:
    print("숫자 형식이 맞지 않습니다.")
```

### 막힘 2: `try` 블록을 너무 넓게 쓰는 것

```python
# 나쁜 예 - 어디서 오류가 났는지 알기 어렵다
try:
    a = int(input("첫 번째 숫자: "))
    b = int(input("두 번째 숫자: "))
    result = a / b
    print(result)
    with open("result.txt", "w") as f:
        f.write(str(result))
except:
    print("오류가 났습니다.")
```

```python
# 좋은 예 - 오류 발생 가능한 부분을 나눠서 처리
try:
    a = int(input("첫 번째 숫자: "))
    b = int(input("두 번째 숫자: "))
except ValueError:
    print("숫자만 입력해 주세요.")
    exit()

try:
    result = a / b
except ZeroDivisionError:
    print("두 번째 숫자는 0이 될 수 없습니다.")
    exit()

print(f"결과: {result}")
```

### 막힘 3: `finally`를 `else`와 헷갈리는 것

- `else`: 오류가 **없었을 때**만 실행
- `finally`: 오류가 있든 없든 **항상** 실행

파일이나 데이터베이스처럼 반드시 닫아야 하는 자원은 `finally`에서 처리한다.

---

## 자주 만나는 에러와 해결법

| 에러 메시지 | 원인 | 해결 방법 |
|-------------|------|-----------|
| `ValueError: invalid literal for int()` | 숫자가 아닌 문자열을 `int()`로 변환 | `try/except ValueError` 처리 |
| `TypeError: can only concatenate str (not "int") to str` | 문자열에 숫자를 `+`로 연결 | `str()`로 변환 후 연결 |
| `KeyError: 'xxx'` | 딕셔너리에 없는 키 조회 | `.get(key, 기본값)` 사용 |
| `FileNotFoundError: [Errno 2] No such file or directory` | 파일 경로나 이름이 틀림 | 경로 확인, `try/except FileNotFoundError` 처리 |
| `ZeroDivisionError: division by zero` | 0으로 나누기 | 나누기 전에 분모가 0인지 확인 |

---

## 확인 체크리스트

- `try / except / else / finally` 각각의 역할을 설명할 수 있는가
- `ValueError`, `TypeError`, `KeyError`, `FileNotFoundError`, `ZeroDivisionError`가 어떤 상황에서 발생하는지 알고 있는가
- `except 예외종류 as e`로 오류 메시지를 받아서 출력할 수 있는가
- `finally`를 사용해서 항상 실행되는 코드를 작성할 수 있는가
- `raise`로 예외를 다시 던지는 코드를 작성할 수 있는가
- 커스텀 예외 클래스를 만들 수 있는가
- 예외 처리를 너무 넓게 쓰지 않도록 주의하고 있는가

---

## 한 번 더 생각해 보기

1. `except:` (예외 종류 없이)를 쓰면 어떤 문제가 생길 수 있을까?
2. `finally`가 없어도 코드가 동작한다면, 언제 `finally`를 꼭 써야 할까?
3. 커스텀 예외를 만들면 어떤 장점이 있을까?
4. 사용자가 잘못 입력했을 때 최대 몇 번까지 다시 입력할 기회를 주는 것이 적당할까?
