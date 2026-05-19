## 이 장에서 배우는 것

- Python에서 날짜와 시간을 다루는 `datetime` 모듈 사용법
- 날짜와 시간을 원하는 형식의 문자열로 바꾸는 방법 (`strftime`)
- 문자열을 날짜/시간 객체로 변환하는 방법 (`strptime`)
- 시간대(타임존)가 무엇인지, 왜 중요한지 이해하기
- `pytz`와 `zoneinfo`를 이용해 한국 시간(KST)과 세계 표준시(UTC)를 변환하기
- 두 날짜/시간 사이의 차이를 계산하는 방법 (`timedelta`)

---

## 먼저 쉬운 설명

우리가 매일 쓰는 앱은 거의 모두 시간을 다룹니다. 주문 내역에 "2025-05-18 오후 3시 22분"이 찍히고, 채팅 메시지에 "방금 전"이라고 뜨고, 항공권 예약 화면에는 "서울 출발 09:00 / 뉴욕 도착 11:30(현지 시각)"이라고 표시됩니다.

이 모든 것이 날짜·시간 처리 코드로 만들어집니다.

처음에는 쉬워 보이지만, **타임존(시간대)** 문제를 만나면 많은 개발자가 고생합니다. 서울은 UTC+9, 뉴욕은 UTC-5라서 같은 순간이라도 두 도시의 시각이 다릅니다. 타임존을 무시하고 코드를 짜면 예약 시간이 14시간 어긋나는 버그가 생기기도 합니다.

이 장에서는 "왜 이렇게 하는지"를 이해하면서 Python datetime을 배웁니다.

---

## 1. `datetime` 모듈 기본 — 지금 시각 가져오기

Python 표준 라이브러리에 포함된 `datetime` 모듈로 현재 날짜와 시간을 가져올 수 있습니다.

```python
# 파일명: 01_datetime_basic.py
from datetime import datetime, date, time

# 현재 날짜와 시간 (타임존 정보 없음 — "naive datetime"이라고 부릅니다)
지금 = datetime.now()
print("현재 날짜+시간:", 지금)
# 예: 2025-05-18 15:22:34.123456

# 날짜만
오늘 = date.today()
print("오늘 날짜:", 오늘)
# 예: 2025-05-18

# 직접 날짜·시간 객체 만들기
생일 = datetime(1995, 8, 15, 9, 30, 0)
print("생일:", 생일)
# 예: 1995-08-15 09:30:00

# 각 속성에 접근하기
print("연도:", 지금.year)
print("월:", 지금.month)
print("일:", 지금.day)
print("시:", 지금.hour)
print("분:", 지금.minute)
print("초:", 지금.second)
```

> **핵심 개념**: `datetime.now()`는 컴퓨터의 로컬 시각을 가져옵니다. 타임존 정보가 없는 이 상태를 **naive datetime**이라고 부릅니다.

---

## 2. 날짜를 문자열로, 문자열을 날짜로 — `strftime` / `strptime`

데이터베이스나 API에서 날짜를 주고받을 때는 문자열로 변환해야 합니다.

```python
# 파일명: 02_format_parse.py
from datetime import datetime

지금 = datetime(2025, 5, 18, 15, 22, 34)

# ── strftime: datetime → 문자열 ──────────────────────────────
# %Y: 4자리 연도, %m: 2자리 월, %d: 2자리 일
# %H: 24시간 시, %M: 분, %S: 초
한국식 = 지금.strftime("%Y년 %m월 %d일 %H시 %M분")
print(한국식)
# 출력: 2025년 05월 18일 15시 22분

iso형식 = 지금.strftime("%Y-%m-%dT%H:%M:%S")
print(iso형식)
# 출력: 2025-05-18T15:22:34

# ── strptime: 문자열 → datetime ──────────────────────────────
# 두 번째 인자의 형식이 문자열과 정확히 일치해야 합니다
문자열 = "2025-05-18 15:22:34"
변환됨 = datetime.strptime(문자열, "%Y-%m-%d %H:%M:%S")
print(type(변환됨))  # <class 'datetime.datetime'>
print(변환됨.year)   # 2025

# 한국식 날짜 문자열 파싱
한국_문자열 = "2025년 05월 18일"
한국_날짜 = datetime.strptime(한국_문자열, "%Y년 %m월 %d일")
print(한국_날짜)
# 출력: 2025-05-18 00:00:00
```

