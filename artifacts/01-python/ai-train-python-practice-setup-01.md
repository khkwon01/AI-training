# ai-train Python Environment Setup Practice 01

## 목표

Chapter 05와 Chapter 07을 연결해서,

- 환경변수
- 경로
- 가상환경
- 라이브러리 설치

를 한 번에 가볍게 연습해 본다.

## 상황

작은 Python 프로젝트를 만든다고 생각해 보자.

프로젝트 안에는:

- `data/` 폴더
- `.venv` 가상환경
- `requirements.txt`

가 필요하다고 가정한다.

## 따라 하기 순서

### 1단계. 프로젝트 폴더 생각하기

예:

```text
my-python-study/
```

### 2단계. 가상환경 만들기

```bash
python3 -m venv .venv
```

### 3단계. 가상환경 활성화하기

```bash
source .venv/bin/activate
```

### 4단계. 라이브러리 설치하기

```bash
pip install requests
```

### 5단계. 설치 목록 저장하기

```bash
pip freeze > requirements.txt
```

### 6단계. data 경로 만들기

```python
from pathlib import Path

data_path = Path("data") / "student.json"
print(data_path)
```

### 7단계. 환경변수 읽기

```python
import os

app_mode = os.environ.get("APP_MODE", "dev")
print(app_mode)
```

## 워크북 빈칸 채우기

```text
내 프로젝트 폴더 이름:
________________________________

내가 설치할 라이브러리:
________________________________

내가 저장할 파일 경로:
________________________________

내가 확인할 환경변수 이름:
________________________________
```

## 확인 체크리스트

- `.venv`를 왜 만드는지 설명할 수 있는가
- `requirements.txt`가 왜 필요한지 설명할 수 있는가
- `Path("data") / "student.json"` 같은 경로를 읽을 수 있는가
- `os.environ.get()`가 어떤 값을 읽는지 말할 수 있는가

