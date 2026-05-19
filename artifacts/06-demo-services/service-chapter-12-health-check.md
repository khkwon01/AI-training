## 이 장에서 배우는 것

- 서비스 헬스 체크(health check)가 무엇인지 이해한다
- Python으로 간단한 헬스 체크 엔드포인트를 만든다
- 업타임 모니터링이 왜 필요한지 설명할 수 있다
- `requests` 라이브러리로 외부 서비스 상태를 자동으로 확인한다
- 헬스 체크 결과를 로그 파일에 기록한다
- 서비스가 죽었을 때 알림을 보내는 기본 구조를 만든다

---

## 먼저 쉬운 설명

여러분이 운영하는 웹 서비스가 새벽 3시에 갑자기 죽었다고 상상해 보세요. 사용자들은 에러 페이지를 보고 있지만, 개발자인 여러분은 자고 있어서 아무것도 모릅니다. 아침에 출근해서야 문제를 알게 되면 이미 수십 명, 수백 명의 사용자가 불편을 겪은 후입니다.

**헬스 체크(Health Check)** 는 "우리 서비스가 지금 살아있나요?"를 자동으로 확인하는 기능입니다. 마치 심장박동 모니터처럼, 일정한 간격으로 서비스의 상태를 확인하고 문제가 생기면 즉시 알려줍니다.

**업타임 모니터링(Uptime Monitoring)** 은 서비스가 얼마나 오래, 안정적으로 살아있었는지를 기록하는 일입니다. "우리 서비스는 이번 달에 99.9% 정상 운영되었습니다"라고 말할 수 있으려면 이 데이터가 필요합니다.

이 장을 마치면 여러분의 서비스에 자동 건강 검진 시스템을 붙일 수 있습니다.

---

## 1. 헬스 체크 엔드포인트 만들기

헬스 체크의 가장 기본은 `/health` 라는 URL에 접근했을 때 서비스 상태를 JSON으로 돌려주는 것입니다.

**`health_server.py`**

```python
from fastapi import FastAPI
import time

app = FastAPI()

# 서버가 시작된 시각을 기록합니다
START_TIME = time.time()


@app.get("/health")
def health_check():
    """서비스가 살아있는지 확인하는 엔드포인트"""
    uptime_seconds = int(time.time() - START_TIME)

    return {
        "status": "ok",
        "uptime_seconds": uptime_seconds,
        "message": "서비스가 정상적으로 작동 중입니다"
    }


@app.get("/")
def root():
    return {"message": "안녕하세요! 메인 서비스입니다"}
```

서버를 실행하면:

```bash
uvicorn health_server:app --reload
```

브라우저에서 `http://localhost:8000/health` 를 열면 다음과 같은 응답을 볼 수 있습니다:

```json
{
  "status": "ok",
  "uptime_seconds": 42,
  "message": "서비스가 정상적으로 작동 중입니다"
}
```

> **핵심 규칙**: HTTP 상태 코드 `200`은 "정상", `500`은 "서버 오류"입니다. 모니터링 도구는 이 숫자를 보고 서비스 상태를 판단합니다.

---

## 2. 데이터베이스 연결 상태도 함께 확인하기

단순히 서버가 켜져 있는지만 확인하는 것은 부족합니다. 데이터베이스에 연결이 안 된다면 서비스는 실질적으로 죽은 것과 같습니다. 더 똑똑한 헬스 체크를 만들어 봅시다.

**`health_server_v2.py`**

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse
import time
import sqlite3
import os

app = FastAPI()
START_TIME = time.time()
DB_PATH = "app_database.db"


def check_database() -> dict:
    """데이터베이스에 실제로 연결되는지 확인합니다"""
    try:
        conn = sqlite3.connect(DB_PATH)
        conn.execute("SELECT 1")  # 가장 간단한 쿼리로 연결 확인
        conn.close()
        return {"status": "ok", "message": "DB 연결 정상"}
    except Exception as e:
        return {"status": "error", "message": f"DB 연결 실패: {str(e)}"}


def check_disk_space() -> dict:
    """디스크 여유 공간을 확인합니다"""
    stat = os.statvfs(".")
    free_gb = (stat.f_bavail * stat.f_frsize) / (1024 ** 3)

    if free_gb < 1.0:
        return {"status": "warning", "free_gb": round(free_gb, 2), "message": "디스크 공간 부족 경고"}
    return {"status": "ok", "free_gb": round(free_gb, 2), "message": "디스크 공간 충분"}


