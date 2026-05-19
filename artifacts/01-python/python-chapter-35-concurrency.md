## 이 장에서 배우는 것

- 동시성(concurrency)과 병렬성(parallelism)의 차이를 설명할 수 있다
- `threading` 모듈로 여러 작업을 동시에 실행할 수 있다
- `multiprocessing` 모듈로 CPU 집약적인 작업을 병렬로 처리할 수 있다
- GIL(Global Interpreter Lock)이 무엇인지, 언제 문제가 되는지 이해한다
- `ThreadPoolExecutor`와 `ProcessPoolExecutor`로 코드를 간결하게 작성할 수 있다
- 스레드 간 데이터 공유 시 발생하는 경쟁 조건(race condition)을 인식하고 막을 수 있다

---

## 먼저 쉬운 설명

요리를 혼자 할 때를 생각해 보자.  
라면을 끓이면서 계란도 삶고 김치도 꺼내야 한다.  
한 가지 일이 끝날 때까지 기다렸다가 다음 일을 하면 너무 느리다.  
실제로는 물이 끓는 동안 다른 준비를 함께 한다.

파이썬 프로그램도 마찬가지다.  
파일을 다운로드하는 동안 다른 파일을 준비하거나,  
무거운 계산을 CPU 여러 개에 나눠서 동시에 처리할 수 있다.

이게 바로 **동시성 프로그래밍**이다.

| 상황 | 적합한 도구 |
|------|-------------|
| 파일 읽기/쓰기, 네트워크 요청처럼 기다리는 시간이 긴 작업 | `threading` |
| 숫자 계산, 이미지 처리처럼 CPU를 많이 쓰는 작업 | `multiprocessing` |

---

## 1. threading — 동시에 여러 일 처리하기

### 1.1 기본 스레드 만들기

```python
# download_files.py
import threading
import time

def 파일_다운로드(파일명: str) -> None:
    print(f"[시작] {파일명} 다운로드 중...")
    time.sleep(2)  # 실제 다운로드를 흉내 낸다
    print(f"[완료] {파일명} 다운로드 끝!")

파일_목록 = ["report_2024.pdf", "data_jan.csv", "image_01.png"]

스레드_목록 = []
for 파일 in 파일_목록:
    t = threading.Thread(target=파일_다운로드, args=(파일,))
    스레드_목록.append(t)
    t.start()

# 모든 스레드가 끝날 때까지 기다린다
for t in 스레드_목록:
    t.join()

print("모든 파일 다운로드 완료!")
```

실행하면 3개 파일이 거의 동시에 시작되어 약 2초 만에 끝난다.  
순차 실행이었다면 6초가 걸렸을 것이다.

### 1.2 ThreadPoolExecutor — 더 깔끔한 방법

```python
# pool_download.py
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def 파일_다운로드(파일명: str) -> str:
    time.sleep(2)
    return f"{파일명} 완료"

파일_목록 = ["report_2024.pdf", "data_jan.csv", "image_01.png", "video_intro.mp4"]

with ThreadPoolExecutor(max_workers=3) as executor:
    # 작업을 제출하고 Future 객체를 받는다
    future_map = {executor.submit(파일_다운로드, 파일): 파일 for 파일 in 파일_목록}

    for future in as_completed(future_map):
        결과 = future.result()
        print(f"✓ {결과}")

print("풀 종료, 모든 작업 완료")
```

`with` 블록을 벗어날 때 자동으로 모든 스레드가 정리된다.  
`max_workers=3`이면 한 번에 최대 3개만 실행된다.

---

## 2. GIL과 경쟁 조건 — 스레드의 함정

### 2.1 GIL이란?

파이썬에는 **GIL(Global Interpreter Lock)**이 있다.  
여러 스레드가 동시에 파이썬 바이트코드를 실행하지 못하게 막는 잠금 장치다.  
덕분에 I/O 대기 작업은 괜찮지만, **CPU 계산**은 스레드를 늘려도 빨라지지 않는다.

### 2.2 경쟁 조건 — 데이터가 망가지는 상황

```python
# race_condition.py
import threading

잔액 = 0  # 공유 변수

def 입금(금액: int, 반복: int) -> None:
    global 잔액
    for _ in range(반복):
        현재 = 잔액
        현재 += 금액
        잔액 = 현재  # 여기서 다른 스레드가 끼어들 수 있다!

t1 = threading.Thread(target=입금, args=(1, 100_000))
t2 = threading.Thread(target=입금, args=(1, 100_000))

t1.start()
t2.start()
t1.join()
t2.join()

print(f"최종 잔액: {잔액}")  # 200000이어야 하지만, 다른 값이 나올 수 있다
```

