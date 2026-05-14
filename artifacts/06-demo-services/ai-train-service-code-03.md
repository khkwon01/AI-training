## 이 장에서 배우는 것

- `json` 모듈을 불러오고 사용하는 방법
- 메모 목록을 JSON 파일에 저장하는 방법 (`json.dump`)
- JSON 파일을 읽어 메모 목록을 복원하는 방법 (`json.load`)
- 파일이 없을 때 안전하게 처리하는 방법
- 앞서 만든 메모 서비스에 저장 기능을 붙이는 방법

---

## 먼저 쉬운 설명

지난 장까지 우리는 메모를 **리스트(list)** 에 저장했습니다. 리스트는 편리하지만, 프로그램을 끄면 모든 내용이 사라집니다. 친구가 열심히 적은 메모가 컴퓨터를 재시작하자마자 없어진다면 얼마나 실망스러울까요?

이 문제를 해결하는 가장 간단한 방법이 **JSON 파일에 저장하기**입니다. JSON은 메모장으로 열어도 읽을 수 있는 텍스트 형식이라서, 파이썬 리스트나 딕셔너리를 그대로 파일에 담을 수 있습니다. 프로그램을 껐다 다시 켜도 파일에서 데이터를 불러오면 그대로 복원됩니다.

---

## 1. `json` 모듈이란?

파이썬에는 JSON을 다루는 도구가 기본으로 내장되어 있습니다. 따로 설치하지 않아도 됩니다.

```python
import json  # 이 한 줄로 JSON 기능을 전부 쓸 수 있습니다
```

`json` 모듈에서 가장 자주 쓰는 함수 두 가지를 기억하세요.

| 함수 | 하는 일 |
|------|---------|
| `json.dump(data, file)` | 파이썬 데이터를 파일에 JSON 형식으로 씁니다 |
| `json.load(file)` | JSON 파일을 읽어 파이썬 데이터로 돌려줍니다 |

---

## 2. 메모 목록을 파일에 저장하기

저장 기능을 함수로 분리하면 나중에 고치기 쉬워집니다. 파일 이름은 `notes_service.py`입니다.

```python
import json

SAVE_FILE = "notes.json"  # 저장할 파일 이름을 상수로 정의합니다

def save_notes(notes):
    """메모 리스트를 JSON 파일에 저장합니다."""
    with open(SAVE_FILE, "w", encoding="utf-8") as f:
        json.dump(notes, f, ensure_ascii=False, indent=2)
    print(f"메모 {len(notes)}개를 '{SAVE_FILE}'에 저장했습니다.")
```

코드를 한 줄씩 읽어봅시다.

- `open(SAVE_FILE, "w", encoding="utf-8")` — 파일을 쓰기(`"w"`) 모드로 엽니다. 한글이 깨지지 않도록 `encoding="utf-8"`을 씁니다.
- `with` 블록이 끝나면 파일이 자동으로 닫힙니다. 직접 `f.close()`를 부를 필요가 없습니다.
- `ensure_ascii=False` — 한글을 `\uC548\uB155` 같은 코드가 아닌 그대로 저장합니다.
- `indent=2` — 파일을 열었을 때 들여쓰기가 생겨 사람이 읽기 편해집니다.

---

## 3. 파일에서 메모 목록 불러오기

프로그램을 시작할 때 파일이 있으면 읽고, 없으면 빈 리스트를 돌려줍니다.

```python
def load_notes():
    """JSON 파일에서 메모 리스트를 불러옵니다. 파일이 없으면 빈 리스트를 반환합니다."""
    try:
        with open(SAVE_FILE, "r", encoding="utf-8") as f:
            notes = json.load(f)
        print(f"저장된 메모 {len(notes)}개를 불러왔습니다.")
        return notes
    except FileNotFoundError:
        print("저장 파일이 없습니다. 새로 시작합니다.")
        return []
```

`try / except FileNotFoundError` 패턴이 핵심입니다. 처음 실행할 때는 파일이 없을 수밖에 없으므로, 오류를 잡아서 빈 리스트를 반환합니다. 오류를 무시하는 게 아니라 **예상되는 상황을 안전하게 처리**하는 것입니다.

---

## 4. 이전 코드와 연결하기

2장에서 만든 `add_note`, `list_notes` 함수에 저장·불러오기를 붙여봅시다.

```python
import json

SAVE_FILE = "notes.json"

def load_notes():
    try:
        with open(SAVE_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return []

def save_notes(notes):
    with open(SAVE_FILE, "w", encoding="utf-8") as f:
        json.dump(notes, f, ensure_ascii=False, indent=2)

# --- 프로그램 시작 시 메모 불러오기 ---
notes = load_notes()

def add_note(text):
    """메모를 추가하고 바로 파일에 저장합니다."""
    notes.append({"id": len(notes) + 1, "text": text})
    save_notes(notes)  # 추가할 때마다 저장!

def list_notes():
    """저장된 메모를 모두 출력합니다."""
    if not notes:
        print("메모가 없습니다.")
        return
    for note in notes:
        print(f"[{note['id']}] {note['text']}")

# --- 간단한 테스트 ---
if __name__ == "__main__":
    add_note("오늘 점심은 비빔밥")
    add_note("파이썬 JSON 공부하기")
    list_notes()
```

