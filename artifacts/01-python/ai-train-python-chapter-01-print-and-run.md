# ai-train Python Basics Chapter 01: Python 실행과 print

## 이 장에서 배우는 것

- Python이 무엇인지, 왜 배우는지
- Python 코드를 어떻게 실행하는지
- `print()`로 화면에 글자를 출력하는 방법
- 따옴표를 쓰는 이유

---

## 먼저 쉬운 설명

Python은 컴퓨터에게 일을 시키는 언어다.

예를 들어 "화면에 안녕이라고 써줘"를 Python으로 표현하면 이렇다.

```python
print("안녕")
```

Python은 왜 배우나?

- 문법이 영어 문장처럼 읽히고, 처음 배우기에 어렵지 않다
- AI 활용, 데이터 처리, 웹 개발 등 다양한 곳에서 쓴다
- 초보자가 "코딩이 뭔지"를 느끼기에 가장 빠른 언어 중 하나다

---

## 1. Python 실행 환경 확인

Python을 실행하려면 VS Code 터미널을 사용한다.

VS Code에서 터미널을 여는 방법:
- 메뉴: **View > Terminal**
- 단축키 (Mac): `Ctrl + `` ` ` (백틱)
- 단축키 (Windows): `Ctrl + `` ` ` (백틱)

터미널이 열리면 Python이 설치되어 있는지 확인한다.

```bash
python3 --version
```

이렇게 나오면 정상이다.

```
Python 3.11.4
```

버전 숫자는 달라도 괜찮다. `Python 3.` 으로 시작하면 된다.

---

## 2. 첫 번째 Python 파일 만들기

VS Code에서 `hello.py` 파일을 만든다.

파일 만드는 순서:
1. VS Code 왼쪽 탐색기(Explorer) 패널에서 마우스 오른쪽 클릭
2. **New File** 선택
3. 파일 이름: `hello.py` 입력 후 Enter

파일 이름 끝에 `.py`를 반드시 붙여야 한다. `.py`가 없으면 Python 파일로 인식하지 않는다.

---

## 3. `print()`란 무엇인가

`print()`는 화면에 내용을 출력하는 명령이다.

```python
print("Hello, Python")
```

실행하면 이렇게 보인다.

```
Hello, Python
```

`print` 뒤에 반드시 괄호 `()`가 있어야 한다.
괄호 안에는 출력할 내용을 넣는다.

---

## 4. 따옴표를 왜 쓰나

글자(텍스트)를 Python에 전달할 때는 따옴표로 감싸야 한다.

```python
print("안녕하세요")
print('Hello')
```

큰따옴표 `"..."` 와 작은따옴표 `'...'` 둘 다 사용할 수 있다. 어느 쪽이든 짝이 맞으면 된다.

따옴표 없이 쓰면 오류가 난다.

```python
print(안녕)   # 오류 발생
```

```
NameError: name '안녕' is not defined
```

Python은 따옴표가 없으면 `안녕`을 변수나 명령으로 이해하려고 한다. 글자를 출력하고 싶을 때는 항상 따옴표로 감싸야 한다.

---

## 5. 여러 줄 출력하기

`print()`를 여러 번 쓰면 여러 줄이 출력된다.

```python
print("이름: Mina")
print("나이: 14")
print("좋아하는 언어: Python")
```

출력 결과:

```
이름: Mina
나이: 14
좋아하는 언어: Python
```

---

## 6. 파일 실행하기

`hello.py`에 코드를 작성한 뒤 VS Code 터미널에서 실행한다.

```bash
python3 hello.py
```

실행 결과가 터미널에 나타난다.

**주의**: `python3 hello.py`를 실행할 때는 터미널의 현재 위치가 `hello.py` 파일이 있는 폴더여야 한다.

현재 위치 확인:
```bash
pwd
```

파일이 있는 폴더로 이동:
```bash
cd python-study
```

---

## 7. 따라 하기 실습

### 실습 1. `hello.py` 파일 만들고 실행하기

1. VS Code에서 `python-study` 폴더를 연다
2. 새 파일 `hello.py`를 만든다
3. 아래 코드를 그대로 입력한다

```python
print("Hello, Python")
print("코딩 공부 시작!")
```

4. 저장한다 (단축키: `Cmd + S` / `Ctrl + S`)
5. 터미널에서 실행한다

```bash
python3 hello.py
```

예상 출력:

```
Hello, Python
코딩 공부 시작!
```

### 실습 2. 자기소개 출력하기

`intro.py` 파일을 새로 만들고, 이름과 좋아하는 것 2가지를 출력해 보자.

```python
print("이름: (내 이름)")
print("좋아하는 것: (내용)")
print("배우고 싶은 것: Python")
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 예시 | 해결 방법 |
|------|----------------|----------|
| `print` 뒤 괄호 없음 (`print "안녕"`) | `SyntaxError` | `print("안녕")`으로 수정 |
| 따옴표를 열고 닫지 않음 (`print("안녕`) | `SyntaxError: EOL while scanning string literal` | 따옴표를 짝 맞춰 닫기 |
| 파일을 저장하지 않고 실행 | 이전 내용이 실행됨 | `Cmd+S` / `Ctrl+S`로 저장 후 실행 |
| 파일 이름에 `.py` 없이 저장 | 터미널에서 `python3 hello` 실행 시 오류 | 파일 이름에 `.py` 포함 |
| 터미널 위치가 파일과 다른 폴더 | `No such file or directory` | `cd` 명령으로 파일 폴더로 이동 |

---

## 확인 체크리스트

- [ ] `python3 --version` 명령이 `Python 3.x` 로 출력되는가
- [ ] VS Code에서 `.py` 파일을 만들 수 있는가
- [ ] `print("내용")`을 작성하고 실행할 수 있는가
- [ ] 글자를 출력할 때 따옴표가 왜 필요한지 설명할 수 있는가
- [ ] `python3 파일이름.py` 형태로 실행할 수 있는가

---

## 한 번 더 생각해 보기

1. `print()` 안에 따옴표 없이 숫자를 쓰면 어떻게 될까? `print(42)` 를 실행해 보자.
2. `print("안녕")` 과 `print('안녕')` 의 결과는 같을까 다를까?
3. `print()` 안에 아무것도 안 쓰면 어떻게 될까?

---

## 다음 장

다음 장에서는 **변수**를 배운다. 변수는 값을 이름 붙여 저장해 두는 방법이다.
`print()` 안에 직접 글자를 쓰는 대신, 변수에 저장한 값을 출력하는 법을 배운다.

## 전체 Python 학습 순서 안내

이 장은 Python의 시작점이다. 전체 챕터는 아래 순서로 이어진다.

| 챕터 | 주제 |
|------|------|
| 01 | print와 실행 ← 지금 여기 |
| 02 | 변수 |
| 03 | 자료형 (int, str, bool) |
| 04 | 파일과 JSON |
| 05 | 디버깅 |
| 06~10 | 프로젝트 구조, 환경, 예외, venv |
| 11 | 조건문 (if/elif/else) |
| 12 | 반복문 (for/while) |
| 13 | 함수 (def, return) |
| 14 | class와 object |
| 15 | module과 import |
| 16 | 터미널/실행버튼/디버그 비교 |

전체 학습 지도: [ai-train-curriculum-python-map.md](../../00-curriculum/ai-train-curriculum-python-map.md)

---

## 참고 자료

- Python Tutorial: Introduction — https://docs.python.org/3/tutorial/introduction.html
- VS Code Python 시작 가이드 — https://code.visualstudio.com/docs/python/python-tutorial
