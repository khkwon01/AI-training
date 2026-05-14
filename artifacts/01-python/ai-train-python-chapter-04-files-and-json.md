# ai-train Python Basics Chapter 02

## 이 장에서 배우는 것

- 파일을 읽고 쓰는 기본 방법
- JSON이 무엇인지
- Python에서 JSON을 다루는 방법
- 아주 간단한 테스트가 왜 필요한지

## 먼저 쉬운 설명

프로그램은 계산만 하는 것이 아니라, 데이터를 저장하고 다시 읽어야 할 때가 많다.

예를 들어:

- 메모를 파일로 저장하기
- 학생 정보나 점수를 JSON으로 저장하기
- 내가 만든 함수가 제대로 동작하는지 확인하기

이 장에서는 이 세 가지를 가장 쉬운 형태로 배운다.

## 1. 파일이란 무엇인가

파일은 데이터를 저장하는 공간이다.

Python에서는 파일을 열고, 내용을 읽고, 다시 저장할 수 있다.

## 2. 파일 쓰기

```python
with open("note.txt", "w", encoding="utf-8") as file:
    file.write("Python 공부 시작")
```

## 3. 파일 읽기

```python
with open("note.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

## 4. JSON이란 무엇인가

JSON은 데이터를 정리해서 저장할 때 자주 쓰는 형식이다.

```json
{
  "name": "Mina",
  "age": 14,
  "favorite_language": "Python"
}
```

## 5. Python에서 JSON 저장하기

```python
import json

student = {
    "name": "Mina",
    "age": 14,
    "favorite_language": "Python"
}

with open("student.json", "w", encoding="utf-8") as file:
    json.dump(student, file, ensure_ascii=False, indent=2)
```

## 6. Python에서 JSON 읽기

```python
import json

with open("student.json", "r", encoding="utf-8") as file:
    student = json.load(file)

print(student["name"])
```

## 7. 간단한 테스트란 무엇인가

테스트는 "이 코드가 내가 기대한 대로 동작하는지 확인하는 과정"이다.

## 8. 아주 간단한 테스트 예시

```python
def add(a, b):
    return a + b


assert add(2, 3) == 5
```

## 9. 따라 하기 실습

### 실습 1. 메모 파일 만들기

```python
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("오늘은 Python 파일 입출력을 배웠다.")
```

### 실습 2. JSON 저장하기

```python
import json

profile = {
    "name": "Jisoo",
    "age": 15,
    "city": "Suwon"
}

with open("profile.json", "w", encoding="utf-8") as file:
    json.dump(profile, file, ensure_ascii=False, indent=2)
```

### 실습 3. JSON 읽기

```python
import json

with open("profile.json", "r", encoding="utf-8") as file:
    profile = json.load(file)

print(profile["city"])
```

### 실습 4. 테스트하기

```python
def multiply(a, b):
    return a * b


assert multiply(3, 4) == 12
```

## 자주 하는 실수

- 파일 모드를 `r`과 `w`로 헷갈리기
- `encoding=\"utf-8\"`을 빼먹기
- `json.dump`와 `json.load`를 혼동하기
- 테스트를 하지 않고 코드가 맞다고 생각하기

## 확인 체크리스트

- 파일을 열고 저장할 수 있는가
- JSON이 무엇인지 말할 수 있는가
- Python에서 JSON을 저장하고 읽을 수 있는가
- 간단한 `assert` 테스트를 쓸 수 있는가

## 한 번 더 생각해 보기

1. 파일과 JSON은 언제 필요할까?
2. 왜 데이터를 JSON으로 저장하면 편할까?
3. 테스트는 왜 초보자에게도 중요한가?

## 교사용 메모

- 강조: 파일은 저장 공간, 변수는 실행 중 값, JSON은 다른 프로그램과도 주고받기 쉬운 형식이라고 설명한다.
- 막힘: `r/w` 모드, `json.dump/load` 방향, 저장된 파일 위치 확인에서 자주 멈춘다.
- 질문 1: `json.dump()`는 넣는 동작일까, 꺼내는 동작일까?
- 질문 2: `assert`가 실패하면 무엇을 다시 확인해야 할까?
