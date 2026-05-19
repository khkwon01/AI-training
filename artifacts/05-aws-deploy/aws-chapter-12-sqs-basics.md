## 이 장에서 배우는 것

- AWS SQS가 무엇인지, 왜 메시지 큐가 필요한지 이해한다
- SQS 큐를 직접 생성하고 설정하는 방법을 익힌다
- Python boto3로 메시지를 보내고 받는 코드를 작성한다
- 표준 큐(Standard Queue)와 FIFO 큐의 차이를 구분한다
- 메시지 처리 후 큐에서 삭제하는 흐름을 완성한다

---

## 먼저 쉬운 설명

카페를 상상해 보세요. 손님이 주문을 하면 직원이 즉시 커피를 만들지 않고 **주문 메모지**를 중간 통에 넣습니다. 바리스타는 자기 속도에 맞춰 그 메모지를 하나씩 꺼내서 커피를 만듭니다.

이게 바로 **메시지 큐(Message Queue)** 입니다.

서비스 A가 서비스 B에게 "이 일 좀 해줘"라고 말할 때, 서비스 B가 바쁘거나 잠깐 꺼져 있어도 메시지를 잃지 않고 안전하게 전달할 수 있습니다. AWS SQS(Simple Queue Service)는 이 역할을 클라우드에서 대신 해주는 서비스입니다.

**언제 SQS를 쓸까요?**
- 이미지 업로드 → 썸네일 자동 생성
- 주문 접수 → 결제 처리 → 배송 알림 순차 처리
- 트래픽이 갑자기 몰렸을 때 시스템이 터지지 않도록 완충

---

## 1. SQS 핵심 개념 이해하기

SQS를 사용하기 전에 반드시 알아야 할 단어들이 있습니다.

| 용어 | 설명 | 비유 |
|------|------|------|
| **Queue** | 메시지를 임시로 보관하는 공간 | 주문 대기 통 |
| **Message** | 큐에 넣는 데이터 | 주문 메모지 |
| **Producer** | 메시지를 보내는 쪽 | 주문 받는 직원 |
| **Consumer** | 메시지를 꺼내 처리하는 쪽 | 바리스타 |
| **Visibility Timeout** | 메시지를 꺼낸 후 처리 완료 전까지 다른 Consumer가 볼 수 없는 시간 | 바리스타가 메모지를 들고 있는 동안 |
| **Dead Letter Queue** | 처리 실패한 메시지가 이동하는 별도 큐 | 문제 주문 보관함 |

### 표준 큐 vs FIFO 큐

```
표준 큐 (Standard Queue)
  - 초당 거의 무제한 메시지 처리
  - 순서 보장 안 됨 (대부분 순서대로지만 가끔 뒤바뀜)
  - 중복 전달 가능성 있음
  - 사용 예: 이메일 발송, 로그 수집

FIFO 큐 (First-In-First-Out)
  - 초당 최대 300 메시지 (배치 처리 시 3,000)
  - 순서 완벽 보장
  - 중복 없음
  - 이름이 반드시 '.fifo'로 끝나야 함
  - 사용 예: 결제 처리, 재고 차감
```

---

## 2. 환경 설정하기

### AWS CLI 및 boto3 설치

```bash
# boto3 설치
pip install boto3

# AWS 자격증명 설정 (처음 한 번만)
aws configure
```

```
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

### Python에서 SQS 클라이언트 만들기

```python
# sqs_client.py
import boto3

# SQS 클라이언트 생성
sqs = boto3.client('sqs', region_name='ap-northeast-2')

# 현재 계정의 모든 큐 목록 확인
response = sqs.list_queues()
queues = response.get('QueueUrls', [])

if queues:
    print("현재 있는 큐 목록:")
    for queue in queues:
        print(f"  - {queue}")
else:
    print("아직 큐가 없습니다.")
```

---

## 3. SQS 큐 생성하기

### 표준 큐 생성

```python
# create_queue.py
import boto3
import json

