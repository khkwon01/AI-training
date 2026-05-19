## 이 장에서 배우는 것

- GitHub Advanced Security(GHAS)가 무엇인지, 왜 필요한지 이해한다
- Code Scanning(코드 스캐닝)을 GitHub Actions 워크플로에 연결하는 방법을 배운다
- CodeQL로 내 코드에서 보안 취약점을 자동으로 찾는 방법을 익힌다
- Secret Scanning(시크릿 스캐닝)으로 비밀번호·API 키 유출을 막는 방법을 배운다
- Dependabot으로 의존성 라이브러리의 보안 패치를 자동화하는 방법을 배운다
- SAST(정적 애플리케이션 보안 테스트)가 CI/CD 파이프라인에서 어떤 역할을 하는지 이해한다

---

## 먼저 쉬운 설명

코드를 짜다 보면 "일단 동작하면 됐지"라는 생각이 들기 쉽습니다. 그런데 실제 서비스에서는 동작하는 코드보다 **안전한 코드**가 훨씬 중요합니다.

보안 문제는 눈에 잘 보이지 않습니다. SQL 인젝션, 하드코딩된 비밀번호, 오래된 라이브러리의 취약점—이런 것들은 코드가 멀쩡히 실행되는 동안에도 해커의 공격 통로가 됩니다.

GitHub는 이 문제를 자동으로 잡아주는 도구들을 제공합니다. 마치 맞춤법 검사기가 글을 쓸 때 틀린 단어를 밑줄로 표시해 주듯이, GitHub의 보안 스캐닝 도구는 코드를 Push할 때마다 위험한 패턴을 찾아 알려줍니다.

이 장에서는 이 도구들을 실제 프로젝트에 연결하는 방법을 단계별로 배웁니다.

---

## 1. SAST란 무엇인가

**SAST(Static Application Security Testing)** 는 코드를 실행하지 않고, 소스 코드 자체를 분석해서 보안 취약점을 찾는 기술입니다.

```
코드 실행 없이 분석
┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐
│  소스 코드   │ ──▶ │  SAST 도구  │ ──▶ │  취약점 리포트 생성  │
│  (Python,   │     │  (CodeQL,   │     │  - SQL Injection     │
│   JS, etc.) │     │   Semgrep)  │     │  - XSS              │
└─────────────┘     └─────────────┘     │  - 하드코딩 비밀번호 │
                                        └─────────────────────┘
```

GitHub에서는 **CodeQL**이 대표적인 SAST 도구입니다. Pull Request를 열거나 코드를 Push할 때 자동으로 실행됩니다.

---

## 2. GitHub Advanced Security 구성 요소

GitHub의 보안 기능은 크게 세 가지입니다.

| 기능 | 역할 | 쉬운 비유 |
|------|------|-----------|
| **Code Scanning** | 코드 안의 보안 버그 탐지 | 코드 맞춤법 검사기 |
| **Secret Scanning** | API 키·비밀번호 유출 탐지 | 민감정보 감시 카메라 |
| **Dependabot** | 라이브러리 취약점 자동 패치 PR | 부품 리콜 알림 서비스 |

```python
# 나쁜 예시 — Secret Scanning이 이걸 잡아냅니다
import boto3

AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"        # ← 위험! 실제 키를 코드에 직접 씀
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxR"  # ← 위험!

client = boto3.client(
    "s3",
    aws_access_key_id=AWS_ACCESS_KEY,
    aws_secret_access_key=AWS_SECRET_KEY,
)
```

```python
# 좋은 예시 — 환경 변수로 분리
import os
import boto3

client = boto3.client(
    "s3",
    aws_access_key_id=os.environ["AWS_ACCESS_KEY_ID"],
    aws_secret_access_key=os.environ["AWS_SECRET_ACCESS_KEY"],
)
```

---

## 3. CodeQL 워크플로 작성하기

CodeQL을 GitHub Actions에 연결하는 워크플로 파일을 작성합니다. 파일 위치는 반드시 `.github/workflows/` 폴더 안이어야 합니다.

```yaml
# .github/workflows/codeql.yml

name: "CodeQL 보안 스캐닝"

on:
  push:
    branches: ["main", "develop"]
  pull_request:
    branches: ["main"]
  schedule:
    # 매주 월요일 오전 9시(KST = UTC+9이므로 UTC 00:00)에 정기 스캔
    - cron: "0 0 * * 1"

jobs:
  analyze:
    name: 코드 분석
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      security-events: write  # ← 보안 결과를 GitHub에 업로드하려면 반드시 필요

    strategy:
      fail-fast: false
      matrix:
        language: ["python", "javascript"]  # 프로젝트 언어에 맞게 수정하세요

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: CodeQL 초기화
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          # 추가 쿼리를 사용하면 더 많은 취약점을 탐지할 수 있습니다
          queries: security-extended

      - name: 자동 빌드
        uses: github/codeql-action/autobuild@v3

      - name: CodeQL 분석 실행
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

> **핵심 포인트**: `permissions`에서 `security-events: write`를 빠뜨리면 스캔 결과가 GitHub Security 탭에 나타나지 않습니다. 가장 흔한 실수입니다.

---

## 4. Secret Scanning 설정하기

Secret Scanning은 Public 저장소에서는 기본으로 활성화되어 있습니다. Private 저장소는 아래 경로에서 직접 켜야 합니다.

```
저장소 → Settings → Security & analysis → Secret scanning → Enable
```

**Push Protection** 기능도 함께 켜면, 비밀 키가 포함된 커밋 자체를 Push 단계에서 막을 수 있습니다.

```bash
# Push Protection이 작동하면 이런 에러가 납니다
$ git push origin main

