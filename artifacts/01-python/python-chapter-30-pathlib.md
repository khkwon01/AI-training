## 이 장에서 배우는 것

- `pathlib.Path`로 파일 경로를 안전하게 다루는 방법
- 파일 읽기, 쓰기, 추가 쓰기 (`read_text`, `write_text`, `open`)
- 디렉터리 만들기, 목록 보기, 삭제하기
- 경로 조합, 확장자 확인, 파일 존재 여부 검사
- 실무에서 자주 쓰는 파일 순회 패턴 (`glob`, `rglob`)

---

## 먼저 쉬운 설명

프로그램을 만들다 보면 반드시 파일을 다뤄야 하는 순간이 옵니다.  
설정 파일을 읽고, 결과를 저장하고, 로그를 쌓는 일이 모두 파일 작업입니다.

파이썬에는 오래전부터 `os`, `os.path` 같은 도구가 있었지만, 코드가 복잡하고 실수하기 쉬웠습니다.  
파이썬 3.4부터 추가된 **`pathlib`** 는 경로를 *문자열이 아닌 객체*로 다루기 때문에 훨씬 읽기 쉽고 안전합니다.

```python
# 예전 방식 — 문자열을 직접 이어 붙여야 해서 실수가 많았습니다
import os
path = os.path.join("data", "report", "result.txt")

# 새로운 방식 — / 연산자로 경로를 자연스럽게 연결합니다
from pathlib import Path
path = Path("data") / "report" / "result.txt"
```

이 장을 마치면 파일을 자신 있게 읽고 쓸 수 있게 됩니다.

---

## 1. Path 객체 만들기와 경로 정보 읽기

`Path()`에 경로 문자열을 넣으면 경로 객체가 만들어집니다.  
`.`을 찍으면 현재 디렉터리, `..`은 부모 디렉터리를 가리킵니다.

```python
from pathlib import Path

# 현재 작업 디렉터리
현재_위치 = Path.cwd()
print(현재_위치)          # /Users/홍길동/프로젝트

# 홈 디렉터리
홈 = Path.home()
print(홈)                 # /Users/홍길동

# 경로 조합 — / 연산자를 사용합니다
파일_경로 = Path("데이터") / "2024년" / "보고서.txt"
print(파일_경로)          # 데이터/2024년/보고서.txt

# 경로 정보 조각내기
p = Path("/Users/홍길동/프로젝트/main.py")
print(p.name)             # main.py          (파일 이름)
print(p.stem)             # main             (확장자 없는 이름)
print(p.suffix)           # .py              (확장자)
print(p.parent)           # /Users/홍길동/프로젝트
print(p.parts)            # ('/', 'Users', '홍길동', '프로젝트', 'main.py')
```

---

## 2. 파일 존재 여부 확인과 타입 검사

파일을 열기 전에 *존재하는지* 먼저 확인하는 습관을 들이세요.  
존재하지 않는 파일을 열면 `FileNotFoundError`가 발생합니다.

```python
from pathlib import Path

설정_파일 = Path("config.json")

# 존재 여부 확인
if 설정_파일.exists():
    print("파일이 있습니다.")
else:
    print("파일이 없습니다. 기본값을 사용합니다.")

# 파일인지 디렉터리인지 구분하기
로그_경로 = Path("logs")
print(로그_경로.is_file())   # False (디렉터리이면)
print(로그_경로.is_dir())    # True

# 확장자로 필터링하는 예시
소스_파일 = Path("app.py")
if 소스_파일.suffix == ".py":
    print(f"{소스_파일.name}은 파이썬 파일입니다.")
```

---

## 3. 파일 읽기와 쓰기

짧은 파일은 `read_text` / `write_text`가 가장 간단합니다.  
줄 단위로 처리하거나 큰 파일을 다룰 때는 `open()`을 사용합니다.

```python
from pathlib import Path

# ── 파일 쓰기 ──────────────────────────────────────────
메모_파일 = Path("오늘의_할_일.txt")

# write_text: 파일이 없으면 만들고, 있으면 덮어씁니다
메모_파일.write_text(
    "1. 파이썬 공부\n2. 운동\n3. 독서\n",
    encoding="utf-8"
)
print("저장 완료!")

# ── 파일 읽기 ──────────────────────────────────────────
내용 = 메모_파일.read_text(encoding="utf-8")
print(내용)

# 줄 단위로 읽기
줄_목록 = 메모_파일.read_text(encoding="utf-8").splitlines()
for 번호, 줄 in enumerate(줄_목록, start=1):
    print(f"  {번호}번째 줄: {줄}")

# ── 내용 추가하기 (append) ────────────────────────────
with 메모_파일.open("a", encoding="utf-8") as f:
    f.write("4. 일기 쓰기\n")

print("추가 완료! 현재 내용:")
print(메모_파일.read_text(encoding="utf-8"))
```

