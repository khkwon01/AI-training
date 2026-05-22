# Chapter 06: 배포 런북과 운영 체크리스트

## 이 장에서 배우는 것

- 런북이 왜 필요한지 (기억에 의존한 배포의 위험)
- 배포 전 체크리스트 (로컬 테스트 통과? 환경변수 설정? IAM 권한?)
- 배포 방법별 단계 (콘솔 직접 편집, zip 파일 업로드)
- 배포 후 검증 체크리스트 (Function URL 응답? CloudWatch 로그 정상?)
- 롤백 방법 (이전 버전으로 되돌리기)
- 주간 운영 점검 항목
- 실습: 체크리스트 따라 실제 배포 → 검증 → 롤백 연습

---

## 왜 필요한가 — 기억에 의존한 배포의 위험

코드를 Lambda에 올리는 건 간단하다. 편집기에 붙여넣고 Deploy를 누르면 된다.

처음에는 잘 된다. 배포할 것이 많지 않고, 모든 걸 기억할 수 있다.

한 달이 지나면 어떻게 될까?

- "환경 변수에 뭐가 있었더라?"
- "테스트는 어떻게 했었지?"
- "오류 나면 어떻게 롤백하더라?"
- "배포 순서가 맞나?"

기억은 틀린다. 피곤할 때, 급할 때, 오랜만에 배포할 때 특히 그렇다.

### 실제로 일어나는 배포 사고

**사고 1: 환경 변수 설정 누락**

새 서버에 배포했는데 `OPENAI_API_KEY` 환경 변수를 설정하는 것을 잊었다. 배포 직후부터 모든 요청이 500 오류를 반환한다. 원인 파악에 30분이 걸렸다.

배포 전 체크리스트에 "환경 변수 확인" 항목이 있었다면 1분 안에 잡을 수 있었다.

**사고 2: 테스트 안 한 코드 배포**

로컬에서 고쳤다고 생각했는데, 실제로 `lambda_function.py`가 아닌 `test.py`를 고쳐놨다. Lambda에는 수정 전 코드가 올라갔다. 배포 직후 테스트를 했다면 바로 알 수 있었다.

**사고 3: 롤백 방법을 몰라서 다운타임 연장**

오류가 난 배포를 되돌리려는데 어떻게 하는지 몰랐다. 구글링하면서 20분을 낭비했다. 미리 롤백 절차를 문서화해뒀다면 5분 안에 끝났다.

### 런북이 해결하는 것

**런북(runbook)**은 "이 서비스를 어떻게 배포하고, 문제가 생기면 어떻게 대응한다"를 미리 정리해둔 절차서다.

런북이 있으면:
- 배포마다 같은 순서로 진행된다 → 실수가 줄어든다
- 오랜만에 배포해도 처음과 같은 품질로 배포할 수 있다
- 나 아닌 다른 사람도 배포할 수 있다
- 문제 발생 시 당황하지 않고 절차대로 대응한다

---

## 1. 배포 전 체크리스트

코드를 Lambda에 올리기 전에 반드시 확인한다. 모두 체크한 후에 배포한다.

### 코드 준비 확인

```
□ 로컬에서 python lambda_function.py를 실행해서 기본 동작을 확인했는가
□ 수정한 파일이 실제로 lambda_function.py인지 확인했는가
□ print(), logging 등 디버그용으로만 쓴 민감 정보 출력을 제거했는가
□ 코드에 API 키, 비밀번호가 하드코딩되어 있지 않은가
```

### AWS 설정 확인

```
□ Lambda 함수에 필요한 환경 변수가 모두 설정되어 있는가
  (Configuration → Environment variables에서 확인)
□ Lambda 실행 역할에 필요한 권한이 있는가
  (Configuration → Permissions에서 확인)
□ Lambda 타임아웃 설정이 처리 시간에 충분한가
  (Configuration → General configuration에서 확인, 기본 3초)
□ 메모리 설정이 충분한가
  (Configuration → General configuration, 기본 128MB)
```

### 변경 내용 파악

