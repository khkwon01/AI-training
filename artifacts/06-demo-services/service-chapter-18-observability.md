## 이 장에서 배우는 것

- 마이크로서비스에서 **로그(logging)** 를 구조적으로 남기는 방법
- **메트릭(metrics)** 을 수집해서 서비스 상태를 숫자로 파악하는 방법
- **분산 추적(distributed tracing)** 으로 요청이 여러 서비스를 거치는 경로를 확인하는 방법
- **헬스 체크(health check)** 엔드포인트를 만들어 서비스 생존 여부를 외부에서 확인하는 방법
- 위 세 가지를 묶어 **관찰 가능성(observability)** 이라고 부르는 이유

---

## 먼저 쉬운 설명

자동차를 운전하다가 갑자기 엔진 경고등이 켜졌다고 상상해 보세요.  
경고등만 있고 계기판이 없다면 "뭔가 잘못됐다"는 것만 알 뿐, 속도가 얼마인지, 연료가 얼마나 남았는지, 엔진 온도가 얼마인지 전혀 모릅니다.

마이크로서비스도 똑같습니다.  
서비스가 죽었을 때 "왜 죽었는지", "언제부터 느려졌는지", "어느 서비스에서 문제가 생겼는지" 알 수 없으면 고치는 데 몇 시간이 걸립니다.

**관찰 가능성**은 서비스의 계기판을 만드는 일입니다.  
로그는 **무슨 일이 있었는지** 기록하고, 메트릭은 **지금 상태가 어떤지** 숫자로 보여주고, 추적은 **요청이 어떤 경로로 흘렀는지** 지도처럼 보여줍니다.

이 세 가지가 갖춰지면 장애가 발생해도 5분 안에 원인을 찾을 수 있습니다.

---

## 1. 구조적 로깅 (Structured Logging)

일반 로그는 사람이 읽기는 쉽지만 기계가 분석하기 어렵습니다.

```python
# 나쁜 예 — 평문 로그
print("사용자 123이 주문 456을 했습니다")
```

구조적 로그는 JSON 형태로 남겨서 검색과 필터링이 쉽습니다.

```python
# order_service/logger.py
import logging
import json
import sys
from datetime import datetime, timezone


class JsonFormatter(logging.Formatter):
    """로그를 JSON 한 줄로 직렬화하는 포매터."""

    def format(self, record: logging.LogRecord) -> str:
        log_obj = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "service": "order-service",
        }
        # extra 필드가 있으면 그대로 포함
        if hasattr(record, "extra_fields"):
            log_obj.update(record.extra_fields)
        return json.dumps(log_obj, ensure_ascii=False)


def get_logger(name: str) -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)

    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(JsonFormatter())
    logger.addHandler(handler)
    return logger
```

```python
# order_service/main.py
from logger import get_logger

logger = get_logger(__name__)


def process_order(user_id: int, order_id: int) -> dict:
    # extra 필드로 컨텍스트 정보를 함께 기록
    extra = {"extra_fields": {"user_id": user_id, "order_id": order_id}}

    logger.info("주문 처리 시작", extra=extra)

    try:
        # 실제 주문 처리 로직이 여기 들어옴
        result = {"status": "success", "order_id": order_id}
        logger.info("주문 처리 완료", extra=extra)
        return result
    except Exception as exc:
        logger.error(
            "주문 처리 실패",
            extra={"extra_fields": {"user_id": user_id, "order_id": order_id, "error": str(exc)}},
        )
        raise
```

실행하면 아래처럼 출력됩니다.

```json
{"timestamp": "2026-05-18T09:00:00+00:00", "level": "INFO", "logger": "__main__", "message": "주문 처리 시작", "service": "order-service", "user_id": 1, "order_id": 42}
```

---

## 2. 메트릭 수집 (Metrics with Prometheus)

메트릭은 서비스를 숫자로 표현합니다.  
가장 널리 쓰이는 도구는 **Prometheus** 와 Python 클라이언트 라이브러리입니다.