> **주의:** `write_text`는 기존 내용을 **완전히 덮어씁니다**.  
> 기존 내용을 유지하면서 추가하려면 `open("a")`를 사용하세요.

---

## 4. 디렉터리 만들기와 삭제

```python
from pathlib import Path

# ── 디렉터리 만들기 ───────────────────────────────────
결과_폴더 = Path("결과물") / "2024년" / "1분기"

# parents=True  : 중간 경로가 없어도 한 번에 만듭니다
# exist_ok=True : 이미 있어도 오류 없이 넘어갑니다
결과_폴더.mkdir(parents=True, exist_ok=True)
print(f"'{결과_폴더}' 폴더 생성 완료")

# ── 디렉터리 내 항목 나열하기 ─────────────────────────
for 항목 in Path(".").iterdir():
    종류 = "폴더" if 항목.is_dir() else "파일"
    print(f"  [{종류}] {항목.name}")

# ── 파일 삭제 ─────────────────────────────────────────
임시_파일 = Path("임시.txt")
임시_파일.write_text("삭제될 내용", encoding="utf-8")
임시_파일.unlink()          # 파일 삭제
print("임시 파일 삭제 완료")

# 없을 수도 있는 파일을 안전하게 삭제할 때
임시_파일.unlink(missing_ok=True)   # 파이썬 3.8+

# ── 빈 디렉터리 삭제 ──────────────────────────────────
빈_폴더 = Path("빈_폴더_테스트")
빈_폴더.mkdir(exist_ok=True)
빈_폴더.rmdir()             # 비어있어야 삭제됩니다
```

---

## 5. glob으로 파일 찾기

`glob`은 특정 패턴에 맞는 파일을 한꺼번에 찾을 때 씁니다.  
`rglob`은 하위 폴더까지 재귀적으로 검색합니다.

```python
from pathlib import Path

프로젝트_루트 = Path(".")

# 현재 폴더의 .py 파일만 찾기
print("=== 현재 폴더의 파이썬 파일 ===")
for py_파일 in 프로젝트_루트.glob("*.py"):
    print(f"  {py_파일.name}")

# 모든 하위 폴더까지 포함해서 .txt 파일 찾기
print("\n=== 전체 폴더의 텍스트 파일 ===")
for txt_파일 in 프로젝트_루트.rglob("*.txt"):
    크기 = txt_파일.stat().st_size
    print(f"  {txt_파일}  ({크기} 바이트)")

# 특정 이름 패턴으로 찾기 — report_로 시작하는 모든 파일
for 보고서 in 프로젝트_루트.rglob("report_*"):
    print(f"  발견: {보고서}")
```

---

## 6. 파일 이동과 이름 바꾸기

```python
from pathlib import Path

# 파일 이름 바꾸기
원본 = Path("초안.txt")
원본.write_text("초안 내용입니다.", encoding="utf-8")

완성본 = 원본.rename("완성본.txt")   # 이름 바꾸기 (원본 Path 반환)
print(완성본.read_text(encoding="utf-8"))

# 다른 폴더로 이동하기
보관_폴더 = Path("보관함")
보관_폴더.mkdir(exist_ok=True)

이동_결과 = 완성본.rename(보관_폴더 / 완성본.name)
print(f"파일 이동 완료: {이동_결과}")

# 확장자만 바꾸기 — with_suffix 활용
원래_경로 = Path("데이터.csv")
새_경로 = 원래_경로.with_suffix(".xlsx")
print(새_경로)   # 데이터.xlsx  (실제 파일은 변경하지 않고 경로 객체만 생성)
```

---

## 따라 하기 실습

### 실습 1 — 일기장 파일 만들기

오늘 날짜 이름의 일기 파일을 만들고 내용을 저장해 보세요.

```python
from pathlib import Path
from datetime import date

# 1단계: 일기 폴더 만들기
일기_폴더 = Path("일기장")
일기_폴더.mkdir(exist_ok=True)

# 2단계: 오늘 날짜로 파일 이름 정하기
오늘 = date.today().strftime("%Y-%m-%d")
일기_파일 = 일기_폴더 / f"{오늘}_일기.txt"

# 3단계: 내용 쓰기
일기_파일.write_text(
    f"날짜: {오늘}\n\n오늘은 파이썬 파일 입출력을 배웠다.\n뿌듯하다!\n",
    encoding="utf-8"
)

print(f"일기 저장 완료: {일기_파일}")
print(일기_파일.read_text(encoding="utf-8"))
```

---

### 실습 2 — 폴더 안 파일 목록 보고서 만들기

실습 1에서 만든 `일기장` 폴더를 스캔해서 목록을 별도 파일로 저장합니다.