```
□ 이번 배포에서 무엇이 추가/변경/삭제됐는지 한 문장으로 설명할 수 있는가
□ 롤백이 필요하면 이전 코드를 찾을 수 있는가
  (GitHub에서 이전 commit, 또는 Lambda Versions 탭)
```

---

## 2. 배포 방법 A — 콘솔 직접 편집

코드가 단일 파일(`lambda_function.py`)이고 외부 패키지 의존성이 없을 때 사용한다.

### Step 1. 현재 버전 기록 (롤백 준비)

```
Lambda 함수 페이지 → [Versions] 탭 확인
```

버전이 없거나 최신이 배포되지 않은 상태라면:
```
Lambda 함수 페이지 → 오른쪽 위 [Actions] 드롭다운 → [Publish new version]
→ Description에 "배포 전 스냅샷 - YYYY-MM-DD" 입력
→ [Publish] 클릭
```

이 단계를 거치면 문제가 생겼을 때 이 버전으로 돌아올 수 있다.

```
배포 전 버전 기록: Version 5  (문제 생기면 이 버전 코드로 복원)
```

### Step 2. 코드 편집

```
Lambda 함수 페이지 → [Code] 탭 → 파일 탐색기에서 lambda_function.py 더블클릭
→ 기존 코드 전체 선택(Ctrl+A / Cmd+A) → 새 코드 붙여넣기
```

### Step 3. Deploy

오른쪽 위 **[Deploy]** 버튼 클릭.

"Changes deployed" 메시지가 나타나면 성공이다.

주의: Deploy를 누르지 않으면 변경사항이 저장만 되고 적용되지 않는다. 저장(Save) ≠ 배포(Deploy).

---

## 3. 배포 방법 B — zip 파일 업로드

외부 패키지(`requests`, `boto3` 등)가 필요할 때, 또는 파일이 여러 개일 때 사용한다.

### Step 1. 로컬에서 배포 패키지 만들기

```bash
# 프로젝트 폴더에서 실행
mkdir deploy_package
pip install requests -t deploy_package/   # 외부 패키지를 폴더에 설치

# Lambda 코드 복사
cp lambda_function.py deploy_package/

# zip 파일 생성 (deploy_package 폴더 안의 파일들을 압축)
cd deploy_package
zip -r ../lambda_deploy.zip .
cd ..
```

완료되면 프로젝트 폴더에 `lambda_deploy.zip` 파일이 생긴다.

### Step 2. Lambda 콘솔에서 zip 업로드

```
Lambda 함수 페이지 → [Code] 탭
→ 오른쪽 위 [Upload from] 드롭다운 → [.zip file]
→ [Upload] 클릭 → lambda_deploy.zip 선택
→ [Save] 클릭
```

저장이 완료되면 자동으로 Deploy된다. 별도로 Deploy 버튼을 누르지 않아도 된다.

### Step 3. 핸들러 설정 확인

zip으로 올릴 때 핸들러 경로가 맞는지 확인한다.

```
Configuration → General configuration → Edit
→ Handler: lambda_function.lambda_handler
```

파일명이 `lambda_function.py`이고 함수명이 `lambda_handler`이면 기본값 그대로 두면 된다.

---

## 4. 배포 후 검증 체크리스트

Deploy가 완료되면 즉시 아래를 확인한다. 문제를 빠르게 잡는 것이 핵심이다.

### 기능 테스트

```
□ 기본 URL 호출이 200 응답을 반환하는가
  curl "https://함수URL/"

□ 주요 기능이 정상 동작하는가
  curl "https://함수URL/?name=테스트"
  curl -X POST "https://함수URL/memos?text=테스트메모"

□ 응답 JSON 형식이 예상과 일치하는가
  {"message": "..."} 형식인가?
```

### 로그 확인

```
□ CloudWatch Logs에 오류([ERROR])가 없는가
  Lambda → Monitor → View CloudWatch logs → 최신 Log stream

□ 배포 직전 대비 로그 패턴이 정상인가
  기존: [INFO] 요청 수신 → [INFO] 처리 완료
  지금도 같은 패턴이 나오는가?
```

### 지표 확인