remote: Push rejected. Secret detected in commit abc1234:
remote:   GitHub Personal Access Token
remote:   File: config/settings.py, Line: 12
remote:
remote: To allow this secret, visit:
remote:   https://github.com/your-org/your-repo/security/secret-scanning/...
```

이 에러가 보이면 커밋에서 비밀 키를 제거한 뒤 다시 Push해야 합니다.

```bash
# 실수로 커밋한 비밀 키를 히스토리에서 제거하는 방법
# (이미 Push 전이라면 아래 명령어로 마지막 커밋을 수정합니다)
git reset HEAD~1          # 마지막 커밋 취소 (파일은 유지)
# settings.py에서 비밀 키 제거 후
git add config/settings.py
git commit -m "fix: 하드코딩된 API 키를 환경 변수로 교체"
```

---

## 5. Dependabot 보안 업데이트 설정하기

Dependabot은 `requirements.txt`, `package.json` 등의 의존성 파일을 감시하다가 취약점이 발견된 라이브러리가 있으면 자동으로 업데이트 PR을 만들어 줍니다.

```yaml
# .github/dependabot.yml

version: 2
updates:
  # Python 패키지 관리
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
    # 한 번에 너무 많은 PR이 열리지 않도록 제한
    open-pull-requests-limit: 5

  # Node.js 패키지 관리
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5

  # GitHub Actions 자체도 최신으로 유지
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
```

Dependabot이 만든 PR은 아래처럼 보입니다.

```
[Dependabot] Bump requests from 2.28.0 to 2.31.0

Bumps requests from 2.28.0 to 2.31.0.

Changelog:
  2.31.0 — [Security] Fix CVE-2023-32681: Proxy-Authorization 헤더 유출 문제 수정

Dependabot compatibility score: 99%
```

---

## 6. 보안 스캔 결과 읽고 처리하기

CodeQL 스캔이 완료되면 **Security 탭 → Code scanning alerts** 에서 결과를 확인합니다.

```python
# CodeQL이 탐지하는 취약점 예시: SQL Injection

import sqlite3

def get_user(username):
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()

    # ❌ 위험: 사용자 입력을 직접 쿼리에 붙여 넣음
    query = "SELECT * FROM users WHERE name = '" + username + "'"
    cursor.execute(query)

    return cursor.fetchone()
```

```python
# ✅ 수정: 파라미터 바인딩 사용

def get_user(username):
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()

    # 물음표(?)가 사용자 입력을 안전하게 처리합니다
    query = "SELECT * FROM users WHERE name = ?"
    cursor.execute(query, (username,))

    return cursor.fetchone()
```

스캔 결과에서 `Dismiss alert`를 클릭하면 알림을 무시할 수 있지만, 반드시 **이유를 선택**해야 합니다 (`False positive` / `Used in tests` / `Won't fix`). 이유 없이 무시하면 나중에 왜 무시했는지 알 수 없게 됩니다.

---

## 따라 하기 실습

### 실습 1 — CodeQL 워크플로 추가하고 첫 스캔 실행하기

1. 실습용 저장소를 만듭니다 (이미 있는 저장소 사용 가능).

2. 아래 명령어로 워크플로 파일을 만듭니다.

```bash
mkdir -p .github/workflows
touch .github/workflows/codeql.yml
```

3. `codeql.yml`에 **섹션 3**의 내용을 붙여 넣고, `language` 항목을 본인 프로젝트 언어에 맞게 수정합니다.

```yaml
matrix:
  language: ["python"]  # JavaScript 프로젝트라면 "javascript"로 변경
```

4. 커밋하고 Push합니다.

```bash
git add .github/workflows/codeql.yml
git commit -m "ci: CodeQL 보안 스캐닝 워크플로 추가"
git push origin main
```

5. GitHub 저장소 → **Actions 탭**으로 이동해서 `CodeQL 보안 스캐닝` 워크플로가 실행되는지 확인합니다.

---

### 실습 2 — 의도적으로 취약한 코드를 Push하고 알림 확인하기

1. 취약점 테스트용 파일을 만듭니다.

```bash
touch app/vulnerable_example.py
```

2. 파일에 SQL Injection 취약 코드를 작성합니다.

