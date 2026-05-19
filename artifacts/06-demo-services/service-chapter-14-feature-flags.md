## 이 장에서 배우는 것

- 피처 플래그(Feature Flag)가 무엇인지, 왜 사용하는지 이해한다
- 딕셔너리와 환경 변수로 간단한 피처 플래그를 구현한다
- 피처 플래그로 새 기능을 안전하게 켜고 끄는 방법을 익힌다
- 실제 서비스에서 사용하는 패턴을 직접 코드로 작성해 본다
- 피처 플래그를 사용할 때 흔히 저지르는 실수를 피한다

---

## 먼저 쉬운 설명

새로운 기능을 만들었는데, 아직 모든 사용자에게 보여주기 무섭다고 느낀 적 있나요?

예를 들어 쇼핑몰에 "AI 추천 상품" 기능을 새로 만들었습니다. 그런데 배포하자마자 오류가 나면 모든 사용자가 피해를 봅니다. 이럴 때 **피처 플래그**를 사용하면, 코드를 배포하면서도 기능을 일단 꺼둔 상태로 운영할 수 있습니다. 준비가 됐을 때 플래그를 켜면 기능이 활성화됩니다.

쉽게 말하면, 피처 플래그는 **코드 속에 있는 전등 스위치**입니다.

```
배포 ──→ 기능 꺼짐 ──→ 테스트 완료 ──→ 스위치 ON ──→ 기능 켜짐
```

이 방식 덕분에 "코드 배포"와 "기능 출시"를 분리할 수 있습니다. 대형 서비스(쿠팡, 카카오 등)가 매일 수십 번 배포하면서도 안정적으로 운영하는 비결 중 하나가 바로 이것입니다.

---

## 1. 가장 단순한 피처 플래그 — 딕셔너리 방식

가장 쉬운 방법은 딕셔너리에 플래그를 정의하는 것입니다.

**파일: `feature_flags.py`**

```python
# 피처 플래그를 딕셔너리로 정의합니다.
# True = 기능 켜짐, False = 기능 꺼짐
FEATURE_FLAGS = {
    "ai_recommendation": False,   # AI 추천 기능 (아직 준비 중)
    "dark_mode": True,            # 다크 모드 (이미 배포 완료)
    "new_checkout_page": False,   # 새 결제 화면 (테스트 중)
}


def is_enabled(flag_name: str) -> bool:
    """주어진 플래그가 켜져 있는지 확인합니다."""
    return FEATURE_FLAGS.get(flag_name, False)
```

**파일: `shop.py`**

```python
from feature_flags import is_enabled


def get_product_list(user_id: int) -> list:
    products = ["상품A", "상품B", "상품C"]

    # AI 추천 기능이 켜져 있을 때만 추천 상품을 추가합니다.
    if is_enabled("ai_recommendation"):
        products += ["AI추천1", "AI추천2"]

    return products


# 실행 예시
print(get_product_list(user_id=42))
# 출력: ['상품A', '상품B', '상품C']
# ai_recommendation이 False이므로 추천 상품은 추가되지 않습니다.
```

> **핵심 포인트:** `FEATURE_FLAGS["ai_recommendation"] = True` 로 바꾸는 것만으로 기능이 켜집니다. 코드 로직을 건드리지 않아도 됩니다.

---

## 2. 환경 변수로 플래그 관리하기

딕셔너리 방식은 코드를 수정해야 합니다. 운영 환경에서는 코드 수정 없이 플래그를 바꾸고 싶을 때가 많습니다. 이럴 때 **환경 변수**를 사용합니다.

**파일: `.env`** (절대 Git에 올리지 마세요)

```
FEATURE_AI_RECOMMENDATION=false
FEATURE_DARK_MODE=true
FEATURE_NEW_CHECKOUT_PAGE=false
```

**파일: `feature_flags.py`**

