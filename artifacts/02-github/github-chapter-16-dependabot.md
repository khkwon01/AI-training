# Chapter 16: Dependabot 보안 관리

## 이 장에서 배우는 것

- Dependabot이 무엇인지, 왜 필요한지 이해한다
- `dependabot.yml` 파일을 직접 작성하고 저장소에 적용한다
- Dependabot이 자동으로 만들어 주는 Pull Request를 검토하고 병합한다
- GitHub Security Alerts 탭에서 취약점을 확인하고 대응한다
- 자동 보안 업데이트(Automated Security Updates)를 켜고 끈다

---

## 먼저 쉬운 설명

여러분이 집에서 스마트폰 앱을 쓴다고 생각해 보세요. 앱 스토어는 "이 앱에 보안 문제가 생겼으니 업데이트하세요"라고 알림을 보냅니다. 업데이트를 안 하면 개인 정보가 새어나갈 수도 있죠.

코드에서 사용하는 라이브러리(다른 사람이 만든 코드 묶음)도 똑같습니다. `requests`, `flask`, `django` 같은 패키지에 보안 취약점이 발견되면, 그냥 두면 내 서비스가 해킹 당할 수 있습니다.

**Dependabot**은 GitHub이 제공하는 자동 감시 로봇입니다. 매일 또는 매주 내 프로젝트의 패키지 버전을 확인하고, 취약점이 있거나 오래된 버전을 발견하면 자동으로 업데이트 PR(Pull Request)을 만들어 줍니다. 내가 직접 일일이 찾아다닐 필요가 없어집니다.

---

## 1. Dependabot이란 무엇인가

Dependabot은 두 가지 역할을 합니다.

| 역할 | 설명 |
|---|---|
| **보안 알림** (Security Alerts) | 알려진 취약점(CVE)이 있는 패키지를 발견하면 즉시 알림 |
| **버전 업데이트** (Version Updates) | 최신 버전이 나오면 자동으로 PR 생성 |

### 어떤 파일을 감시하나

Dependabot은 아래 파일들을 자동으로 읽습니다.

```
# Python
requirements.txt
pyproject.toml
Pipfile

# Node.js
package.json

# GitHub Actions
.github/workflows/*.yml
```

---

## 2. dependabot.yml 파일 작성하기

Dependabot 동작 방식을 직접 설정하려면 저장소의 `.github/dependabot.yml` 파일을 만들어야 합니다.

### 기본 구조

```yaml
# .github/dependabot.yml

version: 2

updates:
  # Python 패키지 (pip) 감시
  - package-ecosystem: "pip"
    directory: "/"          # requirements.txt 가 있는 위치
    schedule:
      interval: "weekly"    # 매주 월요일에 확인
    labels:
      - "dependencies"      # PR에 자동으로 붙는 라벨
    reviewers:
      - "kihyuk-kwon"       # PR 리뷰어 자동 지정

  # GitHub Actions 워크플로우 감시
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"   # 매달 확인
```

### schedule 옵션 비교

```yaml
# 매일 확인 (업데이트가 많은 프로젝트에 적합)
schedule:
  interval: "daily"

# 매주 월요일 오전 9시(한국 시간) 확인
schedule:
  interval: "weekly"
  day: "monday"
  time: "09:00"
  timezone: "Asia/Seoul"

# 매달 1일 확인 (안정성이 중요한 프로젝트에 적합)
schedule:
  interval: "monthly"
```

### 특정 패키지 업데이트를 건너뛰는 방법

```yaml
# .github/dependabot.yml

version: 2

updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    ignore:
      # django는 주요 버전(major) 업그레이드만 무시
      - dependency-name: "django"
        update-types: ["version-update:semver-major"]

      # boto3는 완전히 업데이트 무시 (이유: 사내 호환성 문제)
      - dependency-name: "boto3"
```

---

## 3. Security Alerts 확인하기

### GitHub에서 확인하는 방법

