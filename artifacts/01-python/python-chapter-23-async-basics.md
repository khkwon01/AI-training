## 이 장에서 배우는 것

- `async`와 `await` 키워드가 무엇인지 이해한다
- 동기(sync)와 비동기(async) 코드의 차이를 설명할 수 있다
- `asyncio` 라이브러리를 사용해 간단한 비동기 함수를 작성한다
- `httpx` 또는 `aiohttp`로 API를 비동기로 호출한다
- 여러 API 요청을 동시에 보내 속도를 높이는 방법을 익힌다

---

## 먼저 쉬운 설명

카페에서 커피를 주문한다고 생각해 보세요.

**동기 방식**: 점원이 커피 한 잔을 다 만들 때까지 다음 손님 주문을 받지 않습니다. 손님이 10명이면 10배 느립니다.

**비동기 방식**: 점원이 커피 머신을 돌려놓고, 기다리는 동안 다음 손님 주문을 받습니다. 10명의 커피를 거의 동시에 만듭니다.

API 호출도 마찬가지입니다. 서버 응답을 기다리는 시간이 길기 때문에, 그 시간에 다른 요청을 보내면 훨씬 빨라집니다. `async`/`await`는 파이썬에서 이런 "기다리는 동안 다른 일 하기"를 가능하게 해 주는 문법입니다.

---

## 1. 동기 vs 비동기 — 눈으로 비교하기

동기 코드는 위에서 아래로 순서대로 실행됩니다. 각 줄이 끝나야 다음 줄이 실행됩니다.

```python
# sync_example.py
import time

def 커피_만들기(이름):
    print(f"{이름}의 커피 제조 시작")
    time.sleep(2)  # 2초 기다림 (서버 응답 대기와 같은 상황)
    print(f"{이름}의 커피 완성!")

def main():
    커피_만들기("철수")
    커피_만들기("영희")
    커피_만들기("민준")

main()
# 총 6초 걸림
```

비동기 코드는 기다리는 구간에서 다른 작업을 처리할 수 있습니다.

```python
# async_example.py
import asyncio

async def 커피_만들기(이름):
    print(f"{이름}의 커피 제조 시작")
    await asyncio.sleep(2)  # 2초 기다리지만, 다른 작업 가능
    print(f"{이름}의 커피 완성!")

async def main():
    await asyncio.gather(
        커피_만들기("철수"),
        커피_만들기("영희"),
        커피_만들기("민준"),
    )

asyncio.run(main())
# 총 약 2초 걸림!
```

> **핵심**: `async def`로 함수를 선언하고, 기다려야 하는 곳에 `await`를 붙입니다. `asyncio.gather()`를 쓰면 여러 작업을 동시에 실행합니다.

---

## 2. `async`와 `await` 문법 이해하기

`async def`로 만든 함수를 **코루틴(coroutine)**이라고 부릅니다. 일반 함수와 다르게 작동합니다.

```python
# coroutine_basics.py

# 일반 함수
def 일반_함수():
    return "안녕"

# 코루틴 함수
async def 비동기_함수():
    return "안녕"

# 차이 확인
print(일반_함수())          # "안녕" 출력
print(비동기_함수())        # <coroutine object 비동기_함수 at 0x...> 출력!
                            # 실행되지 않고 코루틴 객체가 반환됨
```

코루틴을 실행하려면 반드시 `await`를 쓰거나, `asyncio.run()`으로 감싸야 합니다.

```python
# coroutine_run.py
import asyncio

async def 인사하기(이름: str) -> str:
    await asyncio.sleep(1)  # 네트워크 지연 흉내
    return f"안녕하세요, {이름}님!"

async def main():
    # await로 코루틴 실행
    결과 = await 인사하기("지수")
    print(결과)  # "안녕하세요, 지수님!"

# asyncio.run()은 async main을 실행하는 진입점
asyncio.run(main())
```

---

## 3. `httpx`로 실제 API 비동기 호출하기

`requests` 라이브러리는 동기 전용입니다. 비동기 HTTP 요청에는 `httpx`를 사용합니다.

```bash
pip install httpx
```

```python
# api_async_basic.py
import asyncio
import httpx

async def 날씨_가져오기(도시: str) -> dict:
    url = "https://wttr.in/{도시}?format=j1".format(도시=도시)
    
    async with httpx.AsyncClient() as client:
        response = await client.get(url, timeout=10.0)
        response.raise_for_status()  # 4xx, 5xx 에러 자동 처리
        return response.json()

async def main():
    print("날씨 요청 중...")
    데이터 = await 날씨_가져오기("Seoul")
    온도 = 데이터["current_condition"][0]["temp_C"]
    print(f"서울 현재 기온: {온도}°C")

asyncio.run(main())
```