```bash
pip install prometheus-client fastapi uvicorn
```

```python
# order_service/metrics.py
from prometheus_client import Counter, Histogram, Gauge, generate_latest, CONTENT_TYPE_LATEST
from fastapi import FastAPI, Response

app = FastAPI()

# 카운터: 계속 증가만 하는 숫자 (예: 요청 횟수)
REQUEST_COUNT = Counter(
    "order_requests_total",
    "전체 주문 요청 수",
    ["method", "endpoint", "status_code"],
)

# 히스토그램: 분포를 측정 (예: 응답 시간)
REQUEST_LATENCY = Histogram(
    "order_request_duration_seconds",
    "요청 처리 시간 (초)",
    ["endpoint"],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 2.0, 5.0],
)

# 게이지: 올라갔다 내려갔다 하는 숫자 (예: 현재 처리 중인 요청)
IN_FLIGHT_REQUESTS = Gauge(
    "order_in_flight_requests",
    "현재 처리 중인 요청 수",
)


@app.post("/orders")
async def create_order(user_id: int, order_id: int):
    IN_FLIGHT_REQUESTS.inc()
    try:
        with REQUEST_LATENCY.labels(endpoint="/orders").time():
            # 실제 비즈니스 로직
            result = {"status": "created", "order_id": order_id}
            REQUEST_COUNT.labels(method="POST", endpoint="/orders", status_code="201").inc()
            return result
    except Exception:
        REQUEST_COUNT.labels(method="POST", endpoint="/orders", status_code="500").inc()
        raise
    finally:
        IN_FLIGHT_REQUESTS.dec()


@app.get("/metrics")
async def metrics():
    """Prometheus가 주기적으로 이 엔드포인트를 수집합니다."""
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

`/metrics` 에 접속하면 Prometheus가 읽을 수 있는 형태로 숫자가 노출됩니다.

```
# HELP order_requests_total 전체 주문 요청 수
# TYPE order_requests_total counter
order_requests_total{endpoint="/orders",method="POST",status_code="201"} 3.0
```

---

## 3. 분산 추적 (Distributed Tracing with OpenTelemetry)

주문 서비스 → 재고 서비스 → 결제 서비스처럼 여러 서비스를 거칠 때 어디서 느려졌는지 추적합니다.

```bash
pip install opentelemetry-sdk opentelemetry-instrumentation-fastapi opentelemetry-exporter-otlp
```

```python
# order_service/tracing.py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import Resource


def setup_tracing(service_name: str) -> trace.Tracer:
    """서비스 시작 시 한 번 호출해서 추적을 설정합니다."""
    resource = Resource.create({"service.name": service_name})
    provider = TracerProvider(resource=resource)

    # Jaeger 또는 OTLP Collector 주소
    exporter = OTLPSpanExporter(endpoint="http://localhost:4317", insecure=True)
    provider.add_span_processor(BatchSpanProcessor(exporter))

    trace.set_tracer_provider(provider)
    return trace.get_tracer(service_name)
```

```python
# order_service/app.py
import httpx
from fastapi import FastAPI
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from tracing import setup_tracing
from opentelemetry import trace

app = FastAPI()
tracer = setup_tracing("order-service")

# FastAPI 요청/응답을 자동으로 스팬(span)으로 기록
FastAPIInstrumentor.instrument_app(app)


@app.post("/orders")
async def create_order(user_id: int, item_id: str):
    # 현재 스팬에 사용자 정의 속성 추가
    current_span = trace.get_current_span()
    current_span.set_attribute("user.id", user_id)
    current_span.set_attribute("item.id", item_id)

    # 재고 서비스 호출 — 자동으로 trace context가 헤더에 전파됨
    with tracer.start_as_current_span("check-inventory") as span:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"http://inventory-service/items/{item_id}/stock"
            )
            span.set_attribute("http.status_code", response.status_code)

    return {"status": "created"}
