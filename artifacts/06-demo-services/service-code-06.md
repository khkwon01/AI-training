## 이 장에서 배우는 것

- 키워드 검색 함수가 무엇인지 설명할 수 있다
- `notes.json`에 저장된 메모 중 특정 단어가 포함된 것만 골라낼 수 있다
- 앞서 만든 저장·불러오기·삭제 흐름과 검색이 어떻게 이어지는지 이해한다
- 검색 결과가 없을 때 어떻게 처리하는지 안다

---

## 먼저 쉬운 설명

지금까지 우리는 메모를 **저장**하고, **불러오고**, **삭제**하는 기능을 만들었습니다.  
그런데 메모가 100개쯤 쌓이면 어떻게 될까요?  
원하는 메모를 찾으려고 전체를 하나씩 눈으로 훑어봐야 할까요?

그래서 필요한 것이 **검색(search)** 입니다.  
검색은 "이 단어가 들어 있는 메모만 보여 줘"라고 컴퓨터에게 부탁하는 것입니다.  
마치 책에서 특정 단어를 Ctrl+F로 찾는 것과 같습니다.

이 장에서는 딱 하나, **키워드 검색 함수** 만 만듭니다.  
작지만, 앞의 다섯 장에서 쌓아 온 `notes.json` 흐름 위에 자연스럽게 얹힙니다.

---

## 1. 지금까지 만든 코드 구조 복습

이전 장에서 완성한 `notes_service.py`는 이런 모양이었습니다.

```python
# notes_service.py  (1~5장에서 만든 것)
import json
import os

FILENAME = "notes.json"

def load_notes():
    if not os.path.exists(FILENAME):
        return []
    with open(FILENAME, "r", encoding="utf-8") as f:
        return json.load(f)

def save_notes(notes):
    with open(FILENAME, "w", encoding="utf-8") as f:
        json.dump(notes, f, ensure_ascii=False, indent=2)

def add_note(text):
    notes = load_notes()
    notes.append(text)
    save_notes(notes)

def delete_note(index):
    notes = load_notes()
    notes.pop(index)
    save_notes(notes)
```

이 파일에 검색 함수를 **추가**하는 것이 이번 장의 목표입니다.  
기존 코드는 전혀 건드리지 않아도 됩니다.

---

## 2. 검색 함수 만들기

**키워드 검색**이란, 메모 목록을 처음부터 끝까지 살펴보면서  
특정 단어(`keyword`)가 포함된 메모만 새 목록으로 모아 돌려주는 것입니다.

```python
# notes_service.py 에 아래 함수를 추가합니다

def search_notes(keyword):
    notes = load_notes()        # ① 저장된 메모 전체를 불러온다
    result = []                 # ② 결과를 담을 빈 목록을 만든다

    for note in notes:          # ③ 메모를 하나씩 꺼낸다
        if keyword in note:     # ④ 키워드가 포함되어 있으면
            result.append(note) # ⑤ 결과 목록에 추가한다

    return result               # ⑥ 결과 목록을 돌려준다
```

> **핵심 한 줄:**  
> `if keyword in note` — 파이썬에서 `in`은 "앞의 것이 뒤의 것 안에 들어 있니?"를 확인합니다.  
> `"고양이" in "나는 고양이를 좋아해"` → `True`

---

## 3. 검색 결과가 없을 때 처리하기

검색 결과가 없으면 빈 목록 `[]`이 돌아옵니다.  
그냥 두어도 오류는 나지 않지만, 사용자가 "왜 아무것도 안 나오지?" 하고 헷갈릴 수 있습니다.  
결과를 출력하는 쪽에서 빈 목록을 친절하게 안내해 주세요.

```python
# main.py  (또는 테스트용 실행 파일)
from notes_service import add_note, search_notes

# 메모 몇 개를 미리 추가합니다
add_note("오늘 고양이를 만났다")
add_note("내일 도서관에 가야 한다")
add_note("고양이 사료를 사야 한다")

# 검색 실행
keyword = "고양이"
found = search_notes(keyword)

if found:                          # 결과가 있을 때
    print(f"'{keyword}' 검색 결과:")
    for i, note in enumerate(found):
        print(f"  {i+1}. {note}")
else:                              # 결과가 없을 때
    print(f"'{keyword}'(이)가 포함된 메모가 없습니다.")
```

**예상 출력:**
```
'고양이' 검색 결과:
  1. 오늘 고양이를 만났다
  2. 고양이 사료를 사야 한다
```

---

## 4. 전체 흐름 한눈에 보기

```
notes.json
    │
    ▼
load_notes()  ←──────────────────────────┐
    │                                    │
    ▼                                    │
[메모1, 메모2, 메모3, ...]               │
    │                                    │
    ▼  keyword in note?                  │
결과 목록 result []                      │
    │                                    │
    ▼                                    │
출력 또는 "없음" 안내      (원본 파일은 변경 없음)
```