**자주 쓰는 형식 코드 정리:**

| 코드 | 의미 | 예시 |
|------|------|------|
| `%Y` | 4자리 연도 | `2025` |
| `%m` | 2자리 월 | `05` |
| `%d` | 2자리 일 | `18` |
| `%H` | 24시간 형식 시 | `15` |
| `%I` | 12시간 형식 시 | `03` |
| `%M` | 분 | `22` |
| `%S` | 초 | `34` |
| `%p` | AM/PM | `PM` |

---

## 3. 날짜 계산 — `timedelta`

두 날짜 사이의 차이를 계산하거나, 날짜에 며칠을 더하거나 빼고 싶을 때 `timedelta`를 사용합니다.

```python
# 파일명: 03_timedelta.py
from datetime import datetime, timedelta

오늘 = datetime(2025, 5, 18)

# 날짜 더하기·빼기
일주일_후 = 오늘 + timedelta(days=7)
print("일주일 후:", 일주일_후)
# 출력: 2025-05-25 00:00:00

어제 = 오늘 - timedelta(days=1)
print("어제:", 어제)
# 출력: 2025-05-17 00:00:00

# 시간 단위로도 가능
두시간_후 = 오늘 + timedelta(hours=2, minutes=30)
print("2시간 30분 후:", 두시간_후)

# ── 두 날짜의 차이 구하기 ──────────────────────────────────────
입사일 = datetime(2023, 3, 2)
오늘_날짜 = datetime(2025, 5, 18)
근속_기간 = 오늘_날짜 - 입사일

print(f"근속 일수: {근속_기간.days}일")
# 출력: 근속 일수: 807일

# D-day 계산
시험일 = datetime(2025, 11, 14)
남은_일수 = (시험일 - 오늘_날짜).days
print(f"시험까지 D-{남은_일수}")
# 출력: 시험까지 D-180
```

---

## 4. 타임존(시간대) 이해하기

**타임존**이란 지구상의 위치에 따라 다르게 적용되는 시간 기준입니다.

- **UTC (협정 세계시)**: 기준이 되는 시간대, UTC+0
- **KST (한국 표준시)**: UTC보다 9시간 빠름, UTC+9
- **PST (미국 태평양 표준시)**: UTC보다 8시간 느림, UTC-8

```
UTC  10:00
KST  19:00  (UTC + 9시간)
PST  02:00  (UTC - 8시간)  ← 전날 새벽!
```

Python에서 타임존 정보가 붙은 datetime을 **aware datetime**, 없는 것을 **naive datetime**이라고 부릅니다. 이 둘을 섞어서 비교하면 오류가 납니다.

---

## 5. `zoneinfo`로 타임존 다루기 (Python 3.9+)

Python 3.9부터는 표준 라이브러리의 `zoneinfo`를 사용할 수 있습니다.

```python
# 파일명: 05_zoneinfo.py
from datetime import datetime
from zoneinfo import ZoneInfo

# ── KST 기준으로 현재 시각 가져오기 ──────────────────────────
kst = ZoneInfo("Asia/Seoul")
지금_kst = datetime.now(tz=kst)
print("한국 시각:", 지금_kst)
# 예: 2025-05-18 15:22:34+09:00

# ── UTC 기준으로 현재 시각 가져오기 ──────────────────────────
utc = ZoneInfo("UTC")
지금_utc = datetime.now(tz=utc)
print("UTC 시각:", 지금_utc)
# 예: 2025-05-18 06:22:34+00:00

# ── KST → UTC 변환 ────────────────────────────────────────────
kst_시각 = datetime(2025, 5, 18, 15, 0, 0, tzinfo=kst)
utc_변환 = kst_시각.astimezone(ZoneInfo("UTC"))
print("KST:", kst_시각)
print("UTC:", utc_변환)
# KST: 2025-05-18 15:00:00+09:00
# UTC: 2025-05-18 06:00:00+00:00

# ── UTC → KST 변환 ────────────────────────────────────────────
utc_시각 = datetime(2025, 5, 18, 6, 0, 0, tzinfo=ZoneInfo("UTC"))
kst_변환 = utc_시각.astimezone(kst)
print("UTC:", utc_시각)
print("KST:", kst_변환)
# UTC: 2025-05-18 06:00:00+00:00
# KST: 2025-05-18 15:00:00+09:00
```