```

---

## 4. 헬스 체크 엔드포인트 (Health Check)

쿠버네티스나 로드 밸런서가 서비스가 살아있는지 주기적으로 확인합니다.  
두 가지 엔드포인트를 만드는 것이 표준입니다.

```python
# order_service/health.py
import asyncio
import time

import asyncpg
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 서비스 시작 시각 기록
START_TIME = time.time()


class HealthStatus(BaseModel):
    status: str          # "healthy" 또는 "unhealthy"
    uptime_seconds: float
    checks: dict[str, str]


@app.get("/health/live")
async def liveness():
    """
    프로세스가 살아있는지 확인합니다.
    단순히 200을 반환하면 됩니다.
    쿠버네티스 livenessProbe에서 사용합니다.
    """
    return {"status": "alive"}


@app.get("/health/ready", response_model=HealthStatus)
async def readiness():
    """
    트래픽을 받을 준비가 됐는지 확인합니다.
    DB 연결 등 의존성을 실제로 체크합니다.
    쿠버네티스 readinessProbe에서 사용합니다.
    """
    checks: dict[str, str] = {}

    # 데이터베이스 연결 체크
    try:
        conn = await asyncpg.connect("postgresql://user:pass@localhost/orders")
        await conn.fetchval("SELECT 1")
        await conn.close()
        checks["database"] = "ok"
    except Exception as exc:
        checks["database"] = f"error: {exc}"

    all_ok = all(v == "ok" for v in checks.values())
    return HealthStatus(
        status="healthy" if all_ok else "unhealthy",
        uptime_seconds=round(time.time() - START_TIME, 1),
        checks=checks,
    )
```

---

## 따라 하기 실습

### 실습 1 — 구조적 로거 적용하기

1. 프로젝트 폴더를 만듭니다.

```bash
mkdir observability-demo && cd observability-demo
python -m venv .venv && source .venv/bin/activate
pip install fastapi uvicorn
```

2. `logger.py` 파일을 만들고 위의 `JsonFormatter` 코드를 그대로 붙여넣습니다.

3. `app.py` 를 만들어 아래처럼 작성하고 실행합니다.

```python
# app.py
from fastapi import FastAPI
from logger import get_logger

app = FastAPI()
logger = get_logger("demo")


@app.get("/ping")
def ping():
    logger.info("ping 요청 수신", extra={"extra_fields": {"endpoint": "/ping"}})
    return {"message": "pong"}
```

```bash
uvicorn app:app --reload
```

4. 다른 터미널에서 `curl http://localhost:8000/ping` 을 실행하고 JSON 로그가 출력되는지 확인합니다.

---

### 실습 2 — 메트릭 엔드포인트 추가하기

실습 1의 `app.py` 에 메트릭을 추가합니다.

```bash
pip install prometheus-client
```

```python
# app.py (수정)
from fastapi import FastAPI, Response
from prometheus_client import Counter, generate_latest, CONTENT_TYPE_LATEST
from logger import get_logger

app = FastAPI()
logger = get_logger("demo")

PING_COUNT = Counter("ping_requests_total", "ping 요청 횟수")


@app.get("/ping")
def ping():
    PING_COUNT.inc()
    logger.info("ping 요청 수신", extra={"extra_fields": {"endpoint": "/ping"}})
    return {"message": "pong"}


@app.get("/metrics")
def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

`curl http://localhost:8000/ping` 을 여러 번 실행한 뒤 `curl http://localhost:8000/metrics` 로 카운터가 올라가는지 확인합니다.

---

### 실습 3 — 헬스 체크 엔드포인트 추가하기

실습 2의 `app.py` 에 헬스 체크를 추가합니다.  
데이터베이스가 없으므로 외부 HTTP 호출을 의존성으로 대신 사용합니다.

