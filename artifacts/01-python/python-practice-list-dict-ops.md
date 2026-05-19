## 이 장에서 배우는 것

- 리스트(list)를 만들고 항목을 추가·삭제·조회하는 방법
- 딕셔너리(dictionary)를 만들고 키-값 쌍을 다루는 방법
- 리스트와 딕셔너리를 함께 사용하는 실용적인 패턴
- 흔히 발생하는 오류 메시지를 읽고 스스로 고치는 능력

---

## 먼저 쉬운 설명

쇼핑 목록을 종이에 쓴다고 상상해 보세요.  
항목을 순서대로 나열하면 **리스트**, 상품 이름 옆에 가격을 적으면 **딕셔너리**입니다.

Python에서 데이터를 다루는 거의 모든 작업이 이 두 가지 자료구조에서 시작됩니다.  
API 응답, CSV 파일, 사용자 입력 — 어디서나 리스트와 딕셔너리가 등장합니다.  
지금 이 둘을 확실히 익혀 두면 앞으로 배울 모든 주제가 훨씬 쉬워집니다.

---

## 1. 리스트 기본 조작

리스트는 **순서가 있는 값의 묶음**입니다. 대괄호 `[]`로 만듭니다.

```python
# 파일명: list_practice.py

# 리스트 생성
fruits = ["사과", "바나나", "딸기"]

# 인덱스로 조회 (0부터 시작)
print(fruits[0])   # 사과
print(fruits[-1])  # 딸기 (뒤에서 첫 번째)

# 항목 추가
fruits.append("포도")
print(fruits)  # ['사과', '바나나', '딸기', '포도']

# 특정 위치에 삽입
fruits.insert(1, "망고")
print(fruits)  # ['사과', '망고', '바나나', '딸기', '포도']

# 항목 삭제
fruits.remove("바나나")
print(fruits)  # ['사과', '망고', '딸기', '포도']

# 인덱스로 삭제
del fruits[0]
print(fruits)  # ['망고', '딸기', '포도']

# 길이 확인
print(len(fruits))  # 3
```

---

## 2. 리스트 슬라이싱과 반복

```python
# 파일명: list_slice.py

scores = [85, 92, 78, 95, 88, 70, 99]

# 슬라이싱: [시작:끝] (끝 인덱스는 포함 안 됨)
print(scores[1:4])   # [92, 78, 95]
print(scores[:3])    # [85, 92, 78]  — 처음부터 3개
print(scores[4:])    # [88, 70, 99]  — 4번 인덱스부터 끝까지

# for 문으로 순회
for score in scores:
    if score >= 90:
        print(f"우수: {score}")

# 리스트 컴프리헨션으로 필터링
high_scores = [s for s in scores if s >= 90]
print(high_scores)  # [92, 95, 99]

# 정렬
scores.sort()
print(scores)       # [70, 78, 85, 88, 92, 95, 99]

scores.sort(reverse=True)
print(scores)       # [99, 95, 92, 88, 85, 78, 70]
```

---

## 3. 딕셔너리 기본 조작

딕셔너리는 **키(key)와 값(value)의 쌍**입니다. 중괄호 `{}`로 만듭니다.

```python
# 파일명: dict_practice.py

# 딕셔너리 생성
student = {
    "name": "김민준",
    "age": 25,
    "major": "컴퓨터공학"
}

# 키로 값 조회
print(student["name"])          # 김민준
print(student.get("age"))       # 25
print(student.get("grade", "없음"))  # 없음 (키가 없으면 기본값 반환)

# 값 추가 / 수정
student["grade"] = "A"
student["age"] = 26
print(student)

# 키 삭제
del student["grade"]
print(student)

# 키 존재 여부 확인
if "name" in student:
    print("이름이 있습니다.")

# 키 목록, 값 목록
print(list(student.keys()))    # ['name', 'age', 'major']
print(list(student.values()))  # ['김민준', 26, '컴퓨터공학']
```

---

## 4. 딕셔너리 반복과 중첩 구조

```python
# 파일명: dict_loop.py

# 딕셔너리 반복
product = {"상품명": "노트북", "가격": 1200000, "재고": 15}

for key, value in product.items():
    print(f"{key}: {value}")

# 리스트 안에 딕셔너리 — 가장 흔한 실전 패턴
students = [
    {"name": "이수현", "score": 88},
    {"name": "박지훈", "score": 95},
    {"name": "최유나", "score": 72},
]

# 점수 80점 이상인 학생만 출력
for s in students:
    if s["score"] >= 80:
        print(f"{s['name']}: {s['score']}점")

# 딕셔너리 안에 딕셔너리
inventory = {
    "apple": {"count": 50, "price": 1500},
    "banana": {"count": 30, "price": 800},
}
print(inventory["apple"]["price"])  # 1500
```

---

## 따라 하기 실습

### 실습 1 — 쇼핑 카트 만들기

`shopping_cart.py` 파일을 만들고 아래 요구사항을 직접 구현해 보세요.

1. 빈 리스트 `cart`를 만드세요.
2. `"우유"`, `"계란"`, `"식빵"` 세 항목을 `append`로 추가하세요.
3. `"계란"`을 `remove`로 삭제하세요.
4. `for` 문으로 카트에 남은 항목을 한 줄씩 출력하세요.