@app.get("/health")
def health_check():
    db_status = check_database()
    disk_status = check_disk_space()

    # 하나라도 오류이면 전체 상태를 오류로 표시합니다
    overall_status = "ok"
    if db_status["status"] == "error" or disk_status["status"] == "error":
        overall_status = "error"
    elif db_status["status"] == "warning" or disk_status["status"] == "warning":
        overall_status = "warning"

    response_body = {
        "status": overall_status,
        "uptime_seconds": int(time.time() - START_TIME),
        "checks": {
            "database": db_status,
            "disk": disk_status
        }
    }

    # 오류 상태일 때는 HTTP 500을 반환합니다
    if overall_status == "error":
        return JSONResponse(status_code=500, content=response_body)

    return response_body
```

응답 예시 (정상):

```json
{
  "status": "ok",
  "uptime_seconds": 120,
  "checks": {
    "database": {"status": "ok", "message": "DB 연결 정상"},
    "disk": {"status": "ok", "free_gb": 15.3, "message": "디스크 공간 충분"}
  }
}
```

---

## 3. 외부 서비스를 주기적으로 감시하는 모니터링 스크립트 만들기

이번에는 반대 입장에서 생각해 봅시다. 내 서비스가 다른 서비스의 헬스 체크 URL을 정기적으로 찌르면서 "살아있니?"를 확인하는 모니터입니다.

**`uptime_monitor.py`**

```python
import requests
import time
import logging
from datetime import datetime

# 로그 설정: 파일과 터미널에 동시에 기록합니다
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler("uptime_log.txt", encoding="utf-8"),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

# 감시할 서비스 목록
SERVICES = [
    {"name": "메인 서비스",    "url": "http://localhost:8000/health"},
    {"name": "결제 서비스",    "url": "http://localhost:8001/health"},
]

INTERVAL_SECONDS = 30   # 30초마다 확인
TIMEOUT_SECONDS = 5     # 5초 안에 응답 없으면 타임아웃


def check_service(service: dict) -> bool:
    """
    서비스 URL에 GET 요청을 보내고 결과를 기록합니다.
    정상이면 True, 문제이면 False를 반환합니다.
    """
    name = service["name"]
    url = service["url"]

    try:
        response = requests.get(url, timeout=TIMEOUT_SECONDS)

        if response.status_code == 200:
            logger.info(f"[정상] {name} | 응답시간: {response.elapsed.total_seconds():.3f}초")
            return True
        else:
            logger.error(f"[오류] {name} | HTTP {response.status_code}")
            return False

    except requests.exceptions.ConnectionError:
        logger.error(f"[오류] {name} | 서버에 연결할 수 없습니다 (ConnectionError)")
        return False

    except requests.exceptions.Timeout:
        logger.error(f"[오류] {name} | 응답 시간 초과 ({TIMEOUT_SECONDS}초)")
        return False

    except Exception as e:
        logger.error(f"[오류] {name} | 예상치 못한 오류: {e}")
        return False


def send_alert(service_name: str, message: str):
    """실제 프로덕션에서는 Slack, 이메일 등으로 알림을 보냅니다"""
    logger.warning(f"🚨 알림 발송: [{service_name}] {message}")
    # 여기에 Slack webhook, 이메일 발송 코드를 추가합니다


def run_monitor():
    """무한 루프로 모든 서비스를 주기적으로 확인합니다"""
    logger.info("업타임 모니터링 시작")

    # 각 서비스의 이전 상태를 기억합니다 (처음엔 정상으로 가정)
    previous_status = {s["name"]: True for s in SERVICES}

    while True:
        for service in SERVICES:
            is_healthy = check_service(service)
            name = service["name"]

            # 상태가 변경되었을 때만 알림을 보냅니다
            if not is_healthy and previous_status[name]:
                send_alert(name, "서비스가 응답하지 않습니다!")
            elif is_healthy and not previous_status[name]:
                send_alert(name, "서비스가 복구되었습니다.")

            previous_status[name] = is_healthy

        time.sleep(INTERVAL_SECONDS)


if __name__ == "__main__":
    run_monitor()