> **팁**: Python 3.8 이하이거나 외부 라이브러리를 써도 된다면 `pytz`를 사용합니다. `pip install pytz`로 설치 후 `import pytz`로 불러옵니다.

---

## 6. `pytz`로 타임존 다루기 (Python 3.8 이하 또는 레거시 코드)

```python
# 파일명: 06_pytz.py
# 설치: pip install pytz
import pytz
from datetime import datetime

kst = pytz.timezone("Asia/Seoul")
utc = pytz.utc

# ── naive datetime에 타임존 붙이기 ───────────────────────────
# 주의: astimezone() 이 아닌 localize()를 써야 합니다!
naive_dt = datetime(2025, 5, 18, 15, 0, 0)
kst_dt = kst.localize(naive_dt)   # ✅ 올바른 방법
print(kst_dt)
# 2025-05-18 15:00:00+09:00

# ── 타임존 변환 ───────────────────────────────────────────────
utc_dt = kst_dt.astimezone(utc)
print("UTC:", utc_dt)
# UTC: 2025-05-18 06:00:00+00:00

# ── 자주 쓰는 타임존 이름 ─────────────────────────────────────
# "Asia/Seoul"       → 한국 (KST, UTC+9)
# "America/New_York" → 미국 동부 (EST/EDT)
# "Europe/London"    → 영국 (GMT/BST)
# "Asia/Tokyo"       → 일본 (JST, UTC+9, 한국과 동일)
```

---

## 7. ISO 8601 형식과 `isoformat()`

API 통신에서는 날짜를 ISO 8601 형식으로 주고받는 것이 표준입니다.

```python
# 파일명: 07_isoformat.py
from datetime import datetime
from zoneinfo import ZoneInfo

kst = ZoneInfo("Asia/Seoul")
지금 = datetime(2025, 5, 18, 15, 22, 34, tzinfo=kst)

# datetime → ISO 8601 문자열
print(지금.isoformat())
# 출력: 2025-05-18T15:22:34+09:00

# ISO 8601 문자열 → datetime (Python 3.7+)
iso_문자열 = "2025-05-18T15:22:34+09:00"
파싱됨 = datetime.fromisoformat(iso_문자열)
print(파싱됨)
# 출력: 2025-05-18 15:22:34+09:00

print(파싱됨.tzinfo)
# 출력: UTC+09:00

# UTC 기준 ISO 형식 (Z는 UTC를 나타냄 — API에서 자주 봄)
# Python 3.11+에서는 "Z" 접미사를 직접 파싱 가능
utc_문자열 = "2025-05-18T06:22:34Z"
# Python 3.11 미만에서는 Z를 +00:00으로 교체해서 파싱
파싱됨2 = datetime.fromisoformat(utc_문자열.replace("Z", "+00:00"))
print(파싱됨2)
```

---

## 따라 하기 실습

### 실습 1 — 나의 생일까지 D-day 계산기 만들기

```python
# 파일명: practice_01_dday.py
from datetime import datetime

def dday_계산(목표일_문자열: str) -> int:
    """
    'YYYY-MM-DD' 형식의 문자열을 받아 오늘부터 며칠 남았는지 반환합니다.
    이미 지난 날짜면 음수를 반환합니다.
    """
    오늘 = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
    목표일 = datetime.strptime(목표일_문자열, "%Y-%m-%d")
    남은_일수 = (목표일 - 오늘).days
    return 남은_일수

# 직접 날짜를 바꿔서 테스트해 보세요
내_생일 = "2025-08-15"
결과 = dday_계산(내_생일)

if 결과 > 0:
    print(f"생일까지 D-{결과}")
elif 결과 == 0:
    print("오늘이 생일입니다!")
else:
    print(f"생일이 {abs(결과)}일 지났습니다.")
```

