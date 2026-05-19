## 이 장에서 배우는 것

- `sys.argv`로 터미널 명령줄 인수를 읽는 방법
- `argparse` 모듈로 옵션과 인수를 체계적으로 처리하는 방법
- `--help` 메시지를 자동으로 생성하는 방법
- 선택 인수(optional)와 필수 인수(positional)의 차이
- 실제 CLI 도구처럼 동작하는 Python 스크립트 작성 방법

---

## 먼저 쉬운 설명

터미널에서 이런 명령을 본 적 있나요?

```
python script.py --name 철수 --age 25
```

이게 바로 **CLI(Command Line Interface)** 입니다. 마우스로 클릭하는 대신, 텍스트로 프로그램에 정보를 전달하는 방식이에요.

왜 배워야 할까요? 실제 개발 현장에서는 서버, 데이터 처리, 자동화 스크립트를 모두 CLI로 실행합니다. `git`, `pip`, `docker` 같은 도구들이 모두 CLI 방식으로 만들어져 있어요. 이걸 만들 줄 알면 "진짜 개발자"처럼 보이는 도구를 직접 제작할 수 있습니다.

---

## 1. `sys.argv` — 가장 단순한 방법

`sys.argv`는 터미널에서 입력한 내용을 **리스트**로 담아주는 변수입니다.

```python
# greet.py
import sys

# sys.argv[0]은 항상 파일 이름
print("전달된 인수 목록:", sys.argv)

if len(sys.argv) > 1:
    name = sys.argv[1]
    print(f"안녕하세요, {name}님!")
else:
    print("이름을 입력해 주세요.")
```

**실행 예시:**

```
$ python greet.py 철수
전달된 인수 목록: ['greet.py', '철수']
안녕하세요, 철수님!
```

```
$ python greet.py
전달된 인수 목록: ['greet.py']
이름을 입력해 주세요.
```

> `sys.argv`는 단순하지만 `--name`, `--help` 같은 옵션을 직접 파싱해야 해서 코드가 복잡해집니다. 그래서 `argparse`를 씁니다.

---

## 2. `argparse` 기본 구조

`argparse`는 Python 표준 라이브러리에 포함된 CLI 파싱 도구입니다. 설치 없이 바로 사용할 수 있어요.

```python
# hello_argparse.py
import argparse

# 1. 파서 객체 생성
parser = argparse.ArgumentParser(description="간단한 인사 프로그램")

# 2. 인수 추가
parser.add_argument("name", help="인사할 사람의 이름")

# 3. 인수 파싱
args = parser.parse_args()

# 4. 사용
print(f"안녕하세요, {args.name}님!")
```

**실행 예시:**

```
$ python hello_argparse.py 영희
안녕하세요, 영희님!

$ python hello_argparse.py --help
usage: hello_argparse.py [-h] name

간단한 인사 프로그램

positional arguments:
  name        인사할 사람의 이름

options:
  -h, --help  show this help message and exit
```

`--help`가 자동으로 생성됩니다! `sys.argv`로는 직접 구현해야 했던 기능이에요.

---

## 3. 위치 인수(positional) vs 선택 인수(optional)

```python
# calculator.py
import argparse

parser = argparse.ArgumentParser(description="간단한 계산기")

# 위치 인수: 반드시 입력해야 함 (앞에 --가 없음)
parser.add_argument("x", type=float, help="첫 번째 숫자")
parser.add_argument("y", type=float, help="두 번째 숫자")

# 선택 인수: 입력 안 해도 됨 (앞에 --가 있음)
parser.add_argument("--op", choices=["add", "sub", "mul", "div"],
                    default="add", help="연산 종류 (기본값: add)")

args = parser.parse_args()

if args.op == "add":
    result = args.x + args.y
elif args.op == "sub":
    result = args.x - args.y
elif args.op == "mul":
    result = args.x * args.y
elif args.op == "div":
    if args.y == 0:
        print("오류: 0으로 나눌 수 없습니다.")
        exit(1)
    result = args.x / args.y

print(f"결과: {result}")
```

**실행 예시:**

```
$ python calculator.py 10 3
결과: 13.0

$ python calculator.py 10 3 --op mul
결과: 30.0

$ python calculator.py 10 3 --op div
결과: 3.3333333333333335

$ python calculator.py 10 0 --op div
오류: 0으로 나눌 수 없습니다.
```

