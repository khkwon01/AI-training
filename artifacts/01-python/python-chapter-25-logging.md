## 이 장에서 배우는 것

- `print()` 대신 `logging` 모듈을 써야 하는 이유
- 로그 레벨(DEBUG, INFO, WARNING, ERROR, CRITICAL) 구분 방법
- JSON 형식 로그를 만들어 운영 환경에서 검색하기 좋게 만드는 법
- 파일과 콘솔에 동시에 로그를 남기는 핸들러 설정
- 민감한 정보(비밀번호, 토큰)를 로그에 노출하지 않는 방법
- 로그 설정을 코드가 아닌 환경 변수로 관리하는 방법

---

## 먼저 쉬운 설명

서비스를 배포하고 나서 오류가 생겼을 때 `print()` 문만 가득하면 어떤 일이 일어날까요?

- 언제 오류가 났는지 시간을 알 수 없어요.
- 어떤 함수에서 문제가 생겼는지 추적하기 어려워요.
- 로그가 너무 많으면 중요한 내용을 찾을 수 없어요.
- 운영 서버에서는 보통 터미널을 직접 볼 수 없어요.

**로그**는 서비스가 스스로 쓰는 일기장이에요. 잘 쓰인 일기는 문제가 생겼을 때 "그때 무슨 일이 있었는지" 빠르게 알려줍니다.

```
[나쁜 예]  주문 처리 완료
[좋은 예]  2026-05-18T10:23:45Z INFO order_service.process  order_id=ORD-991 user_id=42 amount=15000 status=completed
```

두 번째 줄만 봐도 "누가, 언제, 무엇을, 어떤 결과로" 알 수 있죠.

---

## 1. `print()` 를 버리고 `logging` 시작하기

`logging` 모듈은 파이썬 기본 내장 모듈이라 따로 설치하지 않아도 됩니다.

```python
# order_service.py

import logging

# 모듈 단위 로거 생성 — 파일 이름이 로그에 자동으로 찍힘
logger = logging.getLogger(__name__)


def process_order(order_id: str, user_id: int, amount: int) -> dict:
    logger.info("주문 처리 시작", extra={"order_id": order_id, "user_id": user_id})

    if amount <= 0:
        logger.warning(
            "유효하지 않은 금액",
            extra={"order_id": order_id, "amount": amount},
        )
        return {"status": "failed", "reason": "invalid_amount"}

    # 실제 처리 로직 생략
    logger.info("주문 처리 완료", extra={"order_id": order_id, "status": "completed"})
    return {"status": "completed"}
```

> **핵심 규칙:** 로거는 항상 `__name__` 으로 만드세요. 그러면 로그에 `order_service` 처럼 어느 파일에서 기록했는지 자동으로 나타납니다.

---

## 2. 로그 레벨 — 중요도에 따라 나눠 쓰기

| 레벨 | 숫자 | 언제 써요? |
|------|------|-----------|
| `DEBUG` | 10 | 개발 중 상세한 변수 값, 흐름 추적 |
| `INFO` | 20 | 정상 동작 확인 (주문 완료, 로그인 성공) |
| `WARNING` | 30 | 문제는 아니지만 주의가 필요한 상황 |
| `ERROR` | 40 | 기능이 실패했지만 서비스는 계속 동작 |
| `CRITICAL` | 50 | 서비스 전체가 멈출 수 있는 심각한 오류 |

```python
# log_levels_demo.py

import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)


def login(username: str, password: str) -> bool:
    logger.debug("로그인 시도 중 (개발 환경에서만 보임)", extra={"username": username})

    if not username or not password:
        logger.warning("빈 자격증명으로 로그인 시도", extra={"username": username})
        return False

    # DB 조회 생략
    success = True

    if success:
        logger.info("로그인 성공", extra={"username": username})
    else:
        logger.error("로그인 실패 — 자격증명 불일치", extra={"username": username})

    return success
```