```
□ Monitor 탭의 Errors 카운트가 0인가
  (배포 직후 몇 분 후에 확인)

□ Duration이 이전과 비슷한가
  (갑자기 높아졌다면 코드 변경으로 인한 성능 저하)
```

### 검증 실패 시 즉시 롤백

검증 항목 중 하나라도 실패하면 배포를 즉시 롤백한다. 잠시 기다려보는 것은 좋지 않다. 빠르게 롤백해서 서비스를 복구하고, 원인을 분석하는 것이 순서다.

---

## 5. 문제 발생 시 빠른 진단

배포 후 오류가 발생했을 때 원인을 빠르게 찾는 절차다.

### 1분 안에 확인할 것

**CloudWatch Logs 확인:**

```
Lambda → Monitor → View CloudWatch logs
→ 최신 Log stream 클릭
→ [ERROR]가 보이는가?
→ 오류 메시지와 traceback 복사
```

**Lambda Monitor 탭 확인:**

```
Lambda → Monitor 탭
→ Errors 그래프: 배포 시점부터 올라가고 있는가?
→ Duration 그래프: 갑자기 높아졌는가?
→ Throttles 그래프: 제한에 걸리고 있는가?
```

### 오류 유형별 대응

| 오류 | 원인 가능성 | 즉각 조치 |
|------|-----------|---------|
| `500 Internal Server Error` | 코드 오류 | CloudWatch에서 traceback 확인 후 수정 또는 롤백 |
| `Task timed out after X.XX seconds` | 실행 시간 초과 | 타임아웃 설정 늘리거나 코드 최적화 후 재배포 |
| `Unable to import module 'lambda_function'` | 코드 문법 오류 또는 의존성 누락 | 코드 문법 확인, zip 패키지 재확인 |
| `AccessDeniedException` | IAM 권한 부족 | Configuration → Permissions에서 필요한 정책 추가 |
| `[ERROR] KeyError: 'xxx'` | 입력값 처리 오류 | 코드에서 해당 키를 `.get()`으로 안전하게 읽도록 수정 |
| `ModuleNotFoundError: No module named 'xxx'` | 패키지 누락 | zip 배포 시 패키지 포함 여부 확인 |

### 배포 이전과 비교하기

오류가 배포 직후 시작됐다면 원인은 대부분 배포한 코드다.

```bash
# GitHub에서 이전 코드와 현재 코드 비교
git log --oneline -5          # 최근 5개 commit 확인
git diff HEAD~1 HEAD          # 최근 변경 내용 확인
```

무엇이 바뀌었는지 파악하고, 그 변경이 오류와 연관이 있는지 판단한다.

---

## 6. 롤백 방법

### 방법 A. Lambda Versions으로 롤백 (권장)

배포 전에 버전을 만들어뒀다면 가장 빠른 방법이다.

```
Lambda 함수 페이지 → [Versions] 탭
→ 배포 전에 만든 버전 클릭 (예: Version 5)
→ [Code] 탭에서 코드 확인
→ 코드 전체 복사 (Ctrl+A, Ctrl+C)
→ [Qualifiers] 드롭다운에서 [$LATEST] 선택
→ lambda_function.py 편집기에 복사한 코드 붙여넣기
→ [Deploy] 클릭
```

### 방법 B. GitHub에서 이전 코드 가져오기

```bash
# 최근 commit 목록 확인
git log --oneline

# 출력 예시:
# a1b2c3d 새 기능 추가 (방금 배포한 것)
# e4f5g6h 환경 변수 설정 추가 (이전 정상 버전)
# i7j8k9l 초기 Lambda 코드

# 특정 commit의 파일 내용 확인
git show e4f5g6h:lambda_function.py

# 특정 commit의 파일을 현재 폴더로 복원
git show e4f5g6h:lambda_function.py > lambda_function.py
```

이 코드를 Lambda 편집기에 붙여넣고 Deploy.

### 방법 C. 로컬 백업 파일 사용

배포 전에 현재 코드를 백업해두는 습관을 들이면 된다.