---

## 4. 자주 쓰는 인수 타입과 플래그

```python
# file_tool.py
import argparse

parser = argparse.ArgumentParser(description="파일 처리 도구")

# type=int: 정수로 변환
parser.add_argument("--count", type=int, default=10, help="처리할 줄 수")

# type=str, required=True: 필수 선택 인수
parser.add_argument("--input", type=str, required=True, help="입력 파일 경로")

# action="store_true": 플래그 (있으면 True, 없으면 False)
parser.add_argument("--verbose", action="store_true", help="상세 출력 모드")

# nargs="+": 여러 개의 값을 리스트로 받음
parser.add_argument("--tags", nargs="+", help="태그 목록 (여러 개 가능)")

args = parser.parse_args()

if args.verbose:
    print(f"[상세] 입력 파일: {args.input}")
    print(f"[상세] 처리할 줄 수: {args.count}")
    if args.tags:
        print(f"[상세] 태그: {', '.join(args.tags)}")

print(f"'{args.input}' 파일에서 {args.count}줄 처리 완료")
```

**실행 예시:**

```
$ python file_tool.py --input data.txt --count 5 --verbose --tags python cli 초보
[상세] 입력 파일: data.txt
[상세] 처리할 줄 수: 5
[상세] 태그: python, cli, 초보
'data.txt' 파일에서 5줄 처리 완료
```

---

## 따라 하기 실습

### 실습 1 — 이름과 나이를 받는 소개 프로그램 만들기

`introduce.py` 파일을 만들고 다음 조건을 구현하세요.

1. `name` 위치 인수를 받아 이름을 출력한다.
2. `--age` 선택 인수(정수형)를 받아 나이를 출력한다. 기본값은 `0`.
3. `--formal` 플래그가 있으면 "님", 없으면 "야"를 붙인다.

```python
# introduce.py
import argparse

parser = argparse.ArgumentParser(description="자기소개 프로그램")
parser.add_argument("name", help="이름")
parser.add_argument("--age", type=int, default=0, help="나이")
parser.add_argument("--formal", action="store_true", help="격식체 사용")
args = parser.parse_args()

호칭 = "님" if args.formal else "야"
print(f"안녕하세요, {args.name}{호칭}!")
if args.age > 0:
    print(f"나이는 {args.age}살이군요.")
```

**예상 실행 결과:**

```
$ python introduce.py 철수
안녕하세요, 철수야!

$ python introduce.py 영희 --age 28 --formal
안녕하세요, 영희님!
나이는 28살이군요.
```

---

### 실습 2 — 숫자 목록 통계 프로그램 만들기

`stats.py` 파일을 만드세요. 실습 1에서 배운 `nargs`와 `type`을 응용합니다.

```python
# stats.py
import argparse

parser = argparse.ArgumentParser(description="숫자 통계 계산기")
parser.add_argument("numbers", type=float, nargs="+", help="분석할 숫자 목록")
parser.add_argument("--show-all", action="store_true", help="합계, 평균, 최댓값 모두 표시")
args = parser.parse_args()

nums = args.numbers
평균 = sum(nums) / len(nums)

if args.show_all:
    print(f"합계:   {sum(nums)}")
    print(f"평균:   {평균:.2f}")
    print(f"최댓값: {max(nums)}")
    print(f"최솟값: {min(nums)}")
else:
    print(f"평균: {평균:.2f}")
```

**예상 실행 결과:**

```
$ python stats.py 10 20 30 40 50
평균: 30.00

$ python stats.py 10 20 30 40 50 --show-all
합계:   150.0
평균:   30.00
최댓값: 50.0
최솟값: 10.0
```

---

### 실습 3 — 두 프로그램을 합친 성적 분석 도구 만들기

`grade_tool.py` 파일을 만드세요. 실습 1, 2의 개념을 모두 활용합니다.

