## 이 장에서 배우는 것

- 캐싱이 무엇인지, 왜 Lambda에서 특히 중요한지 이해한다
- Lambda 전역 변수를 이용한 인메모리 캐싱을 직접 구현한다
- TTL(Time To Live)을 사용해 캐시 만료를 관리한다
- 캐시 히트(cache hit)와 캐시 미스(cache miss)의 차이를 안다
- 캐시 무효화(invalidation) 개념과 언제 캐시를 지워야 하는지 이해한다

---

## 먼저 쉬운 설명

음식점에서 메뉴판을 생각해 보자.

손님이 올 때마다 주방장이 직접 나와서 "오늘 메뉴가 뭐예요?"라고 물어본다고 하면 너무 비효율적이다. 그래서 메뉴판을 만들어 두고, 손님은 메뉴판을 보면 된다. 메뉴가 바뀔 때만 메뉴판을 새로 만들면 충분하다.

프로그래밍에서 **캐싱(caching)** 이 바로 이 메뉴판 역할이다.

매번 데이터베이스나 외부 API에 요청하는 대신, 한 번 가져온 데이터를 **잠깐 저장해 두고** 다음 요청에 바로 꺼내 쓴다.

Lambda에서는 이게 특히 중요하다. Lambda는 요청이 올 때마다 실행되기 때문에, 매번 DB 연결을 열고 데이터를 가져오면 **시간도 오래 걸리고 비용도 높아진다.** 잘 설계된 캐싱 전략 하나가 응답 시간을 10배 이상 줄여 줄 수 있다.

---

## 1. 캐싱이 왜 필요한가

### 캐싱 없이 매번 DB를 조회하는 코드

```python
# lambda_function.py (캐싱 없는 버전)
import boto3
import json

def lambda_handler(event, context):
    # 요청마다 DB에서 설정값을 새로 가져온다
    dynamodb = boto3.resource("dynamodb")
    table = dynamodb.Table("app-config")

    response = table.get_item(Key={"config_key": "feature_flags"})
    config = response["Item"]

    return {
        "statusCode": 200,
        "body": json.dumps({"flags": config["value"]})
    }
```

위 코드의 문제점:

| 문제 | 설명 |
|------|------|
| 느린 응답 | 매 요청마다 DynamoDB 네트워크 호출 발생 (수십~수백 ms) |
| 비용 증가 | DynamoDB 읽기 요청 횟수만큼 비용 청구 |
| 불필요한 반복 | 설정값은 거의 바뀌지 않는데 매번 새로 읽음 |

### 응답 시간 비교 (개념)

```
캐싱 없음:  [Lambda 시작] → [DB 연결] → [DB 조회 150ms] → [응답] = 총 ~200ms
캐싱 있음:  [Lambda 시작] → [메모리 조회 0.1ms]          → [응답] = 총  ~50ms
```

---

## 2. Lambda 전역 변수 캐싱

Lambda의 중요한 특성이 있다. 같은 Lambda **인스턴스**가 여러 요청을 처리할 때, **전역 변수는 요청 사이에 유지된다.** 이 특성을 이용하면 무료로 간단한 캐싱을 구현할 수 있다.

### 기본 전역 변수 캐싱

```python
# lambda_function.py
import boto3
import json
import time

# 전역 변수 — Lambda 인스턴스가 살아있는 동안 유지된다
_config_cache = None
_cache_loaded_at = 0
CACHE_TTL_SECONDS = 300  # 5분

def _get_config():
    """설정값을 캐시에서 가져오거나, 만료됐으면 DB에서 새로 로드한다."""
    global _config_cache, _cache_loaded_at

    now = time.time()

    # 캐시가 비어있거나 TTL이 지났으면 새로 로드
    if _config_cache is None or (now - _cache_loaded_at) > CACHE_TTL_SECONDS:
        print("캐시 미스 — DynamoDB에서 설정값 로드")
        dynamodb = boto3.resource("dynamodb")
        table = dynamodb.Table("app-config")
        response = table.get_item(Key={"config_key": "feature_flags"})
        _config_cache = response["Item"]
        _cache_loaded_at = now
    else:
        print("캐시 히트 — 저장된 값 사용")

    return _config_cache


def lambda_handler(event, context):
    config = _get_config()

    return {
        "statusCode": 200,
        "body": json.dumps({"flags": config["value"]})
    }
```

