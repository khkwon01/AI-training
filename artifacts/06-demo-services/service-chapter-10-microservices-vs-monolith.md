## 이 장에서 배우는 것

- 모놀리식(Monolith) 아키텍처가 무엇인지 이해한다
- 마이크로서비스(Microservices) 아키텍처가 무엇인지 이해한다
- 두 방식의 장단점을 구체적으로 비교할 수 있다
- 내 프로젝트에 어떤 방식이 맞는지 스스로 판단하는 기준을 갖는다
- 초보자가 마이크로서비스를 너무 일찍 선택했을 때 겪는 문제를 안다

---

## 먼저 쉬운 설명

레스토랑을 떠올려 보세요.

**모놀리식**은 한 명의 요리사가 주문받기, 요리하기, 계산까지 모두 하는 작은 식당입니다. 처음엔 빠르고 간단합니다. 하지만 손님이 100명으로 늘면 그 한 명이 버티지 못합니다.

**마이크로서비스**는 홀 직원, 주방장, 계산원이 각자 역할을 나눈 대형 레스토랑입니다. 효율적이지만, 직원들이 서로 "언제 음식 나와요?", "카드 결제 됐나요?" 하고 계속 소통해야 합니다. 작은 식당에 이 구조를 적용하면 오히려 복잡해집니다.

개발도 마찬가지입니다. **팀 규모, 트래픽, 서비스 복잡도에 따라** 맞는 방식이 다릅니다. 처음부터 마이크로서비스를 선택하면 배포도 어렵고, 디버깅도 힘들고, 혼자 개발하기엔 너무 무겁습니다.

이 장을 마치면 여러분은 "우리 프로젝트는 지금 어떤 구조가 맞을까?"라는 질문에 스스로 답할 수 있게 됩니다.

---

## 1. 모놀리식 아키텍처란?

모놀리식은 모든 기능이 **하나의 코드베이스, 하나의 프로세스**로 실행되는 구조입니다.

```
my-shop/
├── main.py
├── routes/
│   ├── user.py        # 사용자 관련 엔드포인트
│   ├── product.py     # 상품 관련 엔드포인트
│   └── order.py       # 주문 관련 엔드포인트
├── models/
│   ├── user.py
│   ├── product.py
│   └── order.py
└── database.py        # DB 연결 하나
```

```python
# main.py — 모든 기능이 한 애플리케이션 안에 있음
from fastapi import FastAPI
from routes import user, product, order

app = FastAPI()

app.include_router(user.router, prefix="/users")
app.include_router(product.router, prefix="/products")
app.include_router(order.router, prefix="/orders")
```

```python
# routes/order.py — 주문 처리 시 사용자와 상품 정보를 직접 import해서 사용
from models.user import get_user
from models.product import get_product
from database import db

def create_order(user_id: int, product_id: int):
    user = get_user(user_id)       # 같은 프로세스 안에서 직접 호출
    product = get_product(product_id)
    
    if product.stock < 1:
        raise ValueError("재고가 없습니다")
    
    order = {"user": user, "product": product}
    db.save(order)
    return order
```

**모놀리식의 특징:**

| 항목 | 설명 |
|------|------|
| 배포 | `python main.py` 한 번으로 전체 서비스 실행 |
| 통신 | 함수 호출 (네트워크 없음, 빠름) |
| DB | 하나의 데이터베이스 공유 |
| 디버깅 | 에러 로그가 한 곳에 모여 있어 찾기 쉬움 |

---

## 2. 마이크로서비스 아키텍처란?

마이크로서비스는 각 기능을 **독립적인 서비스(프로세스)로 분리**한 구조입니다. 서비스 간 통신은 HTTP API 또는 메시지 큐를 사용합니다.

```
my-shop/
├── user-service/       # 포트 8001에서 실행
│   ├── main.py
│   └── models.py
├── product-service/    # 포트 8002에서 실행
│   ├── main.py
│   └── models.py
├── order-service/      # 포트 8003에서 실행
│   ├── main.py
│   └── models.py
└── docker-compose.yml  # 세 서비스를 함께 띄우는 설정
```

