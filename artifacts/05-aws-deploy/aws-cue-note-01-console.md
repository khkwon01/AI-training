## 이 장에서 배우는 것

- AWS 콘솔에서 Lambda 함수를 직접 테스트하는 방법
- 실행 결과와 로그를 확인하는 위치
- CloudWatch Logs에서 로그를 찾아가는 경로
- 테스트 이벤트를 만들고 저장하는 방법

---

## 먼저 쉬운 설명

코드를 배포했는데 제대로 동작하는지 어떻게 확인할까요?

AWS 콘솔에는 코드를 직접 실행해보고 결과를 즉시 확인할 수 있는 버튼들이 있습니다. 어디를 눌러야 할지만 알면, 복잡한 도구 없이도 내 함수가 잘 돌아가는지 바로 확인할 수 있습니다.

이 장은 그 위치를 짧게 기억하기 위한 **cue note(빠른 참고 메모)** 입니다.

---

## 1. Test — 함수 직접 실행해보기

Lambda 콘솔에서 함수를 열면 상단 탭에 **Test** 버튼이 있습니다.

```
AWS 콘솔 경로:
Lambda → [내 함수 이름] → Test 탭
```

테스트 이벤트는 JSON 형식으로 작성합니다. 아래는 이름을 전달하는 간단한 예시입니다.

```json
{
  "name": "철수",
  "age": 10
}
```

Lambda 함수 코드 예시:

```python
def lambda_handler(event, context):
    name = event.get("name", "이름 없음")
    return {
        "statusCode": 200,
        "body": f"안녕하세요, {name}님!"
    }
```

테스트를 실행하면 화면 아래에 **Execution result** 박스가 나타나고, 함수의 리턴값을 바로 확인할 수 있습니다.

> **기억 포인트:** `Test` 버튼은 함수 페이지 상단, 코드 편집기 바로 위에 있습니다.

---

## 2. Monitor — 실행 통계 한눈에 보기

**Monitor** 탭은 함수가 몇 번 실행되었는지, 오류는 몇 번 났는지 그래프로 보여줍니다.

```
AWS 콘솔 경로:
Lambda → [내 함수 이름] → Monitor 탭
```

탭을 열면 아래 항목들이 차트로 나타납니다:

```
Invocations   : 함수가 호출된 횟수
Errors        : 오류가 발생한 횟수
Duration      : 실행 시간 (밀리초)
Throttles     : 동시 실행 한도 초과 횟수
```

코드에서 오류가 나면 **Errors** 그래프에 빨간 막대가 올라갑니다. 이 그래프를 보면 "언제 문제가 생겼는지"를 빠르게 파악할 수 있습니다.

> **기억 포인트:** `Monitor` 탭은 숫자와 그래프로 함수 상태를 보여주는 대시보드입니다.

---

## 3. View CloudWatch Logs — 로그 직접 읽기

실제 `print()` 출력이나 오류 메시지 전체 내용을 보려면 CloudWatch Logs로 이동해야 합니다.

```
AWS 콘솔 경로 (방법 1 — Monitor 탭에서 바로 이동):
Lambda → [내 함수 이름] → Monitor 탭 → "View CloudWatch logs" 버튼 클릭

AWS 콘솔 경로 (방법 2 — CloudWatch 직접 이동):
CloudWatch → Log groups → /aws/lambda/[내 함수 이름]
```

로그 그룹 안에는 **Log streams** 목록이 있습니다. 가장 최근 항목을 클릭하면 실행 기록이 줄 단위로 나타납니다.

```
# Lambda 로그 예시 출력

START RequestId: a1b2c3d4 ...
이름: 철수
END RequestId: a1b2c3d4
REPORT RequestId: a1b2c3d4  Duration: 3.21 ms  Billed Duration: 4 ms
```

Python 코드에서 `print()`를 쓰면 이 로그에 그대로 기록됩니다:

```python
def lambda_handler(event, context):
    name = event.get("name", "이름 없음")
    print(f"이름: {name}")          # 이 줄이 CloudWatch에 기록됨
    return {"statusCode": 200, "body": f"안녕하세요, {name}님!"}
```

> **기억 포인트:** `View CloudWatch logs`는 Monitor 탭 안에 있는 파란 링크 버튼입니다.

---

## 빠른 위치 cue note