```python
# app.py (수정 — 헬스 체크 추가)
import httpx
import time

START_TIME = time.time()


@app.get("/health/live")
def liveness():
    return {"status": "alive"}


@app.get("/health/ready")
async def readiness():
    checks = {}
    try:
        async with httpx.AsyncClient(timeout=2.0) as client:
            r = await client.get("https://httpbin.org/status/200")
            checks["external_api"] = "ok" if r.status_code == 200 else f"status {r.status_code}"
    except Exception as exc:
        checks["external_api"] = f"error: {exc}"

    return {
        "status": "healthy" if all(v == "ok" for v in checks.values()) else "unhealthy",
        "uptime_seconds": round(time.time() - START_TIME, 1),
        "checks": checks,
    }
```

```bash
pip install httpx
curl http://localhost:8000/health/ready
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `logging.basicConfig()` 와 커스텀 핸들러를 같이 설정 | 로그가 두 번 출력됨 | `basicConfig` 호출을 제거하고 핸들러를 직접 추가 |
| `Counter` 를 함수 안에서 매번 생성 | `ValueError: Duplicated timeseries` | `Counter` 는 모듈 최상단에서 한 번만 생성 |
| `histogram.observe()` 에 초가 아닌 밀리초를 전달 | 메트릭은 수집되지만 값이 1000배 큰 이상한 숫자 | `time.time()` 차이 그대로 전달 (초 단위) |
| `/health/ready` 가 DB 연결 실패 시 500 대신 200 반환 | 로드 밸런서가 unhealthy 서비스에 계속 트래픽 전달 | `all_ok` 가 False 이면 `status_code=503` 을 명시적으로 반환 |
| `OTLPSpanExporter` 주소가 틀림 | `StatusCode.UNAVAILABLE: failed to connect` | 로컬 테스트 시 `insecure=True` 확인, 포트 4317 확인 |
| 로그에 한글 깨짐 | `\uc8fc\ubb38` 형태로 출력됨 | `json.dumps(..., ensure_ascii=False)` 추가 |

---

## 확인 체크리스트

- [ ] `JsonFormatter` 를 적용하면 로그가 JSON 한 줄로 출력된다
- [ ] `extra_fields` 로 `user_id`, `order_id` 같은 컨텍스트 정보를 로그에 포함할 수 있다
- [ ] `Counter`, `Histogram`, `Gauge` 의 차이를 설명할 수 있다
- [ ] `/metrics` 엔드포인트에서 Prometheus 형식의 텍스트를 확인했다
- [ ] `liveness` 와 `readiness` 의 차이를 설명할 수 있다
- [ ] `readiness` 가 의존성 체크에 실패하면 `503` 을 반환해야 한다는 것을 안다
- [ ] 분산 추적에서 **스팬(span)** 이 무엇인지 한 문장으로 설명할 수 있다
- [ ] 세 가지 신호(로그, 메트릭, 추적)를 합쳐 관찰 가능성이라고 부른다는 것을 안다

---

## 한 번 더 생각해 보기

1. 로그 레벨(`DEBUG`, `INFO`, `WARNING`, `ERROR`)을 어떤 기준으로 구분해야 할까요?  
   "사용자가 잘못된 입력을 보냈을 때"는 어느 레벨이 적절할지, "데이터베이스 연결이 끊겼을 때"는 어느 레벨이 적절할지 생각해 보세요.

2. `/health/ready` 가 DB 체크를 할 때 DB가 잠깐 느려지면 헬스 체크도 느려집니다.  
   타임아웃을 얼마로 설정하는 것이 좋을까요? 너무 짧으면 어떤 문제가 생기고, 너무 길면 어떤 문제가 생길까요?

3. 메트릭은 집계된 숫자이고 로그는 개별 이벤트입니다.  
   "초당 500 요청 중 5%가 실패한다"는 상황에서 메트릭만으로 충분할까요, 아니면 로그도 함께 봐야 할까요? 두 가지를 같이 써야 하는 이유를 생각해 보세요.

---

## 다음 장

다음 장에서는 Docker와 docker-compose를 사용해 이 서비스를 컨테이너로 패키징하고 Prometheus, Grafana와 함께 로컬에서 실제 대시보드를 구성하는 방법을 배웁니다.