```bash
# 배포 전 백업
cp lambda_function.py lambda_function_backup_20260521.py

# 필요할 때 복원
cp lambda_function_backup_20260521.py lambda_function.py
```

### 롤백 후 확인

롤백 후에도 반드시 배포 후 검증 체크리스트를 다시 실행한다.

```bash
# 기본 동작 확인
curl "https://함수URL/"

# 이전 정상 동작과 같은 응답이 나오는가?
```

---

## 7. 주간 운영 점검 항목

서비스를 운영하면서 매주 한 번 확인하는 목록이다.

```
□ CloudWatch Logs에 반복되는 오류가 없는가
  (같은 오류 패턴이 계속 나온다면 근본 원인 수정 필요)

□ Lambda Monitor의 에러율이 1% 이하인가
  (Monitor 탭 → Errors / Invocations)

□ Duration이 타임아웃의 50% 이상을 사용하는 케이스가 없는가
  (기본 타임아웃 3초라면, Duration이 1500ms 넘는 케이스가 있는가)

□ 불필요한 Lambda 호출이 없는가
  (예상 외로 Invocations가 많다면 잘못된 반복 호출 확인)

□ IAM 권한이 여전히 최소 수준인가
  (테스트 중 추가한 권한이 있다면 제거)

□ 환경 변수에 불필요한 값이 없는가
  (삭제한 기능의 환경 변수가 남아 있지 않은가)

□ Lambda 메모리 설정이 적절한가
  (Max Memory Used가 Memory Size의 80% 이상이면 증설 검토)
```

---

## 8. 메모 서비스 배포 런북 템플릿

실제 서비스에서 쓸 수 있는 런북 예시다. 이 형식으로 자신의 서비스 런북을 만들면 된다.

```markdown
# memo-service 배포 런북

최종 업데이트: 2026-05-21
담당자: (이름)

## 서비스 정보
- 함수명: memo-service
- 리전: ap-northeast-2 (서울)
- Runtime: Python 3.12
- 메모리: 128MB, 타임아웃: 10초
- Function URL: https://xxxxx.lambda-url.ap-northeast-2.on.aws

## 환경 변수 목록
| Key | 설명 | 기본값 |
|-----|------|--------|
| MAX_MEMOS | 최대 메모 개수 | 100 |
| SERVICE_NAME | 서비스 표시 이름 | 메모 서비스 |

## 필요한 IAM 권한
- AWSLambdaBasicExecutionRole (기본 포함)
- (DynamoDB를 사용한다면) dynamodb:GetItem, PutItem, DeleteItem on memo-table

## 배포 전 체크리스트
- [ ] 로컬 테스트 통과 확인
- [ ] 코드에 비밀값 하드코딩 없음
- [ ] 환경 변수 목록 확인
- [ ] GitHub에 최신 코드 push 완료
- [ ] 배포 전 Lambda 버전 게시 (Publish new version)

## 배포 절차
1. Lambda 함수 페이지 → Code 탭 → lambda_function.py 편집기 열기
2. 기존 코드 전체 선택 → GitHub의 최신 코드 붙여넣기
3. Deploy 클릭 → "Changes deployed" 확인

## 배포 후 검증
1. curl "https://xxxxx.../memos" → 200 OK 확인
2. curl -X POST "https://xxxxx.../memos?text=배포테스트" → 메모 추가 확인
3. CloudWatch Logs 확인 → [ERROR] 없음 확인
4. Monitor 탭 → Errors = 0 확인

## 롤백 절차
1. Lambda → Versions 탭 → 배포 전 버전 코드 복사
2. Code 탭 → lambda_function.py에 붙여넣기 → Deploy
3. 배포 후 검증 재실행

## 주요 연락처
- GitHub 저장소: https://github.com/username/memo-service
- CloudWatch Logs: /aws/lambda/memo-service
```

---

## 따라 하기 실습

### 실습 1. 체크리스트 따라 정상 배포 연습

**목표**: 배포 전 체크리스트를 따르면서 실제 배포를 수행하고, 배포 후 검증까지 완료한다.

**Step 1. 배포할 코드 준비**