실행 결과:

```
저장된 메모 0개를 불러왔습니다.   # 또는 기존 메모 개수
[1] 오늘 점심은 비빔밥
[2] 파이썬 JSON 공부하기
```

저장된 `notes.json` 파일 내용:

```json
[
  {
    "id": 1,
    "text": "오늘 점심은 비빔밥"
  },
  {
    "id": 2,
    "text": "파이썬 JSON 공부하기"
  }
]
```

---

## 따라 하기 실습

### 실습 1 — 저장·불러오기 기본 확인

1. 위 코드를 `notes_service.py`로 저장합니다.
2. 터미널에서 실행합니다.
   ```
   python notes_service.py
   ```
3. 같은 폴더에 `notes.json`이 생겼는지 확인합니다. 메모장이나 VS Code로 열어보세요.

---

### 실습 2 — 프로그램을 껐다 켜도 데이터가 남아있는지 확인

1. `if __name__ == "__main__":` 블록에서 `add_note` 두 줄을 지우고 `list_notes()`만 남깁니다.
   ```python
   if __name__ == "__main__":
       list_notes()
   ```
2. 다시 실행합니다.
   ```
   python notes_service.py
   ```
3. 실습 1에서 입력한 메모가 그대로 출력되면 성공입니다.

---

### 실습 3 — 메모 삭제 후 파일 업데이트

1. `delete_note(note_id)` 함수를 작성합니다.
   ```python
   def delete_note(note_id):
       global notes
       notes = [n for n in notes if n["id"] != note_id]
       save_notes(notes)
       print(f"메모 {note_id}번을 삭제했습니다.")
   ```
2. `if __name__ == "__main__":` 블록에서 `delete_note(1)`을 호출합니다.
3. `notes.json`을 열어 1번 메모가 사라졌는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 실제 오류 메시지 | 해결 방법 |
|------|-----------------|-----------|
| `open`에 `encoding` 생략 | `UnicodeDecodeError: 'cp949' codec can't decode byte ...` | `encoding="utf-8"` 을 반드시 추가하세요 |
| `"w"` 모드로 읽으려 할 때 | `io.UnsupportedOperation: not readable` | 읽을 땐 `"r"`, 쓸 땐 `"w"` 입니다 |
| `FileNotFoundError` 처리 안 함 | `FileNotFoundError: [Errno 2] No such file or directory: 'notes.json'` | `try / except FileNotFoundError` 로 감싸세요 |
| `json.dump` 결과를 변수에 저장하려 함 | 변수가 `None` | `json.dump`는 반환값이 없습니다. 파일 객체에 직접 씁니다 |
| `indent` 값을 문자열로 씀 | `TypeError: 'str' object cannot be interpreted as an integer` | `indent=2` 처럼 정수로 씁니다 |

---

## 확인 체크리스트

- [ ] `import json` 한 줄로 JSON 모듈을 사용할 수 있다는 것을 안다.
- [ ] `json.dump`와 `json.load`의 차이를 말할 수 있다.
- [ ] `with open(...) as f:` 패턴이 파일을 자동으로 닫아준다는 것을 안다.
- [ ] `ensure_ascii=False`가 없으면 한글이 이상하게 저장된다는 것을 안다.
- [ ] `FileNotFoundError`를 `try / except`로 잡아 빈 리스트를 반환하는 코드를 직접 쓸 수 있다.
- [ ] `notes_service.py`를 실행한 뒤 `notes.json` 파일을 직접 열어 내용을 확인했다.
- [ ] 프로그램을 껐다 다시 켜도 메모가 유지되는 것을 확인했다.

---

## 한 번 더 생각해 보기

1. `save_notes`를 `add_note` 안에서 호출합니다. 메모를 100개 연속으로 추가한다면 파일을 100번 씁니다. 성능 면에서 어떤 문제가 생길까요? 어떻게 바꾸면 좋을까요?

2. `notes.json` 파일을 텍스트 편집기로 직접 열어서 내용을 일부러 깨뜨린 뒤 (예: 중괄호를 지워서) 프로그램을 실행하면 어떤 오류가 발생하나요? 그 오류를 `except` 로 잡으려면 어떤 예외 클래스를 써야 할까요?

3. 지금은 `id`를 `len(notes) + 1`로 만듭니다. 메모를 하나 삭제하고 새 메모를 추가하면 `id`가 중복될 수 있습니다. 어떤 방식으로 `id`를 생성하면 중복을 피할 수 있을까요?

---

## 다음 장

다음 장에서는 지금까지 만든 저장 기능을 Flask 웹 서버와 연결해서, 브라우저에서 메모를 추가하고 삭제할 수 있는 작은 HTTP API를 완성합니다.