sqs = boto3.client('sqs', region_name='ap-northeast-2')

# 표준 큐 생성
response = sqs.create_queue(
    QueueName='my-order-queue',
    Attributes={
        # 메시지 최대 보관 기간 (초 단위, 기본 4일, 최대 14일)
        'MessageRetentionPeriod': '86400',  # 1일
        # 메시지를 꺼낸 후 다시 보이기까지의 시간 (초)
        'VisibilityTimeout': '30',
        # 메시지 최대 크기 (바이트, 최대 256KB)
        'MaximumMessageSize': '262144',
    }
)

queue_url = response['QueueUrl']
print(f"큐가 생성되었습니다: {queue_url}")
```

### FIFO 큐 생성

```python
# create_fifo_queue.py
import boto3

sqs = boto3.client('sqs', region_name='ap-northeast-2')

# FIFO 큐 이름은 반드시 '.fifo'로 끝나야 합니다
response = sqs.create_queue(
    QueueName='my-payment-queue.fifo',
    Attributes={
        'FifoQueue': 'true',
        # 동일한 내용의 메시지 중복 방지 (5분 내)
        'ContentBasedDeduplication': 'true',
        'VisibilityTimeout': '60',
    }
)

print(f"FIFO 큐 생성: {response['QueueUrl']}")
```

---

## 4. 메시지 보내기 (Producer)

### 단일 메시지 전송

```python
# send_message.py
import boto3
import json
from datetime import datetime

sqs = boto3.client('sqs', region_name='ap-northeast-2')

QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/123456789012/my-order-queue'

def send_order(order_data: dict) -> str:
    """주문 데이터를 SQS에 전송합니다."""
    response = sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(order_data, ensure_ascii=False),
        # 메시지에 추가 속성 붙이기 (필터링에 활용)
        MessageAttributes={
            'OrderType': {
                'DataType': 'String',
                'StringValue': order_data.get('type', 'normal')
            },
            'Priority': {
                'DataType': 'Number',
                'StringValue': str(order_data.get('priority', 1))
            }
        }
    )
    message_id = response['MessageId']
    print(f"메시지 전송 완료. ID: {message_id}")
    return message_id

# 실제 사용 예시
order = {
    'order_id': 'ORD-20260518-001',
    'customer_id': 'user-42',
    'items': [
        {'product': '아메리카노', 'qty': 2, 'price': 4500},
        {'product': '크루아상', 'qty': 1, 'price': 3200}
    ],
    'total': 12200,
    'type': 'cafe',
    'priority': 2,
    'created_at': datetime.now().isoformat()
}

send_order(order)
```

### 여러 메시지 한꺼번에 보내기 (배치)

```python
# send_batch_messages.py
import boto3
import json

sqs = boto3.client('sqs', region_name='ap-northeast-2')

QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/123456789012/my-order-queue'

def send_orders_batch(orders: list) -> None:
    """최대 10개의 주문을 한꺼번에 전송합니다."""
    # SQS 배치는 한 번에 최대 10개까지만 가능
    entries = [
        {
            'Id': str(i),  # 배치 내 고유 ID (숫자 or 문자열)
            'MessageBody': json.dumps(order, ensure_ascii=False)
        }
        for i, order in enumerate(orders[:10])
    ]

    response = sqs.send_message_batch(
        QueueUrl=QUEUE_URL,
        Entries=entries
    )

    성공 = len(response.get('Successful', []))
    실패 = len(response.get('Failed', []))
    print(f"전송 결과 - 성공: {성공}개, 실패: {실패}개")

    if response.get('Failed'):
        for fail in response['Failed']:
            print(f"  실패 항목 ID: {fail['Id']}, 이유: {fail['Message']}")
```

---

## 5. 메시지 받아서 처리하기 (Consumer)

### 메시지 수신 및 처리

```python
# consume_messages.py
import boto3
import json
import time

sqs = boto3.client('sqs', region_name='ap-northeast-2')

QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/123456789012/my-order-queue'

def process_order(order_data: dict) -> bool:
    """실제 주문 처리 로직. 성공 시 True, 실패 시 False를 반환합니다."""
    print(f"  주문 처리 중: {order_data['order_id']}")
    print(f"  고객: {order_data['customer_id']}")
    print(f"  총액: {order_data['total']:,}원")
    # 실제 비즈니스 로직이 여기에 들어갑니다
    return True

def consume_messages():
    """큐에서 메시지를 지속적으로 가져와 처리합니다."""
    print("메시지 수신 대기 중...")

    while True:
        # 메시지 가져오기 (Long Polling 사용 권장)
        response = sqs.receive_message(
            QueueUrl=QUEUE_URL,
            MaxNumberOfMessages=5,      # 한 번에 최대 10개
            WaitTimeSeconds=20,          # Long Polling: 메시지가 올 때까지 최대 20초 대기
            MessageAttributeNames=['All'] # 메시지 속성도 함께 가져오기
        )

        messages = response.get('Messages', [])

        if not messages:
            print("처리할 메시지 없음. 다시 대기합니다...")
            continue

        for message in messages:
            receipt_handle = message['ReceiptHandle']  # 삭제에 필요한 핸들
            body = json.loads(message['Body'])

            print(f"\n메시지 수신: {message['MessageId']}")

            # 처리 성공 여부에 따라 삭제 or 유지
            if process_order(body):
                # 처리 완료 후 반드시 삭제해야 중복 처리 방지!
                sqs.delete_message(
                    QueueUrl=QUEUE_URL,
                    ReceiptHandle=receipt_handle
                )
                print(f"  메시지 삭제 완료 ✓")
            else:
                print(f"  처리 실패. Visibility Timeout 후 재시도됩니다.")

consume_messages()
```

---

## 6. 큐 모니터링 및 관리

```python
# queue_monitor.py
import boto3

sqs = boto3.client('sqs', region_name='ap-northeast-2')

QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/123456789012/my-order-queue'

def get_queue_stats() -> dict:
    """큐의 현재 상태를 조회합니다."""
    response = sqs.get_queue_attributes(
        QueueUrl=QUEUE_URL,
        AttributeNames=[
            'ApproximateNumberOfMessages',           # 처리 대기 중인 메시지 수
            'ApproximateNumberOfMessagesNotVisible', # 처리 중인 메시지 수 (꺼내갔지만 아직 삭제 안 됨)
            'ApproximateNumberOfMessagesDelayed',    # 지연된 메시지 수
        ]
    )

    attrs = response['Attributes']
    stats = {
        '대기 중': int(attrs['ApproximateNumberOfMessages']),
        '처리 중': int(attrs['ApproximateNumberOfMessagesNotVisible']),
        '지연됨': int(attrs['ApproximateNumberOfMessagesDelayed']),
    }

    print("=== 큐 현황 ===")
    for key, value in stats.items():
        print(f"  {key}: {value}개")

    return stats

get_queue_stats()
```

---

## 따라 하기 실습

### 실습 1 — 주문 접수 시스템 만들기

`order_producer.py` 파일을 만들고 아래 코드를 완성하세요. 표준 큐를 생성한 뒤, 3개의 가상 주문 메시지를 전송합니다.

```python
# order_producer.py
import boto3
import json
from datetime import datetime

sqs = boto3.client('sqs', region_name='ap-northeast-2')

# TODO 1: 'cafe-order-queue' 이름으로 표준 큐를 생성하세요
response = sqs.create_queue(
    QueueName='cafe-order-queue',
    Attributes={
        'MessageRetentionPeriod': '3600',  # 1시간
        'VisibilityTimeout': '30',
    }
)
QUEUE_URL = response['QueueUrl']
print(f"큐 생성: {QUEUE_URL}")