### 코드 흐름 이해하기

```
첫 번째 요청:
  _config_cache = None  →  DB 조회  →  _config_cache에 저장  →  응답

두 번째~N번째 요청 (5분 이내):
  _config_cache에 값 있음  →  바로 반환  →  응답 (DB 조회 없음!)

5분 후 요청:
  TTL 초과  →  DB 재조회  →  캐시 갱신  →  응답
```

---

## 3. TTL과 캐시 무효화

**TTL(Time To Live)** 은 캐시가 유효한 시간이다. 시간이 지나면 캐시를 버리고 새 데이터를 가져온다.

### TTL 설계 기준

| 데이터 종류 | 권장 TTL | 이유 |
|-------------|----------|------|
| 앱 설정값 | 5~10분 | 자주 안 바뀜, 오래 캐시해도 됨 |
| 사용자 프로필 | 1~2분 | 가끔 바뀔 수 있음 |
| 실시간 재고 | 10~30초 | 자주 바뀌므로 짧게 |
| 주식 가격 | 캐싱 하지 않음 | 실시간이 핵심 |

### 수동 캐시 무효화

```python
# cache_manager.py
import time

class SimpleCache:
    def __init__(self, ttl_seconds: int):
        self._data = {}
        self._timestamps = {}
        self.ttl = ttl_seconds

    def get(self, key: str):
        """캐시에서 값을 가져온다. 만료됐거나 없으면 None 반환."""
        if key not in self._data:
            return None

        age = time.time() - self._timestamps[key]
        if age > self.ttl:
            # 만료된 캐시 삭제
            del self._data[key]
            del self._timestamps[key]
            return None

        return self._data[key]

    def set(self, key: str, value):
        """값을 캐시에 저장한다."""
        self._data[key] = value
        self._timestamps[key] = time.time()

    def invalidate(self, key: str):
        """특정 키의 캐시를 강제로 삭제한다."""
        self._data.pop(key, None)
        self._timestamps.pop(key, None)

    def clear(self):
        """전체 캐시를 비운다."""
        self._data.clear()
        self._timestamps.clear()


# 전역 캐시 인스턴스 — Lambda 인스턴스 생명주기 동안 유지
_cache = SimpleCache(ttl_seconds=300)


def lambda_handler(event, context):
    action = event.get("action", "get")

    if action == "invalidate":
        # 관리자가 캐시를 강제로 비울 때
        _cache.invalidate("user_config")
        return {"statusCode": 200, "body": "캐시 무효화 완료"}

    # 캐시에서 먼저 시도
    config = _cache.get("user_config")
    if config is None:
        # 캐시 미스: DB에서 로드
        config = {"feature_a": True, "feature_b": False}  # 실제로는 DB 조회
        _cache.set("user_config", config)

    return {"statusCode": 200, "body": str(config)}
```

---

## 4. DB 연결도 캐싱한다

캐싱은 데이터만이 아니다. **DB 연결 자체**도 비용이 크다. Lambda에서 DB 클라이언트를 전역으로 선언하면 재사용된다.

```python
# lambda_function.py
import boto3
import json

# Lambda 인스턴스 시작 시 한 번만 실행 — 연결 재사용
dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("users")

# 데이터 캐시도 전역으로
_user_cache = {}


def lambda_handler(event, context):
    user_id = event["pathParameters"]["userId"]

    # 캐시 확인
    if user_id in _user_cache:
        print(f"캐시 히트: user_id={user_id}")
        return {
            "statusCode": 200,
            "body": json.dumps(_user_cache[user_id])
        }

    # 캐시 미스: DB 조회 (연결은 이미 열려있음)
    print(f"캐시 미스: user_id={user_id}")
    response = table.get_item(Key={"user_id": user_id})

    if "Item" not in response:
        return {"statusCode": 404, "body": "사용자를 찾을 수 없습니다"}

    user = response["Item"]
    _user_cache[user_id] = user  # 캐시에 저장

    return {
        "statusCode": 200,
        "body": json.dumps(user)
    }
```