**확인 포인트**: `replace(hour=0, ...)` 를 쓰는 이유가 무엇인지 생각해 보세요. (힌트: 시·분·초까지 포함하면 "오늘"의 D-day가 0이 아닐 수 있습니다.)

---

### 실습 2 — 세계 시각 변환기 만들기

실습 1에서 날짜 계산에 익숙해졌으니, 이번에는 타임존을 추가합니다.

```python
# 파일명: practice_02_world_clock.py
from datetime import datetime
from zoneinfo import ZoneInfo

도시_타임존 = {
    "서울":    "Asia/Seoul",
    "도쿄":    "Asia/Tokyo",
    "런던":    "Europe/London",
    "뉴욕":    "America/New_York",
    "로스앤젤레스": "America/Los_Angeles",
}

def 세계_시각_출력(기준_도시: str = "서울"):
    tz = ZoneInfo(도시_타임존[기준_도시])
    기준_시각 = datetime.now(tz=tz)

    print(f"\n[ 기준: {기준_도시} {기준_시각.strftime('%Y-%m-%d %H:%M:%S %Z')} ]")
    print("-" * 45)

    for 도시, 타임존_이름 in 도시_타임존.items():
        변환_시각 = 기준_시각.astimezone(ZoneInfo(타임존_이름))
        print(f"  {도시:<12} {변환_시각.strftime('%Y-%m-%d %H:%M:%S')}")

세계_시각_출력("서울")
```

**예상 출력:**
```
[ 기준: 서울 2025-05-18 15:22:34 KST ]
---------------------------------------------
  서울         2025-05-18 15:22:34
  도쿄         2025-05-18 15:22:34
  런던         2025-05-18 07:22:34
  뉴욕         2025-05-18 02:22:34
  로스앤젤레스  2025-05-17 23:22:34
```

---

### 실습 3 — 주문 시각 기록 시스템 만들기

앞선 두 실습을 합쳐서 실무에 가까운 코드를 작성합니다.

```python
# 파일명: practice_03_order_system.py
from datetime import datetime
from zoneinfo import ZoneInfo

KST = ZoneInfo("Asia/Seoul")
UTC = ZoneInfo("UTC")

def 주문_생성(상품명: str) -> dict:
    """주문 시각을 UTC로 저장하고 KST로 표시합니다."""
    utc_시각 = datetime.now(tz=UTC)  # DB 저장용: 항상 UTC

    return {
        "상품명": 상품명,
        "주문_시각_utc": utc_시각.isoformat(),        # DB에 저장하는 값
        "주문_시각_kst": utc_시각.astimezone(KST).strftime("%Y-%m-%d %H:%M:%S KST"),  # 화면 표시용
    }

def 주문_이력_출력(주문_목록: list[dict]):
    print(f"\n{'상품명':<15} {'주문 시각 (한국 시간)'}")
    print("-" * 40)
    for 주문 in 주문_목록:
        print(f"{주문['상품명']:<15} {주문['주문_시각_kst']}")

# 테스트
주문들 = [
    주문_생성("아메리카노"),
    주문_생성("크로와상"),
    주문_생성("치즈케이크"),
]

주문_이력_출력(주문들)
print("\n[DB 저장 형태 (UTC ISO 8601)]")
for 주문 in 주문들:
    print(f"  {주문['상품명']}: {주문['주문_시각_utc']}")
```

> **실무 포인트**: 데이터베이스에는 **항상 UTC로 저장**하고, 사용자에게 보여줄 때 현지 시간으로 변환하는 패턴이 표준입니다.

---

## 자주 하는 실수