```

실행하면 터미널에 이런 로그가 나타납니다:

```
2026-05-18 14:30:00,123 [INFO] 업타임 모니터링 시작
2026-05-18 14:30:00,456 [INFO] [정상] 메인 서비스 | 응답시간: 0.023초
2026-05-18 14:30:00,789 [ERROR] [오류] 결제 서비스 | 서버에 연결할 수 없습니다 (ConnectionError)
2026-05-18 14:30:00,790 [WARNING] 🚨 알림 발송: [결제 서비스] 서비스가 응답하지 않습니다!
```

---

## 4. 업타임 통계 계산하기

모니터링 로그가 쌓이면 "이번 주 가동률이 몇 %였는지"를 계산할 수 있어야 합니다.

**`uptime_stats.py`**

```python
import re
from datetime import datetime
from collections import defaultdict


def parse_log_file(log_path: str) -> dict:
    """
    uptime_log.txt 를 읽어서 서비스별 정상/오류 횟수를 셉니다.
    반환 예: {"메인 서비스": {"ok": 100, "error": 2}}
    """
    stats = defaultdict(lambda: {"ok": 0, "error": 0})

    # 로그 형식: "2026-05-18 14:30:00,456 [INFO] [정상] 메인 서비스 | ..."
    ok_pattern = re.compile(r"\[정상\] (.+?) \|")
    error_pattern = re.compile(r"\[오류\] (.+?) \|")

    try:
        with open(log_path, "r", encoding="utf-8") as f:
            for line in f:
                ok_match = ok_pattern.search(line)
                error_match = error_pattern.search(line)

                if ok_match:
                    stats[ok_match.group(1)]["ok"] += 1
                elif error_match:
                    stats[error_match.group(1)]["error"] += 1

    except FileNotFoundError:
        print(f"로그 파일을 찾을 수 없습니다: {log_path}")
        return {}

    return dict(stats)


def calculate_uptime_percentage(stats: dict) -> None:
    """서비스별 업타임 퍼센트를 출력합니다"""
    print("\n📊 업타임 리포트")
    print("-" * 40)

    for service_name, counts in stats.items():
        total = counts["ok"] + counts["error"]
        if total == 0:
            continue

        uptime_pct = (counts["ok"] / total) * 100
        print(f"{service_name}")
        print(f"  총 확인 횟수: {total}회")
        print(f"  정상: {counts['ok']}회 / 오류: {counts['error']}회")
        print(f"  업타임: {uptime_pct:.2f}%")
        print()


if __name__ == "__main__":
    stats = parse_log_file("uptime_log.txt")
    calculate_uptime_percentage(stats)
```

출력 예시:

```
📊 업타임 리포트
----------------------------------------
메인 서비스
  총 확인 횟수: 102회
  정상: 100회 / 오류: 2회
  업타임: 98.04%

결제 서비스
  총 확인 횟수: 102회
  정상: 95회 / 오류: 7회
  업타임: 93.14%