```python
from pathlib import Path

일기_폴더 = Path("일기장")
목록_파일 = Path("일기_목록.txt")

# 파일 목록 수집
줄_목록 = ["=== 일기 파일 목록 ===\n"]
for 파일 in sorted(일기_폴더.glob("*.txt")):
    크기 = 파일.stat().st_size
    줄_목록.append(f"- {파일.name}  ({크기} 바이트)\n")

# 목록 저장
목록_파일.write_text("".join(줄_목록), encoding="utf-8")
print("목록 저장 완료!")
print(목록_파일.read_text(encoding="utf-8"))
```

---

### 실습 3 — 오래된 파일 보관함으로 옮기기

2023년 이전 일기 파일을 `보관함` 폴더로 이동하는 스크립트를 만듭니다.

```python
from pathlib import Path

일기_폴더 = Path("일기장")
보관_폴더 = Path("일기장") / "보관함"
보관_폴더.mkdir(exist_ok=True)

이동_수 = 0
for 파일 in 일기_폴더.glob("*.txt"):
    # 파일 이름이 "2023-" 으로 시작하면 이동
    if 파일.stem.startswith("2023-"):
        대상 = 보관_폴더 / 파일.name
        파일.rename(대상)
        print(f"이동: {파일.name} → 보관함/")
        이동_수 += 1

print(f"\n총 {이동_수}개 파일을 보관함으로 이동했습니다.")
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 원인 | 해결 방법 |
|------|------------|------|----------|
| 없는 파일 열기 | `FileNotFoundError: [Errno 2] No such file or directory: 'data.txt'` | 파일 경로가 잘못됐거나 파일이 없음 | `exists()` 로 먼저 확인하거나 `try/except` 사용 |
| 중간 경로 없이 폴더 만들기 | `FileNotFoundError: [Errno 2] No such file or directory: 'a/b/c'` | 부모 폴더가 없는데 `parents=True` 누락 | `mkdir(parents=True, exist_ok=True)` 사용 |
| 이미 있는 폴더 다시 만들기 | `FileExistsError: [Errno 17] File exists: '결과물'` | `exist_ok=False`(기본값)인 상태에서 중복 생성 | `mkdir(exist_ok=True)` 추가 |
| 인코딩 지정 안 함 | `UnicodeDecodeError: 'cp949' codec can't decode byte...` | Windows에서 UTF-8 파일을 기본 인코딩으로 읽음 | 항상 `encoding="utf-8"` 명시 |
| 비어 있지 않은 폴더 삭제 | `OSError: [Errno 66] Directory not empty: '폴더명'` | `rmdir()`은 빈 폴더만 삭제 가능 | `shutil.rmtree(폴더명)` 사용 (주의: 복구 불가) |
| `rename` 후 원본 변수 재사용 | `FileNotFoundError` | `rename`은 새 Path를 반환하며 원본 경로는 더 이상 유효하지 않음 | 반환값을 새 변수에 저장: `새_경로 = 파일.rename(...)` |
| 경로에 `\` 사용 (Windows) | `SyntaxError` 또는 경로 오류 | `\n`, `\t` 등 이스케이프 문자로 해석 | `/` 연산자 또는 `r"경로\파일"` 형식 사용 |

---

## 확인 체크리스트

- [ ] `Path("폴더") / "파일.txt"` 처럼 `/` 연산자로 경로를 조합할 수 있다
- [ ] `.name`, `.stem`, `.suffix`, `.parent` 속성의 차이를 설명할 수 있다
- [ ] `exists()`, `is_file()`, `is_dir()`로 경로 상태를 확인할 수 있다
- [ ] `write_text`와 `open("a")`의 차이(덮어쓰기 vs 추가)를 구분할 수 있다
- [ ] `mkdir(parents=True, exist_ok=True)`가 왜 필요한지 설명할 수 있다
- [ ] `glob("*.py")`와 `rglob("*.py")`의 차이를 안다
- [ ] 파일 읽고 쓸 때 `encoding="utf-8"`을 항상 지정하는 이유를 설명할 수 있다
- [ ] `unlink()`와 `rmdir()`의 적용 대상 차이를 알고 있다

---

## 한 번 더 생각해 보기

1. **`write_text`와 `open("w")`는 결과가 같아 보이는데, 언제 어느 것을 쓰면 좋을까요?**  
   힌트: 파일 크기, 예외 처리, 코드 가독성 관점에서 생각해 보세요.

2. **여러 사람이 동시에 같은 파일에 쓰면 어떤 문제가 생길까요?**  
   힌트: 파일 잠금(lock), 레이스 컨디션이라는 키워드로 검색해 보세요.

3. **`Path.cwd()`가 반환하는 경로는 스크립트 파일의 위치와 항상 같을까요?**  
   힌트: 터미널에서 스크립트를 실행하는 위치를 바꿔가며 직접 확인해 보세요.

---

## 다음 장

다음 장에서는 파일에 구조화된 데이터를 저장하는 방법인 **JSON과 CSV 파일 다루기**를 배웁니다.