> **주의:** 이 패턴은 메모리를 사용한다. 사용자가 매우 많다면 캐시 크기를 제한해야 한다. 간단한 방법은 캐시 키 수가 일정 이상이면 전체를 비우는 것이다.

---

## 따라 하기 실습

### 실습 1 — 캐싱 없는 기본 Lambda 작성

`artifacts/06-demo-services/caching-demo/lambda_no_cache.py` 파일을 만든다.

```python
# lambda_no_cache.py
import json
import time

def get_product_price(product_id: str) -> dict:
    """실제 DB 조회를 흉내 낸다 (0.2초 지연)."""
    time.sleep(0.2)  # DB 조회 시간 시뮬레이션
    prices = {
        "apple": 1500,
        "banana": 800,
        "orange": 2000,
    }
    return {"product_id": product_id, "price": prices.get(product_id, 0)}


def lambda_handler(event, context):
    start = time.time()

    product_id = event.get("product_id", "apple")
    result = get_product_price(product_id)  # 매번 DB 조회

    elapsed = round((time.time() - start) * 1000, 1)
    result["elapsed_ms"] = elapsed

    return {"statusCode": 200, "body": json.dumps(result)}
```

이 파일을 저장한 뒤 `lambda_handler({"product_id": "apple"}, {})` 를 세 번 실행해 보자. 매번 200ms 이상 걸린다는 것을 확인한다.

---

### 실습 2 — 전역 변수 캐싱 추가

`artifacts/06-demo-services/caching-demo/lambda_with_cache.py` 파일을 만든다. 실습 1 코드에 캐싱을 추가한다.

```python
# lambda_with_cache.py
import json
import time

# 전역 캐시 딕셔너리
_price_cache = {}
_cache_timestamps = {}
CACHE_TTL = 60  # 60초


def get_product_price(product_id: str) -> dict:
    """캐시를 확인하고, 없으면 DB에서 가져온다."""
    now = time.time()

    # 1. 캐시 히트 확인
    if product_id in _price_cache:
        age = now - _cache_timestamps[product_id]
        if age < CACHE_TTL:
            print(f"[캐시 히트] {product_id}, 캐시 나이: {age:.1f}초")
            return _price_cache[product_id]
        else:
            print(f"[캐시 만료] {product_id}")

    # 2. 캐시 미스 — DB 조회
    print(f"[캐시 미스] {product_id} — DB 조회 중...")
    time.sleep(0.2)  # DB 조회 시뮬레이션
    prices = {"apple": 1500, "banana": 800, "orange": 2000}
    data = {"product_id": product_id, "price": prices.get(product_id, 0)}

    # 3. 캐시에 저장
    _price_cache[product_id] = data
    _cache_timestamps[product_id] = now

    return data


def lambda_handler(event, context):
    start = time.time()

    product_id = event.get("product_id", "apple")
    result = get_product_price(product_id)

    elapsed = round((time.time() - start) * 1000, 1)
    result["elapsed_ms"] = elapsed

    return {"statusCode": 200, "body": json.dumps(result)}
```

같은 `product_id`로 세 번 호출해 보자. 첫 번째만 200ms이고, 이후에는 1ms 미만이 되는 것을 확인한다.

---

### 실습 3 — 캐시 무효화 엔드포인트 추가

실습 2 파일에 캐시를 강제로 비우는 기능을 추가한다.