### 2.3 Lock으로 고치기

```python
# safe_bank.py
import threading

잔액 = 0
lock = threading.Lock()  # 잠금 객체

def 안전_입금(금액: int, 반복: int) -> None:
    global 잔액
    for _ in range(반복):
        with lock:  # 한 번에 하나의 스레드만 이 블록 실행
            잔액 += 금액

t1 = threading.Thread(target=안전_입금, args=(1, 100_000))
t2 = threading.Thread(target=안전_입금, args=(1, 100_000))

t1.start()
t2.start()
t1.join()
t2.join()

print(f"최종 잔액: {잔액}")  # 항상 200000
```

---

## 3. multiprocessing — CPU 작업을 진짜로 병렬 처리하기

### 3.1 기본 프로세스 만들기

```python
# cpu_work.py
import multiprocessing
import time

def 소수_세기(범위_끝: int) -> int:
    """2부터 범위_끝까지 소수 개수를 센다."""
    def 소수_판별(n: int) -> bool:
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True

    return sum(1 for n in range(2, 범위_끝) if 소수_판별(n))

if __name__ == "__main__":  # 반드시 필요!
    시작 = time.time()

    p1 = multiprocessing.Process(target=소수_세기, args=(500_000,))
    p2 = multiprocessing.Process(target=소수_세기, args=(500_000,))

    p1.start()
    p2.start()
    p1.join()
    p2.join()

    print(f"소요 시간: {time.time() - 시작:.2f}초")
```

> **중요:** `if __name__ == "__main__":` 없이 실행하면 윈도우에서 무한 루프가 생긴다.

### 3.2 ProcessPoolExecutor와 결과 수집

```python
# prime_pool.py
from concurrent.futures import ProcessPoolExecutor
import time

def 소수_판별(n: int) -> bool:
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

def 범위_소수_목록(시작: int, 끝: int) -> list[int]:
    return [n for n in range(시작, 끝) if 소수_판별(n)]

if __name__ == "__main__":
    구간_목록 = [(2, 250_000), (250_000, 500_000), (500_000, 750_000), (750_000, 1_000_000)]

    시작 = time.time()
    with ProcessPoolExecutor() as executor:
        결과들 = list(executor.map(lambda 구간: 범위_소수_목록(*구간), 구간_목록))

    전체_소수 = [소수 for 구간_결과 in 결과들 for 소수 in 구간_결과]
    print(f"소수 개수: {len(전체_소수)}, 소요 시간: {time.time() - 시작:.2f}초")
```

### 3.3 프로세스 간 데이터 공유

프로세스는 메모리를 공유하지 않는다. 데이터를 주고받으려면 `Queue`를 사용한다.

```python
# process_queue.py
from multiprocessing import Process, Queue

def 작업자(큐: Queue, 작업_번호: int) -> None:
    결과 = 작업_번호 ** 2
    큐.put({"번호": 작업_번호, "결과": 결과})

if __name__ == "__main__":
    결과_큐: Queue = Queue()
    프로세스들 = [Process(target=작업자, args=(결과_큐, i)) for i in range(5)]

    for p in 프로세스들:
        p.start()
    for p in 프로세스들:
        p.join()

    while not 결과_큐.empty():
        print(결과_큐.get())
```

---

## 4. threading vs multiprocessing — 언제 무엇을 쓸까?

```python
# benchmark_compare.py
import threading
import multiprocessing
import time

def cpu_작업(n: int) -> int:
    return sum(i * i for i in range(n))

def io_작업(초: float) -> None:
    time.sleep(초)

if __name__ == "__main__":
    # I/O 작업: threading이 유리
    print("=== I/O 작업 비교 ===")
    시작 = time.time()
    스레드들 = [threading.Thread(target=io_작업, args=(1.0,)) for _ in range(4)]
    for t in 스레드들: t.start()
    for t in 스레드들: t.join()
    print(f"threading: {time.time() - 시작:.2f}초")

    시작 = time.time()
    프로세스들 = [multiprocessing.Process(target=io_작업, args=(1.0,)) for _ in range(4)]
    for p in 프로세스들: p.start()
    for p in 프로세스들: p.join()
    print(f"multiprocessing: {time.time() - 시작:.2f}초")

    # CPU 작업: multiprocessing이 유리
    print("\n=== CPU 작업 비교 ===")
    시작 = time.time()
    스레드들 = [threading.Thread(target=cpu_작업, args=(1_000_000,)) for _ in range(4)]
    for t in 스레드들: t.start()
    for t in 스레드들: t.join()
    print(f"threading: {time.time() - 시작:.2f}초")

    시작 = time.time()
    프로세스들 = [multiprocessing.Process(target=cpu_작업, args=(1_000_000,)) for _ in range(4)]
    for p in 프로세스들: p.start()
    for p in 프로세스들: p.join()
    print(f"multiprocessing: {time.time() - 시작:.2f}초")
```