```python
# order-service/main.py — 다른 서비스 데이터가 필요하면 HTTP로 요청
import httpx
from fastapi import FastAPI, HTTPException

app = FastAPI()

USER_SERVICE_URL = "http://user-service:8001"
PRODUCT_SERVICE_URL = "http://product-service:8002"

@app.post("/orders")
async def create_order(user_id: int, product_id: int):
    # 사용자 서비스에 HTTP 요청
    async with httpx.AsyncClient() as client:
        user_resp = await client.get(f"{USER_SERVICE_URL}/users/{user_id}")
        if user_resp.status_code != 200:
            raise HTTPException(status_code=404, detail="사용자를 찾을 수 없습니다")
        
        # 상품 서비스에 HTTP 요청
        product_resp = await client.get(f"{PRODUCT_SERVICE_URL}/products/{product_id}")
        if product_resp.status_code != 200:
            raise HTTPException(status_code=404, detail="상품을 찾을 수 없습니다")
    
    user = user_resp.json()
    product = product_resp.json()
    
    return {"user": user, "product": product, "status": "주문 완료"}
```

```yaml
# docker-compose.yml — 세 서비스를 각각 컨테이너로 실행
version: "3.9"
services:
  user-service:
    build: ./user-service
    ports:
      - "8001:8001"

  product-service:
    build: ./product-service
    ports:
      - "8002:8002"

  order-service:
    build: ./order-service
    ports:
      - "8003:8003"
    depends_on:
      - user-service
      - product-service
```

**마이크로서비스의 특징:**

| 항목 | 설명 |
|------|------|
| 배포 | 서비스마다 별도로 배포, 독립 업데이트 가능 |
| 통신 | HTTP, gRPC, 메시지 큐 (네트워크 지연 발생) |
| DB | 서비스마다 별도 DB 권장 |
| 디버깅 | 에러가 여러 서비스에 분산되어 추적이 복잡 |

---

## 3. 두 방식 비교: 언제 무엇을 선택할까?

```python
# 판단 기준 체크리스트 (코드가 아닌 의사결정 도구)
decision_guide = {
    "팀 규모": {
        "1~3명": "모놀리식 권장",
        "5명 이상, 팀별 독립 배포 필요": "마이크로서비스 고려"
    },
    "트래픽": {
        "하루 수천 건 이하": "모놀리식으로 충분",
        "특정 기능만 트래픽 폭증": "해당 기능만 분리 고려"
    },
    "도메인 복잡도": {
        "기능이 서로 강하게 연결됨": "모놀리식 유리",
        "기능이 독립적으로 운영 가능": "마이크로서비스 유리"
    },
    "배포 주기": {
        "전체 서비스 동시 배포": "모놀리식 간단",
        "서비스마다 다른 배포 주기": "마이크로서비스 유리"
    }
}

# 초보자를 위한 현실적인 조언
print("👉 혼자 또는 소규모 팀이라면: 모놀리식으로 시작하세요")
print("👉 나중에 특정 기능만 분리하는 '모듈러 모놀리스' 방식도 있습니다")
print("👉 마이크로서비스는 운영 복잡도가 높아 경험 없이는 독이 됩니다")
```

**시각적 비교표:**

| 비교 항목 | 모놀리식 | 마이크로서비스 |
|-----------|----------|----------------|
| 초기 개발 속도 | 빠름 | 느림 (인프라 설정 포함) |
| 배포 복잡도 | 낮음 | 높음 |
| 장애 격리 | 어려움 (하나 죽으면 전체) | 쉬움 (해당 서비스만 영향) |
| 팀 독립성 | 낮음 | 높음 |
| 모니터링 | 간단 | 복잡 (분산 추적 필요) |
| 초보자 추천 | ✅ 강력 추천 | ⚠️ 경험 후 도입 권장 |

---

## 4. 마이크로서비스 도입 시 실제로 추가되는 것들

마이크로서비스를 선택하면 **기능 개발 외에** 다음 것들을 추가로 관리해야 합니다.

```python
# 마이크로서비스 환경에서 필요한 추가 인프라 예시

# 1. API 게이트웨이 — 외부 요청을 각 서비스로 라우팅
# 2. 서비스 디스커버리 — 서비스 주소를 동적으로 찾는 방법
# 3. 분산 추적 — 요청이 어느 서비스에서 실패했는지 추적

# 예: 분산 추적 없이 에러가 나면 어디서 실패했는지 모름
# order-service 로그:
# ERROR: 주문 생성 실패 - Connection timeout

# 원인이 user-service인지, product-service인지, DB인지 바로 알 수 없음
# → Jaeger, Zipkin 같은 분산 추적 도구 필요

# 예: 서비스 간 인증도 별도로 처리해야 함
import jwt

def verify_internal_request(token: str):
    """서비스끼리 통신할 때도 인증 토큰이 필요합니다"""
    try:
        payload = jwt.decode(token, "내부-서비스-시크릿", algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise Exception("서비스 토큰이 만료되었습니다")
```