| 실수 | 발생하는 오류 메시지 | 해결 방법 |
|------|------------------|-----------|
| naive datetime과 aware datetime을 비교 | `TypeError: can't compare offset-naive and offset-aware datetimes` | 두 datetime 모두 타임존을 붙이거나 둘 다 타임존을 제거하세요 |
| `strptime` 형식 문자열이 실제 문자열과 다름 | `ValueError: time data '2025/05/18' does not match format '%Y-%m-%d'` | 구분자(`-`, `/`, `.`)와 형식 코드를 문자열에 맞게 수정하세요 |
| `pytz`에서 `replace()` 로 타임존 붙이기 | 썸머타임(DST) 처리가 잘못될 수 있음, 오류 없이 틀린 값 반환 | `pytz`에서는 반드시 `localize()`를 사용하세요 |
| 날짜 문자열에 시간 부분 없이 `%H:%M:%S` 형식 지정 | `ValueError: time data '2025-05-18' does not match format '%Y-%m-%d %H:%M:%S'` | 형식 문자열에서 시간 부분을 제거하거나, 날짜 문자열에 시간을 추가하세요 |
| `timedelta`로 월·연도 단위 계산 시도 | `TypeError: unsupported type for timedelta` | `timedelta`는 일(days)/초(seconds)/마이크로초만 지원합니다. 월·연도 계산은 `dateutil.relativedelta`를 쓰세요 |
| `datetime.now()` 와 `datetime.utcnow()` 를 혼용 | 오류 없이 9시간 차이 발생 | 타임존을 명시하는 `datetime.now(tz=UTC)` 만 사용하세요. `utcnow()`는 naive datetime을 반환하므로 피하는 것이 좋습니다 |

---

## 확인 체크리스트

- [ ] `datetime.now()` 와 `datetime.now(tz=...)` 의 차이를 설명할 수 있다
- [ ] `strftime("%Y-%m-%d %H:%M:%S")` 로 현재 시각을 원하는 형식의 문자열로 출력할 수 있다
- [ ] `strptime("2025-05-18", "%Y-%m-%d")` 로 날짜 문자열을 `datetime` 객체로 변환할 수 있다
- [ ] `timedelta(days=7)` 를 더하거나 빼서 날짜를 계산할 수 있다
- [ ] 두 `datetime` 객체를 빼서 `.days` 로 일수 차이를 구할 수 있다
- [ ] `ZoneInfo("Asia/Seoul")` 로 한국 표준시(KST) 타임존을 만들 수 있다
- [ ] `.astimezone()` 으로 KST ↔ UTC 변환을 할 수 있다
- [ ] DB 저장은 UTC, 사용자 표시는 KST로 하는 이유를 설명할 수 있다
- [ ] `naive datetime` 과 `aware datetime` 이 무엇인지 구분할 수 있다
- [ ] `isoformat()` 과 `fromisoformat()` 으로 ISO 8601 형식을 다룰 수 있다

---

## 한 번 더 생각해 보기

1. **서울 사무소와 뉴욕 사무소를 연결하는 회의 예약 시스템**을 만든다고 가정합니다. 사용자가 "서울 시간 오전 10시"에 회의를 잡으면 뉴욕 참여자에게는 어떤 시각으로 알림을 보내야 할까요? 코드로 어떻게 구현할지 흐름을 적어보세요.

2. 아래 코드를 실행하면 어떤 문제가 생길 수 있을까요? 그리고 어떻게 고쳐야 할까요?
   ```python
   from datetime import datetime
   import pytz

   kst = pytz.timezone("Asia/Seoul")
   dt = datetime(2025, 5, 18, 15, 0, 0).replace(tzinfo=kst)  # ← 이 줄이 문제입니다
   print(dt)
   ```

3. 쇼핑몰 주문 시스템에서 "오늘 자정까지 주문하면 내일 배송"이라는 기능을 구현하려 합니다. 자정을 어떤 타임존 기준으로 정의해야 할지, 그리고 해외 고객이 접속할 경우 어떻게 처리해야 할지 생각해 보세요.

---

## 다음 장

다음 장에서는 `requests` 라이브러리로 외부 API를 호출하는 방법을 배우며, 이 장에서 배운 ISO 8601 날짜 형식이 실제 API 응답에서 어떻게 사용되는지 직접 확인합니다.