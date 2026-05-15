# ai-train Python 실습: 조건문 + 반복문 + 함수 집중 연습

## 대상 챕터

Chapter 11 (조건문), Chapter 12 (반복문), Chapter 13 (함수)

---

## 이 실습에서 연습하는 것

조건문, 반복문, 함수는 Python의 핵심이다. 이 세 가지를 따로 배웠다면, 이제 함께 써보자.

각 문제는 작은 것부터 시작해서 점점 결합해간다.

---

## Part 1. 조건문 단독 연습

### 문제 1-1. 삼각형 판별

세 변의 길이를 입력받아 삼각형이 될 수 있는지 판별하는 함수를 만든다.

**조건**: 두 변의 합이 나머지 한 변보다 크면 삼각형이 된다.

```python
def is_triangle(a, b, c):
    # 여기에 코드 작성
    pass

# 테스트
print(is_triangle(3, 4, 5))   # True
print(is_triangle(1, 2, 10))  # False
print(is_triangle(5, 5, 5))   # True
```

### 문제 1-2. 윤년 판별

연도를 입력받아 윤년 여부를 반환하는 함수를 만든다.

**윤년 조건**:
- 4로 나누어 떨어지고
- 100으로 나누어 떨어지지 않거나
- 400으로 나누어 떨어지면 윤년

```python
def is_leap_year(year):
    # 여기에 코드 작성
    pass

print(is_leap_year(2000))  # True
print(is_leap_year(1900))  # False
print(is_leap_year(2024))  # True
print(is_leap_year(2023))  # False
```

---

## Part 2. 반복문 단독 연습

### 문제 2-1. 피보나치 수열

n번째까지의 피보나치 수를 리스트로 반환하는 함수를 만든다.

피보나치: 0, 1, 1, 2, 3, 5, 8, 13 ...

```python
def fibonacci(n):
    # 여기에 코드 작성
    pass

print(fibonacci(8))   # [0, 1, 1, 2, 3, 5, 8, 13]
print(fibonacci(1))   # [0]
print(fibonacci(0))   # []
```

### 문제 2-2. 단어 빈도 세기

단어 리스트를 받아 각 단어가 몇 번 등장하는지 딕셔너리로 반환하는 함수를 만든다.

```python
def count_words(words):
    # 여기에 코드 작성
    pass

result = count_words(["apple", "banana", "apple", "cherry", "banana", "apple"])
print(result)   # {"apple": 3, "banana": 2, "cherry": 1}

print(count_words([]))   # {}
```

---

## Part 3. 조건문 + 반복문 결합

### 문제 3-1. 소수 찾기

2부터 n까지의 소수를 모두 리스트로 반환하는 함수를 만든다.

소수: 1과 자기 자신만으로 나누어 떨어지는 수

```python
def find_primes(n):
    # 여기에 코드 작성
    pass

print(find_primes(20))   # [2, 3, 5, 7, 11, 13, 17, 19]
print(find_primes(2))    # [2]
print(find_primes(1))    # []
```

### 문제 3-2. FizzBuzz

1부터 n까지 출력하되:
- 3의 배수면 "Fizz"
- 5의 배수면 "Buzz"
- 15의 배수면 "FizzBuzz"
- 나머지는 숫자 그대로

결과를 리스트로 반환한다.

```python
def fizzbuzz(n):
    # 여기에 코드 작성
    pass

print(fizzbuzz(15))
# [1, 2, "Fizz", 4, "Buzz", "Fizz", 7, 8, "Fizz", "Buzz", 11, "Fizz", 13, 14, "FizzBuzz"]
```

---

## Part 4. 함수 설계 연습

### 문제 4-1. 리스트 통계 함수 세트

하나의 숫자 리스트에서 여러 통계를 구하는 함수를 각각 만들되, 빈 리스트에 안전하게 동작해야 한다.

```python
def my_sum(numbers):
    """리스트의 합계를 반환. 빈 리스트면 0."""
    pass

def my_average(numbers):
    """리스트의 평균을 반환. 빈 리스트면 0."""
    pass

def my_max(numbers):
    """리스트의 최댓값을 반환. 빈 리스트면 None."""
    pass

def my_min(numbers):
    """리스트의 최솟값을 반환. 빈 리스트면 None."""
    pass

# 테스트
data = [5, 2, 8, 1, 9, 3]
print(my_sum(data))      # 28
print(my_average(data))  # 4.666...
print(my_max(data))      # 9
print(my_min(data))      # 1

# 빈 리스트 테스트
print(my_sum([]))        # 0
print(my_max([]))        # None
```

### 문제 4-2. 텍스트 분석 함수

문자열을 받아 다양한 정보를 반환하는 함수를 만든다.

```python
def analyze_text(text):
    """
    텍스트를 분석해서 딕셔너리로 반환한다.
    반환값:
        - char_count: 전체 문자 수 (공백 포함)
        - word_count: 단어 수
        - line_count: 줄 수
        - most_common_char: 가장 많이 등장한 문자 (공백 제외)
    """
    pass

text = """Python is fun
I love coding
Python is great"""

result = analyze_text(text)
print(result["char_count"])       # 전체 문자 수
print(result["word_count"])       # 단어 수
print(result["line_count"])       # 줄 수
print(result["most_common_char"]) # 'n' (또는 다른 문자)
```

---

## Part 5. 세 가지 모두 결합

### 문제 5-1. 숫자 분류기

숫자 리스트를 받아 짝수/홀수/양수/음수로 분류해서 딕셔너리로 반환하는 함수를 만든다. 0은 짝수이고 양수도 음수도 아니다.

```python
def classify_numbers(numbers):
    """
    반환값:
        - even: 짝수 리스트
        - odd: 홀수 리스트
        - positive: 양수 리스트
        - negative: 음수 리스트
        - zero_count: 0의 개수
    """
    pass

data = [-3, -2, -1, 0, 1, 2, 3, 4, 5]
result = classify_numbers(data)

print(result["even"])      # [-2, 0, 2, 4]
print(result["odd"])       # [-3, -1, 1, 3, 5]
print(result["positive"])  # [1, 2, 3, 4, 5]
print(result["negative"])  # [-3, -2, -1]
print(result["zero_count"])# 1
```

### 문제 5-2. 성적 처리 파이프라인

학생 이름과 점수 딕셔너리를 받아 처리하는 함수 파이프라인을 만든다.

```python
def get_grade(score):
    """점수 → 등급 반환"""
    pass

def process_scores(score_dict):
    """
    score_dict: {"Mina": 92, "Tom": 75, "Jisoo": 88, ...}
    반환값:
        - results: [{"name": ..., "score": ..., "grade": ...}, ...]
        - class_average: 반 전체 평균
        - top_student: 최고 점수 학생 이름
        - grade_distribution: {"A": 1, "B": 2, ...}
    """
    pass

scores = {"Mina": 92, "Tom": 75, "Jisoo": 88, "Alex": 61, "Yuna": 85}
result = process_scores(scores)

for r in result["results"]:
    print(f"{r['name']}: {r['score']}점 ({r['grade']})")

print(f"반 평균: {result['class_average']:.1f}")
print(f"최고 학생: {result['top_student']}")
print(f"등급 분포: {result['grade_distribution']}")
```

---

## 정답 확인 방법

각 함수를 작성한 뒤 제공된 테스트 코드를 실행해서 결과를 확인한다.

모두 통과했으면 AI에게 아래를 요청해본다:

```
위 함수들 중 하나를 골라 엣지 케이스 테스트를 3개 더 추가해줘.
내가 놓친 케이스가 있는지 확인하고 싶어.
```