---

## 4. 여러 API를 동시에 호출하기 — `asyncio.gather()`

`asyncio.gather()`가 비동기의 핵심입니다. 여러 코루틴을 동시에 실행하고 모든 결과를 기다립니다.

```python
# api_concurrent.py
import asyncio
import httpx
import time

CITIES = ["Seoul", "Tokyo", "London", "Paris", "New York"]

async def 도시_온도_가져오기(client: httpx.AsyncClient, 도시: str) -> str:
    url = f"https://wttr.in/{도시}?format=%t"
    response = await client.get(url, timeout=10.0)
    온도 = response.text.strip()
    return f"{도시}: {온도}"

async def main():
    시작 = time.time()
    
    # 하나의 AsyncClient를 재사용하는 것이 효율적
    async with httpx.AsyncClient() as client:
        작업들 = [도시_온도_가져오기(client, 도시) for 도시 in CITIES]
        결과들 = await asyncio.gather(*작업들)
    
    for 결과 in 결과들:
        print(결과)
    
    print(f"\n총 소요 시간: {time.time() - 시작:.2f}초")
    # 동기였다면 5배 더 걸렸을 것

asyncio.run(main())
```

---

## 5. 에러 처리와 타임아웃

실제 서비스에서는 네트워크 오류나 타임아웃을 반드시 처리해야 합니다.

```python
# api_error_handling.py
import asyncio
import httpx

async def 안전하게_가져오기(url: str) -> dict | None:
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(url, timeout=5.0)
            response.raise_for_status()
            return response.json()
    
    except httpx.TimeoutException:
        print(f"[타임아웃] {url} — 5초 내에 응답 없음")
        return None
    
    except httpx.HTTPStatusError as e:
        print(f"[HTTP 오류] {e.response.status_code}: {url}")
        return None
    
    except httpx.RequestError as e:
        print(f"[연결 오류] {url}: {e}")
        return None

async def main():
    urls = [
        "https://jsonplaceholder.typicode.com/todos/1",  # 정상
        "https://jsonplaceholder.typicode.com/todos/9999",  # 404
        "https://존재하지않는도메인12345.com/api",  # 연결 오류
    ]
    
    결과들 = await asyncio.gather(*[안전하게_가져오기(u) for u in urls])
    
    for url, 결과 in zip(urls, 결과들):
        if 결과:
            print(f"성공: {결과}")

asyncio.run(main())
```

---

## 따라 하기 실습

### 실습 1 — 나만의 첫 비동기 함수 만들기

`practice_async_01.py` 파일을 만들고 아래를 따라 작성해 보세요.

```python
# practice_async_01.py
import asyncio

async def 숫자_세기(이름: str, 초: int):
    """주어진 초 동안 기다린 후 완료 메시지를 출력합니다."""
    print(f"[시작] {이름} — {초}초 걸림")
    await asyncio.sleep(초)
    print(f"[완료] {이름}!")

async def main():
    # TODO: asyncio.gather()를 사용해 세 작업을 동시에 실행해 보세요
    # 힌트: 숫자_세기("작업A", 3), 숫자_세기("작업B", 1), 숫자_세기("작업C", 2)
    pass

asyncio.run(main())
```

**예상 출력** (순서가 동시에 시작되고, 빠른 것이 먼저 완료됩니다):
```
[시작] 작업A — 3초 걸림
[시작] 작업B — 1초 걸림
[시작] 작업C — 2초 걸림
[완료] 작업B!
[완료] 작업C!
[완료] 작업A!
```

---

### 실습 2 — 공개 API에서 데이터 가져오기

`practice_async_02.py` 파일을 만들어 JSONPlaceholder API에서 게시글 3개를 동시에 가져옵니다.

```python
# practice_async_02.py
import asyncio
import httpx

BASE_URL = "https://jsonplaceholder.typicode.com"

async def 게시글_가져오기(client: httpx.AsyncClient, 게시글_id: int) -> dict:
    response = await client.get(f"{BASE_URL}/posts/{게시글_id}")
    response.raise_for_status()
    return response.json()

async def main():
    async with httpx.AsyncClient() as client:
        # TODO: 게시글 id 1, 2, 3을 동시에 가져와서 제목을 출력하세요
        # 힌트: asyncio.gather()와 리스트 컴프리헨션을 활용하세요
        pass

asyncio.run(main())
```

