## 이 장에서 배우는 것

- AI에게 서비스 전체를 맡기는 방식 vs. 단계별로 함께 만드는 방식의 차이
- 아이디어를 AI가 이해할 수 있는 프롬프트로 변환하는 법
- 첫 번째 코드를 받아서 실제로 실행해보는 경험
- 오류가 났을 때 AI에게 다시 물어보는 디버깅 루프
- 기능을 하나씩 추가하며 서비스를 점진적으로 완성하는 흐름

---

## 먼저 쉬운 설명

AI로 코드를 짜는 방법은 크게 두 가지입니다.

**방법 A — 한 번에 다 맡기기**
"할 일 관리 프로그램 전체를 만들어줘"라고 한 번에 요청하는 방식입니다.
빠르게 결과물이 나오지만, 코드가 길고 복잡해서 어디가 어떻게 동작하는지 이해하기 어렵습니다.
오류가 생겼을 때 어디를 고쳐야 할지 막막해집니다.

**방법 B — 단계별로 함께 만들기**
"먼저 할 일을 추가하는 기능만 만들어줘" → 실행 → "이번엔 삭제 기능을 추가해줘" → 실행 → …
처음엔 느려 보이지만, 각 단계에서 무슨 일이 일어나는지 눈으로 확인하면서 진행합니다.
오류가 생겨도 범위가 작아서 찾기 쉽습니다.

이 챕터에서는 **방법 B**를 사용해서 **할 일 관리 CLI(커맨드라인 앱)**를 처음부터 완성합니다.

> **CLI란?** 마우스 없이 터미널에 글자를 입력해서 사용하는 프로그램입니다.
> `python todo.py add "우유 사기"` 처럼 입력하면 동작합니다.

---

## 1. 프로젝트 기획 — AI에게 아이디어 설명하기

### 1-1. 기획 프롬프트를 쓰기 전에 생각할 것

AI에게 코드를 요청하기 전에 **세 가지**를 먼저 정합니다.

| 질문 | 예시 답변 |
|------|----------|
| 이 프로그램이 무엇을 하는가? | 할 일 목록을 관리한다 |
| 누가 어떻게 사용하는가? | 터미널에서 명령어로 사용한다 |
| 꼭 필요한 기능은 무엇인가? | 추가, 조회, 완료 표시, 삭제 |

### 1-2. 기획용 프롬프트 예시

아래 프롬프트를 그대로 AI(Claude, ChatGPT 등)에게 붙여넣어 보세요.

```
나는 파이썬을 막 배우기 시작한 초보자야.
터미널에서 사용하는 간단한 할 일 관리 프로그램을 만들고 싶어.

프로그램 이름: todo.py
필요한 기능:
1. 할 일 추가: python todo.py add "할 일 내용"
2. 목록 보기: python todo.py list
3. 완료 표시: python todo.py done 1  (1번 항목을 완료로 표시)
4. 삭제: python todo.py delete 1  (1번 항목 삭제)

데이터는 todo.json 파일에 저장해줘.

지금은 전체 코드를 주지 말고,
이 프로그램의 구조(파일 구성, 주요 함수 목록)만 먼저 설명해줘.
초보자가 이해할 수 있도록 짧게 설명해줘.
```

> **왜 전체 코드를 바로 달라고 하지 않나요?**
> 구조를 먼저 파악하면 나중에 코드가 길어져도 어디를 찾아야 할지 알 수 있습니다.
> AI가 제안한 구조가 맘에 들지 않으면 이 단계에서 방향을 바꿀 수 있습니다.

### 1-3. AI 응답 검증 방법

AI가 구조를 설명해주면 아래 항목을 확인하세요.

- [ ] 파일이 몇 개인지 명확한가?
- [ ] 각 기능이 어떤 함수로 구현되는지 나와 있는가?
- [ ] 데이터 저장 방식(JSON, CSV 등)이 명확한가?
- [ ] 이해가 안 되는 단어가 있으면 바로 물어보기

```
방금 설명한 구조에서 "argparse"가 뭔지 모르겠어.
초보자한테 한 줄로 설명해줘.
```

---

## 2. 첫 번째 코드 받기 — add 기능부터