```python
import json
import os

def lambda_handler(event, context):
    print("[INFO] === 배포 테스트 v2 ===")

    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    version = os.environ.get("APP_VERSION", "1.0.0")

    result = {
        "message": f"안녕하세요, {name}님!",
        "version": version
    }

    print(f"[INFO] 응답: {result}")
    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

**Step 2. 배포 전 체크리스트 실행**

아래 항목을 하나씩 확인하고 실제로 체크한다.

```
□ 코드에 API 키나 비밀번호가 없는가 → 확인 완료
□ 환경 변수 APP_VERSION이 설정되어 있는가
  → Configuration → Environment variables에서 APP_VERSION = 2.0.0 추가
□ Lambda 타임아웃이 충분한가 → 기본 3초, 이 코드는 충분함
```

**Step 3. 배포 전 버전 게시**

```
Lambda → [Actions] → [Publish new version]
→ Description: "배포 전 스냅샷 2026-05-21"
→ [Publish]
```

**Step 4. 코드 Deploy**

편집기에 위 코드 붙여넣기 → [Deploy]

**Step 5. 배포 후 검증**

```bash
# 기본 호출
curl "https://함수URL/"
# 예상 응답: {"message": "안녕하세요, World님!", "version": "2.0.0"}

# name 파라미터
curl "https://함수URL/?name=철수"
# 예상 응답: {"message": "안녕하세요, 철수님!", "version": "2.0.0"}
```

CloudWatch Logs에서 `[INFO] === 배포 테스트 v2 ===` 로그 확인.

---

### 실습 2. 의도적인 오류 배포 → 감지 → 롤백 연습

**목표**: 오류가 있는 코드를 배포했을 때 어떻게 감지하고 롤백하는지 전체 흐름을 연습한다.

**Step 1. 오류 코드 배포**

```python
import json

def lambda_handler(event, context):
    print("[INFO] 오류 테스트 코드")

    # 의도적 오류: Lambda에서 발생하는 RuntimeError
    raise RuntimeError("이것은 롤백 테스트를 위한 의도적인 오류입니다")

    return {
        "statusCode": 200,
        "body": json.dumps({"message": "이 줄은 실행되지 않음"})
    }
```

Deploy 클릭.

**Step 2. 오류 확인**

```bash
curl "https://함수URL/"
# 응답: {"message": "Internal Server Error"} + HTTP 500
```

**Step 3. CloudWatch에서 오류 로그 확인**

```
Lambda → Monitor → View CloudWatch logs → 최신 Log stream
```

다음 로그를 찾는다:
```
[INFO] 오류 테스트 코드
[ERROR] RuntimeError: 이것은 롤백 테스트를 위한 의도적인 오류입니다
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 6, in lambda_handler
    raise RuntimeError("이것은 롤백 테스트를 위한 의도적인 오류입니다")
