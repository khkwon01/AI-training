# ai-train Vibe Coding Chapter 04: 구현부터 배포까지 전체 워크플로우

## 이 장에서 배우는 것

- AI 코딩 → GitHub PR → AWS 배포까지 전체 흐름을 한 번에 실습하기
- 각 단계에서 AI를 어떻게 활용하는지
- 단계 사이에서 검증을 어떻게 하는지
- 실제 작업처럼 issue에서 시작해서 배포 확인까지 마치는 방법

---

## 전체 흐름

```
1. GitHub issue 생성
        ↓
2. branch 생성
        ↓
3. AI로 코드 초안 생성 (Vibe coding)
        ↓
4. 코드 검증 (repair loop + assert)
        ↓
5. commit → push → PR 생성
        ↓
6. AI/사람 코드 리뷰
        ↓
7. PR merge
        ↓
8. Lambda에 배포
        ↓
9. CloudWatch로 동작 확인
```

---

## 오늘 만들 기능

메모 서비스에 **메모 수정 기능**을 추가한다.

`PUT /memos?id=1&text=새내용` 형태로 호출하면 해당 번호 메모를 수정한다.

---

## Step 1. GitHub issue 생성

GitHub 저장소에서:

```
제목: 메모 수정 기능 추가

## 목적
저장된 메모의 내용을 수정할 수 있어야 합니다.

## 상세
- PUT /memos?id=번호&text=새내용 형태로 요청
- 존재하지 않는 번호면 오류 반환
- 성공 시 수정된 내용 확인 응답 반환

Label: enhancement
```

issue 번호 확인 (예: #3)

---

## Step 2. branch 생성

```bash
git checkout -b feature/3-edit-memo
```

---

## Step 3. AI로 코드 초안 생성

Claude 또는 Copilot에게:

```
Python Lambda 함수에 메모 수정 기능을 추가하고 싶어.

현재 구조:
- memos = [] 로 전역 리스트에 메모 저장
- lambda_handler(event, context) 가 HTTP 메서드와 경로로 라우팅

추가할 기능:
- HTTP 메서드: PUT
- 쿼리 파라미터: id (1-based 번호), text (새 내용)
- id가 범위 밖이면 {"error": "잘못된 번호"}, 400 반환
- 성공 시 {"message": "수정됨", "id": id, "new_text": text}, 200 반환

edit_memo(index, new_text) 함수와 lambda_handler에서 PUT 처리하는 부분을 만들어줘.
```

---

## Step 4. 받은 초안 검증

AI가 준 코드를 그대로 쓰기 전에 확인한다.

### 예상 초안

```python
def edit_memo(index, new_text):
    try:
        idx = int(index) - 1
        if idx < 0 or idx >= len(memos):
            return {"error": f"잘못된 번호: 1~{len(memos)} 사이로 입력"}, 400
        if not new_text or not new_text.strip():
            return {"error": "빈 내용으로 수정할 수 없습니다"}, 400
        old = memos[idx]
        memos[idx] = new_text.strip()
        return {"message": "수정됨", "id": int(index), "old": old, "new": new_text.strip()}, 200
    except (ValueError, TypeError):
        return {"error": "id는 숫자여야 합니다"}, 400
```

### 로컬 테스트

```python
# 로컬에서 함수만 테스트
memos = ["Python 공부", "GitHub 실습", "AWS 배포"]

# 정상 케이스
result, status = edit_memo(2, "GitHub PR 연습")
assert status == 200
assert memos[1] == "GitHub PR 연습"

# 범위 오류
result, status = edit_memo(10, "없는 메모")
assert status == 400

# 빈 내용
result, status = edit_memo(1, "")
assert status == 400

# 숫자가 아닌 id
result, status = edit_memo("abc", "내용")
assert status == 400

print("모든 테스트 통과!")
```

### lambda_handler에 PUT 라우팅 추가

```python
elif method == "PUT":
    index = params.get("id")
    text = params.get("text", "")
    body, status = edit_memo(index, text)
```

---

## Step 5. commit → push → PR

```bash
git add lambda_function.py
git commit -m "feature: 메모 수정 기능 추가 (#3)"
git push origin feature/3-edit-memo
```

PR 만들기:

```
제목: 메모 수정 기능 추가

## 변경 내용
- edit_memo(index, new_text) 함수 추가
- PUT /memos?id=번호&text=새내용 라우팅 추가

## 테스트
- 정상 수정: PUT /memos?id=1&text=새내용 → 200
- 범위 오류: PUT /memos?id=99 → 400
- 빈 내용: PUT /memos?id=1&text= → 400

Closes #3
```

---

## Step 6. AI 코드 리뷰 요청

PR 설명에 Claude나 Copilot에게 리뷰 요청을 붙인다.

또는 AI에게 직접:

```
아래 Python 코드를 리뷰해줘.
Lambda API에서 메모를 수정하는 함수야.

[edit_memo 코드 붙여넣기]

확인해줄 것:
1. 엣지 케이스가 모두 처리됐는가
2. 오류 응답 형식이 일관성 있는가
3. 개선할 점이 있는가
```

AI 리뷰 결과를 읽고 필요하면 수정한다.

---

## Step 7. PR merge

검토가 끝나면 GitHub에서 **Merge pull request** 클릭.

issue #3이 자동으로 closed 상태가 되는지 확인.

---

## Step 8. Lambda에 배포

merge된 코드를 Lambda에 반영한다.

```
1. AWS Lambda 콘솔 → memo-service 함수 열기
2. 코드 편집기에서 edit_memo 함수와 PUT 라우팅 추가
3. Deploy 클릭
```

또는 GitHub에서 직접 코드를 복사해서 Lambda 편집기에 붙여넣는다.

---

## Step 9. 동작 확인

### Function URL로 테스트

```bash
# 메모 추가
curl -X POST "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos?text=Python+공부"
curl -X POST "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos?text=GitHub+실습"

# 목록 확인
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos"

# 1번 메모 수정
curl -X PUT "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos?id=1&text=Python+고급+공부"

# 수정 확인
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos"
```

### CloudWatch Logs 확인

Lambda → Monitor → View CloudWatch logs 에서:
- `[INPUT]`, `[PROCESS]`, `[OUTPUT]` 로그 확인
- PUT 요청의 입력과 출력이 예상대로인지 확인

---

## 전체 흐름 체크리스트

- [ ] GitHub issue를 만들고 번호를 확인했는가
- [ ] issue 번호를 포함한 branch를 만들었는가
- [ ] AI로 초안을 받고 로컬에서 assert 테스트를 통과시켰는가
- [ ] commit 메시지에 issue 번호를 포함했는가
- [ ] PR 설명에 변경 내용, 테스트 방법, `Closes #번호`를 작성했는가
- [ ] AI 리뷰를 요청하고 결과를 확인했는가
- [ ] Lambda에 배포하고 실제 URL로 동작을 확인했는가
- [ ] CloudWatch에서 로그를 확인했는가

---

## 한 번 더 생각해 보기

1. 이 흐름에서 가장 시간이 오래 걸리는 단계는 어디인가? 어떻게 빠르게 할 수 있을까?
2. assert 테스트를 로컬에서 통과시킨 코드가 Lambda에서 실패할 수 있는 이유는 무엇인가?
3. issue → branch → PR → deploy 흐름을 따르면 팀 협업에서 어떤 장점이 있을까?
