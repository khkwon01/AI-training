# Python Cumulative Practice 01 Answer Guide

## 이 문서의 목적

종합 연습문제를 풀다가 막혔을 때, 학습자가 완전한 정답을 바로 보기 전에 힌트를 얻을 수 있게 돕는다.

## 힌트 1. class부터 차근차근 만들기

먼저 `Student` class를 만들고, 필요한 정보가 무엇인지 생각해 보자.

- 이름
- 나이
- 좋아하는 언어

이 세 가지가 먼저 들어가면 충분하다.

## 힌트 2. 객체를 만든 뒤 값 확인하기

객체를 만든 다음에는 `print()`로 값이 잘 들어갔는지 먼저 확인해 보자.

예:

```python
print(student1.name)
print(student1.age)
```

## 힌트 3. JSON 저장 전에는 딕셔너리로 바꾸기

초보자에게는 class 객체 자체를 바로 저장하는 것보다,
먼저 딕셔너리로 바꾼 뒤 JSON으로 저장하는 방식이 이해하기 쉽다.

## 힌트 4. 저장 후에는 다시 읽기

파일 저장만 하고 끝내지 말고,
반드시 다시 읽어서 값이 맞는지 확인해 보자.

## 힌트 5. assert로 마지막 확인하기

정답이라고 생각되면 `assert`로 마지막 확인을 해 보자.

```python
assert loaded_data["name"] == "Mina"
```

## 정답 예시

```python
import json


class Student:
    def __init__(self, name, age, favorite_language):
        self.name = name
        self.age = age
        self.favorite_language = favorite_language

    def introduce(self):
        return f"저는 {self.name}이고, {self.age}살이며, {self.favorite_language}를 좋아합니다."


student1 = Student("Mina", 14, "Python")

student_data = {
    "name": student1.name,
    "age": student1.age,
    "favorite_language": student1.favorite_language
}

with open("student1.json", "w", encoding="utf-8") as file:
    json.dump(student_data, file, ensure_ascii=False, indent=2)

with open("student1.json", "r", encoding="utf-8") as file:
    loaded_data = json.load(file)

assert loaded_data["name"] == "Mina"
assert loaded_data["favorite_language"] == "Python"
```

## 정답을 볼 때 확인할 것

- 왜 class를 먼저 만들었는가
- 왜 JSON 저장 전에 딕셔너리로 바꿨는가
- 왜 마지막에 assert로 확인했는가