### 2-1. 첫 코드 요청 프롬프트

구조를 확인했으면 이제 첫 번째 기능 코드를 요청합니다.
**한 번에 모든 기능을 요청하지 않고, add 기능 하나만 먼저 만듭니다.**

```
좋아. 이제 코드를 짜보자.
todo.py 파일을 만들되, 지금은 "할 일 추가(add)" 기능만 구현해줘.

조건:
- python todo.py add "할 일 내용" 으로 실행한다
- 데이터는 todo.json에 저장한다
- 저장 형식: [{"id": 1, "text": "할 일", "done": false}]
- 파이썬 초보자가 읽을 수 있도록 각 줄에 짧은 주석을 달아줘
- 실행 예시도 함께 보여줘
```

### 2-2. AI가 주는 코드 예시

AI는 아래와 같은 코드를 줄 것입니다.

```python
# todo.py — 할 일 추가 기능

import json        # JSON 파일을 읽고 쓰기 위한 모듈
import sys         # 터미널 인수(argv)를 읽기 위한 모듈
import os          # 파일 존재 여부 확인용 모듈

TODO_FILE = "todo.json"  # 데이터를 저장할 파일 이름

def load_todos():
    """저장된 할 일 목록을 파일에서 읽어온다."""
    if not os.path.exists(TODO_FILE):
        return []  # 파일이 없으면 빈 목록 반환
    with open(TODO_FILE, "r", encoding="utf-8") as f:
        return json.load(f)

def save_todos(todos):
    """할 일 목록을 파일에 저장한다."""
    with open(TODO_FILE, "w", encoding="utf-8") as f:
        json.dump(todos, f, ensure_ascii=False, indent=2)

def add_todo(text):
    """새 할 일을 목록에 추가한다."""
    todos = load_todos()

    # 새 항목의 id는 현재 목록 길이 + 1
    new_id = len(todos) + 1
    new_item = {"id": new_id, "text": text, "done": False}

    todos.append(new_item)
    save_todos(todos)
    print(f"추가됨: [{new_id}] {text}")

# --- 프로그램 진입점 ---
if __name__ == "__main__":
    # sys.argv[1]이 명령어, sys.argv[2]가 내용
    if len(sys.argv) < 3:
        print("사용법: python todo.py add '할 일 내용'")
        sys.exit(1)

    command = sys.argv[1]   # "add"
    argument = sys.argv[2]  # "할 일 내용"

    if command == "add":
        add_todo(argument)
    else:
        print(f"알 수 없는 명령어: {command}")
```

### 2-3. 코드를 받으면 반드시 할 것

1. 코드를 파일로 저장한다 (`todo.py`)
2. 바로 실행해본다
3. 결과가 기대와 같은지 확인한다

```bash
# 터미널에서 실행
python todo.py add "우유 사기"
# 예상 출력: 추가됨: [1] 우유 사기

python todo.py add "운동하기"
# 예상 출력: 추가됨: [2] 운동하기

cat todo.json
# 예상 출력:
# [
#   {"id": 1, "text": "우유 사기", "done": false},
#   {"id": 2, "text": "운동하기", "done": false}
# ]
```

---

## 3. 오류가 났을 때 — AI와 함께 디버깅하기

### 3-1. 흔히 마주치는 오류들

코드를 실행하다 보면 오류가 납니다. 당황하지 마세요. 오류 메시지를 그대로 AI에게 붙여넣으면 됩니다.

**오류 1: 파이썬이 설치되지 않은 경우**
```
'python'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는
배치 파일이 아닙니다.
```

**오류 2: 파일 인코딩 문제 (Windows)**
```
UnicodeDecodeError: 'cp949' codec can't decode byte 0xec in position 0
```

**오류 3: 인수를 빠뜨린 경우**
```
IndexError: list index out of range
```

### 3-2. 디버깅 프롬프트 예시

오류가 나면 아래 형식으로 AI에게 물어보세요.

```
아래 코드를 실행했더니 오류가 났어.

[실행한 명령어]
python todo.py add 우유 사기

[오류 메시지]
IndexError: list index out of range

[내가 사용하는 환경]
- 운영체제: Windows 11
- 파이썬 버전: 3.11

어디가 문제인지, 어떻게 고치면 되는지 알려줘.
수정된 코드 부분만 보여줘 (전체 코드 말고).
```

