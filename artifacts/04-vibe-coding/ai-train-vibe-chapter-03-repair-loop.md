# ai-train Vibe Coding Chapter 03: AI 코드 수정 루프 (Repair Loop)

## 이 장에서 배우는 것

- AI가 준 코드가 틀렸을 때 어떻게 대화를 이어가는지
- 오류 → 분석 → 수정 요청 → 검증의 반복 루프
- AI와 대화할 때 좋은 피드백 주는 방법
- 직접 고치는 것과 AI에게 다시 요청하는 것을 언제 구분할지
- 최종 코드가 내 의도대로 동작하는지 검증하는 방법

---

## 먼저 쉬운 설명

AI가 항상 완벽한 코드를 주지는 않는다.

오류가 나거나, 실행은 되는데 결과가 틀리거나, 내가 원한 방식과 다를 수 있다.

이럴 때 "AI가 틀렸네"하고 포기하거나 다시 처음부터 요청하는 것은 비효율적이다.

**수정 루프**는 AI와 대화를 이어가면서 코드를 점점 올바르게 만들어가는 방법이다.

```
요청 → AI 초안 → 오류/문제 발견 → 수정 요청 → 확인 → 반복
```

---

## 1. 오류가 났을 때

### Step 1. 오류 메시지 복사

터미널에서 오류 메시지 전체를 복사한다. 마지막 줄만 아니라 전체가 필요하다.

```
Traceback (most recent call last):
  File "memo.py", line 15, in load_from_file
    memos = json.load(f)
  File "/usr/lib/python3/dist-packages/json/__init__.py", line 293, in load
    return loads(fp.read(),
json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (pos 0)
```

### Step 2. 오류 + 코드 + 상황을 함께 전달

```
아래 코드를 실행했더니 오류가 났어.

오류 메시지:
json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (pos 0)

코드:
def load_from_file():
    with open("memos.json", "r", encoding="utf-8") as f:
        memos = json.load(f)

상황:
처음 실행할 때는 memos.json 파일이 비어있어.
빈 파일이 있을 때도 오류 없이 동작해야 해.

어떻게 고쳐야 할까?
```

### Step 3. 수정 코드 확인 후 적용

AI가 준 수정 코드를 그대로 복사하기 전에 반드시 읽는다.

- 어디가 바뀌었는지 확인
- 내가 원하는 방향과 맞는지 확인
- 이해 안 되는 부분은 설명 요청

---

## 2. 실행은 되는데 결과가 틀릴 때

오류 없이 실행됐지만 결과가 기대와 다른 경우.

### 예시 시나리오

AI가 만든 `search_memo` 함수가 있다:

```python
def search_memo(memos, keyword):
    return [m for m in memos if keyword in m]
```

테스트해보니 문제 발견:
```python
memos = ["Python 공부", "python 연습", "GitHub 실습"]
print(search_memo(memos, "python"))
# 출력: ['python 연습']   ← "Python 공부"가 빠짐!
```

대소문자를 구분해서 "Python 공부"가 빠졌다.

### 수정 요청

```
search_memo 함수가 대소문자를 구분하고 있어.
"python"으로 검색하면 "Python 공부"도 나와야 하는데 안 나오네.
대소문자 구분 없이 검색하도록 수정해줘.
```

---

## 3. 내가 원하는 방식이 아닐 때

AI가 올바른 코드를 줬지만, 내 프로젝트 스타일이나 요구사항과 다를 때.

```
코드는 동작하는데, 우리 프로젝트에서는 함수가 결과를 print하지 않고
리스트로 반환해야 해. 출력은 main.py에서 직접 할 거야.
반환값만 주는 방식으로 바꿔줘.
```

---

## 4. Repair Loop 전체 예시

**목표**: 메모에서 날짜를 파싱하는 함수 만들기

**Round 1 - 첫 요청:**
```
Python으로 "2026-05-15 Python 공부" 형태의 문자열에서
날짜 부분을 datetime 객체로 변환하는 parse_date_from_memo(text) 함수를 만들어줘.
```

AI가 준 코드:
```python
from datetime import datetime

def parse_date_from_memo(text):
    date_str = text.split(" ")[0]
    return datetime.strptime(date_str, "%Y-%m-%d")
```

**Round 2 - 오류 발견:**
```python
print(parse_date_from_memo("Python 공부"))
# ValueError: time data 'Python' does not match format '%Y-%m-%d'
```

