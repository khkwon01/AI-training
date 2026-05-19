# Chapter 12: 반복문

## 이 장에서 배우는 것

- 반복문이 왜 필요한지
- `for`로 리스트나 범위를 반복하는 방법
- `while`로 조건이 참인 동안 반복하는 방법
- `break`와 `continue`로 반복을 제어하는 방법
- `range()`를 사용하는 방법

---

## 먼저 쉬운 설명

같은 작업을 10번 해야 한다면 코드를 10번 복사해야 할까?

```python
print("안녕")
print("안녕")
print("안녕")
# ... 10번
```

반복문을 쓰면 이렇게 된다.

```python
for i in range(10):
    print("안녕")
```

반복문은 "이 코드를 여러 번 실행해라"를 간결하게 표현하는 방법이다.

---

## 1. for 반복문

`for`는 리스트, 문자열, `range()` 등에서 하나씩 꺼내며 반복한다.

### 리스트 반복

```python
fruits = ["apple", "banana", "orange"]

for fruit in fruits:
    print(fruit)
```

출력:
```
apple
banana
orange
```

### 문자열 반복

```python
for char in "Python":
    print(char)
```

출력:
```
P
y
t
h
o
n
```

---

## 2. range()

숫자 범위를 반복할 때 사용한다.

```python
# range(끝): 0부터 끝-1까지
for i in range(5):
    print(i)   # 0, 1, 2, 3, 4

# range(시작, 끝): 시작부터 끝-1까지
for i in range(1, 6):
    print(i)   # 1, 2, 3, 4, 5

# range(시작, 끝, 간격): 간격만큼 건너뛰며
for i in range(0, 10, 2):
    print(i)   # 0, 2, 4, 6, 8
```

---

## 3. enumerate(): 번호와 함께 반복

인덱스와 값을 동시에 사용할 때 쓴다.

```python
subjects = ["Python", "GitHub", "AWS"]

for i, subject in enumerate(subjects, start=1):
    print(f"{i}. {subject}")
```

출력:
```
1. Python
2. GitHub
3. AWS
```

---

## 4. while 반복문

조건이 `True`인 동안 계속 반복한다.

```python
count = 0

while count < 5:
    print(f"count = {count}")
    count += 1
```

출력:
```
count = 0
count = 1
count = 2
count = 3
count = 4
```

`count += 1` 을 빠뜨리면 무한 반복이 된다. 반드시 조건이 `False`가 되는 시점을 만들어야 한다.

### 사용자 입력 받기

```python
while True:
    text = input("입력 (종료하려면 quit): ")
    if text == "quit":
        break
    print(f"입력받음: {text}")
```

---

## 5. break와 continue

### break: 반복 완전 종료

```python
for i in range(10):
    if i == 5:
        break
    print(i)
# 0, 1, 2, 3, 4 출력 후 종료
```

### continue: 현재 반복만 건너뜀

```python
for i in range(10):
    if i % 2 == 0:
        continue   # 짝수는 건너뜀
    print(i)
# 1, 3, 5, 7, 9 출력
```

---

## 6. 중첩 반복문

반복문 안에 반복문을 넣을 수 있다.

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} x {j} = {i * j}")
    print()
```

출력:
```
1 x 1 = 1
1 x 2 = 2
1 x 3 = 3

2 x 1 = 2
...
```

중첩이 깊어지면 읽기 어려워지므로, 가능하면 함수로 나누는 것이 좋다.

---

## 7. 따라 하기 실습

### 실습 1. 리스트 합계 구하기

`calculate.py` 파일을 만들고:

```python
scores = [85, 92, 78, 90, 88]
total = 0

for score in scores:
    total += score

average = total / len(scores)
print(f"합계: {total}")
print(f"평균: {average:.1f}")
```

### 실습 2. 숫자 맞추기 게임

```python
import random

answer = random.randint(1, 10)
attempts = 0

while True:
    guess = int(input("1~10 사이 숫자를 맞춰보세요: "))
    attempts += 1

    if guess == answer:
        print(f"정답! {attempts}번 만에 맞췄습니다.")
        break
    elif guess < answer:
        print("더 큰 숫자입니다.")
    else:
        print("더 작은 숫자입니다.")
```

### 실습 3. 앞 장 grade.py에 반복 추가

여러 점수를 리스트로 만들어 각 점수의 등급을 출력해 보자.

```python
scores = [95, 72, 88, 61, 79]

for score in scores:
    if score >= 90:
        grade = "A"
    elif score >= 80:
        grade = "B"
    elif score >= 70:
        grade = "C"
    else:
        grade = "F"
    print(f"{score}점 → {grade}")
```

---

## 자주 하는 실수

| 실수 | 증상 | 해결 방법 |
|------|------|----------|
| `while True` 에서 `break` 빠짐 | 무한 반복으로 프로그램 멈춤 | `Ctrl+C`로 강제 종료, `break` 조건 추가 |
| `range(5)` 가 0~4임을 모름 | 범위가 예상보다 하나 적음 | `range(1, 6)` 으로 1~5 범위 설정 |
| for 변수를 반복 중에 수정 | 예상 밖의 동작 | 리스트를 직접 수정할 때는 복사본 사용 |
| `continue` 와 `break` 혼동 | 반복이 예상과 다르게 동작 | `break`=전체 종료, `continue`=이번 회차만 스킵 |

---

## 확인 체크리스트

- [ ] `for`와 `while`의 차이를 설명할 수 있는가
- [ ] `range(1, 11)` 이 몇 개의 숫자를 만드는지 말할 수 있는가
- [ ] `break`와 `continue`를 언제 쓰는지 구분할 수 있는가
- [ ] 리스트의 합계를 `for`로 구할 수 있는가

---

## 한 번 더 생각해 보기

1. `for`를 써야 할 때와 `while`을 써야 할 때를 어떻게 구분할까?
2. `range(10)` 과 `range(0, 10)` 은 같은 결과를 줄까?
3. 무한 루프가 왜 위험한지, 어떻게 안전하게 쓸 수 있을지 생각해보자.

---

## 다음 장

다음 장에서는 함수를 배운다. 반복문으로 여러 번 실행하는 코드를 함수로 묶으면 훨씬 깔끔해진다.

---

## 참고 자료

- Python Tutorial: More Control Flow Tools — https://docs.python.org/3/tutorial/controlflow.html