```python
# lambda_with_cache.py 하단에 lambda_handler를 아래로 교체

def lambda_handler(event, context):
    start = time.time()

    # 캐시 무효화 요청 처리
    if event.get("action") == "invalidate":
        product_id = event.get("product_id")
        if product_id:
            _price_cache.pop(product_id, None)
            _cache_timestamps.pop(product_id, None)
            return {"statusCode": 200, "body": f"{product_id} 캐시 삭제 완료"}
        else:
            _price_cache.clear()
            _cache_timestamps.clear()
            return {"statusCode": 200, "body": "전체 캐시 삭제 완료"}

    # 일반 조회
    product_id = event.get("product_id", "apple")
    result = get_product_price(product_id)
    elapsed = round((time.time() - start) * 1000, 1)
    result["elapsed_ms"] = elapsed

    return {"statusCode": 200, "body": json.dumps(result)}
```

아래 순서로 테스트한다.

```python
# 1. 첫 조회 (캐시 미스, ~200ms)
lambda_handler({"product_id": "apple"}, {})

# 2. 재조회 (캐시 히트, ~0ms)
lambda_handler({"product_id": "apple"}, {})

# 3. 캐시 무효화
lambda_handler({"action": "invalidate", "product_id": "apple"}, {})

# 4. 다시 조회 (캐시 미스, ~200ms)
lambda_handler({"product_id": "apple"}, {})
```

---

## 자주 하는 실수

| 실수 | 발생하는 에러 또는 증상 | 해결 방법 |
|------|------------------------|-----------|
| `global` 선언을 빠뜨림 | 전역 변수를 함수 안에서 수정했는데 반영 안 됨 | 값을 재할당하는 변수 앞에 `global _cache_var` 선언 추가 |
| TTL 없이 캐싱 | 오래된 데이터가 계속 반환됨, 데이터 불일치 | `time.time()` 기반 TTL 체크 로직 추가 |
| 캐시 크기 무제한 | Lambda 메모리 초과 → `MemoryError` | 최대 항목 수 제한 후 초과 시 `_cache.clear()` 호출 |
| Lambda 콜드 스타트 시 캐시 없음 | 트래픽 급증 시 갑자기 응답이 느려짐 | 정상 동작이다. 캐시는 웜 인스턴스에서만 유효함을 이해하고 설계 |
| 민감한 데이터를 전역에 저장 | 다른 요청에서 개인정보가 노출될 수 있음 | 사용자별 민감 데이터는 전역 캐시에 저장하지 말 것 |
| `KeyError: 'Item'` | DynamoDB에 없는 키 조회 후 바로 `response["Item"]` 접근 | `"Item" in response` 로 먼저 확인 |

---

## 확인 체크리스트

- [ ] 캐싱이 왜 필요한지 말로 설명할 수 있다
- [ ] Lambda 전역 변수가 요청 사이에 유지된다는 것을 이해했다
- [ ] `_cache_var = None` 을 함수 밖(전역)에 선언했다
- [ ] TTL을 초 단위로 정하고 `time.time()` 으로 나이를 계산했다
- [ ] 캐시 히트와 캐시 미스를 로그로 구분해서 출력했다
- [ ] 캐시 무효화 경로(`action == "invalidate"`)를 구현했다
- [ ] 실습 3의 4단계 테스트를 모두 실행하고 응답 시간 차이를 확인했다

---

## 한 번 더 생각해 보기

1. Lambda 인스턴스가 두 개 이상 동시에 실행되면 캐시 상태가 어떻게 될까? 인스턴스 A와 B의 캐시가 서로 다를 수 있는 상황을 상상해 보자. 이걸 해결하려면 어떤 방법이 있을까?

2. TTL을 너무 길게 잡으면 어떤 문제가 생길까? 반대로 너무 짧게 잡으면? 본인이 만드는 서비스에서 적당한 TTL은 어느 정도일지 생각해 보자.

3. 지금까지 배운 전역 변수 캐싱은 Lambda 인스턴스 하나에만 존재한다. 수백 개의 Lambda 인스턴스가 동시에 실행된다면, 모든 인스턴스가 공유하는 캐시가 필요할 텐데 그런 서비스로는 무엇이 있을까?

---

## 다음 장

다음 장에서는 여러 Lambda 인스턴스가 공유하는 외부 캐시 서비스인 **ElastiCache(Redis)** 를 Lambda에 연결하는 방법을 배운다.