# TODO 2: 아래 주문 3개를 큐에 전송하세요
orders = [
    {'order_id': 'A001', 'menu': '아메리카노', 'size': 'L', 'price': 5000},
    {'order_id': 'A002', 'menu': '카페라떼', 'size': 'M', 'price': 5500},
    {'order_id': 'A003', 'menu': '녹차프라푸치노', 'size': 'L', 'price': 6500},
]

for order in orders:
    sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(order, ensure_ascii=False)
    )
    print(f"주문 전송: {order['order_id']} - {order['menu']}")
```

**실행 방법:**
```bash
python order_producer.py
```

**예상 출력:**
```
큐 생성: https://sqs.ap-northeast-2.amazonaws.com/...
주문 전송: A001 - 아메리카노
주문 전송: A002 - 카페라떼
주문 전송: A003 - 녹차프라푸치노
```

---

### 실습 2 — 바리스타 처리 시스템 만들기

`order_consumer.py` 파일을 만들어서 실습 1에서 넣은 메시지를 꺼내고 처리한 후 삭제하세요.

```python
# order_consumer.py
import boto3
import json
import time

sqs = boto3.client('sqs', region_name='ap-northeast-2')

# 실습 1에서 출력된 URL을 여기에 붙여넣으세요
QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/여기에_실제_URL_입력'

def make_coffee(order: dict) -> None:
    """커피 제조 시뮬레이션"""
    print(f"  ☕ {order['menu']} ({order['size']}) 제조 중...")
    time.sleep(1)  # 제조 시간 시뮬레이션
    print(f"  완료! 가격: {order['price']:,}원")

print("바리스타 시작!")

# 큐에서 모든 메시지를 처리할 때까지 반복
processed = 0
while True:
    response = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=1,
        WaitTimeSeconds=5
    )

    messages = response.get('Messages', [])
    if not messages:
        print(f"\n모든 주문 처리 완료! 총 {processed}건")
        break

    message = messages[0]
    order = json.loads(message['Body'])

    print(f"\n주문 접수: {order['order_id']}")
    make_coffee(order)

    # 처리 완료 후 큐에서 삭제
    sqs.delete_message(
        QueueUrl=QUEUE_URL,
        ReceiptHandle=message['ReceiptHandle']
    )
    processed += 1
```

**실행 방법:**
```bash
python order_consumer.py
```

---

### 실습 3 — Dead Letter Queue 연결하기

`setup_dlq.py` 파일을 만들어서 처리 실패한 메시지를 별도 큐로 보내는 구조를 설정하세요.

```python
# setup_dlq.py
import boto3
import json

sqs = boto3.client('sqs', region_name='ap-northeast-2')

# 1단계: Dead Letter Queue 먼저 생성
dlq_response = sqs.create_queue(QueueName='cafe-order-dlq')
dlq_url = dlq_response['QueueUrl']

# DLQ의 ARN(고유 주소)을 가져옵니다
dlq_attrs = sqs.get_queue_attributes(
    QueueUrl=dlq_url,
    AttributeNames=['QueueArn']
)
dlq_arn = dlq_attrs['Attributes']['QueueArn']
print(f"DLQ ARN: {dlq_arn}")

# 2단계: 메인 큐에 DLQ 연결 설정 업데이트
# maxReceiveCount: 몇 번 실패하면 DLQ로 보낼지
redrive_policy = {
    'deadLetterTargetArn': dlq_arn,
    'maxReceiveCount': '3'  # 3번 실패 시 DLQ로 이동
}

sqs.set_queue_attributes(
    QueueUrl='https://sqs.ap-northeast-2.amazonaws.com/여기에_실제_URL_입력/cafe-order-queue',
    Attributes={
        'RedrivePolicy': json.dumps(redrive_policy)
    }
)

