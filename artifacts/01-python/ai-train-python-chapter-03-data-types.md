# ai-train Python Basics Chapter 03: 자료형

## 이 장에서 배우는 것

- 자료형이 무엇인지, 왜 구분하는지
- 숫자(int, float), 문자열(str), 불리언(bool) 이해하기
- 자료형을 섞을 때 생기는 오류와 해결법
- 변수에 어떤 자료형이 들어 있는지 확인하는 방법

---

## 먼저 쉬운 설명

현실에서도 데이터의 종류에 따라 할 수 있는 일이 다르다.

- 숫자 10과 숫자 20은 더할 수 있다 → 30
- 이름 "Mina"와 "Kim"은 더할 수 없다 (합계가 없다)
- 이름 "Mina"와 숫자 14를 더하면 의미가 없다

Python도 마찬가지다. 데이터의 종류(자료형)에 따라 할 수 있는 연산이 다르다. 자료형을 모르고 쓰면 뜻밖의 오류가 생기거나, 원하지 않는 결과가 나온다.

---

## 1. 숫자형: int와 float

### int (정수)

소수점 없는 숫자다.

```python
age = 14
score = 100
count = -3
```

### float (실수)

소수점이 있는 숫자다.

```python
height = 163.5
weight = 52.0
temperature = -1.5
```

### 숫자 연산

```python
a = 10
b = 3

print(a + b)   # 13  (더하기)
print(a - b)   # 7   (빼기)
print(a * b)   # 30  (곱하기)
print(a / b)   # 3.333...  (나누기, 결과는 float)
print(a // b)  # 3   (몫)
print(a % b)   # 1   (나머지)
```

`/` 는 항상 소수점 결과를 준다. `//` 는 소수점을 버린 몫만 준다.

---

## 2. 문자열: str

글자, 단어, 문장을 담는 자료형이다. 따옴표로 감싼다.

```python
name = "Mina"
greeting = "안녕하세요"
sentence = "Python is fun"
```

### 문자열 이어 붙이기

문자열끼리는 `+`로 이어 붙일 수 있다.

```python
first = "Hello"
second = " Python"
print(first + second)
```

출력:
```
Hello Python
```

### 문자열과 숫자는 바로 더할 수 없다

```python
name = "Mina"
age = 14
print(name + age)   # 오류!
```

```
TypeError: can only concatenate str (not "int") to str
```

글자와 숫자를 함께 출력하고 싶을 때는 두 가지 방법이 있다.

방법 1 — 콤마로 구분:
```python
print(name, "나이:", age)
```

방법 2 — f-string (권장):
```python
print(f"{name}의 나이는 {age}살입니다.")
```

f-string은 `f"..."` 형태로 쓰고, `{}` 안에 변수를 넣는다. 가장 읽기 쉬운 방법이다.

---

## 3. 불리언: bool

참(`True`) 또는 거짓(`False`) 둘 중 하나만 가지는 자료형이다.

```python
is_student = True
is_raining = False
```

`True`와 `False`는 반드시 첫 글자가 대문자여야 한다. `true`나 `false`로 쓰면 오류가 난다.

### 비교 연산의 결과는 bool

```python
score = 85
print(score >= 80)   # True
print(score == 100)  # False
print(score != 0)    # True
```

비교 연산자:
- `==` : 같다
- `!=` : 다르다
- `>` : 크다
- `<` : 작다
- `>=` : 크거나 같다
- `<=` : 작거나 같다

---

## 4. 자료형 확인하기

변수에 어떤 자료형이 들어 있는지 모를 때는 `type()`을 쓴다.

```python
age = 14
name = "Mina"
height = 163.5
is_student = True

print(type(age))        # <class 'int'>
print(type(name))       # <class 'str'>
print(type(height))     # <class 'float'>
print(type(is_student)) # <class 'bool'>
```

---

## 5. 자료형 변환

자료형을 바꿔야 할 때가 있다.

```python
age_str = "14"        # 문자열 "14"
age_int = int(age_str)  # 정수 14로 변환

print(type(age_str))  # <class 'str'>
print(type(age_int))  # <class 'int'>
```

주요 변환 함수:
- `int("14")` → 정수 14
- `float("3.14")` → 실수 3.14
- `str(14)` → 문자열 `"14"`

변환할 수 없는 값을 변환하려 하면 오류가 난다.

```python
int("안녕")  # ValueError: invalid literal for int()
```

---

## 6. 따라 하기 실습

앞 장의 `intro.py`와 `calc.py`를 이어서 사용한다.

### 실습 1. 자료형 확인하기

`types.py` 파일을 새로 만들고 아래를 입력한다.

```python
name = "Jisoo"
age = 16
height = 165.0
is_student = True

print(type(name))
print(type(age))
print(type(height))
print(type(is_student))
```

저장하고 실행해서 각 자료형을 확인한다.

### 실습 2. f-string으로 자기소개 출력하기

`intro.py`를 열고 아래처럼 수정한다.

```python
name = "Mina"
age = 14
favorite = "Python"

print(f"안녕하세요. 저는 {name}이고, {age}살입니다.")
print(f"배우고 싶은 것: {favorite}")
```

### 실습 3. 자료형 변환 실습

`calc.py`를 열고 아래를 추가한다.

```python
score_str = "85"
score_int = int(score_str)
print(f"점수: {score_int}")
print(f"100점 만점 기준 달성 여부: {score_int >= 80}")
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|------|-----------|----------|
| 문자열과 숫자를 `+`로 연결 | `TypeError: can only concatenate str (not "int") to str` | f-string 또는 `str()` 변환 사용 |
| `true` / `false` 소문자로 쓰기 | `NameError: name 'true' is not defined` | 반드시 `True` / `False` 대문자로 |
| `==` 대신 `=` 로 비교 | 비교 결과 아닌 값 저장 발생 | 비교는 `==`, 저장은 `=` 구분 |
| 숫자처럼 보이는 문자열을 바로 연산 | `TypeError` | `int()` 또는 `float()`으로 변환 |

---

## 확인 체크리스트

- [ ] int, float, str, bool 네 가지 자료형을 말할 수 있는가
- [ ] `type()`으로 자료형을 확인할 수 있는가
- [ ] f-string으로 변수와 글자를 함께 출력할 수 있는가
- [ ] 문자열과 숫자를 `+`로 연결할 수 없는 이유를 설명할 수 있는가
- [ ] `int("14")`와 `str(14)`가 무엇을 하는지 설명할 수 있는가

---

## 한 번 더 생각해 보기

1. `14 / 2`의 결과는 int일까 float일까? 실행해서 확인해 보자.
2. `"10" + "20"` 을 실행하면 무엇이 출력될까? 숫자 30일까, 글자 `"1020"`일까?
3. `type(True)` 를 실행하면 무엇이 나올까?

---

## 다음 장

다음 장에서는 **조건문**을 배운다. 자료형에서 배운 `True` / `False` 를 활용해서 "이럴 때는 이렇게, 저럴 때는 저렇게"를 코드로 표현하는 방법을 익힌다.

---

## 참고 자료

- Python Built-in Types — https://docs.python.org/3/library/stdtypes.html
- Python Tutorial: Numbers — https://docs.python.org/3/tutorial/introduction.html#numbers
- Python Tutorial: Strings — https://docs.python.org/3/tutorial/introduction.html#strings