1. 저장소 페이지 상단 탭에서 **Security** 클릭
2. 왼쪽 사이드바에서 **Dependabot alerts** 클릭
3. 취약점 목록에서 각 항목을 클릭하면 상세 정보 확인 가능

알림 예시 화면에서 보이는 정보:

```
패키지명: requests 2.25.0
취약점:  CVE-2023-32681 (보안 등급: Medium)
설명:    HTTP 리다이렉트 시 인증 헤더가 유출될 수 있음
권고 조치: requests >= 2.31.0 으로 업그레이드
```

### 심각도(Severity) 등급 이해하기

| 등급 | 색상 | 의미 | 권장 대응 시간 |
|---|---|---|---|
| Critical | 빨강 | 즉시 악용 가능한 취약점 | 즉시 |
| High | 주황 | 높은 위험도 | 48시간 이내 |
| Medium | 노랑 | 조건부 위험 | 1주일 이내 |
| Low | 파랑 | 낮은 위험도 | 다음 릴리즈까지 |

---

## 4. Dependabot PR 검토하고 병합하기

Dependabot이 만든 PR을 보면 이런 형태입니다.

```
PR 제목: Bump requests from 2.25.0 to 2.31.0

변경 파일: requirements.txt

- requests==2.25.0
+ requests==2.31.0

릴리즈 노트:
- 2.31.0: 보안 취약점 CVE-2023-32681 수정
- 2.28.0: urllib3 의존성 업데이트
```

### 병합 전 확인 사항

```bash
# 1. 로컬에서 브랜치를 받아 테스트
git fetch origin
git checkout dependabot/pip/requests-2.31.0

# 2. 가상환경 업데이트 후 테스트 실행
pip install -r requirements.txt
python -m pytest tests/

# 3. 문제 없으면 GitHub에서 Merge 버튼 클릭
```

### requirements.txt 예시 (실제 프로젝트 파일)

```
# requirements.txt

# 웹 프레임워크
flask==2.3.3

# HTTP 클라이언트
requests==2.31.0      # Dependabot이 2.25.0 → 2.31.0 으로 올려줌

# 데이터 검증
pydantic==2.1.1

# 환경 변수 관리
python-dotenv==1.0.0
```

---

## 5. 자동 보안 업데이트 켜기

Security Alerts가 발생했을 때 Dependabot이 자동으로 PR을 만들도록 설정할 수 있습니다.

### 설정 방법 (GitHub 웹 UI)

```
저장소 → Settings → Security → Code security and analysis

아래 항목을 Enable 클릭:
✅ Dependabot alerts
✅ Dependabot security updates    ← 이것을 켜면 자동 PR 생성
✅ Dependabot version updates
```

### 알림 이메일 수신 설정

```
GitHub 우측 상단 프로필 → Settings
→ Notifications
→ "Dependabot alerts" 항목에서
   ✅ Email 체크
```

---

## 따라 하기 실습

### 실습 1: dependabot.yml 파일 만들기

아래 구조로 새 파일을 만드세요.

```bash
# 프로젝트 루트에서 실행
mkdir -p .github
touch .github/dependabot.yml
```

`.github/dependabot.yml` 파일에 아래 내용을 붙여넣습니다.

```yaml
version: 2

updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Seoul"
    labels:
      - "dependencies"
      - "security"
    open-pull-requests-limit: 5
```

커밋하고 푸시합니다.

```bash
git add .github/dependabot.yml
git commit -m "chore: add dependabot configuration"
git push origin main
```

### 실습 2: 취약한 패키지 버전으로 테스트 환경 만들기

보안 알림이 실제로 어떻게 뜨는지 확인하기 위해 낮은 버전의 패키지를 넣어봅니다.

```
# requirements-test.txt  (실습용 파일, 실제 서비스에 쓰지 않음)

# 취약점이 알려진 구버전 (확인용)
requests==2.20.0
```