> **팁:** `우유 사기`처럼 공백이 있는 내용은 반드시 따옴표로 감싸야 합니다.
> `python todo.py add "우유 사기"`

### 3-3. AI 응답에서 주의할 점

| 상황 | 주의사항 |
|------|----------|
| AI가 코드 전체를 다시 준 경우 | 무엇이 바뀌었는지 직접 비교하고 이유를 물어보기 |
| AI가 새로운 라이브러리 설치를 권장한 경우 | 왜 필요한지 먼저 물어보고 판단하기 |
| 수정해도 같은 오류가 나는 경우 | 오류 메시지를 다시 붙여넣어 반복 요청하기 |
| AI의 설명이 너무 어려운 경우 | "초등학생한테 설명하듯이 다시 설명해줘"라고 요청하기 |

---

## 4. 기능 추가 — list, done, delete

### 4-1. 기능 추가 프롬프트 예시

add 기능이 동작하는 것을 확인했으면, 이제 나머지 기능을 추가합니다.

```
add 기능이 잘 동작해. 이제 기존 todo.py에 "list" 기능을 추가해줘.

요구사항:
- python todo.py list 로 실행
- 완료된 항목은 [✓], 미완료는 [ ] 로 표시
- 목록이 비어있으면 "할 일이 없습니다." 출력

기존 코드의 load_todos(), save_todos() 함수는 그대로 쓰고,
list_todos() 함수와 메인 분기 처리만 추가해줘.
```

```
이번엔 "done" 기능을 추가해줘.

요구사항:
- python todo.py done 1 로 실행 (1번 항목 완료 표시)
- 해당 id가 없으면 "해당 번호의 항목이 없습니다." 출력
- 완료 처리 후 "완료: [1] 우유 사기" 형식으로 출력

done_todo(item_id) 함수를 추가하는 방식으로 구현해줘.
```

```
마지막으로 "delete" 기능을 추가해줘.

요구사항:
- python todo.py delete 1 로 실행
- 삭제 전에 "정말 삭제할까요? (y/n)" 확인 메시지 출력
- y 입력 시 삭제, 그 외 입력 시 취소
- 삭제 후 남은 항목의 id를 1부터 다시 정렬해줘
```

### 4-2. 완성된 코드 예시

```python
# todo.py — 할 일 관리 CLI (완성본)

import json
import sys
import os

TODO_FILE = "todo.json"

def load_todos():
    if not os.path.exists(TODO_FILE):
        return []
    with open(TODO_FILE, "r", encoding="utf-8") as f:
        return json.load(f)

def save_todos(todos):
    with open(TODO_FILE, "w", encoding="utf-8") as f:
        json.dump(todos, f, ensure_ascii=False, indent=2)

def add_todo(text):
    todos = load_todos()
    new_id = len(todos) + 1
    todos.append({"id": new_id, "text": text, "done": False})
    save_todos(todos)
    print(f"추가됨: [{new_id}] {text}")

def list_todos():
    todos = load_todos()
    if not todos:
        print("할 일이 없습니다.")
        return
    for item in todos:
        mark = "✓" if item["done"] else " "
        print(f"[{mark}] {item['id']}. {item['text']}")

def done_todo(item_id):
    todos = load_todos()
    for item in todos:
        if item["id"] == item_id:
            item["done"] = True
            save_todos(todos)
            print(f"완료: [{item_id}] {item['text']}")
            return
    print("해당 번호의 항목이 없습니다.")

def delete_todo(item_id):
    todos = load_todos()
    target = next((t for t in todos if t["id"] == item_id), None)
    if not target:
        print("해당 번호의 항목이 없습니다.")
        return
    confirm = input(f"'{target['text']}' 를 삭제할까요? (y/n): ")
    if confirm.lower() != "y":
        print("취소됐습니다.")
        return
    todos = [t for t in todos if t["id"] != item_id]
    # id를 1부터 다시 정렬
    for i, item in enumerate(todos, start=1):
        item["id"] = i
    save_todos(todos)
    print(f"삭제됨: {target['text']}")

# --- 진입점 ---
if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("사용법: python todo.py [add|list|done|delete] [인수]")
        sys.exit(1)

    command = sys.argv[1]

    if command == "add":
        if len(sys.argv) < 3:
            print("사용법: python todo.py add '할 일 내용'")
        else:
            add_todo(sys.argv[2])
    elif command == "list":
        list_todos()
    elif command == "done":
        if len(sys.argv) < 3:
            print("사용법: python todo.py done [번호]")
        else:
            done_todo(int(sys.argv[2]))
    elif command == "delete":
        if len(sys.argv) < 3:
            print("사용법: python todo.py delete [번호]")
        else:
            delete_todo(int(sys.argv[2]))
    else:
        print(f"알 수 없는 명령어: {command}")
```

