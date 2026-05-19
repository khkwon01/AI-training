# Python venv/pip Practice 01

## 목표

가상환경을 만들고, 라이브러리를 설치하고, `requirements.txt`까지 저장하는 흐름을 직접 따라 해 본다.

## 따라 하기 순서

### 1단계. 가상환경 만들기

```bash
python3 -m venv .venv
```

### 2단계. 가상환경 활성화하기

```bash
source .venv/bin/activate
```

### 3단계. 라이브러리 설치하기

```bash
pip install requests
```

### 4단계. 설치 확인하기

```bash
pip list
```

### 5단계. 설치 목록 저장하기

```bash
pip freeze > requirements.txt
```

## 결과로 확인할 것

- `.venv` 폴더가 생겼는가
- `pip list`에서 `requests`가 보이는가
- `requirements.txt` 파일이 생겼는가

## 자주 하는 실수

- 가상환경을 켜지 않고 설치하기
- 설치 후 `requirements.txt`를 저장하지 않기
- 어느 폴더에서 작업 중인지 확인하지 않기

## 확인 질문

1. 왜 `venv`를 먼저 만들까?
2. 왜 설치 목록을 파일로 남길까?
3. 다른 사람이 이 프로젝트를 실행하려면 무엇이 필요할까?