---

### 실습 3 — 에러 처리가 포함된 API 클라이언트

`practice_async_03.py` 파일에서 실습 2를 발전시킵니다. 존재하지 않는 게시글(id=9999)을 포함시키고, `try/except`로 에러를 처리해 프로그램이 중단되지 않도록 만들어 보세요.

```python
# practice_async_03.py
import asyncio
import httpx

BASE_URL = "https://jsonplaceholder.typicode.com"

async def 안전하게_게시글_가져오기(client: httpx.AsyncClient, 게시글_id: int) -> dict | None:
    # TODO: try/except로 HTTPStatusError와 RequestError를 처리하세요
    # 에러 발생 시 None을 반환하고 에러 메시지를 출력하세요
    pass

async def main():
    ids = [1, 5, 9999, 10]  # 9999는 존재하지 않음
    
    async with httpx.AsyncClient() as client:
        결과들 = await asyncio.gather(
            *[안전하게_게시글_가져오기(client, id) for id in ids]
        )
    
    for 게시글 in 결과들:
        if 게시글:
            print(f"제목: {게시글['title'][:30]}...")

asyncio.run(main())
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| `async def` 함수를 `await` 없이 호출 | `RuntimeWarning: coroutine 'xxx' was never awaited` | 호출 앞에 `await` 추가 |
| 일반 함수 안에서 `await` 사용 | `SyntaxError: 'await' outside async function` | 함수를 `async def`로 변경 |
| `asyncio.run()` 안에서 다시 `asyncio.run()` 호출 | `RuntimeError: This event loop is already running` | 중첩 금지; 안쪽은 `await`만 사용 |
| `requests` 라이브러리를 비동기에서 사용 | 코드가 블로킹됨, 속도 개선 없음 | `httpx.AsyncClient` 또는 `aiohttp` 사용 |
| `AsyncClient`를 요청마다 새로 생성 | 성능 저하, 연결 낭비 | `async with httpx.AsyncClient() as client:` 블록 하나로 재사용 |
| `gather()` 안의 코루틴 하나가 실패하면 전체 취소됨 | `ExceptionGroup` 또는 중간에 프로그램 종료 | `return_exceptions=True` 옵션 사용 |
| `time.sleep()` 을 비동기 함수에서 사용 | 이벤트 루프 전체가 멈춤 | `await asyncio.sleep()` 사용 |

---

## 확인 체크리스트

- [ ] `async def`와 일반 `def`의 차이를 말로 설명할 수 있다
- [ ] `await`를 빠뜨렸을 때 어떤 경고가 뜨는지 알고 있다
- [ ] `asyncio.gather()`로 여러 코루틴을 동시에 실행할 수 있다
- [ ] `asyncio.run()`이 진입점 역할을 한다는 것을 이해한다
- [ ] `httpx.AsyncClient`를 사용해 GET 요청을 보낼 수 있다
- [ ] `try/except`로 `httpx.TimeoutException`과 `HTTPStatusError`를 처리할 수 있다
- [ ] `time.sleep()` 대신 `await asyncio.sleep()`을 써야 하는 이유를 안다
- [ ] 실습 3의 코드를 스스로 완성해 정상 출력을 확인했다

---

## 한 번 더 생각해 보기

1. **동시성 vs 병렬성**: `asyncio.gather()`는 진짜로 동시에 실행되는 걸까요, 아니면 빠르게 번갈아가며 실행되는 걸까요? 파이썬의 GIL(Global Interpreter Lock)과 관련해서 생각해 보세요.

2. **언제 비동기를 쓰면 좋을까요?**: 파일 읽기/쓰기나 데이터베이스 쿼리에도 비동기가 도움이 될까요? CPU를 많이 쓰는 계산(예: 이미지 변환)에는 어떨까요?

3. **에러 전파**: `asyncio.gather(작업1(), 작업2(), 작업3())`에서 `작업2`가 예외를 발생시키면 `작업1`과 `작업3`은 어떻게 될까요? `return_exceptions=True`를 추가하면 어떻게 달라지는지 직접 실험해 보세요.

---

## 다음 장

다음 장에서는 비동기 API 호출을 실제 서비스에 적용하기 위해 **FastAPI로 비동기 REST API 서버를 만드는 방법**을 배웁니다.