> **운영 환경 팁:** 운영 서버에서는 레벨을 `INFO` 로 설정해 `DEBUG` 메시지가 쌓이지 않게 합니다.

---

## 3. JSON 형식 로그 — 검색과 분석을 쉽게

운영 환경에서는 Elasticsearch, CloudWatch, Datadog 같은 도구로 로그를 수집합니다. 이 도구들은 JSON 형식 로그를 훨씬 잘 처리해요.

```python
# logging_config.py

import json
import logging
import traceback
from datetime import datetime, timezone


class JsonFormatter(logging.Formatter):
    """모든 로그를 JSON 한 줄로 출력하는 포매터."""

    def format(self, record: logging.LogRecord) -> str:
        log_data: dict = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
        }

        # extra= 로 넘긴 키-값 포함
        for key, value in record.__dict__.items():
            if key not in logging.LogRecord.__dict__ and not key.startswith("_"):
                log_data[key] = value

        # 예외 정보가 있으면 추가
        if record.exc_info:
            log_data["exception"] = traceback.format_exception(*record.exc_info)

        return json.dumps(log_data, ensure_ascii=False)


def setup_logger(name: str, level: str = "INFO") -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(getattr(logging, level.upper(), logging.INFO))

    if not logger.handlers:
        handler = logging.StreamHandler()
        handler.setFormatter(JsonFormatter())
        logger.addHandler(handler)

    return logger
```

```python
# main.py
from logging_config import setup_logger

logger = setup_logger(__name__, level="INFO")
logger.info("서버 시작", extra={"port": 8080, "env": "production"})
```

출력 결과:
```json
{"timestamp": "2026-05-18T10:23:45+00:00", "level": "INFO", "logger": "__main__", "message": "서버 시작", "port": 8080, "env": "production"}
```

---

## 4. 핸들러 — 콘솔과 파일에 동시에 기록하기

```python
# file_logging.py

import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path

LOG_DIR = Path("logs")
LOG_DIR.mkdir(exist_ok=True)  # logs/ 폴더가 없으면 자동 생성


def get_logger(name: str) -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)

    # 콘솔 핸들러 — 터미널에 출력
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)  # 콘솔은 INFO 이상만
    console_handler.setFormatter(
        logging.Formatter("%(asctime)s %(levelname)-8s %(name)s  %(message)s")
    )

    # 파일 핸들러 — 최대 5MB, 최대 3개 파일 보관
    file_handler = RotatingFileHandler(
        LOG_DIR / "app.log",
        maxBytes=5 * 1024 * 1024,  # 5MB
        backupCount=3,
        encoding="utf-8",
    )
    file_handler.setLevel(logging.DEBUG)  # 파일은 DEBUG 이상 모두
    file_handler.setFormatter(
        logging.Formatter("%(asctime)s %(levelname)-8s %(name)s  %(message)s")
    )

    logger.addHandler(console_handler)
    logger.addHandler(file_handler)
    return logger
```

> **`RotatingFileHandler` 를 쓰는 이유:** 로그 파일이 무한정 커지면 디스크가 가득 찹니다. 5MB 가 넘으면 `app.log.1`, `app.log.2` 로 자동 교체해 줍니다.

---

## 5. 민감한 정보 보호 — 절대 로그에 담으면 안 되는 것

```python
# safe_logging.py

import logging
import re

logger = logging.getLogger(__name__)

SENSITIVE_FIELDS = {"password", "token", "secret", "card_number", "cvv"}


def mask_sensitive(data: dict) -> dict:
    """민감한 키의 값을 마스킹해서 반환."""
    return {
        key: "***MASKED***" if key.lower() in SENSITIVE_FIELDS else value
        for key, value in data.items()
    }


def register_user(payload: dict) -> dict:
    # 절대 하면 안 되는 것
    # logger.info(f"회원가입 요청: {payload}")  ← 비밀번호가 그대로 노출됨!

    # 올바른 방법
    safe_payload = mask_sensitive(payload)
    logger.info("회원가입 요청", extra=safe_payload)

    return {"status": "ok"}


# 테스트
register_user({
    "username": "kim_gildong",
    "password": "super_secret_123",  # 로그에 찍히지 않아야 함
    "email": "gildong@example.com",
})
```