RuntimeError: 이것은 롤백 테스트를 위한 의도적인 오류입니다
```

**Step 4. Versions 탭에서 이전 버전 코드 복원**

```
Lambda → [Versions] 탭
→ 실습 1에서 만든 버전 클릭
→ [Code] 탭 → lambda_function.py 코드 전체 복사
→ [Qualifiers] 드롭다운 → [$LATEST] 선택
→ lambda_function.py 편집기에 복사한 코드 붙여넣기
→ [Deploy]
```

**Step 5. 롤백 후 검증**

```bash
curl "https://함수URL/?name=철수"
# 정상 응답: {"message": "안녕하세요, 철수님!", "version": "2.0.0"}
```

성공하면 롤백이 완료된 것이다.

---

### 실습 3. 나만의 런북 작성하기

**목표**: 위 템플릿을 참고해서 자신의 함수에 맞는 런북을 작성한다.

프로젝트 폴더에 `RUNBOOK.md` 파일을 만들고 다음 항목을 채운다.

```
1. 서비스 정보 (함수명, 리전, URL)
2. 환경 변수 목록 (현재 설정된 것들)
3. 배포 전 체크리스트 (나의 상황에 맞게 수정)
4. 배포 절차 (단계별로)
5. 배포 후 검증 방법 (실제 curl 명령어 포함)
6. 롤백 절차
```

작성 후 git에 커밋:

```bash
git add RUNBOOK.md
git commit -m "docs: memo-service 배포 런북 추가"
git push
```

---

## 자주 막히는 지점

### 막히는 지점 1. "Deploy 버튼을 눌렀는데 적용이 안 됐어요"

**확인**: "Changes deployed" 초록색 메시지가 나타났는가?
나타나지 않았다면 Deploy 버튼을 다시 클릭한다.

또는 페이지를 새로 고침 후 다시 시도한다.

### 막히는 지점 2. "Versions 탭이 없어요"

Versions 탭은 Lambda 함수 페이지 상단 탭 목록에 있다:
```
[Code] [Test] [Monitor] [Configuration] [Aliases] [Versions]
```

탭이 많으면 오른쪽으로 스크롤하거나 화면을 넓혀야 보인다.

### 막히는 지점 3. "Publish new version이 회색으로 비활성화되어 있어요"

코드 변경 후 Deploy를 하지 않았거나, 이미 $LATEST가 마지막 게시 버전과 동일한 경우다.

코드에 아무 변경(스페이스 추가 등)을 하고 Deploy한 뒤 다시 시도한다.

### 막히는 지점 4. "롤백 후에도 오류가 계속 나요"

롤백 후 URL을 호출했을 때 브라우저 또는 터미널이 캐시된 응답을 보여줄 수 있다. 다음을 시도한다:
- 터미널에서 curl 재실행
- 브라우저 강제 새로 고침 (Ctrl+Shift+R / Cmd+Shift+R)
- 다른 브라우저에서 접속

여전히 오류라면 CloudWatch Logs에서 롤백 후의 최신 로그를 다시 확인한다.

---

## 자주 하는 실수

| 실수 | 결과 | 예방법 |
|------|------|--------|
| Deploy를 빼먹음 | 코드가 변경되지 않은 상태로 서비스됨 | 항상 "Changes deployed" 메시지 확인 |
| 배포 전 버전 게시 안 함 | 롤백할 버전이 없음 | 배포 전 Publish new version 습관화 |
| 배포 후 테스트 안 함 | 오류를 사용자가 먼저 발견 | 배포 직후 즉시 curl로 테스트 |
| 환경 변수 확인 빠뜨림 | 새 환경 변수 미설정으로 500 오류 | 배포 전 체크리스트에 환경 변수 항목 포함 |
| 오류 발생 시 기다림 | 다운타임 연장 | 문제 있으면 즉시 롤백 결정 |

---

## 확인 체크리스트

- [ ] 배포 전 체크리스트를 모두 확인하고 배포할 수 있는가
- [ ] Deploy 후 curl로 기본 동작을 즉시 테스트하는가
- [ ] CloudWatch Logs에서 오류 여부를 확인하는가
- [ ] Lambda Versions 탭에서 이전 버전으로 롤백하는 방법을 설명할 수 있는가
- [ ] 나만의 런북을 RUNBOOK.md 파일로 작성했는가

---

## 한 번 더 생각해 보기

1. 런북 없이 배포하다가 문제가 생기면 어떤 일이 일어날까?
2. 배포 후 몇 분이 지나서 오류가 발생했다면 배포가 원인일 수도 있을까?
3. 팀원이 내 서비스를 대신 배포해야 한다면 런북에 무엇이 더 있어야 할까?

---

## 다음 장

이것으로 AWS Lambda 기초 배포 파트가 마무리된다. 지금까지 배운 것:
- Lambda 함수 만들기와 Function URL
- 로깅과 CloudWatch 모니터링
- 환경 변수와 보안
- 배포 런북과 운영 체크리스트

다음은 실제 서비스를 확장하는 방법(DynamoDB 연동, API Gateway 등)을 배운다.

---

## 참고 자료

- AWS Lambda 버전 관리 — https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html
- AWS Lambda 모니터링 — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html
- Lambda 배포 패키지 — https://docs.aws.amazon.com/lambda/latest/dg/python-package.html