**추가로 공부해야 할 것들:**

```
모놀리식 → 마이크로서비스 전환 시 학습 필요 목록
├── Docker / Kubernetes
├── API Gateway (Kong, AWS API Gateway 등)
├── 서비스 메시 (Istio 등)
├── 분산 추적 (Jaeger, OpenTelemetry)
├── 메시지 큐 (Kafka, RabbitMQ)
└── 각 서비스별 CI/CD 파이프라인
```

---

## 따라 하기 실습

### 실습 1: 모놀리식 쇼핑몰 구조 만들기

다음 파일 구조를 직접 만들고 실행해 보세요.

```bash
mkdir monolith-shop
cd monolith-shop
pip install fastapi uvicorn
```

```python
# monolith-shop/main.py
from fastapi import FastAPI, HTTPException

app = FastAPI()

# 간단한 인메모리 데이터
users_db = {1: {"id": 1, "name": "김철수", "email": "kim@example.com"}}
products_db = {1: {"id": 1, "name": "파이썬 책", "price": 30000, "stock": 5}}
orders_db = []

@app.get("/users/{user_id}")
def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="사용자 없음")
    return users_db[user_id]

@app.get("/products/{product_id}")
def get_product(product_id: int):
    if product_id not in products_db:
        raise HTTPException(status_code=404, detail="상품 없음")
    return products_db[product_id]

@app.post("/orders")
def create_order(user_id: int, product_id: int):
    # 같은 프로세스 안에서 직접 함수 호출 — 네트워크 없음
    user = get_user(user_id)
    product = get_product(product_id)
    
    if products_db[product_id]["stock"] < 1:
        raise HTTPException(status_code=400, detail="재고 없음")
    
    products_db[product_id]["stock"] -= 1
    order = {"user": user["name"], "product": product["name"], "status": "완료"}
    orders_db.append(order)
    return order
```

```bash
# 실행
uvicorn main:app --reload

# 테스트
curl -X POST "http://localhost:8000/orders?user_id=1&product_id=1"
# 결과: {"user": "김철수", "product": "파이썬 책", "status": "완료"}
```

---

### 실습 2: 같은 기능을 마이크로서비스 구조로 분리해 보기

실습 1의 코드를 두 개의 서비스로 나눠 봅니다. **얼마나 복잡해지는지 직접 느끼는 것이 목표**입니다.

```bash
mkdir microservices-shop
cd microservices-shop
mkdir user-service order-service
```

```python
# microservices-shop/user-service/main.py
from fastapi import FastAPI, HTTPException

app = FastAPI()
users_db = {1: {"id": 1, "name": "김철수", "email": "kim@example.com"}}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="사용자 없음")
    return users_db[user_id]
```

```python
# microservices-shop/order-service/main.py
import httpx
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.post("/orders")
async def create_order(user_id: int):
    # 이제 HTTP 요청으로 사용자 정보를 가져와야 함
    async with httpx.AsyncClient() as client:
        try:
            resp = await client.get(
                f"http://localhost:8001/users/{user_id}",
                timeout=5.0  # 타임아웃 설정 필수
            )
        except httpx.ConnectError:
            # user-service가 꺼져 있으면 이 에러 발생
            raise HTTPException(status_code=503, detail="사용자 서비스에 연결할 수 없습니다")
        
        if resp.status_code != 200:
            raise HTTPException(status_code=404, detail="사용자 없음")
        
        user = resp.json()
    
    return {"user": user["name"], "status": "완료"}
```

```bash
# 터미널 1: user-service 실행
cd user-service && uvicorn main:app --port 8001

# 터미널 2: order-service 실행
cd order-service && pip install httpx && uvicorn main:app --port 8003

# 테스트 — user-service가 켜져 있어야만 작동함
curl -X POST "http://localhost:8003/orders?user_id=1"
```

---

### 실습 3: 의사결정 시트 작성하기

아래 템플릿을 `architecture-decision.md` 파일로 저장하고, **여러분의 실제 또는 가상의 프로젝트**에 맞게 채워 보세요.