```python
import os


def is_enabled(flag_name: str) -> bool:
    """
    환경 변수에서 피처 플래그 값을 읽습니다.
    환경 변수 이름 규칙: FEATURE_<대문자_플래그명>
    예: is_enabled("dark_mode") → 환경 변수 FEATURE_DARK_MODE 를 읽음
    """
    env_key = f"FEATURE_{flag_name.upper()}"
    value = os.getenv(env_key, "false")
    return value.lower() == "true"


# 사용 예시
if __name__ == "__main__":
    print(is_enabled("dark_mode"))          # True
    print(is_enabled("ai_recommendation"))  # False
```

**파일: `shop.py`**

```python
from feature_flags import is_enabled


def render_theme(user_id: int) -> str:
    if is_enabled("dark_mode"):
        return "dark-theme.css"
    return "light-theme.css"


print(render_theme(user_id=1))
# 출력: dark-theme.css
```

> **실무 팁:** 서버를 배포하지 않고, 환경 변수만 바꿔도 기능이 즉시 활성화/비활성화됩니다. AWS, GCP 같은 클라우드 서비스의 "환경 변수 설정" 화면에서 바로 바꿀 수 있습니다.

---

## 3. 사용자별로 다르게 적용하기 — 점진적 출시(Gradual Rollout)

새 기능을 처음에는 전체 사용자의 10%에게만 보여주고, 문제가 없으면 50%, 100%로 늘리는 방식입니다. 이것을 **점진적 출시**라고 합니다.

**파일: `feature_flags.py`**

```python
import hashlib


def is_enabled_for_user(flag_name: str, user_id: int, rollout_percent: int) -> bool:
    """
    사용자 ID를 기준으로 일정 비율의 사용자에게만 기능을 활성화합니다.
    같은 사용자는 항상 같은 결과를 받습니다(결정론적).
    """
    if rollout_percent <= 0:
        return False
    if rollout_percent >= 100:
        return True

    # 사용자 ID와 플래그 이름을 조합해서 해시를 만듭니다.
    key = f"{flag_name}:{user_id}"
    hash_value = int(hashlib.md5(key.encode()).hexdigest(), 16)

    # 해시 값을 100으로 나눈 나머지가 rollout_percent 미만이면 활성화
    return (hash_value % 100) < rollout_percent


# 실행 예시
for uid in [1, 2, 3, 100, 999]:
    result = is_enabled_for_user("new_checkout_page", user_id=uid, rollout_percent=30)
    print(f"사용자 {uid}: {'활성화' if result else '비활성화'}")
```

**출력 예시:**
```
사용자 1: 비활성화
사용자 2: 활성화
사용자 3: 비활성화
사용자 100: 활성화
사용자 999: 비활성화
```

> **왜 해시를 사용하나요?** 랜덤을 사용하면 같은 사용자가 새로고침할 때마다 결과가 달라집니다. 해시를 사용하면 동일한 사용자는 항상 동일한 경험을 하게 됩니다.

---

## 4. 클래스로 정리하기 — 실무형 패턴

여러 방식을 하나의 클래스로 묶으면 코드가 훨씬 깔끔해집니다.

**파일: `feature_flags.py`**

```python
import os
import hashlib
from dataclasses import dataclass, field


@dataclass
class FeatureFlag:
    name: str
    enabled: bool = False
    rollout_percent: int = 100  # enabled=True일 때 몇 %의 사용자에게 적용할지


class FeatureFlagManager:
    def __init__(self):
        self._flags: dict[str, FeatureFlag] = {}

    def register(self, flag: FeatureFlag) -> None:
        """플래그를 등록합니다."""
        self._flags[flag.name] = flag

    def is_enabled(self, flag_name: str, user_id: int | None = None) -> bool:
        """플래그가 활성화되어 있는지 확인합니다."""
        flag = self._flags.get(flag_name)

        # 등록되지 않은 플래그는 항상 비활성화
        if flag is None:
            return False

        # 전역으로 꺼진 경우
        if not flag.enabled:
            return False

        # user_id 없이 호출하면 전체 활성화로 간주
        if user_id is None:
            return True

        # 점진적 출시 비율 계산
        key = f"{flag_name}:{user_id}"
        hash_value = int(hashlib.md5(key.encode()).hexdigest(), 16)
        return (hash_value % 100) < flag.rollout_percent


# 앱 전역에서 사용할 매니저 인스턴스
flags = FeatureFlagManager()
flags.register(FeatureFlag(name="ai_recommendation", enabled=False))
flags.register(FeatureFlag(name="dark_mode", enabled=True, rollout_percent=100))
flags.register(FeatureFlag(name="new_checkout_page", enabled=True, rollout_percent=20))
```

