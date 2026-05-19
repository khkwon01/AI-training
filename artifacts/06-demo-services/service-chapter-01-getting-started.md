# Chapter 01: 작은 서비스 처음부터 만들기

## 이 장에서 배우는 것

- "서비스"가 무엇인지, 작은 것부터 시작해도 되는 이유
- 기능을 정의하고 코드로 구현하고 실행하는 전체 흐름
- 메모 관리 서비스를 단계별로 만들어가는 방법
- 각 기능을 함수로 나누는 이유
- 앞에서 배운 Python, GitHub, AI 도구를 실제로 연결하는 방법

---

## 먼저 쉬운 설명

지금까지 배운 것들을 하나씩 떠올려 보자.

- Python으로 변수, 함수, 파일 다루기
- GitHub으로 코드 저장하고 관리하기
- AI에게 코드 도움 요청하기

이 모든 것을 "서비스 하나 만들기"라는 목표 아래 연결할 시간이다.

서비스라고 해서 거창한 게 아니다. "메모를 추가하고, 보고, 지울 수 있는 프로그램" 하나면 충분하다. 이 작은 서비스를 완성하는 과정에서 앞에서 배운 모든 개념이 실제로 쓰인다.

---

## 1. 만들 서비스 정의하기

### 메모 관리 서비스 명세

| 기능 | 설명 | 함수 이름 |
|------|------|----------|
| 메모 추가 | 새 메모를 저장한다 | `add_memo` |
| 메모 목록 보기 | 저장된 메모를 번호와 함께 출력한다 | `show_memos` |
| 메모 삭제 | 번호로 특정 메모를 삭제한다 | `delete_memo` |
| 파일에 저장 | 메모를 JSON 파일에 저장한다 | `save_to_file` |
| 파일에서 불러오기 | JSON 파일에서 메모를 불러온다 | `load_from_file` |

이 5개 함수를 하나씩 만들고 연결하면 서비스가 완성된다.

---

## 2. 프로젝트 구조 잡기

먼저 폴더와 파일을 만든다.

```
memo-service/
├── memo.py        ← 메인 코드
├── memos.json     ← 데이터 저장 파일 (자동 생성됨)
└── README.md      ← 서비스 설명
```

VS Code에서 `memo-service` 폴더를 열고 `memo.py` 파일을 만든다.

---

## 3. 단계별 구현

### 3-1. 메모 추가 함수

```python
# memo.py

memos = []  # 메모를 저장하는 리스트

def add_memo(text):
    """새 메모를 리스트에 추가한다."""
    if not text.strip():
        print("빈 메모는 추가할 수 없습니다.")
        return
    memos.append(text.strip())
    print(f"✓ 메모 추가됨: {text}")
```

테스트:

```python
add_memo("Python 공부하기")
add_memo("GitHub 연습하기")
add_memo("")  # 빈 메모 테스트
print(memos)
```

실행 결과:
```
✓ 메모 추가됨: Python 공부하기
✓ 메모 추가됨: GitHub 연습하기
빈 메모는 추가할 수 없습니다.
['Python 공부하기', 'GitHub 연습하기']
```

### 3-2. 메모 목록 보기 함수

```python
def show_memos():
    """저장된 메모를 번호와 함께 출력한다."""
    if not memos:
        print("저장된 메모가 없습니다.")
        return
    print("\n--- 메모 목록 ---")
    for i, memo in enumerate(memos, start=1):
        print(f"{i}. {memo}")
    print(f"총 {len(memos)}개\n")
```

### 3-3. 메모 삭제 함수

```python
def delete_memo(number):
    """번호로 특정 메모를 삭제한다."""
    if number < 1 or number > len(memos):
        print(f"잘못된 번호입니다. 1~{len(memos)} 사이로 입력하세요.")
        return
    removed = memos.pop(number - 1)
    print(f"✓ 삭제됨: {removed}")
```

### 3-4. 파일 저장/불러오기 함수

```python
import json
import os

FILE_PATH = "memos.json"

def save_to_file():
    """메모를 JSON 파일에 저장한다."""
    with open(FILE_PATH, "w", encoding="utf-8") as f:
        json.dump(memos, f, ensure_ascii=False, indent=2)
    print(f"✓ {len(memos)}개 메모를 파일에 저장했습니다.")

def load_from_file():
    """JSON 파일에서 메모를 불러온다."""
    global memos
    if not os.path.exists(FILE_PATH):
        return  # 파일이 없으면 그냥 넘어감
    with open(FILE_PATH, "r", encoding="utf-8") as f:
        memos = json.load(f)
    print(f"✓ {len(memos)}개 메모를 불러왔습니다.")
```