```python
# shopping_cart.py — 완성 예시
cart = []
cart.append("우유")
cart.append("계란")
cart.append("식빵")

cart.remove("계란")

print("=== 장바구니 ===")
for item in cart:
    print(f"- {item}")
```

---

### 실습 2 — 학생 성적 관리

실습 1을 기반으로 `grade_manager.py`를 만드세요.

1. 딕셔너리 리스트로 학생 3명의 이름과 점수를 저장하세요.
2. 전체 평균 점수를 계산해 출력하세요.
3. 가장 높은 점수를 받은 학생 이름을 출력하세요.

```python
# grade_manager.py — 완성 예시
students = [
    {"name": "정하은", "score": 91},
    {"name": "오태양", "score": 85},
    {"name": "한소연", "score": 97},
]

total = sum(s["score"] for s in students)
average = total / len(students)
print(f"평균 점수: {average:.1f}")

top = max(students, key=lambda s: s["score"])
print(f"최고 득점자: {top['name']} ({top['score']}점)")
```

---

### 실습 3 — 재고 관리 시스템

실습 2를 확장해 `inventory_manager.py`를 만드세요.

1. 딕셔너리 안에 딕셔너리 형태로 상품 3개의 재고와 가격을 저장하세요.
2. 재고가 20개 미만인 상품을 "재고 부족"으로 표시해 출력하세요.
3. 전체 재고 수량 합계를 출력하세요.

```python
# inventory_manager.py — 완성 예시
inventory = {
    "커피": {"count": 45, "price": 12000},
    "녹차": {"count": 8,  "price": 9000},
    "홍차": {"count": 17, "price": 10500},
}

for name, info in inventory.items():
    status = "재고 부족 ⚠️" if info["count"] < 20 else "정상"
    print(f"{name}: {info['count']}개 — {status}")

total_count = sum(info["count"] for info in inventory.values())
print(f"\n전체 재고: {total_count}개")
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 원인 | 해결 방법 |
|------|------------|------|-----------|
| 없는 인덱스 접근 | `IndexError: list index out of range` | 리스트 길이보다 큰 인덱스 사용 | `len()`으로 길이 확인 후 접근 |
| 없는 키로 접근 | `KeyError: 'grade'` | 딕셔너리에 해당 키가 없음 | `dict.get(key)` 또는 `if key in dict` 사용 |
| 리스트에 없는 값 삭제 | `ValueError: list.remove(x): x not in list` | `remove()`에 없는 값 전달 | `if value in list` 확인 후 삭제 |
| 인덱스 0 착각 | 의도와 다른 값 출력 | 인덱스가 1이 아닌 0부터 시작 | `print(list[0])`으로 먼저 확인 |
| 딕셔너리 순서 착각 | 의도와 다른 순서 출력 | 옛날 Python(3.7 이전)은 순서 보장 없음 | Python 3.7+ 사용, 순서 필요 시 `list` 사용 |
| 문자열과 정수 키 혼용 | `KeyError: 1` | `dict["1"]`과 `dict[1]`은 다른 키 | 키 타입을 일관되게 유지 |

---

## 확인 체크리스트

- [ ] 리스트를 `[]`로 생성하고 `append()`로 항목을 추가할 수 있다.
- [ ] 인덱스가 0부터 시작하며 `-1`이 마지막 항목임을 안다.
- [ ] `remove(값)`과 `del list[인덱스]`의 차이를 설명할 수 있다.
- [ ] 슬라이싱 `[1:3]`이 인덱스 1과 2만 반환함을 안다.
- [ ] 딕셔너리를 `{}`로 생성하고 키-값 쌍을 추가·수정·삭제할 수 있다.
- [ ] `dict.get(key, 기본값)`으로 `KeyError` 없이 안전하게 조회할 수 있다.
- [ ] `for key, value in dict.items()`로 딕셔너리를 순회할 수 있다.
- [ ] 리스트 안에 딕셔너리를 넣는 구조를 직접 작성할 수 있다.
- [ ] 실습 3의 `inventory_manager.py`를 오류 없이 실행했다.

---

## 한 번 더 생각해 보기

1. **리스트 vs 딕셔너리**: 학생 100명의 이름만 저장할 때와, 이름·나이·점수를 함께 저장할 때 각각 어떤 자료구조가 더 적합한가요? 그 이유는 무엇인가요?

2. **불변성 고민**: `fruits.sort()`는 원본 리스트를 바꾸지만, `sorted(fruits)`는 새 리스트를 반환합니다. 어떤 상황에서 원본을 바꾸지 않는 것이 중요할까요?

3. **실전 적용**: 여러분이 자주 쓰는 앱(배달, 쇼핑, SNS)에서 리스트와 딕셔너리가 어떻게 쓰일지 한 가지 예를 떠올려 보세요. 데이터 구조를 직접 코드로 설계해 보세요.

---

## 다음 장

다음 장에서는 지금 만든 리스트와 딕셔너리를 **파일로 저장하고 불러오는 방법(JSON 읽기·쓰기)**을 배웁니다.