**파일: `shop.py`**

```python
from feature_flags import flags


def get_checkout_page(user_id: int) -> str:
    if flags.is_enabled("new_checkout_page", user_id=user_id):
        return "새로운 결제 화면"
    return "기존 결제 화면"


# 여러 사용자 테스트
for uid in range(1, 6):
    page = get_checkout_page(user_id=uid)
    print(f"사용자 {uid}: {page}")
```

---

## 따라 하기 실습

### 실습 1 — 딕셔너리 피처 플래그 만들기

다음 구조로 파일을 만들어 보세요.

```
my_app/
├── feature_flags.py
└── main.py
```

**`feature_flags.py`** 에 아래 코드를 작성하세요:

```python
FEATURE_FLAGS = {
    "show_banner": True,
    "beta_search": False,
}

def is_enabled(flag_name: str) -> bool:
    return FEATURE_FLAGS.get(flag_name, False)
```

**`main.py`** 에 아래 코드를 작성하고 실행해 보세요:

```python
from feature_flags import is_enabled, FEATURE_FLAGS

def render_page() -> None:
    if is_enabled("show_banner"):
        print("[배너] 여름 세일 진행 중!")

    if is_enabled("beta_search"):
        print("[베타] 새로운 검색 기능을 사용 중입니다.")
    else:
        print("[기본] 기존 검색 기능을 사용 중입니다.")

render_page()

# 직접 플래그를 바꿔보세요
print("\n--- beta_search 켜기 ---")
FEATURE_FLAGS["beta_search"] = True
render_page()
```

**예상 출력:**
```
[배너] 여름 세일 진행 중!
[기본] 기존 검색 기능을 사용 중입니다.

--- beta_search 켜기 ---
[배너] 여름 세일 진행 중!
[베타] 새로운 검색 기능을 사용 중입니다.
```

---

### 실습 2 — 환경 변수로 플래그 제어하기

실습 1의 `feature_flags.py`를 환경 변수 방식으로 바꿔보세요.

**`feature_flags.py`** 를 수정하세요:

```python
import os

def is_enabled(flag_name: str) -> bool:
    env_key = f"FEATURE_{flag_name.upper()}"
    return os.getenv(env_key, "false").lower() == "true"
```

터미널에서 환경 변수를 설정하고 실행해 보세요:

```bash
# macOS / Linux
export FEATURE_SHOW_BANNER=true
export FEATURE_BETA_SEARCH=false
python main.py

# Windows PowerShell
$env:FEATURE_SHOW_BANNER="true"
$env:FEATURE_BETA_SEARCH="false"
python main.py
```

코드를 전혀 수정하지 않고도 환경 변수만으로 기능이 켜지고 꺼지는 것을 확인하세요.

---

### 실습 3 — 점진적 출시 체험하기

다음 파일을 만들어 사용자 100명 중 몇 명에게 기능이 활성화되는지 직접 세어보세요.

**`rollout_test.py`**

```python
import hashlib

def is_enabled_for_user(flag_name: str, user_id: int, rollout_percent: int) -> bool:
    if rollout_percent <= 0:
        return False
    if rollout_percent >= 100:
        return True
    key = f"{flag_name}:{user_id}"
    hash_value = int(hashlib.md5(key.encode()).hexdigest(), 16)
    return (hash_value % 100) < rollout_percent


# 사용자 1~100번에게 30% 출시 시뮬레이션
enabled_users = []
for uid in range(1, 101):
    if is_enabled_for_user("new_checkout_page", user_id=uid, rollout_percent=30):
        enabled_users.append(uid)

print(f"활성화된 사용자 수: {len(enabled_users)} / 100")
print(f"활성화된 사용자 ID: {enabled_users[:10]} ...")  # 앞 10명만 출력

# rollout_percent를 10, 50, 80으로 바꿔가며 실험해 보세요.
```