### 3-5. 메뉴 루프로 연결하기

```python
def run():
    """메모 서비스 메인 루프."""
    load_from_file()  # 시작할 때 저장된 메모 불러오기
    
    while True:
        print("\n[메모 서비스]")
        print("1. 메모 추가")
        print("2. 메모 목록 보기")
        print("3. 메모 삭제")
        print("4. 저장하고 종료")
        
        choice = input("선택 (1~4): ").strip()
        
        if choice == "1":
            text = input("메모 내용: ")
            add_memo(text)
        elif choice == "2":
            show_memos()
        elif choice == "3":
            show_memos()
            try:
                num = int(input("삭제할 번호: "))
                delete_memo(num)
            except ValueError:
                print("숫자를 입력하세요.")
        elif choice == "4":
            save_to_file()
            print("종료합니다.")
            break
        else:
            print("1~4 중에서 선택하세요.")

if __name__ == "__main__":
    run()
```

---

## 4. 전체 실행해보기

```bash
python3 memo.py
```

```
[메모 서비스]
1. 메모 추가
2. 메모 목록 보기
3. 메모 삭제
4. 저장하고 종료
선택 (1~4):
```

1을 눌러 메모 추가, 2를 눌러 목록 확인, 4를 눌러 저장 후 종료.
다시 실행하면 저장된 메모가 불러와진다.

---

## 5. GitHub에 올리기

```bash
git init
git add memo.py README.md
git commit -m "메모 서비스 초기 구현"
git remote add origin https://github.com/username/memo-service.git
git push -u origin main
```

---

## 6. AI로 기능 추가해보기

기본 서비스가 완성됐으면 AI에게 기능 추가를 요청해본다.

```
지금까지 만든 메모 서비스에 검색 기능을 추가하고 싶어.
memos 리스트에서 특정 단어가 포함된 메모만 출력하는
search_memo(keyword) 함수를 만들어줘.
```

받은 코드를 읽고, 테스트하고, 메뉴에 "5. 메모 검색" 항목을 직접 추가해본다.

---

## 따라 하기 실습

### 실습 1. memo.py 처음부터 직접 작성하기

위 코드를 그대로 복사하지 않고 각 함수를 직접 타이핑하며 작성한다.

### 실습 2. 전체 실행 및 기능 테스트

- 메모 3개 추가
- 목록 확인
- 1개 삭제
- 저장 후 종료
- 다시 실행해서 메모가 불러와지는지 확인

### 실습 3. GitHub에 올리기

저장소를 만들고 `memo.py`를 commit → push.

### 실습 4. AI로 검색 기능 추가하기

AI에게 `search_memo(keyword)` 함수를 요청하고, 받은 코드를 서비스에 연결한다.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| `global memos` 없이 리스트 재할당 | 함수 밖 memos가 바뀌지 않음 | `load_from_file` 안에 `global memos` 추가 |
| `enumerate` 시작을 0으로 | 번호가 0부터 시작해서 헷갈림 | `enumerate(memos, start=1)` 사용 |
| 파일 저장 없이 종료 | 다음 실행 시 메모가 없음 | 종료 전 `save_to_file()` 호출 |
| 잘못된 번호 입력 시 오류 | `IndexError` 발생 | `delete_memo` 안에 범위 체크 추가 |

---

## 확인 체크리스트

- [ ] 5개 함수를 모두 작성하고 각각 테스트했는가
- [ ] 메뉴 루프에서 모든 기능이 동작하는가
- [ ] 종료 후 다시 실행하면 저장된 메모가 불러와지는가
- [ ] GitHub에 commit하고 push했는가
- [ ] AI에게 새 기능을 요청하고 서비스에 추가했는가

---

## 한 번 더 생각해 보기

1. `if __name__ == "__main__":` 가 없으면 어떤 문제가 생길까?
2. 메모를 JSON 대신 텍스트 파일(`.txt`)에 저장하면 어떤 단점이 있을까?
3. 지금 서비스에 없는 기능 중 가장 먼저 추가하고 싶은 것은 무엇인가?

---

## 다음 장

다음 장에서는 이 서비스를 AWS Lambda에 올려서 인터넷에서 사용할 수 있게 만든다.

---

## 참고 자료

- Python json 모듈 — https://docs.python.org/3/library/json.html
- Python os 모듈 — https://docs.python.org/3/library/os.html