```

---

## 따라 하기 실습

### 실습 1 — 기본 헬스 체크 서버 실행하기

1. 새 폴더를 만들고 필요한 패키지를 설치합니다.

```bash
mkdir health-check-practice
cd health-check-practice
pip install fastapi uvicorn requests
```

2. 위의 `health_server.py` 코드를 그대로 파일로 저장합니다.

3. 서버를 실행합니다.

```bash
uvicorn health_server:app --reload
```

4. 새 터미널 창을 열고 헬스 체크를 직접 호출해 봅니다.

```bash
curl http://localhost:8000/health
```

**확인 목표**: `"status": "ok"` 응답이 오면 성공입니다.

---

### 실습 2 — 데이터베이스 연결 포함 헬스 체크로 업그레이드하기

1. 실습 1의 서버를 종료하고, `health_server_v2.py` 파일을 만듭니다.

2. SQLite 데이터베이스 파일을 직접 생성합니다.

```bash
python -c "import sqlite3; sqlite3.connect('app_database.db').close(); print('DB 생성 완료')"
```

3. `health_server_v2.py` 로 서버를 다시 실행합니다.

```bash
uvicorn health_server_v2:app --reload
```

4. 헬스 체크를 호출해 `checks.database.status` 가 `"ok"` 인지 확인합니다.

```bash
curl http://localhost:8000/health
```

5. 이번에는 DB 파일을 삭제하고 다시 호출해 봅니다. HTTP 500 응답이 오는지 확인하세요.

```bash
rm app_database.db
curl -i http://localhost:8000/health
```

**확인 목표**: DB가 없을 때 `"status": "error"` 와 함께 HTTP 500이 오면 성공입니다.

---

### 실습 3 — 업타임 모니터 실행 후 통계 출력하기

1. 실습 2의 서버를 그대로 켜 둔 상태에서, 새 터미널에서 `uptime_monitor.py` 를 실행합니다. (`INTERVAL_SECONDS` 를 `5` 로 줄이면 빠르게 테스트할 수 있습니다.)

```bash
python uptime_monitor.py
```

2. 약 1분 동안 로그가 쌓이는 것을 지켜봅니다. 중간에 서버를 껐다가 켜서 오류 로그도 만들어 봅니다 (Ctrl+C 로 서버 중단).

3. 로그가 충분히 쌓이면 모니터를 Ctrl+C 로 종료하고 통계를 계산합니다.

```bash
python uptime_stats.py
```

**확인 목표**: 업타임 퍼센트가 출력되면 성공입니다. 서버를 껐다 켰다 했다면 100% 미만의 숫자를 볼 수 있습니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|---|---|---|
| FastAPI 없이 실행 | `ModuleNotFoundError: No module named 'fastapi'` | `pip install fastapi uvicorn` 실행 |
| 포트가 이미 사용 중 | `ERROR: [Errno 48] Address already in use` | `lsof -i :8000` 으로 사용 중인 프로세스 확인 후 종료, 또는 `--port 8001` 로 다른 포트 사용 |
| `curl` 연결 거부 | `curl: (7) Failed to connect to localhost port 8000` | 서버가 실행 중인지 확인. 다른 터미널에서 uvicorn이 켜져 있어야 함 |
| DB 경로 오류 | `OperationalError: unable to open database file` | DB 파일 경로가 스크립트와 같은 폴더인지 확인. `os.getcwd()` 로 현재 경로 출력 |
| 타임아웃 설정 누락 | 응답 없이 영원히 대기 | `requests.get(url, timeout=5)` 처럼 항상 `timeout` 값을 명시 |
| 상태 변경 감지 안 됨 | 알림이 계속 오거나 아예 안 옴 | `previous_status` 딕셔너리를 루프 밖에 선언했는지 확인. 루프 안에 있으면 매번 초기화됨 |
| 로그 파일 인코딩 오류 | `UnicodeDecodeError: 'ascii' codec can't decode` | `open(log_path, encoding="utf-8")` 처럼 `encoding="utf-8"` 명시 |

---

## 확인 체크리스트

- [ ] `/health` 엔드포인트에 GET 요청을 보내면 JSON 응답이 온다
- [ ] 정상 상태일 때 HTTP 상태 코드 `200` 이 반환된다
- [ ] 데이터베이스 연결 실패 시 HTTP 상태 코드 `500` 이 반환된다
- [ ] `uptime_monitor.py` 가 30초(또는 설정한 간격)마다 서비스를 확인한다
- [ ] 서비스 상태가 정상 → 오류로 바뀔 때만 알림 함수가 호출된다 (매번 호출되지 않는다)
- [ ] `uptime_log.txt` 파일에 정상/오류 로그가 기록된다
- [ ] `uptime_stats.py` 로 업타임 퍼센트를 계산할 수 있다
- [ ] `requests.get()` 에 항상 `timeout` 파라미터를 설정했다
- [ ] 서버를 껐을 때 `ConnectionError` 를 코드에서 잡아서 처리한다

---

## 한 번 더 생각해 보기

1. **알림을 언제 보내야 할까요?** 서비스가 오류 응답을 딱 1번 보냈을 때 바로 알림을 보내는 것이 좋을까요, 아니면 연속 3번 오류가 난 후에 보내는 것이 좋을까요? 각각 어떤 장단점이 있는지 생각해 보세요.

2. **헬스 체크 URL 자체가 느려진다면?** 서비스가 응답은 하지만 평소 20ms 걸리던 것이 갑자기 2000ms (2초)씩 걸린다면, 이것을 "정상"으로 봐야 할까요? 응답 시간도 모니터링에 포함하려면 코드를 어떻게 바꾸면 될까요?

3. **모니터링 스크립트 자체가 죽으면?** `uptime_monitor.py` 가 실행 중에 예기치 않게 종료되어 버린다면, 우리는 어떻게 알 수 있을까요? 모니터를 감시하는 또 다른 모니터가 필요한 걸까요? 이 문제를 해결하는 실용적인 방법은 무엇이 있을지 생각해 보세요.

---

## 다음 장

다음 장에서는 이번에 만든 헬스 체크와 모니터링을 AWS CloudWatch 및 Slack Webhook과 연결하여, 서비스 장애 시 실시간으로 팀에 알림을 보내는 프로덕션 수준의 알림 파이프라인을 구축합니다.