# Chapter 05: AI 코드 검증 체크리스트

## 이 장에서 배우는 것

- AI가 만든 코드를 수락하기 전에 확인해야 할 항목
- 코드가 동작하는 것과 올바른 것의 차이
- 직접 실행해서 검증하는 방법
- AI 코드에서 자주 나타나는 문제 패턴
- 검증 후 GitHub에 올리기까지의 흐름

---

## 먼저 쉬운 설명

AI가 만든 코드가 실행됐다. 그러면 완성인가?

그렇지 않다. 코드가 **실행되는 것**과 **올바른 것**은 다르다.

```python
def get_average(numbers):
    return sum(numbers) / len(numbers)

# 실행됨 → True
print(get_average([1, 2, 3]))   # 2.0

# 올바름 → ?
print(get_average([]))          # ZeroDivisionError!
```

이 함수는 정상 케이스에서는 잘 동작하지만, 빈 리스트를 넣으면 오류가 난다. AI는 이런 엣지 케이스를 놓칠 때가 많다.

---

## 1. 검증 체크리스트

AI가 만든 코드를 수락하기 전에 아래를 확인한다.

### 기본 확인

- [ ] **코드를 읽었는가** — 각 줄이 무엇을 하는지 이해하고 있는가
- [ ] **실제로 실행했는가** — 눈으로만 보지 않고 직접 실행했는가
- [ ] **원하는 결과가 나오는가** — 예상한 출력이 나오는가

### 입력 검증

- [ ] **정상 입력** — 일반적인 값으로 예상대로 동작하는가
- [ ] **빈 값** — `[]`, `""`, `0`, `None` 을 넣으면 어떻게 되는가
- [ ] **잘못된 타입** — 숫자 자리에 문자열, 문자열 자리에 숫자를 넣으면 어떻게 되는가
- [ ] **경계값** — 최솟값, 최댓값, 딱 1개일 때는 어떻게 동작하는가

### 코드 품질

- [ ] **이해할 수 없는 코드** — 설명 없이는 이해하기 어려운 부분이 있는가
- [ ] **하드코딩된 값** — 변수로 관리해야 할 것이 직접 숫자나 문자로 박혀있는가
- [ ] **보안 문제** — 비밀번호, API 키 등이 코드에 직접 노출되어 있는가

---

## 2. 자주 나타나는 AI 코드 문제 패턴

### 패턴 1. 빈 컬렉션 처리 누락

```python
# AI가 준 코드 (문제 있음)
def first_item(items):
    return items[0]

# 검증
first_item([1, 2, 3])   # 1 ← 정상
first_item([])          # IndexError!

# 수정된 코드
def first_item(items):
    if not items:
        return None
    return items[0]
```

### 패턴 2. 파일/리소스 처리 누락

```python
# AI가 준 코드 (문제 있음)
def read_config():
    with open("config.json") as f:
        return json.load(f)

# 검증: 파일이 없으면?
# FileNotFoundError!

# 수정된 코드
def read_config():
    if not os.path.exists("config.json"):
        return {}
    with open("config.json") as f:
        return json.load(f)
```

### 패턴 3. 타입 불일치

```python
# AI가 준 코드 (문제 있음)
def greet(name):
    return "안녕하세요, " + name + "님!"

# 검증
greet("Mina")    # "안녕하세요, Mina님!" ← 정상
greet(14)        # TypeError: can only concatenate str to str

# 수정된 코드
def greet(name):
    return f"안녕하세요, {name}님!"
```

### 패턴 4. 0 나누기

```python
# AI가 준 코드 (문제 있음)
def percentage(part, total):
    return (part / total) * 100

# 검증
percentage(0, 0)   # ZeroDivisionError!

# 수정된 코드
def percentage(part, total):
    if total == 0:
        return 0
    return (part / total) * 100
```

---

## 3. 검증 실습 방법

### 테스트 케이스 직접 만들기

함수를 받으면 아래 형태로 테스트를 작성한다.

```python
# 검증할 함수
def search_memo(memos, keyword):
    return [m for m in memos if keyword.lower() in m.lower()]

# 테스트 케이스
test_memos = ["Python 공부", "GitHub 실습", "python 연습"]

# 정상 케이스
assert search_memo(test_memos, "Python") == ["Python 공부", "python 연습"]

# 빈 리스트
assert search_memo([], "Python") == []

# 없는 키워드
assert search_memo(test_memos, "AWS") == []

# 빈 키워드
assert search_memo(test_memos, "") == test_memos

print("모든 테스트 통과!")
```

`assert`는 조건이 `False`이면 오류를 발생시킨다. 테스트가 통과하면 다음 줄로 넘어간다.

---

## 4. 검증 후 GitHub까지의 흐름

```
1. AI에게 코드 요청
       ↓
2. 검증 체크리스트 확인
   - 읽기
   - 다양한 입력으로 실행
   - 엣지 케이스 테스트
       ↓
3. 문제 발견 시 → AI에게 수정 요청 (repair loop)
       ↓
4. 검증 통과 시 → 코드 저장
       ↓
5. git add → commit → push
       ↓
6. PR 생성 → 검토
```

---

## 5. 따라 하기 실습

### 실습 1. 아래 코드에서 문제 찾기

AI가 만든 것처럼 아래 코드를 받았다고 가정하고 검증 체크리스트를 적용해본다.

```python
def calculate_bmi(weight, height):
    bmi = weight / (height ** 2)
    if bmi < 18.5:
        return "저체중"
    elif bmi < 25:
        return "정상"
    else:
        return "과체중"
```

테스트할 케이스:
- 정상 입력: `calculate_bmi(70, 1.75)`
- 키 0: `calculate_bmi(70, 0)`
- 음수: `calculate_bmi(-70, 1.75)`
- 문자열: `calculate_bmi("70", 1.75)`

각각 실행해서 어떤 문제가 있는지 파악하고, AI에게 수정을 요청한다.

### 실습 2. 검증 통과 후 GitHub에 올리기

실습 1에서 수정한 코드를 검증 체크리스트를 통과시킨 후:

```bash
git checkout -b fix/bmi-validation
git add bmi.py
git commit -m "BMI 함수에 입력 검증 추가"
git push origin fix/bmi-validation
```

GitHub에서 PR을 만들고, 변경 내용을 설명한다.

---

## 확인 체크리스트

- [ ] AI가 만든 코드를 그냥 실행하지 않고 먼저 읽는 습관이 있는가
- [ ] 정상 케이스와 엣지 케이스(빈 값, 잘못된 타입) 모두 테스트하는가
- [ ] `assert`로 간단한 테스트를 작성할 수 있는가
- [ ] 문제를 발견하면 AI에게 구체적으로 설명하고 수정을 요청하는가
- [ ] 검증이 끝난 코드를 GitHub에 commit하는가

---

## 한 번 더 생각해 보기

1. AI가 만든 코드의 어떤 부분에서 문제가 가장 자주 발생할까?
2. 테스트를 미리 작성하면 어떤 장점이 있을까?
3. AI에게 "이 코드의 문제점을 찾아줘"라고 요청하면 AI가 스스로의 코드를 잘 검토할 수 있을까?

---

## 다음 장

다음 장에서는 AI 도구와 GitHub를 연결해서 PR에서 AI review를 받는 방법을 배운다.

---

## 참고 자료

- Python assert 문서 — https://docs.python.org/3/reference/simple_stmts.html#assert
- Python unittest 시작 — https://docs.python.org/3/library/unittest.html