```bash
git add requirements-test.txt
git commit -m "test: add old dependency for security alert demo"
git push origin main
```

GitHub 저장소의 **Security → Dependabot alerts** 탭에서 알림이 생성되는지 수분 내로 확인합니다.

### 실습 3: Dependabot PR 검토 및 처리하기

실습 2 이후 Dependabot이 자동으로 생성한 PR을 처리합니다.

1. 저장소의 **Pull requests** 탭으로 이동합니다.
2. `Bump requests from 2.20.0 to ...` 형태의 PR을 클릭합니다.
3. **Files changed** 탭에서 변경 내용을 확인합니다.
4. CI 테스트(Actions)가 통과했는지 확인합니다.
5. 이상 없으면 **Merge pull request** 버튼을 클릭합니다.
6. 병합 후 **Security → Dependabot alerts** 에서 해당 알림이 `Resolved` 상태로 바뀌었는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| `dependabot.yml` 위치를 잘못 설정 | Dependabot이 아무 PR도 만들지 않음 | `.github/dependabot.yml` 경로인지 확인. `.github` 폴더가 루트에 있어야 함 |
| `directory` 값을 잘못 지정 | `No supported configuration file found` | `requirements.txt`가 있는 실제 경로로 수정. 루트라면 `"/"` |
| YAML 들여쓰기 오류 | `Error parsing dependabot.yml: mapping values are not allowed here` | 탭 대신 스페이스 2칸 사용. YAML은 탭 문자를 허용하지 않음 |
| Dependabot PR을 무작정 병합 | 서비스 장애 또는 테스트 실패 | 반드시 CI 테스트 통과 후 병합. Breaking change 릴리즈 노트 확인 필수 |
| `open-pull-requests-limit` 초과 | 새 PR이 생성되지 않음 | 기존 Dependabot PR을 먼저 처리하거나 limit 값을 높임 |
| Security Alerts 탭이 안 보임 | 탭 자체가 없음 | 저장소 Settings → Security에서 `Dependabot alerts` 활성화 필요 |

---

## 확인 체크리스트

- [ ] `.github/dependabot.yml` 파일이 저장소 루트의 `.github` 폴더 안에 있다
- [ ] `version: 2` 가 파일 맨 위에 있다
- [ ] `package-ecosystem` 값이 내 프로젝트 언어와 맞다 (`pip`, `npm`, `github-actions` 등)
- [ ] `schedule.interval` 을 `weekly` 또는 `monthly` 로 설정했다
- [ ] GitHub 저장소 Settings에서 `Dependabot alerts` 와 `Dependabot security updates` 가 활성화되어 있다
- [ ] Security → Dependabot alerts 탭에서 현재 알림 목록을 확인했다
- [ ] Critical/High 등급 취약점이 있다면 대응 계획을 세웠다
- [ ] Dependabot PR을 병합하기 전에 CI 테스트 결과를 확인하는 습관을 들였다

---

## 한 번 더 생각해 보기

1. **Dependabot을 `daily`로 설정하면 어떤 장단점이 있을까요?** 업데이트 속도와 PR 관리 부담 사이에서 어떤 균형이 적절할지 팀 규모와 함께 생각해 보세요.

2. **`ignore` 옵션으로 특정 패키지 업데이트를 막으면 언제까지 막아야 할까요?** 호환성 문제로 업그레이드를 미루다가 더 큰 취약점이 쌓이는 상황을 어떻게 예방할 수 있을지 생각해 보세요.

3. **Dependabot이 만들어 준 PR이 CI 테스트에서 실패했습니다.** 이 경우 그냥 PR을 닫아야 할까요, 아니면 다른 방법이 있을까요? 어떤 정보를 먼저 확인할지 순서를 생각해 보세요.

---

## 다음 장

다음 장에서는 GitHub Actions에서 보안에 민감한 API 키와 비밀번호를 안전하게 관리하는 **Actions Secrets** 활용법을 배웁니다.