---

## 따라 하기 실습

### 실습 1 — 멀티스레드 로그 수집기 만들기

`log_collector.py` 파일을 만들어 여러 서버에서 동시에 로그를 수집하는 프로그램을 작성한다.

```python
# log_collector.py
import threading
import time
import random
from datetime import datetime

결과_lock = threading.Lock()
수집된_로그: list[dict] = []

def 서버_로그_수집(서버명: str) -> None:
    """실제 네트워크 요청 대신 sleep으로 지연을 흉내 낸다."""
    지연 = random.uniform(0.5, 2.0)
    time.sleep(지연)

    로그 = {
        "서버": 서버명,
        "시각": datetime.now().strftime("%H:%M:%S"),
        "상태": random.choice(["정상", "경고", "오류"]),
        "응답시간_ms": int(지연 * 1000),
    }

    with 결과_lock:
        수집된_로그.append(로그)
        print(f"[수집완료] {서버명}: {로그['상태']}")

서버_목록 = ["web-01", "web-02", "db-primary", "db-replica", "cache-01"]

시작 = time.time()
스레드들 = [threading.Thread(target=서버_로그_수집, args=(서버,)) for 서버 in 서버_목록]
for t in 스레드들: t.start()
for t in 스레드들: t.join()

print(f"\n총 소요 시간: {time.time() - 시작:.2f}초")
print(f"수집된 로그 수: {len(수집된_로그)}")
for 로그 in sorted(수집된_로그, key=lambda x: x["서버"]):
    print(f"  {로그['서버']}: {로그['상태']} ({로그['응답시간_ms']}ms)")
```

실행 후 `총 소요 시간`이 약 2초 이내인지 확인한다.  
순차 실행이면 최대 10초가 걸린다.

---

### 실습 2 — 멀티프로세싱 이미지 처리기 만들기

실습 1에서 수집한 로그 개념을 확장해, `image_processor.py`로 CPU 집약 작업을 병렬 처리한다.

```python
# image_processor.py
from concurrent.futures import ProcessPoolExecutor, as_completed
import time

def 이미지_필터_적용(파일명: str, 강도: int) -> dict:
    """실제 이미지 처리 대신 무거운 계산으로 흉내 낸다."""
    # 픽셀 변환 계산 시뮬레이션
    결과 = sum(i * 강도 % 255 for i in range(500_000))
    return {"파일": 파일명, "처리결과": 결과 % 255, "강도": 강도}

이미지_작업 = [
    ("photo_001.jpg", 3),
    ("photo_002.jpg", 5),
    ("photo_003.jpg", 7),
    ("banner_main.png", 2),
    ("thumbnail_01.png", 4),
    ("thumbnail_02.png", 6),
]

if __name__ == "__main__":
    시작 = time.time()
    with ProcessPoolExecutor() as executor:
        futures = {
            executor.submit(이미지_필터_적용, 파일, 강도): 파일
            for 파일, 강도 in 이미지_작업
        }

        for future in as_completed(futures):
            결과 = future.result()
            print(f"✓ {결과['파일']} — 강도 {결과['강도']} 적용 완료")

    print(f"\n총 소요 시간: {time.time() - 시작:.2f}초")
    print(f"처리된 파일 수: {len(이미지_작업)}")
```

---

### 실습 3 — 혼합 파이프라인 만들기

실습 1과 실습 2를 합쳐 `pipeline.py`를 만든다. I/O는 스레드, CPU는 프로세스로 분리한다.