**예상 출력:**
```
활성화된 사용자 수: 30 / 100  # 대략 30명 (해시에 따라 약간 다를 수 있음)
활성화된 사용자 ID: [2, 5, 11, 18, 23, 24, 31, 37, 42, 48] ...
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| 플래그 이름 오타 | 기능이 항상 꺼져 있음, 오류 없이 `False` 반환 | `FEATURE_FLAGS.get(name, False)` 대신 키가 없으면 `KeyError`를 내는 `FEATURE_FLAGS[name]` 으로 개발 중에 검증 |
| 환경 변수 값을 `True`/`False`(대문자)로 설정 | 기능이 항상 꺼짐 | `os.getenv(...).lower() == "true"` 로 소문자 비교 필수 |
| 피처 플래그를 삭제하지 않고 영원히 유지 | 코드가 복잡해지고 `if is_enabled(...)` 분기가 쌓임 | 기능 출시 완료 후 플래그와 분기 코드를 함께 삭제 |
| `rollout_percent`에 `random()` 사용 | 같은 사용자가 새로고침할 때마다 기능이 켜졌다 꺼졌다 함 | 반드시 `user_id` 기반 해시를 사용해 결정론적으로 처리 |
| 플래그가 너무 많아짐 | 어떤 플래그가 켜져 있는지 파악 불가 | 플래그 목록을 한 파일에 모으고, 담당자·만료일을 주석으로 명시 |
| 환경 변수 설정 후 서버를 재시작하지 않음 | 환경 변수를 바꿔도 반영이 안 됨 | 환경 변수를 코드 시작 시 읽는 경우, 반드시 프로세스를 재시작해야 함 |

---

## 확인 체크리스트

- [ ] `feature_flags.py` 파일을 만들고 `is_enabled()` 함수를 직접 구현했다
- [ ] `FEATURE_FLAGS["플래그명"] = True/False` 를 바꿔 기능이 켜지고 꺼지는 것을 확인했다
- [ ] 환경 변수 방식으로 코드 수정 없이 플래그를 바꿔봤다
- [ ] `rollout_percent=10`, `50`, `100` 으로 바꿔가며 활성화 비율이 달라지는 것을 확인했다
- [ ] 해시 방식을 사용해 같은 사용자는 항상 같은 결과를 받는다는 것을 이해했다
- [ ] 피처 플래그는 기능 출시 완료 후 코드에서 제거해야 한다는 것을 안다
- [ ] `FeatureFlagManager` 클래스를 직접 타이핑해보고 실행했다

---

## 한 번 더 생각해 보기

1. 피처 플래그를 사용하면 "배포"와 "기능 출시"를 분리할 수 있다고 했습니다. 이것이 팀 협업에서 어떤 장점을 가져올 수 있을까요? 예를 들어 백엔드 개발자와 프론트엔드 개발자가 동시에 같은 기능을 개발할 때 어떻게 활용할 수 있을지 생각해 보세요.

2. 실습 3에서 `rollout_percent=30` 으로 설정하면 정확히 30명이 활성화될까요, 아니면 그렇지 않을 수도 있을까요? 해시 함수의 특성을 고려해서 이유를 설명해 보세요.

3. 피처 플래그는 "기술적 부채"가 될 수 있다고도 합니다. 플래그를 출시 후에도 제거하지 않으면 어떤 문제가 생길지 생각해 보세요. 팀 규칙으로 어떻게 이 문제를 예방할 수 있을까요?

---

## 다음 장

다음 장에서는 피처 플래그를 데이터베이스나 외부 서비스(예: LaunchDarkly)에 저장해 실시간으로 값을 바꾸는 **동적 피처 플래그** 패턴을 배웁니다.