출력:
```
INFO  safe_logging  username=kim_gildong password=***MASKED*** email=gildong@example.com
```

---

## 6. 환경 변수로 로그 레벨 관리하기

```python
# config.py

import logging
import os


def configure_logging() -> None:
    """환경 변수 LOG_LEVEL 로 로그 레벨을 결정."""
    level_name = os.getenv("LOG_LEVEL", "INFO").upper()
    level = getattr(logging, level_name, logging.INFO)

    logging.basicConfig(
        level=level,
        format="%(asctime)s %(levelname)-8s %(name)s  %(message)s",
        datefmt="%Y-%m-%dT%H:%M:%S",
    )

    logging.getLogger("urllib3").setLevel(logging.WARNING)  # 라이브러리 노이즈 줄이기
    logging.getLogger("boto3").setLevel(logging.WARNING)
```

터미널에서 레벨 바꾸기:
```bash
# 개발 환경
LOG_LEVEL=DEBUG python main.py

# 운영 환경
LOG_LEVEL=INFO python main.py
```

> 코드를 수정하지 않고 환경 변수만 바꿔서 로그 상세도를 조절할 수 있어요.

---

## 따라 하기 실습

### 실습 1 — `print()` 를 `logging` 으로 바꾸기

`payment_service.py` 파일을 만들고 아래 코드를 붙여넣으세요.

```python
# payment_service.py  (리팩토링 전)

def charge(user_id: int, amount: int) -> bool:
    print(f"결제 시작: 사용자 {user_id}, 금액 {amount}")
    if amount <= 0:
        print("오류: 금액이 0 이하입니다")
        return False
    print("결제 완료")
    return True
```

`print()` 를 모두 `logging` 으로 교체하세요. 완성 기준:
- 파일 상단에 `logger = logging.getLogger(__name__)` 이 있어야 해요.
- `amount <= 0` 은 `logger.warning()` 으로, 나머지는 `logger.info()` 로 바꾸세요.
- `extra={"user_id": user_id, "amount": amount}` 를 추가하세요.

---

### 실습 2 — JSON 포매터 연결하기

실습 1 에서 만든 `payment_service.py` 에 실습 3의 `logging_config.py` 의 `setup_logger` 를 적용하세요.

```python
# payment_service.py  (실습 2)

from logging_config import setup_logger  # 위에서 만든 파일

logger = setup_logger(__name__, level="DEBUG")


def charge(user_id: int, amount: int) -> bool:
    logger.info("결제 시작", extra={"user_id": user_id, "amount": amount})
    if amount <= 0:
        logger.warning("유효하지 않은 금액", extra={"user_id": user_id, "amount": amount})
        return False
    logger.info("결제 완료", extra={"user_id": user_id})
    return True


if __name__ == "__main__":
    charge(42, 15000)
    charge(43, -500)  # 경고 로그가 찍혀야 해요
```

예상 출력:
```json
{"timestamp": "...", "level": "INFO", "message": "결제 시작", "user_id": 42, "amount": 15000}
{"timestamp": "...", "level": "WARNING", "message": "유효하지 않은 금액", "user_id": 43, "amount": -500}
{"timestamp": "...", "level": "INFO", "message": "결제 완료", "user_id": 42}
```

---

### 실습 3 — 파일 핸들러 + 민감 정보 마스킹 통합

`user_service.py` 를 만들어 파일 로깅과 마스킹을 동시에 적용하세요.