---

## 5. 서비스 완성 확인 — 전체 시나리오 테스트

### 5-1. 테스트 프롬프트

기능을 모두 추가했으면 AI에게 테스트 시나리오를 요청합니다.

```
todo.py 가 완성됐어.
초보자가 직접 따라할 수 있는 테스트 시나리오를 만들어줘.

형식:
- 실행 명령어
- 예상 출력

add 2개, list, done 1개, list 다시, delete 1개, list 다시
이 순서로 만들어줘.
```

### 5-2. 테스트 실행 예시

```bash
# 1. 할 일 추가
python todo.py add "장보기"
# 출력: 추가됨: [1] 장보기

python todo.py add "독서 30분"
# 출력: 추가됨: [2] 독서 30분

# 2. 목록 확인
python todo.py list
# 출력:
# [ ] 1. 장보기
# [ ] 2. 독서 30분

# 3. 1번 완료 처리
python todo.py done 1
# 출력: 완료: [1] 장보기

# 4. 목록 재확인
python todo.py list
# 출력:
# [✓] 1. 장보기
# [ ] 2. 독서 30분

# 5. 1번 삭제
python todo.py delete 1
# 출력: '장보기' 를 삭제할까요? (y/n): y
#       삭제됨: 장보기

# 6. 최종 목록 확인
python todo.py list
# 출력:
# [ ] 1. 독서 30분
```

---

## 따라 하기 실습

### 실습 1: 기획하고 구조 잡기

**목표:** AI와 대화하며 `todo.py`의 구조를 정한다.

1. 새 폴더 `my-todo/` 를 만든다
2. 아래 프롬프트를 AI에게 보낸다

```
파이썬 초보자야. 터미널에서 동작하는 할 일 관리 프로그램을 만들고 싶어.
기능: 추가, 조회, 완료 표시, 삭제
데이터는 JSON 파일에 저장

전체 코드는 아직 필요 없고,
- 파일 구성 (몇 개의 파일이 필요한가)
- 필요한 함수 목록과 각 함수가 하는 일
만 알려줘. 표 형식으로 정리해줘.
```

3. AI의 답변을 읽고 이해가 안 되는 단어를 하나씩 물어본다
4. 구조가 이해됐으면 `구조.md` 파일로 정리해둔다

---

### 실습 2: add 기능 구현 및 실행

**목표:** `todo.py`에 add 기능을 구현하고 실제로 실행해본다.

1. 실습 1에서 정한 구조를 AI에게 보여주며 요청한다

```
아까 정한 구조대로, 지금은 add 기능만 구현해줘.
[실습 1에서 받은 구조 내용을 여기에 붙여넣기]

초보자용 주석 포함, 실행 예시 포함.
```

2. 받은 코드를 `my-todo/todo.py` 로 저장한다
3. 터미널에서 실행한다

```bash
cd my-todo
python todo.py add "첫 번째 할 일"
python todo.py add "두 번째 할 일"
cat todo.json
```

4. 오류가 나면 오류 메시지를 AI에게 그대로 붙여넣어 해결한다

---

### 실습 3: 나머지 기능 추가 및 전체 테스트

**목표:** list, done, delete 기능을 순서대로 추가하고 전체 흐름을 테스트한다.

1. list → done → delete 순서로 각각 AI에게 요청한다
   (기능 하나 추가 → 바로 실행 테스트 → 다음 기능 추가)
