# ai-train Python Basics Chapter 03

## 이 장에서 배우는 것

- 오류가 왜 생기는지
- 오류 메시지를 어떻게 읽는지
- 초보자가 가장 많이 보는 오류 예시
- `print()`로 디버깅하는 가장 쉬운 방법

## 먼저 쉬운 설명

Python을 배우다 보면 오류는 반드시 나온다.

하지만 오류는 "실패"가 아니라  
"무엇을 고쳐야 하는지 알려 주는 힌트"라고 생각하면 훨씬 배우기 쉽다.

## 1. 오류는 왜 생길까

- 변수 이름을 잘못 썼을 때
- 괄호나 따옴표를 빠뜨렸을 때
- 숫자와 문자를 잘못 섞었을 때
- 없는 값을 읽으려고 했을 때

## 2. 오류 메시지 읽기

예:

```python
print(message)
```

```python
NameError: name 'message' is not defined
```

이 말은 `message`라는 이름을 찾지 못했다는 뜻이다.

## 3. 자주 보는 오류 1: NameError

```python
print(user_name)
```

해결:

- 변수를 먼저 만들었는지 확인한다
- 이름을 다르게 쓰지 않았는지 본다

## 4. 자주 보는 오류 2: TypeError

```python
age = 14
message = "나이는 " + age
```

해결:

```python
age = 14
message = "나이는 " + str(age)
```

## 5. 자주 보는 오류 3: IndexError

```python
numbers = [10, 20, 30]
print(numbers[5])
```

해결:

- 리스트 길이를 먼저 확인한다
- 존재하는 위치만 읽는다

## 6. 가장 쉬운 디버깅 방법: print()

```python
def add_scores(a, b):
    print("a =", a)
    print("b =", b)
    result = a + b
    print("result =", result)
    return result
```

## 7. 따라 하기 실습

### 실습 1. NameError 고치기

```python
print(student_name)
```

### 실습 2. TypeError 고치기

```python
age = 15
print("나이: " + age)
```

### 실습 3. print로 값 확인하기

```python
def multiply(a, b):
    result = a * b
    return result
```

## 자주 하는 실수

- 오류 메시지를 읽지 않고 바로 포기하기
- 문제 줄을 보지 않고 아무 줄이나 고치기
- 여러 군데를 한 번에 바꾸기
- 중간값 확인 없이 감으로 수정하기

## 확인 체크리스트

- 오류 종류를 읽을 수 있는가
- 오류가 난 줄을 찾을 수 있는가
- `NameError`, `TypeError`, `IndexError` 차이를 말할 수 있는가
- `print()`로 중간값을 확인할 수 있는가

## 한 번 더 생각해 보기

1. 오류 메시지는 왜 도움이 될까?
2. 왜 여러 군데를 한꺼번에 고치면 더 헷갈릴까?
3. 디버깅은 왜 코딩 실력과 연결될까?

## 교사용 메모

- 강조: 오류는 실패가 아니라 힌트라고 먼저 말하고, 오류 종류와 문제 줄부터 찾게 한다.
- 막힘: 문제 줄이 아닌 다른 줄을 고치거나, `TypeError`를 자료형 문제로 연결하지 못하는 경우가 많다.
- 질문 1: `NameError`를 보면 가장 먼저 무엇을 확인해야 할까?
- 질문 2: `print()`를 중간에 넣으면 어떤 값을 확인할 수 있을까?