```python
# grade_tool.py
import argparse

parser = argparse.ArgumentParser(description="학생 성적 분석 도구")
parser.add_argument("student", help="학생 이름")
parser.add_argument("scores", type=float, nargs="+", help="과목별 점수")
parser.add_argument("--pass-score", type=float, default=60.0, help="합격 기준 점수 (기본값: 60)")
parser.add_argument("--verbose", action="store_true", help="상세 출력")
args = parser.parse_args()

평균 = sum(args.scores) / len(args.scores)
합격여부 = "합격" if 평균 >= args.pass_score else "불합격"

if args.verbose:
    print(f"학생: {args.student}")
    print(f"점수: {args.scores}")
    print(f"합격 기준: {args.pass_score}점")

print(f"{args.student} 학생 평균: {평균:.1f}점 → {합격여부}")
```

**예상 실행 결과:**

```
$ python grade_tool.py 홍길동 85 90 78 --verbose
학생: 홍길동
점수: [85.0, 90.0, 78.0]
합격 기준: 60.0점
홍길동 학생 평균: 84.3점 → 합격

$ python grade_tool.py 임꺽정 45 55 50 --pass-score 55
임꺽정 학생 평균: 50.0점 → 불합격
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 원인 | 해결 방법 |
|------|------------|------|-----------|
| 위치 인수를 빠뜨림 | `error: the following arguments are required: name` | 필수 인수를 입력하지 않음 | 인수를 명령에 추가하거나 `default` 값 설정 |
| 숫자 타입 미지정 | `TypeError: unsupported operand type(s) for +: 'int' and 'str'` | `type=int` 없이 숫자 연산 시도 | `add_argument`에 `type=int` 또는 `type=float` 추가 |
| `--`를 빠뜨림 | `error: unrecognized arguments: verbose` | `--verbose` 대신 `verbose`로 입력 | 선택 인수 앞에 `--` 붙이기 |
| `choices` 범위 초과 | `error: argument --op: invalid choice: 'mod'` | `choices` 목록에 없는 값 입력 | 지정된 선택지 중 하나를 사용하거나 `choices` 목록 확장 |
| `nargs`를 빠뜨리고 여러 값 입력 | `error: unrecognized arguments: 20 30` | 단일 값 인수에 여러 값 전달 | `nargs="+"` 또는 `nargs="*"` 추가 |
| `required=True`인데 생략 | `error: the following arguments are required: --input` | 필수 선택 인수를 입력하지 않음 | `--input 파일명` 형태로 반드시 입력 |

---

## 확인 체크리스트

스스로 확인해 보세요.

- [ ] `sys.argv`를 출력했을 때 첫 번째 요소가 파일 이름임을 안다
- [ ] `argparse.ArgumentParser()`로 파서를 생성할 수 있다
- [ ] `add_argument("name")`(위치 인수)와 `add_argument("--name")`(선택 인수)의 차이를 설명할 수 있다
- [ ] `type=int`, `type=float`을 사용하여 숫자를 올바르게 받을 수 있다
- [ ] `action="store_true"`로 플래그(True/False) 인수를 만들 수 있다
- [ ] `nargs="+"`로 여러 값을 리스트로 받을 수 있다
- [ ] `python script.py --help` 실행 시 도움말이 자동으로 출력되는 것을 확인했다
- [ ] `required=True`와 `default=값`의 용도를 각각 설명할 수 있다
- [ ] 실습 3(`grade_tool.py`)을 오류 없이 실행했다

---

## 한 번 더 생각해 보기

1. `sys.argv`를 직접 쓰는 방법과 `argparse`를 쓰는 방법의 장단점은 무엇일까요? 어떤 상황에서 `sys.argv`가 더 나을 수 있을까요?

2. `grade_tool.py`에서 `--pass-score` 기준을 0~100 사이로 제한하려면 코드를 어떻게 바꿔야 할까요? `argparse`만으로는 부족하다면 어떻게 보완할 수 있을까요?

3. 터미널에서 `pip install`, `git commit -m "메시지"` 같은 명령을 보면 어떤 부분이 위치 인수이고 어떤 부분이 선택 인수인지 이제 구별할 수 있나요? 주변의 CLI 명령들을 분석해 보세요.

---

## 다음 장

다음 장에서는 `pathlib`와 `os` 모듈을 사용하여 파일과 디렉토리를 다루는 방법을 배웁니다 — CLI 도구에서 실제 파일을 읽고 쓰는 실전 기술입니다.