```markdown
# 아키텍처 의사결정 기록 (ADR)

## 프로젝트명: [여기에 프로젝트 이름]

## 현재 상황
- 팀 규모: [명]
- 예상 월간 사용자: [명]
- 핵심 기능 수: [개]

## 체크리스트
- [ ] 팀원이 5명 이상이고 기능별로 독립 배포가 필요하다
- [ ] 특정 기능만 트래픽이 집중되어 독립적인 스케일링이 필요하다
- [ ] 기능들이 서로 독립적으로 운영 가능하다
- [ ] Docker, Kubernetes 운영 경험이 있다
- [ ] 분산 추적, API 게이트웨이를 설정할 여력이 있다

## 결정
체크된 항목이 3개 미만: **모놀리식으로 시작**
체크된 항목이 4개 이상: **마이크로서비스 고려**

## 최종 선택: [모놀리식 / 마이크로서비스]
## 이유: [한 줄 설명]
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| 마이크로서비스 전환 후 서비스 간 통신 설정 누락 | `httpx.ConnectError: [Errno 111] Connection refused` | 호출하는 서비스의 URL과 포트가 올바른지 확인, `docker-compose.yml`의 `depends_on` 설정 확인 |
| 서비스 간 타임아웃 미설정 | 주문 요청이 30초 이상 걸리다가 멈춤 | `httpx.AsyncClient(timeout=5.0)` 처럼 타임아웃을 명시적으로 설정 |
| 모놀리식인데 불필요하게 DB를 분리 | `sqlalchemy.exc.OperationalError: no such table` | 모놀리식에서는 DB를 분리할 이유가 없음. 하나의 DB와 스키마로 시작 |
| 작은 프로젝트에 마이크로서비스 도입 | 배포 스크립트만 500줄, 기능 개발 못 함 | 팀 3명 이하, MVP 단계에서는 모놀리식 고수 |
| 서비스 분리 기준 없이 잘게 쪼갬 | 함수 하나짜리 서비스 20개 생성 | "이 기능이 독립적으로 배포될 필요가 있나?"를 기준으로 분리 여부 결정 |
| 에러 발생 시 어느 서비스가 문제인지 모름 | 로그가 3개 서비스에 분산, 추적 불가 | 분산 추적 도구(OpenTelemetry) 도입 또는 모놀리식 유지 |

---

## 확인 체크리스트

- [ ] 모놀리식 아키텍처에서 서비스 간 통신이 함수 호출임을 이해한다
- [ ] 마이크로서비스에서 서비스 간 통신이 HTTP(네트워크)임을 이해한다
- [ ] 실습 1의 모놀리식 쇼핑몰 코드를 직접 실행해 보았다
- [ ] 실습 2에서 마이크로서비스 구조로 바꿨을 때 복잡도가 늘어남을 직접 느꼈다
- [ ] `httpx.ConnectError`가 왜 발생하는지 설명할 수 있다
- [ ] 팀 규모, 트래픽, 배포 독립성을 기준으로 아키텍처를 선택할 수 있다
- [ ] 실습 3의 의사결정 시트를 내 프로젝트에 맞게 작성해 보았다
- [ ] 마이크로서비스 도입 시 추가로 필요한 인프라(API 게이트웨이, 분산 추적 등)를 나열할 수 있다

---

## 한 번 더 생각해 보기

1. **"처음부터 마이크로서비스로 만들면 나중에 더 편하지 않을까?"** — 실제로 Amazon, Netflix도 처음엔 모놀리식으로 시작했습니다. 왜 그랬을까요? 처음부터 마이크로서비스로 만드는 것이 항상 유리하지 않은 이유를 실습 2의 경험을 바탕으로 설명해 보세요.

2. **서비스를 어떻게 나눠야 할까요?** — 쇼핑몰에서 "알림 발송 기능"을 별도 서비스로 분리하는 것이 좋을까요, 모놀리식 안에 두는 것이 좋을까요? 어떤 조건이 충족되면 분리가 의미 있을지 생각해 보세요.

3. **모놀리식이 죽으면 전체가 죽는다** — 모놀리식의 단점 중 하나는 하나의 버그가 전체 서비스를 다운시킬 수 있다는 점입니다. 이 문제를 마이크로서비스로 전환하지 않고 모놀리식 안에서 줄일 수 있는 방법이 있을까요?

---

## 다음 장

다음 장에서는 모놀리식에서 마이크로서비스로 **단계적으로 전환하는 실전 전략** — 스트랭글러 패턴(Strangler Fig Pattern)을 다룹니다.