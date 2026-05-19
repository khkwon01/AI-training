# Quick Check 03: Chapter 11~15

## 대상 챕터

- Chapter 11: 조건문
- Chapter 12: 반복문
- Chapter 13: 함수
- Chapter 14: class와 object
- Chapter 15: module과 import

---

## Part 1. 조건문 (Chapter 11)

### Q1. 아래 코드의 출력 결과는?

```python
x = 15

if x > 20:
    print("A")
elif x > 10:
    print("B")
elif x > 5:
    print("C")
else:
    print("D")
```

① A  ② B  ③ C  ④ D

---

### Q2. 아래 코드에서 오류가 발생하는 이유는?

```python
score = 75
if score = 75:
    print("맞다")
```

① `score`가 정의되지 않았다  
② `=` 대신 `==` 를 써야 한다  
③ `print()` 에 문자열이 없다  
④ `if` 문 뒤에 `:` 가 없다

---

### Q3. 아래 조건이 `True`가 되려면 `age`의 값은?

```python
if age >= 18 and age < 65:
    print("근로 가능 연령")
```

① 15  ② 17  ③ 30  ④ 65

---

## Part 2. 반복문 (Chapter 12)

### Q4. 아래 코드의 출력 결과는?

```python
for i in range(2, 8, 2):
    print(i, end=" ")
```

① `2 4 6`  
② `2 4 6 8`  
③ `0 2 4 6`  
④ `2 3 4 5 6 7`

---

### Q5. 아래 코드에서 `break`가 실행될 때의 `i` 값은?

```python
for i in range(10):
    if i * i > 30:
        break
print(i)
```

① 5  ② 6  ③ 7  ④ 30

---

### Q6. 아래 코드로 출력되는 숫자 개수는?

```python
count = 0
n = 1
while n <= 100:
    if n % 7 == 0:
        count += 1
    n += 1
print(count)
```

① 13  ② 14  ③ 15  ④ 16

---

## Part 3. 함수 (Chapter 13)

### Q7. 아래 함수의 반환값은?

```python
def mystery(x, y=10):
    return x * 2 + y

print(mystery(3))
```

① 6  ② 13  ③ 16  ④ 오류

---

### Q8. `print`와 `return`의 차이를 올바르게 설명한 것은?

① `print`는 함수 안에서만 쓸 수 있다  
② `return`은 화면에 출력하고 `print`는 값을 반환한다  
③ `print`는 화면에 출력하고, `return`은 함수 호출부에 값을 돌려준다  
④ 둘 다 동일한 역할을 한다

---

### Q9. 아래 코드의 문제점은?

```python
def add(a, b):
    print(a + b)

result = add(3, 4)
print(result * 2)
```

① 함수 이름이 잘못됐다  
② `add` 함수가 값을 반환하지 않아서 `result`가 `None`이 된다  
③ `3`과 `4`는 숫자가 아니다  
④ `result * 2` 는 불가능하다

---

## Part 4. class와 object (Chapter 14)

### Q10. 아래 코드의 출력 결과는?

```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def bark(self):
        return f"{self.name}이(가) 짖습니다!"

dog = Dog("뽀삐", 3)
print(dog.bark())
```

① `뽀삐이(가) 짖습니다!`  
② `Dog.bark()`  
③ `name이(가) 짖습니다!`  
④ 오류

---

### Q11. `__init__` 메서드에 대한 설명으로 올바른 것은?

① `__init__`은 class를 삭제할 때 호출된다  
② `__init__`은 object가 생성될 때 자동으로 호출된다  
③ `__init__`은 직접 호출해야 실행된다  
④ `__init__`이 없으면 class를 만들 수 없다

---

### Q12. 아래 코드에서 `self.name`과 `name`의 차이는?

```python
class Student:
    def __init__(self, name):
        self.name = name   # (A)

    def greet(self):
        print(self.name)   # (B)
```

① (A)와 (B) 모두 매개변수 `name`을 가리킨다  
② (A)는 object의 속성에 저장, (B)는 그 속성을 읽음  
③ (A)와 (B)는 완전히 같다  
④ (B)는 오류다

---

## Part 5. module과 import (Chapter 15)

### Q13. `greetings.py` 파일의 `say_hello` 함수를 가져오는 올바른 방법은?

```python
# greetings.py
def say_hello(name):
    return f"안녕, {name}!"
```

① `import say_hello from greetings`  
② `from greetings import say_hello`  
③ `include greetings.say_hello`  
④ `greetings.import say_hello`

---

### Q14. `if __name__ == "__main__":` 블록이 실행되는 경우는?

① 이 파일이 다른 파일에서 `import` 될 때  
② 이 파일이 직접 실행될 때  
③ 항상 실행된다  
④ 절대 실행되지 않는다

---

### Q15. 아래 코드의 출력 결과는?

```python
import random

numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(len(numbers))
```

① 실행할 때마다 다른 길이  
② 항상 `5`  
③ 항상 `0`  
④ 오류

---

## 서술형

### Q16. 다음 상황에서 `for`와 `while` 중 어느 것이 더 적합한지 이유와 함께 설명하시오.

> 리스트 `[3, 1, 4, 1, 5, 9, 2, 6]` 의 각 요소를 출력한다.

---

### Q17. 아래 함수에서 문제가 될 수 있는 상황과 개선 방법을 설명하시오.

```python
def get_first(items):
    return items[0]
```

---

### Q18. 아래 `BankAccount` class에 `transfer(other_account, amount)` 메서드를 추가하는 코드를 작성하시오. 잔액이 부족하면 "잔액 부족" 메시지를 출력해야 한다.

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            print("잔액 부족")
        else:
            self.balance -= amount

    # 여기에 transfer 메서드 추가
```

---

## 정답은 별도 파일을 참고하세요

[Quick Check 03 정답](python-quick-check-03-answers.md)
