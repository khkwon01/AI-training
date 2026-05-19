# Quick Check 03: 정답 및 해설

## 객관식 정답

| 번호 | 정답 | 핵심 포인트 |
|------|------|-----------|
| Q1 | ② B | `x=15`는 `x > 10` 조건을 만족하는 첫 번째 elif |
| Q2 | ② | 조건 비교는 `==`, 값 저장은 `=` |
| Q3 | ③ 30 | `18 <= age < 65` 범위에 해당 |
| Q4 | ① `2 4 6` | `range(2, 8, 2)` → 2, 4, 6 (8 미포함) |
| Q5 | ② 6 | `6*6=36 > 30`이 처음 True가 되는 시점 |
| Q6 | ② 14 | 1~100 사이 7의 배수: 7,14,21,...,98 → 14개 |
| Q7 | ③ 16 | `mystery(3)` → `3*2 + 10 = 16` |
| Q8 | ③ | `print`=화면 출력, `return`=값 반환 |
| Q9 | ② | `print(a+b)` 는 반환값 없음 → `result`가 `None` |
| Q10 | ① | `dog.name = "뽀삐"` 가 `self.name`에 저장됨 |
| Q11 | ② | `__init__`은 object 생성 시 자동 호출 |
| Q12 | ② | `self.name = name`은 속성 저장, `self.name` 읽기 |
| Q13 | ② | `from 모듈 import 함수` 가 올바른 문법 |
| Q14 | ② | 직접 실행할 때만 `__name__ == "__main__"` |
| Q15 | ② 항상 5 | `shuffle`은 순서만 섞고 길이는 유지 |

---

## 상세 해설

### Q4 해설: range(2, 8, 2)

```python
for i in range(2, 8, 2):
    print(i, end=" ")
# 출력: 2 4 6
```

`range(시작, 끝, 간격)` 에서 끝 값(8)은 포함되지 않는다. 2, 4, 6 이후 다음은 8인데 8 < 8이 False이므로 종료.

### Q5 해설: break 시점

```python
# i=0: 0*0=0 > 30? No
# i=1: 1*1=1 > 30? No
# ...
# i=5: 5*5=25 > 30? No
# i=6: 6*6=36 > 30? Yes → break
```

`print(i)` 는 break 이후 실행되므로 `i=6` 출력.

### Q9 해설: print vs return

```python
def add(a, b):
    print(a + b)   # 화면에 7 출력, 반환값 없음

result = add(3, 4)   # result = None
print(result * 2)    # TypeError: unsupported operand type(s) for *: 'NoneType' and 'int'
```

`return` 으로 수정:
```python
def add(a, b):
    return a + b   # 값을 반환

result = add(3, 4)   # result = 7
print(result * 2)    # 14
```

---

## 서술형 모범 답안

### Q16. for vs while 선택

**정답: `for` 가 더 적합하다.**

이유: 리스트 `[3, 1, 4, 1, 5, 9, 2, 6]`의 요소 개수가 명확히 정해져 있고, 각 요소를 순서대로 하나씩 처리하는 경우에는 `for`가 더 간결하고 명확하다.

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]
for n in numbers:
    print(n)
```

`while`은 반복 횟수가 정해지지 않거나, 조건에 따라 언제 멈출지 모를 때 더 적합하다.

---

### Q17. get_first 함수 문제점

**문제 상황:** `items`가 빈 리스트일 때 `IndexError` 발생

```python
get_first([])   # IndexError: list index out of range
```

**개선 방법:**

```python
def get_first(items):
    if not items:
        return None   # 또는 기본값 반환
    return items[0]
```

또는 예외 처리:
```python
def get_first(items):
    try:
        return items[0]
    except IndexError:
        return None
```

---

### Q18. transfer 메서드 작성

```python
def transfer(self, other_account, amount):
    """다른 계좌로 이체한다."""
    if amount <= 0:
        print("이체 금액은 0보다 커야 합니다")
        return
    if amount > self.balance:
        print("잔액 부족")
        return
    self.balance -= amount
    other_account.balance += amount
    print(f"{amount}원 이체 완료. {self.owner} 잔액: {self.balance}원")
```

**테스트:**
```python
account_a = BankAccount("Mina", 50000)
account_b = BankAccount("Tom", 10000)

account_a.transfer(account_b, 20000)
# 20000원 이체 완료. Mina 잔액: 30000원

print(account_b.balance)   # 30000
account_a.transfer(account_b, 100000)
# 잔액 부족
```