```python
# app/vulnerable_example.py
# 주의: 이 파일은 CodeQL 탐지 테스트 전용입니다

import sqlite3

def search_product(name):
    conn = sqlite3.connect("shop.db")
    # CodeQL이 이 줄을 탐지합니다
    query = "SELECT * FROM products WHERE name = '" + name + "'"
    return conn.execute(query).fetchall()
```

3. Push 후 **Security 탭 → Code scanning alerts** 에서 `SQL query built from user-controlled sources` 알림이 생성되는지 확인합니다 (스캔 완료까지 약 2~5분 소요).

4. 알림 상세 페이지에서 `Show paths` 버튼을 눌러 데이터 흐름(Data flow)을 확인합니다.

---

### 실습 3 — Dependabot 설정 후 취약 라이브러리 시뮬레이션

1. `dependabot.yml` 파일을 만듭니다.

```bash
touch .github/dependabot.yml
```

2. **섹션 5**의 내용을 붙여 넣고 커밋합니다.

```bash
git add .github/dependabot.yml
git commit -m "ci: Dependabot 자동 보안 업데이트 설정"
git push origin main
```

3. 오래된 라이브러리가 포함된 `requirements.txt`를 만듭니다.

```bash
# requirements.txt
requests==2.20.0   # CVE-2018-18074 취약점이 있는 버전
```

4. Push 후 **Security 탭 → Dependabot alerts** 에서 알림이 생성되는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| `permissions`에 `security-events: write` 누락 | Code scanning 결과가 Security 탭에 안 보임 | `codeql.yml`의 `permissions` 블록에 `security-events: write` 추가 |
| `language` 항목을 프로젝트 언어와 다르게 설정 | `No source code was seen during the build` | `matrix.language`를 실제 언어(`python`, `javascript`, `java` 등)로 수정 |
| Private 저장소에서 Secret Scanning이 안 켜짐 | 비밀 키를 Push해도 알림이 안 옴 | Settings → Security & analysis → Secret scanning → Enable |
| `.github/dependabot.yml` 경로가 틀림 | Dependabot이 동작하지 않음 | 파일 위치가 반드시 `.github/dependabot.yml`이어야 함 (루트의 `dependabot.yml`은 인식 안 됨) |
| 이미 유출된 비밀 키를 히스토리에서 제거하지 않음 | 키를 파일에서 지워도 `git log`에 남아 있음 | `git reset`으로 커밋을 되돌리거나 `git filter-repo`로 히스토리를 정리하고, 즉시 키를 폐기·재발급 |
| CodeQL 분석이 너무 오래 걸림 | Actions에서 60분 이상 실행 중 | `autobuild` 대신 언어별 수동 빌드 단계를 추가하거나, `paths` 필터로 스캔 범위를 좁힘 |

---

## 확인 체크리스트

- [ ] `.github/workflows/codeql.yml` 파일이 저장소에 존재한다
- [ ] `permissions`에 `security-events: write`가 포함되어 있다
- [ ] `matrix.language`가 프로젝트의 실제 언어와 일치한다
- [ ] Actions 탭에서 CodeQL 워크플로가 성공(초록색 체크)으로 완료되었다
- [ ] Security 탭 → Code scanning alerts 에서 스캔 결과를 확인할 수 있다
- [ ] `.github/dependabot.yml` 파일이 올바른 경로에 존재한다
- [ ] Private 저장소라면 Secret Scanning이 Settings에서 활성화되어 있다
- [ ] 코드 안에 하드코딩된 비밀번호나 API 키가 없다
- [ ] 발견된 Code scanning alert를 하나 이상 열어 데이터 흐름을 읽어봤다

---

## 한 번 더 생각해 보기

1. **SAST가 모든 보안 문제를 잡아줄 수 있을까요?** SAST는 코드를 실행하지 않고 분석합니다. 로그인한 사용자만 접근 가능한 기능의 권한 오류, 혹은 잘못된 서버 설정처럼 "코드 외부"에 있는 문제는 SAST로 잡기 어렵습니다. SAST 외에 어떤 보완 방법이 있을지 생각해 보세요.

2. **Dependabot이 자동으로 만들어 준 PR을 무조건 Merge해도 될까요?** 라이브러리 버전이 올라가면 기존 코드가 동작하지 않을 수 있습니다(Breaking change). Dependabot PR을 안전하게 Merge하려면 어떤 조건이 갖춰져 있어야 할까요?

3. **비밀 키가 이미 GitHub에 Push되었다면, 파일에서 지우는 것만으로 충분할까요?** git의 히스토리에는 삭제한 내용도 남아 있습니다. 만약 실수로 AWS 키를 Push했다면, 파일 수정 외에 반드시 해야 할 일은 무엇일까요?

---

## 다음 장

다음 장에서는 GitHub Actions를 활용해 테스트 자동화와 배포 파이프라인을 연결하는 완전한 CI/CD 워크플로를 구성하는 방법을 배웁니다.