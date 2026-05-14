# ai-train Python Basics Chapter 08

## 이 장에서 배우는 것

- comprehension이 무엇인지
- 간단한 리스트 변환을 더 짧게 쓰는 방법
- `datetime`으로 날짜와 시간을 다루는 기본 방법

## 먼저 쉬운 설명

이 장은 Python 기초를 조금 더 익힌 다음에 보면 좋은 내용이다.

- comprehension: 반복문을 더 짧게 쓰는 방법
- datetime: 날짜와 시간을 다루는 방법

이라고 생각하면 된다.

## 1. comprehension이란 무엇인가

### 일반 반복문

```python
numbers = [1, 2, 3, 4]
result = []

for number in numbers:
    result.append(number * 2)
```

### comprehension

```python
numbers = [1, 2, 3, 4]
result = [number * 2 for number in numbers]
```

## 2. datetime이란 무엇인가

```python
from datetime import datetime

now = datetime.now()
print(now)
```

## 3. 날짜 형식 바꾸기

```python
from datetime import datetime

now = datetime.now()
print(now.strftime("%Y-%m-%d"))
```

## 4. 따라 하기 실습

### 실습 1. comprehension 써 보기

```python
numbers = [1, 2, 3]
result = [number + 1 for number in numbers]
print(result)
```

### 실습 2. 오늘 날짜 출력하기

```python
from datetime import datetime

today = datetime.now()
print(today.strftime("%Y-%m-%d"))
```

## 교사용 메모

- 강조: comprehension은 "짧고 규칙이 같은 경우"에만 편하다고 설명하면 충분하다.
- 막힘: comprehension을 너무 길게 쓰거나 `datetime` import를 빼먹는 경우가 많다.
- 질문 1: `[1, 2, 3]`을 `[2, 3, 4]`로 바꾸려면 어떻게 쓸까?
- 질문 2: `strftime("%Y-%m-%d")`는 어떤 모양의 날짜를 만들까?

## 자주 하는 실수

- comprehension을 너무 길게 써서 오히려 읽기 어렵게 만들기
- `datetime`을 import하지 않고 바로 쓰기
- 날짜 형식 문자열을 헷갈리기

## 확인 체크리스트

- comprehension이 무엇인지 설명할 수 있는가
- 간단한 리스트 변환을 comprehension으로 쓸 수 있는가
- `datetime.now()`를 사용할 수 있는가
- 날짜를 문자열로 바꿔 출력할 수 있는가

## 한 번 더 생각해 보기

1. 언제 comprehension이 편할까?
2. 언제는 그냥 반복문이 더 읽기 쉬울까?
3. 날짜와 시간을 다루는 기능은 어디에 쓸 수 있을까?