검색은 파일을 **읽기만** 합니다.  
저장·삭제와 달리 `notes.json`을 절대 변경하지 않습니다.

---

## 따라 하기 실습

### 실습 1 — 함수 추가하고 기본 검색 실행하기

1. 이전 장에서 만든 `notes_service.py`를 엽니다.
2. 파일 맨 아래에 `search_notes` 함수를 붙여 넣습니다.
3. 아래 파일을 새로 만들어 실행합니다.

```python
# run_search.py
from notes_service import add_note, search_notes

add_note("파이썬 공부하기")
add_note("장보기: 우유, 달걀")
add_note("파이썬 프로젝트 제출")

result = search_notes("파이썬")
print("검색 결과:", result)
```

**예상 출력:**
```
검색 결과: ['파이썬 공부하기', '파이썬 프로젝트 제출']
```

---

### 실습 2 — 없는 키워드 검색해 보기

실습 1을 실행한 뒤, 키워드를 없는 단어로 바꿔 봅니다.

```python
# run_search.py 아래에 추가
result2 = search_notes("자전거")

if result2:
    print("검색 결과:", result2)
else:
    print("검색 결과가 없습니다.")
```

**예상 출력:**
```
검색 결과가 없습니다.
```

---

### 실습 3 — 대소문자·공백에 주의하며 검색하기

파이썬의 `in`은 대소문자를 구분합니다.  
아래 코드를 추가해서 직접 확인해 보세요.

```python
# run_search.py 아래에 추가
add_note("Python is fun")

result3 = search_notes("python")   # 소문자
result4 = search_notes("Python")   # 대문자 P

print("소문자 python 검색:", result3)
print("대문자 Python 검색:", result4)
```

**예상 출력:**
```
소문자 python 검색: []
대문자 Python 검색: ['Python is fun']
```

> 대소문자를 무시하고 검색하려면 `if keyword.lower() in note.lower():` 로 바꾸면 됩니다.  
> 이건 다음 단계에서 도전해 보세요!

---

## 자주 하는 실수

| 실수 | 오류 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `search_notes`를 임포트하지 않음 | `ImportError: cannot import name 'search_notes'` | `notes_service.py`에 함수를 추가했는지 확인한다 |
| `notes.json`이 없는 상태에서 검색 | 오류 없이 빈 목록 `[]` 반환 | 정상 동작이다. 먼저 `add_note`로 메모를 추가하자 |
| `keyword in notes` 처럼 목록 전체에 `in` 사용 | 결과가 항상 `False` 또는 정확히 일치할 때만 `True` | 반복문 안에서 `keyword in note` (개별 메모)로 바꾼다 |
| 대소문자 차이로 검색 결과 없음 | 오류 없이 빈 목록 `[]` 반환 | `keyword.lower() in note.lower()` 로 변환 후 비교한다 |
| `result`를 반환하지 않고 함수 종료 | 함수 밖에서 `None`이 반환됨 | 함수 마지막에 `return result` 가 있는지 확인한다 |

---

## 확인 체크리스트

- [ ] `search_notes(keyword)` 함수를 `notes_service.py`에 추가했다
- [ ] `load_notes()`로 메모를 불러온 뒤, `for` 반복문으로 하나씩 확인하는 구조를 이해했다
- [ ] `keyword in note` 가 무엇을 검사하는지 설명할 수 있다
- [ ] 검색 결과가 없을 때 빈 목록 `[]`이 돌아오는 것을 직접 확인했다
- [ ] 검색 함수는 `notes.json` 파일을 변경하지 않는다는 것을 알고 있다
- [ ] 실습 1~3을 모두 실행하고 예상 출력과 일치하는 것을 확인했다

---

## 한 번 더 생각해 보기

1. `search_notes("고양이")` 와 `search_notes("고 양이")` 의 결과가 다른 이유는 무엇일까요?  
   공백 한 칸이 검색 결과에 어떤 영향을 줄지 실제로 실험해 보세요.

2. 지금 검색 함수는 키워드가 **하나**일 때만 동작합니다.  
   만약 "파이썬"과 "공부" 두 단어가 **모두** 들어간 메모를 찾고 싶다면 코드를 어떻게 바꿔야 할까요?

3. `search_notes`는 `notes.json`을 읽기만 합니다.  
   그런데 만약 메모가 10,000개라면 매번 파일 전체를 읽는 것이 빠를까요, 느릴까요?  
   더 빠르게 만들 수 있는 방법이 있을지 생각해 보세요.

---

## 다음 장

다음 장에서는 저장·불러오기·삭제·검색 기능을 하나의 간단한 메뉴 인터페이스로 묶어서, 터미널에서 직접 선택해 실행할 수 있는 작은 완성형 서비스를 만들어 봅니다.