print("DLQ 연결 완료! 3번 처리 실패한 메시지는 cafe-order-dlq로 이동합니다.")
```

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 원인 | 해결 방법 |
|------|-----------------|------|-----------|
| 메시지 처리 후 삭제 안 함 | 메시지가 계속 반복 수신됨 | `delete_message()` 호출 누락 | 처리 성공 후 반드시 `delete_message()` 호출 |
| FIFO 큐 이름 규칙 위반 | `InvalidParameterValue: ... must end with .fifo` | 큐 이름이 `.fifo`로 끝나지 않음 | 큐 이름을 `my-queue.fifo`로 변경 |
| 리전 불일치 | `EndpointResolutionError` | 클라이언트 리전과 큐 URL 리전이 다름 | `boto3.client('sqs', region_name='ap-northeast-2')` 명시 |
| AWS 자격증명 없음 | `NoCredentialsError: Unable to locate credentials` | `aws configure` 미실행 또는 환경변수 미설정 | `aws configure` 실행 또는 IAM 역할 확인 |
| 권한 부족 | `AccessDenied: User is not authorized to perform: sqs:SendMessage` | IAM 정책에 SQS 권한 없음 | IAM 사용자/역할에 `AmazonSQSFullAccess` 또는 최소 권한 정책 추가 |
| 메시지 크기 초과 | `InvalidParameterValue: Message too large` | 메시지 본문이 256KB 초과 | 큰 데이터는 S3에 저장 후 SQS엔 S3 경로만 전송 |
| Short Polling 남용 | 비용 과다, CPU 낭비 | `WaitTimeSeconds=0` (기본값) 사용 | `WaitTimeSeconds=20` 으로 Long Polling 설정 |
| ReceiptHandle 혼동 | `ReceiptHandleIsInvalid` | MessageId로 삭제 시도 | `message['ReceiptHandle']` 사용 (MessageId 아님) |

---

## 확인 체크리스트

- [ ] SQS가 무엇인지, 왜 서비스 간 직접 호출 대신 큐를 쓰는지 설명할 수 있다
- [ ] 표준 큐와 FIFO 큐의 차이를 알고 상황에 맞게 선택할 수 있다
- [ ] `boto3`로 SQS 클라이언트를 생성하고 큐를 만들 수 있다
- [ ] `send_message()`로 메시지를 전송하고, `MessageBody`에 JSON을 넣는 방법을 안다
- [ ] `receive_message()`로 메시지를 수신하고, `WaitTimeSeconds`의 의미를 안다
- [ ] 메시지 처리 후 `delete_message()`로 삭제해야 하는 이유를 설명할 수 있다
- [ ] `ReceiptHandle`이 무엇이고, `MessageId`와 어떻게 다른지 안다
- [ ] Dead Letter Queue(DLQ)의 개념과 언제 쓰는지 설명할 수 있다
- [ ] `NoCredentialsError`가 왜 발생하는지 알고 해결할 수 있다
- [ ] 실습 3개를 순서대로 실행해서 메시지가 정상 처리됨을 확인했다

---

## 한 번 더 생각해 보기

1. **만약 Consumer가 메시지를 꺼내고 처리하던 중에 서버가 갑자기 꺼진다면 어떻게 될까요?** `VisibilityTimeout`이 이 문제를 어떻게 해결하는지 생각해 보세요. 처리 시간이 30초 넘게 걸리는 작업이라면 어떻게 설정해야 할까요?

2. **한 큐를 여러 Consumer가 동시에 처리하면 어떻게 될까요?** 예를 들어 바리스타 5명이 동시에 같은 주문 큐를 보고 있다면, 같은 주문을 두 명이 동시에 만들 수 있을까요? SQS는 이 문제를 어떻게 처리하나요?

3. **메시지 처리에 계속 실패하는 주문이 생긴다면 어떻게 해야 할까요?** Dead Letter Queue에 쌓인 메시지를 어떻게 확인하고, 문제를 수정한 뒤 재처리할 수 있을지 방법을 생각해 보세요.

---

## 다음 장

다음 장에서는 SQS와 AWS Lambda를 연결해서 메시지가 도착하면 서버 없이 자동으로 코드가 실행되는 **이벤트 드리븐 아키텍처**를 구축하는 방법을 배웁니다.