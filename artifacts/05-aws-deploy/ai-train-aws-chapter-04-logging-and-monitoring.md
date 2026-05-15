# ai-train AWS Chapter 04: 로깅과 모니터링

## 이 장에서 배우는 것

- Lambda 함수의 로그가 어디에 저장되는지
- CloudWatch Logs에서 로그를 찾고 읽는 방법
- `print()`로 유용한 로그를 남기는 방법
- 오류가 났을 때 로그로 원인을 찾는 방법
- 기본 모니터링 지표 확인하기

---

## 먼저 쉬운 설명

코드가 내 컴퓨터에서 실행될 때는 터미널에서 오류를 바로 볼 수 있다.

하지만 Lambda는 서버에서 실행된다. 오류가 나도 터미널이 없다.

그래서 Lambda는 모든 출력과 오류를 **CloudWatch Logs**에 자동으로 저장한다.

로그를 잘 남기는 것은 배포된 코드를 관리하는 핵심 기술이다.

---

## 1. Lambda 로그가 저장되는 곳

Lambda 함수가 실행될 때마다:
- `print()` 출력 → CloudWatch Logs에 저장
- 오류 메시지 → CloudWatch Logs에 저장
- 실행 시간, 메모리 사용량 → CloudWatch Logs에 저장

### 로그 찾기 경로

```
AWS 콘솔 → Lambda → 함수 선택 → Monitor 탭 → View CloudWatch logs
```

또는:
```
AWS 콘솔 → CloudWatch → Logs → Log groups → /aws/lambda/함수이름
```

---

## 2. 로그 읽기

CloudWatch Logs에서 **Log stream** 목록이 보인다. 가장 최근 스트림을 클릭한다.

각 로그 항목의 구조:

```
START RequestId: abc-123 Version: $LATEST
실행됨: 안녕하세요, Mina님!        ← print() 출력
END RequestId: abc-123
REPORT RequestId: abc-123  Duration: 2.34 ms  Billed Duration: 3 ms  Memory Size: 128 MB  Max Memory Used: 36 MB
```

| 항목 | 의미 |
|------|------|
| `START` | 함수 실행 시작 |
| 중간 줄 | `print()` 로 남긴 로그 |
| `END` | 함수 실행 종료 |
| `REPORT` | 실행 시간, 메모리 사용량 요약 |

---

## 3. 유용한 로그 남기기

단순히 `print("실행됨")` 보다 더 유용한 정보를 남기는 패턴이다.

```python
import json

def lambda_handler(event, context):
    # 입력 데이터 로그
    print(f"[INPUT] event: {json.dumps(event)}")

    name = event.get("queryStringParameters", {}).get("name", "World")

    # 처리 과정 로그
    print(f"[PROCESS] name={name}")

    result = {"message": f"안녕하세요, {name}님!"}

    # 결과 로그
    print(f"[OUTPUT] result: {result}")

    return {
        "statusCode": 200,
        "body": json.dumps(result, ensure_ascii=False)
    }
```

로그 출력 예:
```
[INPUT] event: {"queryStringParameters": {"name": "Mina"}}
[PROCESS] name=Mina
[OUTPUT] result: {'message': '안녕하세요, Mina님!'}
```

---

## 4. 오류 로그 읽기

오류가 발생하면 로그에 traceback이 남는다.

```
START RequestId: abc-123
[ERROR] KeyError: 'name'
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 5, in lambda_handler
    name = event["name"]  ← 오류 위치
KeyError: 'name'
END RequestId: abc-123
```

읽는 방법:
1. `[ERROR]` 줄: 오류 종류와 메시지
2. `Traceback` 아래: 오류가 발생한 파일과 줄 번호
3. 마지막 줄: 구체적인 오류 내용

---

## 5. try/except로 오류 로그 개선하기

```python
import json
import traceback

def lambda_handler(event, context):
    try:
        name = event.get("queryStringParameters", {}).get("name", "World")
        result = {"message": f"안녕하세요, {name}님!"}
        print(f"[SUCCESS] {result}")
        return {
            "statusCode": 200,
            "body": json.dumps(result, ensure_ascii=False)
        }
    except Exception as e:
        print(f"[ERROR] {type(e).__name__}: {e}")
        print(traceback.format_exc())
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Internal Server Error"})
        }
```

오류가 나도 500 응답을 반환하고, 로그에 상세 오류 정보를 남긴다.

---

## 6. 기본 모니터링 지표 확인

Lambda 함수 페이지 → **Monitor** 탭에서 그래프로 확인할 수 있다.

| 지표 | 의미 | 주의 기준 |
|------|------|----------|
| Invocations | 실행 횟수 | 비정상적으로 많으면 확인 |
| Duration | 실행 시간 (ms) | 설정한 타임아웃에 가까워지면 최적화 필요 |
| Errors | 오류 횟수 | 0이 아니면 로그 확인 |
| Throttles | 동시 실행 제한에 걸린 횟수 | 자주 발생하면 동시 실행 한도 증가 고려 |

---

## 7. 따라 하기 실습

### 실습 1. 로그 확인하기

1. `hello-python` 함수의 URL로 몇 번 호출
2. Monitor 탭 → View CloudWatch logs
3. 최근 Log stream 클릭
4. `print()` 로 남긴 로그와 REPORT 줄 확인

### 실습 2. 로그 패턴 추가하기

함수에 `[INPUT]`, `[PROCESS]`, `[OUTPUT]` 로그를 추가하고 Deploy.

URL을 다시 호출해서 CloudWatch에서 구조화된 로그가 보이는지 확인.

### 실습 3. 의도적으로 오류 만들고 로그 확인하기

```python
def lambda_handler(event, context):
    name = event["name"]   # queryStringParameters 없이 직접 접근 → KeyError
    return {"statusCode": 200, "body": f"안녕, {name}"}
```

위 코드로 바꾸고 Deploy → URL 호출 → CloudWatch에서 오류 traceback 확인.

확인 후 올바른 코드로 복구.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| 로그 그룹이 없음 | CloudWatch에 로그가 안 보임 | 함수를 한 번이라도 실행해야 로그 그룹 생성 |
| 로그가 너무 많음 | 찾기 어려움 | 최근 Log stream만 확인, 필터 사용 |
| 민감 정보를 로그에 남김 | 비밀번호, API 키 등이 로그에 노출 | 민감 정보는 절대 print()로 출력하지 않음 |
| 로그 없이 디버깅 시도 | 원인 파악 어려움 | 충분한 `print()` 로그를 먼저 추가 |

---

## 확인 체크리스트

- [ ] CloudWatch Logs에서 Lambda 로그를 찾을 수 있는가
- [ ] `[INPUT]`, `[PROCESS]`, `[OUTPUT]` 패턴으로 로그를 남길 수 있는가
- [ ] 오류 로그에서 파일명, 줄 번호, 오류 내용을 읽을 수 있는가
- [ ] Monitor 탭에서 기본 지표(Invocations, Errors)를 확인할 수 있는가

---

## 한 번 더 생각해 보기

1. 로그를 너무 많이 남기면 어떤 문제가 생길까?
2. 민감한 정보(비밀번호, 카드 번호)를 로그에 남기면 왜 위험한가?
3. 오류가 났을 때 `traceback`을 로그에 남기는 것이 왜 중요한가?

---

## 다음 장

다음 장에서는 지금까지 만든 Python 서비스를 Lambda에 올려서 URL로 사용하는 전체 흐름을 실습한다.

---

## 참고 자료

- AWS CloudWatch Logs — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html
- Lambda 모니터링 — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html