```python
# pipeline.py
import threading
from concurrent.futures import ProcessPoolExecutor
import time
import random

# --- 1단계: 스레드로 데이터 수집 ---
수집_데이터: list[int] = []
수집_lock = threading.Lock()

def 데이터_수집(소스_번호: int) -> None:
    time.sleep(random.uniform(0.1, 0.5))  # I/O 대기 흉내
    값 = random.randint(100_000, 500_000)
    with 수집_lock:
        수집_데이터.append(값)

print("=== 1단계: 스레드로 데이터 수집 ===")
스레드들 = [threading.Thread(target=데이터_수집, args=(i,)) for i in range(8)]
for t in 스레드들: t.start()
for t in 스레드들: t.join()
print(f"수집된 데이터: {수집_데이터}")

# --- 2단계: 프로세스로 CPU 작업 처리 ---
def 무거운_계산(n: int) -> int:
    return sum(i * i for i in range(n))

print("\n=== 2단계: 프로세스로 계산 처리 ===")
if __name__ == "__main__":
    시작 = time.time()
    with ProcessPoolExecutor() as executor:
        결과들 = list(executor.map(무거운_계산, 수집_데이터))
    print(f"계산 결과 합계: {sum(결과들)}")
    print(f"파이프라인 소요 시간: {time.time() - 시작:.2f}초")
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 올바른 해결책 |
|------|----------------------|--------------|
| `if __name__ == "__main__":` 없이 multiprocessing 실행 | `RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase.` | 항상 `if __name__ == "__main__":` 블록 안에서 프로세스를 시작한다 |
| `join()` 없이 스레드 결과를 바로 사용 | 빈 리스트, 0, `None` 등 예상치 못한 값 | 반드시 `t.join()` 또는 `future.result()`로 완료를 기다린다 |
| Lock 없이 공유 변수 수정 | 결과가 실행마다 달라진다 (비결정적) | 공유 변수는 `with lock:` 블록 안에서만 수정한다 |
| CPU 작업에 threading 사용 | 순차 실행보다 느리거나 같은 속도 | CPU 집약 작업은 `multiprocessing` 또는 `ProcessPoolExecutor`를 사용한다 |
| 프로세스 사이에서 일반 리스트 공유 | 변경사항이 다른 프로세스에 반영되지 않는다 | `multiprocessing.Queue` 또는 `multiprocessing.Manager().list()`를 사용한다 |
| `start()` 없이 `join()` 호출 | `RuntimeError: cannot join thread before it is started` | `t.start()` 먼저 호출한 다음 `t.join()`을 호출한다 |
| 너무 많은 스레드/프로세스 생성 | 메모리 부족, OS 오류 | `ThreadPoolExecutor(max_workers=N)` 으로 최대 개수를 제한한다 |

---

## 확인 체크리스트

- [ ] `threading.Thread(target=함수, args=(인자,))`로 스레드를 만들고 `.start()`, `.join()`을 순서대로 호출할 수 있다
- [ ] `ThreadPoolExecutor`로 여러 작업을 제출하고 `as_completed()`로 결과를 받을 수 있다
- [ ] GIL 때문에 파이썬 스레드가 CPU 작업에서 진짜 병렬 실행이 안 된다는 것을 설명할 수 있다
- [ ] 경쟁 조건이 무엇인지 예시와 함께 설명할 수 있다
- [ ] `threading.Lock()`으로 공유 변수를 안전하게 보호할 수 있다
- [ ] `multiprocessing.Process`와 `ProcessPoolExecutor`의 차이를 안다
- [ ] `if __name__ == "__main__":` 가 왜 필요한지 설명할 수 있다
- [ ] `multiprocessing.Queue`로 프로세스 사이에 데이터를 주고받을 수 있다
- [ ] I/O 작업과 CPU 작업 각각에 적합한 도구를 고를 수 있다
- [ ] 실습 3의 파이프라인 코드를 스스로 처음부터 다시 작성할 수 있다

---

## 한 번 더 생각해 보기

1. **스레드 10개를 만들면 항상 10배 빨라질까?**  
   I/O 대기가 많은 작업이라면 어느 정도 빨라지지만, CPU 작업이라면 GIL 때문에 오히려 느려질 수도 있다. 내 작업이 I/O 중심인지 CPU 중심인지 먼저 파악하는 것이 왜 중요할까?

2. **Lock을 너무 많이 걸면 어떤 문제가 생길까?**  
   Lock이 없으면 경쟁 조건이 생기고, Lock이 너무 많거나 잘못 걸면 **데드락(deadlock)**이 생겨 프로그램이 영원히 멈출 수 있다. 두 스레드가 서로 상대방의 Lock이 풀리기를 기다리는 상황을 직접 코드로 표현해 볼 수 있을까?

3. **`ProcessPoolExecutor`는 몇 개의 프로세스를 기본으로 만들까?**  
   기본값은 CPU 코어 수다. 내 컴퓨터의 코어 수를 `multiprocessing.cpu_count()`로 확인해 보고, 프로세스 수를 코어 수보다 많이 늘리면 성능이 어떻게 변하는지 `benchmark_compare.py`를 수정해서 실험해 보자.

---

## 다음 장

다음 장에서는 `asyncio`를 사용한 **비동기 프로그래밍**을 배운다. 스레드 없이도 수천 개의 I/O 작업을 동시에 처리하는 현대적인 방법을 익힌다.