```python
# user_service.py

import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path
from safe_logging import mask_sensitive  # 실습 5에서 만든 파일

Path("logs").mkdir(exist_ok=True)

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

handler = RotatingFileHandler("logs/user_service.log", maxBytes=1_000_000, backupCount=2)
handler.setFormatter(logging.Formatter("%(asctime)s %(levelname)s %(message)s"))
logger.addHandler(handler)


def create_user(data: dict) -> dict:
    logger.info("사용자 생성 요청 수신", extra=mask_sensitive(data))
    # ... 생성 로직 생략
    logger.info("사용자 생성 완료", extra={"username": data["username"]})
    return {"status": "created"}


create_user({"username": "hong_gildong", "password": "p@ssw0rd", "email": "hong@test.com"})
```

`logs/user_service.log` 파일을 열어 `password` 가 `***MASKED***` 로 저장됐는지 확인하세요.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|----------------------|-----------|
| `logging.basicConfig()` 를 여러 번 호출 | 로그가 두 번 찍히거나 아예 안 찍힘 | 앱 시작 지점에서 **딱 한 번**만 호출 |
| `logger = logging.getLogger("root")` 사용 | 다른 라이브러리 로그까지 전부 출력됨 | `getLogger(__name__)` 으로 교체 |
| `f-string` 을 로그 메시지에 사용 | `logger.debug(f"값: {big_obj}")` — 레벨이 꺼져도 문자열 변환 실행 | `logger.debug("값: %s", big_obj)` 또는 `extra={}` 사용 |
| `extra` 키가 `LogRecord` 기본 속성과 겹침 | `KeyError: 'message'` 또는 `ValueError` | `message`, `name`, `level` 같은 예약어는 키로 쓰지 않기 |
| 핸들러를 매번 새로 추가 | 로그가 호출 횟수만큼 중복 출력됨 | `if not logger.handlers:` 조건 추가 |
| 비밀번호를 `extra` 에 그대로 전달 | 로그 파일에 평문 비밀번호 노출 | `mask_sensitive()` 함수로 마스킹 후 전달 |
| 파일 핸들러에 `encoding` 미지정 | `UnicodeEncodeError: 'cp949' codec...` | `RotatingFileHandler(..., encoding="utf-8")` |

---

## 확인 체크리스트

- [ ] `print()` 를 모두 `logging` 모듈로 교체했다.
- [ ] 각 파일에 `logger = logging.getLogger(__name__)` 을 선언했다.
- [ ] 상황에 맞는 레벨(DEBUG / INFO / WARNING / ERROR / CRITICAL)을 사용했다.
- [ ] 로그 메시지에 `extra={}` 로 문맥 정보(ID, 금액 등)를 포함했다.
- [ ] JSON 포매터를 적용해 한 줄 JSON 형식으로 출력된다.
- [ ] 비밀번호, 토큰 등 민감한 값이 로그에 찍히지 않는다.
- [ ] 로그 레벨을 코드가 아닌 환경 변수(`LOG_LEVEL`)로 설정한다.
- [ ] `RotatingFileHandler` 로 로그 파일 크기를 제한했다.
- [ ] 핸들러 중복 추가를 방지하는 조건(`if not logger.handlers:`)이 있다.

---

## 한 번 더 생각해 보기

1. **운영 환경에서 `DEBUG` 레벨을 항상 켜두면 어떤 문제가 생길까요?** 성능과 보안 두 가지 관점에서 생각해 보세요.

2. **JSON 형식 로그와 일반 텍스트 로그의 차이는 무엇일까요?** 팀에 Elasticsearch 같은 로그 분석 도구가 있다면 어떤 형식이 더 유리할까요?

3. **서드파티 라이브러리(`requests`, `boto3` 등)도 `logging` 모듈을 씁니다.** 이 라이브러리들의 DEBUG 로그가 우리 앱 로그에 섞이지 않게 하려면 어떻게 하면 좋을까요?

---

## 다음 장

다음 장에서는 로그를 기반으로 **Prometheus + Grafana 메트릭 대시보드**를 구성해 서비스 상태를 실시간으로 모니터링하는 방법을 배웁니다.