```
날짜로 시작하지 않는 메모를 넣었더니 오류가 났어.

오류: ValueError: time data 'Python' does not match format '%Y-%m-%d'

날짜가 없는 메모는 None을 반환하도록 수정해줘.
```

AI가 수정한 코드:
```python
from datetime import datetime

def parse_date_from_memo(text):
    try:
        date_str = text.split(" ")[0]
        return datetime.strptime(date_str, "%Y-%m-%d")
    except ValueError:
        return None
```

**Round 3 - 검증:**
```python
print(parse_date_from_memo("2026-05-15 Python 공부"))  # datetime 객체
print(parse_date_from_memo("Python 공부"))              # None
print(parse_date_from_memo(""))                         # None
```

모든 케이스가 예상대로 동작하면 완료.

---

## 5. 직접 고치기 vs AI에게 요청하기

| 상황 | 추천 방법 |
|------|----------|
| 오타나 간단한 변수명 수정 | 직접 수정 |
| 로직을 이해했고 한 줄만 고치면 됨 | 직접 수정 |
| 왜 오류인지 모름 | AI에게 요청 |
| 더 나은 방법이 있는지 모름 | AI에게 요청 |
| 오류 수정 후 다른 부분에 영향을 주는지 모름 | AI에게 확인 요청 |

직접 고칠 수 있다면 고치는 게 더 빠르다. 하지만 원인을 모르면 AI에게 설명을 먼저 요청한다.

---

## 6. 최종 검증 체크리스트

코드가 완성됐다고 생각할 때 아래를 확인한다.

- [ ] **정상 케이스**: 일반적인 입력으로 실행했을 때 예상한 결과가 나오는가
- [ ] **빈 입력**: 빈 리스트, 빈 문자열, `0` 등을 넣으면 어떻게 되는가
- [ ] **잘못된 입력**: 숫자 자리에 문자열, None 등을 넣으면 어떻게 되는가
- [ ] **경계값**: 최솟값, 최댓값 근처에서 올바르게 동작하는가
- [ ] **이해 확인**: 코드의 각 줄이 무엇을 하는지 설명할 수 있는가

---

## 7. 따라 하기 실습

### 실습 1. 의도적으로 오류 만들고 수정 루프 돌리기

아래 코드에는 3가지 버그가 있다. AI에게 하나씩 수정을 요청하면서 루프를 경험해본다.

```python
def calculate_stats(numbers):
    total = 0
    for n in numbers:
        total = total + n
    average = total / len(numbers)   # 버그 1: 빈 리스트면 ZeroDivisionError
    minimum = numbers[0]             # 버그 2: 빈 리스트면 IndexError
    for n in numbers:
        if n < minimum:
            minimum = n
    return average, minimum, total

# 버그 3: 반환값 순서가 average, minimum, total인데 아래에서 잘못 받고 있음
total, avg, mn = calculate_stats([5, 3, 8, 1])
print(f"합계: {total}, 평균: {avg}, 최솟값: {mn}")
```

AI에게 각 버그를 설명하고 수정을 요청한다.

### 실습 2. 결과가 틀린 경우 수정 요청

AI에게 "1~100 사이의 소수를 모두 출력하는 함수"를 요청하고, 결과를 직접 검증한다. 잘못된 소수가 포함되어 있으면 구체적으로 어떤 숫자가 틀렸는지 알려주고 수정을 요청한다.

---

## 확인 체크리스트

- [ ] 오류 메시지를 코드와 상황과 함께 AI에게 전달할 수 있는가
- [ ] "실행은 되는데 결과가 틀린" 케이스를 AI에게 설명할 수 있는가
- [ ] AI가 수정한 코드를 적용하기 전에 읽고 이해하는가
- [ ] 완성된 코드를 다양한 입력으로 직접 검증하는가

---

## 한 번 더 생각해 보기

1. AI가 같은 수정을 3번 해도 계속 틀린다면 어떻게 해야 할까?
2. "오류가 없는 코드 = 올바른 코드"가 아닌 이유는 무엇인가?
3. AI와의 대화를 어느 시점에 끊고 혼자 해결을 시도해야 할까?

---

## 다음 장

다음 장에서는 지금까지 배운 Python, GitHub, AI 도구를 모두 연결해서 작은 서비스를 처음부터 완성하는 실습을 한다.

---

## 참고 자료

- Claude 오류 디버깅 가이드 — https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