2. 모든 기능이 완성되면 아래 테스트 시나리오를 처음부터 끝까지 실행한다

```bash
python todo.py add "공부하기"
python todo.py add "운동하기"
python todo.py add "책 읽기"
python todo.py list
python todo.py done 2
python todo.py list
python todo.py delete 1
python todo.py list
```

3. 결과가 예상과 다른 부분이 있으면 AI에게 물어본다
4. 완성된 `todo.py`를 `my-todo/` 폴더에 저장한다

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|------|------------|----------|
| 공백 있는 내용을 따옴표 없이 입력 | `IndexError: list index out of range` | `python todo.py add "우유 사기"` — 따옴표 필수 |
| 한글 저장 오류 (Windows) | `UnicodeEncodeError: 'cp949'` | `open()` 에 `encoding="utf-8"` 추가 확인 |
| `python` 명령어를 못 찾음 | `'python'은(는) 내부 명령이 아닙니다` | `python3` 로 바꿔서 실행, 또는 Python 설치 확인 |
| id에 숫자 대신 문자열 전달 | `TypeError: '<' not supported between instances of 'int' and 'str'` | `int(sys.argv[2])` 로 형변환 확인 |
| todo.json이 깨진 경우 | `json.decoder.JSONDecodeError` | `todo.json` 파일 삭제 후 재실행 |
| AI가 준 코드를 부분만 복사 | 실행은 되지만 기능이 빠짐 | 코드 블록 전체를 복사했는지 확인 |
| 기능 추가 시 기존 코드를 덮어씀 | 이전 기능이 사라짐 | AI에게 "기존 코드에 추가하는 방식으로" 명시 |

---

## 확인 체크리스트

- [ ] AI에게 구조를 먼저 물어보고, 코드를 나중에 요청했다
- [ ] `python todo.py add "할 일"` 이 정상 동작하고 `todo.json`이 생성된다
- [ ] `python todo.py list` 가 목록을 출력한다
- [ ] `python todo.py done 1` 이 완료 표시를 업데이트한다
- [ ] `python todo.py delete 1` 이 삭제 확인 후 항목을 제거한다
- [ ] 오류가 났을 때 메시지를 그대로 AI에게 붙여넣어 해결했다
- [ ] 기능 하나 추가할 때마다 바로 실행해서 확인했다
- [ ] 전체 테스트 시나리오를 처음부터 끝까지 실행했다

---

## 한 번 더 생각해 보기

1. **"한 번에 다 맡기기"와 "단계별로 함께 만들기"** 중 어떤 방식이 더 편했나요? 오류가 났을 때 어떤 차이가 있었나요?

2. AI가 코드를 줬을 때, 코드를 **그냥 복사**하는 것과 **한 줄씩 읽어보는 것** 중 어떤 게 나중에 더 도움이 될까요? 이번 실습에서 어느 정도까지 코드를 이해했나요?

3. 이번에 만든 `todo.py`에 **기능을 하나 더 추가**한다면 무엇을 추가하고 싶나요? AI에게 어떻게 요청할지 프롬프트를 스스로 작성해보세요.

---

## 혼자 해보기 미션 (숙제)

이번 챕터에서 만든 `todo.py`를 기반으로 아래 미션 중 하나를 선택해서 직접 해보세요.

**미션 A (쉬움):** 우선순위 기능 추가
- `python todo.py add "공부하기" high` 처럼 우선순위(high/medium/low)를 함께 저장
- `list` 출력 시 우선순위도 같이 표시

**미션 B (보통):** 마감일 기능 추가
- `python todo.py add "과제 제출" 2026-05-25` 처럼 마감일 저장
- `list` 출력 시 오늘 이후 마감인 항목에 `⚠️ 마감 임박` 표시

**미션 C (도전):** `search` 기능 추가
- `python todo.py search "공부"` 로 키워드 검색
- 검색 결과만 출력

> **힌트:** 각 미션 전에 "요구사항을 먼저 정리한 프롬프트"를 써서 AI에게 보내보세요.

---

## 다음 장

다음 장에서는 이번에 만든 CLI 프로그램에 **AI가 자동으로 오류를 잡고 코드를 개선하는 방법(AI 리뷰 루프)**을 배웁니다.