```
┌─────────────────────────────────────────────────────┐
│              Lambda 함수 콘솔 빠른 참고              │
├──────────────────┬──────────────────────────────────┤
│ 하고 싶은 것     │ 어디를 누르나요?                 │
├──────────────────┼──────────────────────────────────┤
│ 함수 직접 실행   │ 상단 탭 > Test                   │
│ 실행 통계 보기   │ 상단 탭 > Monitor                │
│ print 로그 보기  │ Monitor > View CloudWatch logs   │
│ 로그 직접 찾기   │ CloudWatch > Log groups          │
│                  │   > /aws/lambda/[함수이름]       │
└──────────────────┴──────────────────────────────────┘
```

---

## 따라 하기 실습

### 실습 1 — 테스트 이벤트 만들고 실행하기

1. AWS 콘솔에서 Lambda를 열고, 미리 만들어둔 함수 `hello-beginner`를 클릭합니다.
2. 상단의 **Test** 탭을 클릭합니다.
3. "Create new test event"를 선택하고, 이벤트 이름을 `myFirstTest`로 입력합니다.
4. JSON 내용을 아래처럼 입력하고 **Save** 후 **Test** 버튼을 누릅니다.

```json
{
  "name": "영희",
  "message": "안녕!"
}
```

5. 화면 아래 **Execution result: succeeded** 메시지와 함수의 응답값이 보이면 성공입니다.

---

### 실습 2 — Monitor 탭에서 실행 기록 확인하기

1. 실습 1에서 테스트를 실행한 함수 `hello-beginner` 페이지에서 **Monitor** 탭을 클릭합니다.
2. **Invocations** 그래프에 방금 실행한 기록이 표시되는지 확인합니다.
3. **Errors** 항목이 0인지 확인합니다. 빨간 막대가 없으면 정상입니다.

> 그래프가 바로 안 보일 수 있습니다. 1~2분 후 새로고침하면 반영됩니다.

---

### 실습 3 — CloudWatch Logs에서 print 출력 찾기

1. **Monitor** 탭에서 **View CloudWatch logs** 버튼을 클릭합니다.
2. Log streams 목록에서 가장 위에 있는 항목(가장 최근 실행)을 클릭합니다.
3. 로그 줄 중에서 실습 1에서 `print()`로 출력한 내용이 보이는지 찾아봅니다.

```
# 이런 줄을 찾으면 성공!
이름: 영희
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| 테스트 JSON 형식이 틀림 | `Invalid JSON` 또는 파란 저장 버튼이 비활성화 | JSON 문법 확인: 키는 `"쌍따옴표"`, 마지막 항목에 쉼표 없어야 함 |
| 로그가 CloudWatch에 없음 | Log streams 목록이 비어 있음 | 함수에 CloudWatch Logs 권한이 없는 것 — IAM 역할에 `AWSLambdaBasicExecutionRole` 정책 추가 |
| Monitor 그래프가 안 보임 | 그래프가 비어 있거나 "No data" | 테스트 실행 후 1~2분 대기 후 새로고침 |
| 테스트 실행 후 결과가 안 보임 | 화면이 그대로 | 페이지를 아래로 스크롤 — Execution result 박스는 코드 편집기 아래에 나타남 |
| 이전 테스트 이벤트가 없어짐 | 이벤트 목록이 비어 있음 | 테스트 이벤트는 함수에 저장됨 — 함수를 삭제하고 다시 만들면 사라짐 |

---

## 확인 체크리스트

- [ ] Lambda 콘솔에서 **Test** 탭의 위치를 말할 수 있다
- [ ] 테스트 이벤트를 JSON으로 직접 작성하고 저장할 수 있다
- [ ] 테스트 실행 후 **Execution result** 박스에서 결과를 읽을 수 있다
- [ ] **Monitor** 탭에서 Invocations와 Errors 수치를 확인할 수 있다
- [ ] **View CloudWatch logs** 버튼이 Monitor 탭 안에 있다는 것을 안다
- [ ] CloudWatch Log streams에서 가장 최근 로그를 열 수 있다
- [ ] `print()`로 출력한 내용이 CloudWatch 로그에 기록된다는 것을 안다

---

## 한 번 더 생각해 보기

1. **Test** 탭과 **Monitor** 탭의 차이는 무엇인가요? 각각 어떤 상황에서 먼저 열어보게 될까요?

2. CloudWatch Logs에 로그가 아예 없다면 가장 먼저 의심해야 할 원인은 무엇일까요?

3. 함수를 10번 실행했는데 Monitor의 Errors 그래프에 3번 빨간 막대가 표시됐습니다. 다음 단계로 무엇을 해야 할까요?

---

## 다음 장

다음 장에서는 CloudWatch Logs에서 오류 메시지를 읽고 해석하는 방법을 배웁니다.