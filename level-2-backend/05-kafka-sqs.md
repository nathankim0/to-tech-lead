# Kafka / SQS

## 1. 한 줄 요약

Kafka와 SQS는 **비동기 메시지 전달**을 위한 시스템으로, 서비스 간 결합도를 낮추고 대용량 이벤트를 안정적으로 처리한다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

#### 메시지 큐 = 택배 물류 센터

| 택배 시스템 | 메시지 큐 |
|:---|:---|
| 판매자 | Producer (메시지 생산자) |
| 물류 센터 | Message Queue (Kafka/SQS) |
| 배송 기사 | Consumer (메시지 소비자) |
| 택배 상자 | Message (이벤트/데이터) |
| 송장 번호 | Message ID / Offset |

판매자가 택배를 물류 센터에 맡기면, 배송 기사가 순서대로 배달합니다. 판매자는 배송 기사를 기다릴 필요 없이 다음 택배를 보낼 수 있죠. 이것이 **비동기 처리**입니다.

#### 동기 vs 비동기

```
동기 처리 (Synchronous)
══════════════════════════════════════════════════════════════

    주문 API  ────────────────────────────────────────────────>
       │
       │ 1. 주문 저장 (DB)                          ┐
       │ 2. 결제 처리 (PG)                          │ 사용자 대기
       │ 3. 재고 차감 (Inventory)                   │ (3초)
       │ 4. 이메일 발송 (Email)                     │
       │ 5. 푸시 알림 (FCM)                         ┘
       │
       └─────────────────────────────────────────> 응답
                                                  (총 3초 소요)


비동기 처리 (Asynchronous)
══════════════════════════════════════════════════════════════

    주문 API  ─────────────────────────────>
       │
       │ 1. 주문 저장 (DB)          ┐
       │ 2. 결제 처리 (PG)          │ 사용자 대기
       │ 3. 이벤트 발행 (Kafka)     ┘ (0.5초)
       │
       └──────────────────────────> 응답 (총 0.5초 소요)

       ※ 아래는 백그라운드에서 비동기 처리

    ┌─────────────────┐
    │   Kafka/SQS     │
    │   (Message Q)   │
    └───────┬─────────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
  재고    이메일   푸시
  차감    발송     알림
```

### 왜 알아야 할까?

대규모 서비스에서는 모든 작업을 동기로 처리할 수 없습니다.
- 푸시 알림이 **느려도 API 응답은 빨라야** 합니다
- 서버 장애 시에도 **메시지가 유실되면 안 됩니다**
- 트래픽 폭주 시 **안정적으로 부하를 분산**해야 합니다

---

## 3. 구조 다이어그램

### 메시지 큐 기본 구조

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Message Queue Architecture                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Producers                 Queue                   Consumers       │
│   ─────────                ───────                 ───────────      │
│                                                                     │
│   ┌─────────┐                                     ┌─────────┐       │
│   │ Order   │ ──┐                           ┌──── │ Email   │       │
│   │ Service │   │                           │     │ Service │       │
│   └─────────┘   │      ┌─────────────┐      │     └─────────┘       │
│                 │      │             │      │                       │
│   ┌─────────┐   ├─────>│ Message     │──────┤     ┌─────────┐       │
│   │ Payment │   │      │ Queue       │      │     │ Push    │       │
│   │ Service │ ──┤      │             │      ├──── │ Service │       │
│   └─────────┘   │      │ ┌─┐┌─┐┌─┐┌─┐│      │     └─────────┘       │
│                 │      │ │1││2││3││4││      │                       │
│   ┌─────────┐   │      │ └─┘└─┘└─┘└─┘│      │     ┌─────────┐       │
│   │ User    │ ──┘      └─────────────┘      └──── │ Analytics│      │
│   │ Service │                                     │ Service │       │
│   └─────────┘                                     └─────────┘       │
│                                                                     │
│   특징:                                                              │
│   - 생산자/소비자 분리 (Decoupling)                                  │
│   - 비동기 처리 (Async)                                              │
│   - 메시지 보장 (At-least-once)                                      │
│   - 부하 분산 (Load Balancing)                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Kafka 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Kafka Architecture                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                         Kafka Cluster                               │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                                                             │   │
│   │   Topic: order-events                                       │   │
│   │   ┌─────────────────────────────────────────────────────┐   │   │
│   │   │                                                     │   │   │
│   │   │   Partition 0    Partition 1    Partition 2         │   │   │
│   │   │   ┌───────────┐  ┌───────────┐  ┌───────────┐       │   │   │
│   │   │   │ 0 │ 1 │ 2 │  │ 0 │ 1 │ 2 │  │ 0 │ 1 │ 2 │       │   │   │
│   │   │   └───────────┘  └───────────┘  └───────────┘       │   │   │
│   │   │        │              │              │               │   │   │
│   │   │        ▼              ▼              ▼               │   │   │
│   │   │   Broker 1       Broker 2       Broker 3            │   │   │
│   │   │   (Leader)       (Leader)       (Leader)            │   │   │
│   │   │        │              │              │               │   │   │
│   │   │   Replica ───── Replica ───── Replica               │   │   │
│   │   │                                                     │   │   │
│   │   └─────────────────────────────────────────────────────┘   │   │
│   │                                                             │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│   Producer                                   Consumer Group         │
│   ┌────────────┐                            ┌────────────────────┐  │
│   │ Order API  │                            │ Consumer A         │  │
│   │            │ ──── Partition Key ─────── │ (Partition 0)      │  │
│   │ key: user1 │     (user_id 기준 분배)    ├────────────────────┤  │
│   │ key: user2 │                            │ Consumer B         │  │
│   └────────────┘                            │ (Partition 1, 2)   │  │
│                                             └────────────────────┘  │
│                                                                     │
│   핵심 개념:                                                         │
│   - Topic: 메시지 카테고리 (주문, 결제, 알림...)                      │
│   - Partition: Topic을 나눈 단위 (병렬 처리)                         │
│   - Offset: 파티션 내 메시지 순번                                    │
│   - Consumer Group: 같은 그룹은 파티션 분배하여 처리                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### AWS SQS 구조

```
┌─────────────────────────────────────────────────────────────────────┐
│                          AWS SQS Types                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Standard Queue                     FIFO Queue                     │
│   ──────────────                     ──────────                     │
│                                                                     │
│   ┌─────────────────┐               ┌─────────────────┐             │
│   │     SQS         │               │    SQS FIFO     │             │
│   │                 │               │                 │             │
│   │  ┌─┐ ┌─┐ ┌─┐   │               │  ┌─┐ ┌─┐ ┌─┐   │             │
│   │  │3│ │1│ │2│   │  ←            │  │1│ │2│ │3│   │  ← 순서대로  │
│   │  └─┘ └─┘ └─┘   │  순서 보장 X   │  └─┘ └─┘ └─┘   │              │
│   │                 │               │                 │             │
│   └─────────────────┘               └─────────────────┘             │
│                                                                     │
│   - 높은 처리량                      - 순서 보장 (FIFO)              │
│   - 최소 1회 전달                    - 정확히 1회 전달               │
│   - 순서 보장 X                      - 처리량 제한 (300 TPS)         │
│   - 무제한 TPS                       - 메시지 그룹 ID 필요           │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   SQS + Lambda 연동 (서버리스)                                       │
│   ────────────────────────────                                      │
│                                                                     │
│   ┌────────────┐     ┌────────────┐     ┌────────────┐             │
│   │  Producer  │────>│    SQS     │────>│   Lambda   │             │
│   │            │     │   Queue    │     │ (Consumer) │             │
│   └────────────┘     └────────────┘     └─────┬──────┘             │
│                                               │                     │
│                           ┌───────────────────┼───────────────┐     │
│                           ▼                   ▼               ▼     │
│                      ┌────────┐          ┌────────┐      ┌────────┐ │
│                      │   S3   │          │   DB   │      │  SNS   │ │
│                      └────────┘          └────────┘      └────────┘ │
│                                                                     │
│   - 서버 관리 불필요                                                 │
│   - 자동 스케일링                                                    │
│   - 비용 효율적 (실행 시간만 과금)                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Dead Letter Queue (DLQ)

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Dead Letter Queue (DLQ)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   처리 실패 메시지를 별도 큐로 이동                                   │
│                                                                     │
│   ┌────────────┐     ┌────────────┐     ┌────────────┐             │
│   │  Producer  │────>│ Main Queue │────>│  Consumer  │             │
│   └────────────┘     └──────┬─────┘     └─────┬──────┘             │
│                             │                 │                     │
│                             │                 │ 처리 실패            │
│                             │                 │ (3회 재시도)        │
│                             │                 ▼                     │
│                             │           ┌───────────┐               │
│                             └──────────>│    DLQ    │               │
│                                         │ (Dead     │               │
│                                         │  Letter   │               │
│                                         │  Queue)   │               │
│                                         └─────┬─────┘               │
│                                               │                     │
│                                               ▼                     │
│                                         ┌───────────┐               │
│                                         │ 모니터링  │               │
│                                         │ & 알림    │               │
│                                         └───────────┘               │
│                                                                     │
│   활용:                                                              │
│   - 실패 메시지 분석                                                 │
│   - 수동 재처리                                                      │
│   - 장애 알림                                                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 예시 1: 주문 이벤트 발행 (Kafka Producer)

```python
# Kafka Producer 예시 (Python)
from kafka import KafkaProducer
import json

KAFKA_BOOTSTRAP_SERVERS = ['kafka1:9092', 'kafka2:9092']
ORDER_TOPIC = 'order-events'

class OrderEventProducer:
    def __init__(self):
        self.producer = KafkaProducer(
            bootstrap_servers=KAFKA_BOOTSTRAP_SERVERS,
            value_serializer=lambda v: json.dumps(v).encode('utf-8'),
            key_serializer=lambda k: k.encode('utf-8'),
            acks='all',  # 모든 replica에 저장 확인
            retries=3
        )

    def publish_order_created(self, order: Order):
        """주문 생성 이벤트 발행"""
        event = {
            'event_type': 'ORDER_CREATED',
            'order_id': order.id,
            'user_id': order.user_id,
            'items': [
                {'product_id': item.product_id, 'quantity': item.quantity}
                for item in order.items
            ],
            'total_price': order.total_price,
            'timestamp': datetime.now().isoformat()
        }

        # user_id를 key로 사용 → 같은 사용자의 주문은 같은 파티션으로
        self.producer.send(
            topic=ORDER_TOPIC,
            key=str(order.user_id),
            value=event
        )
        self.producer.flush()  # 즉시 전송 보장

    def publish_order_cancelled(self, order_id: int, reason: str):
        """주문 취소 이벤트 발행"""
        event = {
            'event_type': 'ORDER_CANCELLED',
            'order_id': order_id,
            'reason': reason,
            'timestamp': datetime.now().isoformat()
        }
        self.producer.send(topic=ORDER_TOPIC, value=event)
```

### 예시 2: 이벤트 소비 (Kafka Consumer)

```python
# Kafka Consumer 예시 (Python)
from kafka import KafkaConsumer
import json

class OrderEventConsumer:
    def __init__(self, group_id: str):
        self.consumer = KafkaConsumer(
            ORDER_TOPIC,
            bootstrap_servers=KAFKA_BOOTSTRAP_SERVERS,
            group_id=group_id,  # Consumer Group
            value_deserializer=lambda v: json.loads(v.decode('utf-8')),
            auto_offset_reset='earliest',  # 처음부터 읽기
            enable_auto_commit=False  # 수동 커밋
        )

    def start_consuming(self):
        """이벤트 소비 시작"""
        for message in self.consumer:
            try:
                event = message.value
                self._handle_event(event)

                # 처리 성공 시 offset 커밋
                self.consumer.commit()

            except Exception as e:
                logger.error(f"이벤트 처리 실패: {e}")
                # DLQ로 전송하거나 재시도 로직

    def _handle_event(self, event: dict):
        event_type = event.get('event_type')

        if event_type == 'ORDER_CREATED':
            self._handle_order_created(event)
        elif event_type == 'ORDER_CANCELLED':
            self._handle_order_cancelled(event)

    def _handle_order_created(self, event: dict):
        """주문 생성 처리 - 이메일 발송"""
        order_id = event['order_id']
        user_id = event['user_id']

        # 사용자 정보 조회
        user = user_service.get_user(user_id)

        # 이메일 발송
        email_service.send_order_confirmation(
            to=user.email,
            order_id=order_id,
            items=event['items'],
            total_price=event['total_price']
        )


# 서비스별로 다른 Consumer Group 사용
# → 같은 이벤트를 여러 서비스가 각자 처리
email_consumer = OrderEventConsumer(group_id='email-service')
inventory_consumer = OrderEventConsumer(group_id='inventory-service')
analytics_consumer = OrderEventConsumer(group_id='analytics-service')
```

### 예시 3: AWS SQS 활용 (푸시 알림)

```python
# AWS SQS with boto3
import boto3
import json

PUSH_QUEUE_URL = 'https://sqs.ap-northeast-2.amazonaws.com/123456789/push-notification-queue'

sqs = boto3.client('sqs', region_name='ap-northeast-2')

class PushNotificationQueue:

    def send_push_message(self, user_id: str, title: str, body: str, data: dict = None):
        """푸시 알림 메시지 전송"""
        message = {
            'user_id': user_id,
            'title': title,
            'body': body,
            'data': data or {},
            'timestamp': datetime.now().isoformat()
        }

        response = sqs.send_message(
            QueueUrl=PUSH_QUEUE_URL,
            MessageBody=json.dumps(message),
            MessageGroupId=user_id,  # FIFO Queue용
            MessageDeduplicationId=f"{user_id}-{datetime.now().timestamp()}"
        )

        return response['MessageId']

    def receive_messages(self, max_messages: int = 10):
        """메시지 수신 (Polling)"""
        response = sqs.receive_message(
            QueueUrl=PUSH_QUEUE_URL,
            MaxNumberOfMessages=max_messages,
            WaitTimeSeconds=20,  # Long Polling (비용 절감)
            VisibilityTimeout=30  # 처리 제한 시간
        )

        messages = response.get('Messages', [])
        return messages

    def delete_message(self, receipt_handle: str):
        """처리 완료 후 메시지 삭제"""
        sqs.delete_message(
            QueueUrl=PUSH_QUEUE_URL,
            ReceiptHandle=receipt_handle
        )

    def process_messages(self):
        """메시지 처리 루프"""
        while True:
            messages = self.receive_messages()

            for message in messages:
                try:
                    body = json.loads(message['Body'])

                    # 푸시 알림 발송
                    fcm_service.send_push(
                        user_id=body['user_id'],
                        title=body['title'],
                        body=body['body'],
                        data=body['data']
                    )

                    # 성공 시 삭제
                    self.delete_message(message['ReceiptHandle'])

                except Exception as e:
                    logger.error(f"푸시 처리 실패: {e}")
                    # VisibilityTimeout 후 자동 재처리
                    # maxReceiveCount 초과 시 DLQ로 이동
```

### 예시 4: 이벤트 기반 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Event-Driven Architecture                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   주문 서비스                                                        │
│   ┌──────────────────┐                                              │
│   │  POST /orders    │                                              │
│   │  ───────────────  │                                              │
│   │  1. 주문 저장    │                                              │
│   │  2. 결제 처리    │                                              │
│   │  3. 이벤트 발행  │ ─────┐                                       │
│   └──────────────────┘      │                                       │
│                             │                                       │
│                             ▼                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                       Kafka                                 │   │
│   │                                                             │   │
│   │   Topic: order-events                                       │   │
│   │   ┌─────────────────────────────────────────────────────┐   │   │
│   │   │ ORDER_CREATED │ ORDER_PAID │ ORDER_SHIPPED │ ...    │   │   │
│   │   └─────────────────────────────────────────────────────┘   │   │
│   │                                                             │   │
│   └───────────────────────────┬─────────────────────────────────┘   │
│                               │                                     │
│         ┌─────────────────────┼─────────────────────┐              │
│         │                     │                     │              │
│         ▼                     ▼                     ▼              │
│   ┌───────────┐        ┌───────────┐        ┌───────────┐          │
│   │ 재고      │        │ 이메일    │        │ 분석      │          │
│   │ 서비스    │        │ 서비스    │        │ 서비스    │          │
│   │           │        │           │        │           │          │
│   │ 재고 차감 │        │ 확인 메일 │        │ 이벤트    │          │
│   │           │        │ 발송      │        │ 로깅      │          │
│   └───────────┘        └───────────┘        └───────────┘          │
│                                                                     │
│   장점:                                                              │
│   - 서비스 간 결합도 낮음                                            │
│   - 독립적 배포/확장 가능                                            │
│   - 장애 전파 방지                                                   │
│   - 이벤트 리플레이 가능                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. 장단점

### Kafka vs SQS 비교

| 구분 | Apache Kafka | AWS SQS |
|:---|:---|:---|
| **관리** | 직접 운영 (또는 MSK) | 완전 관리형 |
| **처리량** | 초당 수백만 메시지 | 무제한 (Standard) |
| **메시지 보관** | 설정된 기간 보관 | 최대 14일 |
| **순서 보장** | 파티션 내 보장 | FIFO만 보장 |
| **메시지 재처리** | Offset 이동으로 가능 | 어려움 |
| **복잡도** | 높음 | 낮음 |
| **비용** | 인프라 비용 | 사용량 기반 |
| **적합한 경우** | 실시간 스트리밍, 대용량 | 간단한 큐잉, 서버리스 |

### 메시지 전달 보장

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Message Delivery Guarantees                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   At-Most-Once (최대 1회)                                            │
│   ─────────────────────────                                         │
│   - 메시지 유실 가능                                                 │
│   - 중복 없음                                                        │
│   - 로그, 메트릭 등 유실 괜찮은 데이터                                │
│                                                                     │
│   At-Least-Once (최소 1회)                                           │
│   ─────────────────────────                                         │
│   - 메시지 유실 없음                                                 │
│   - 중복 가능 (Consumer가 멱등성 처리 필요)                           │
│   - 대부분의 비즈니스 이벤트                                         │
│   - Kafka 기본 설정, SQS Standard                                   │
│                                                                     │
│   Exactly-Once (정확히 1회)                                          │
│   ─────────────────────────                                         │
│   - 메시지 유실 없음                                                 │
│   - 중복 없음                                                        │
│   - 구현 복잡, 성능 저하                                             │
│   - Kafka Transactions, SQS FIFO                                    │
│   - 금융 거래 등 정확성 필수                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 사용 시나리오

```
Kafka 사용                           SQS 사용
─────────────────────────────────────────────────────────────────────

✅ 실시간 이벤트 스트리밍             ✅ 간단한 작업 큐
   (클릭스트림, 로그)                    (이메일 발송, 썸네일 생성)

✅ 이벤트 소싱                        ✅ Lambda 트리거
   (상태 변경 기록 보관)                 (서버리스 아키텍처)

✅ 여러 Consumer가 같은 데이터 필요   ✅ AWS 서비스 연동
   (분석, 검색, 알림 각각 소비)          (S3, Lambda, SNS)

✅ 순서 보장 필수                     ✅ 빠른 구축
   (같은 사용자의 이벤트 순서)           (인프라 관리 부담 X)

✅ 대용량 데이터 처리                 ✅ 비용 최적화
   (초당 수백만 이벤트)                  (사용한 만큼만 비용)
```

---

## 6. 내 생각

> 이 섹션은 학습 후 본인의 생각을 정리하는 공간입니다.

```
Q1. 현재 서비스에서 비동기로 처리할 수 있는 작업은 무엇인가?


Q2. 푸시 알림이 중복 발송되면 어떤 문제가 생길까? 어떻게 방지할까?


Q3. 메시지 큐가 장애나면 어떻게 대응해야 할까?


```

---

## 7. 추가 질문

더 깊이 학습하기 위한 질문들입니다.

### 설계 관련
- [ ] 이벤트 스키마 버전 관리는 어떻게 할까?

> **답변**: 메시지 큐의 이벤트 스키마가 변경되면 Consumer가 파싱에 실패할 수 있습니다. **버전 관리 전략**: (1) **Schema Registry 사용**: Confluent Schema Registry(Kafka)나 AWS Glue Schema Registry를 사용하면 스키마를 중앙 관리하고 호환성을 검증합니다. (2) **Wrapper 포맷**: 메시지에 `{"version": 2, "data": {...}}`처럼 버전을 명시하고 Consumer가 버전별로 처리합니다. (3) **하위 호환성 유지**: 새 필드 추가만 허용(Optional), 필드 삭제/타입 변경 금지. (4) **Topic 분리**: 큰 변경 시 `order-events-v2` 같은 새 토픽 생성. **모바일 앱 관점**: 푸시 알림 메시지 포맷이 변경되면 구버전 앱에서 크래시가 발생할 수 있습니다. 알림 페이로드도 버전 필드를 포함하거나, 앱 버전별로 다른 페이로드를 보내는 전략이 필요합니다.

- [ ] Saga 패턴과 메시지 큐의 관계는?

> **답변**: **Saga 패턴**은 마이크로서비스 간 분산 트랜잭션을 메시지 기반으로 처리하는 패턴입니다. 전통적인 2PC(Two-Phase Commit 대신 **보상 트랜잭션**을 사용합니다. **예시: 주문 프로세스**: (1) Order Service: 주문 생성 → `OrderCreated` 이벤트 발행 (2) Payment Service: 결제 처리 → 성공 시 `PaymentCompleted`, 실패 시 `PaymentFailed` (3) Inventory Service: 재고 차감 → 성공 시 `InventoryReserved`, 실패 시 `InventoryFailed` (4) 중간에 실패하면 보상 이벤트로 롤백: `OrderCancelled`, `PaymentRefunded`. **Choreography vs Orchestration**: Choreography는 각 서비스가 이벤트를 듣고 반응(Kafka 적합), Orchestration은 중앙 조정자가 순서 제어(별도 Saga Manager). **모바일 앱 관점**: 주문 API가 바로 "성공"을 반환해도 실제 처리는 진행 중일 수 있습니다. 앱에서는 주문 상태를 폴링하거나 푸시 알림으로 최종 결과를 받아야 합니다.

```
Saga 패턴 흐름 (주문 예시)
───────────────────────────────────────────────────────

성공 시나리오:
[주문 생성] → [결제 성공] → [재고 차감] → [배송 예약] → 완료

실패 시나리오 (재고 부족):
[주문 생성] → [결제 성공] → [재고 부족!]
                              ↓
              [결제 환불] ← [주문 취소]  (보상 트랜잭션)
```

- [ ] CQRS (Command Query Responsibility Segregation)란?

> **답변**: **CQRS**는 데이터의 **쓰기(Command)**와 **읽기(Query)** 모델을 분리하는 패턴입니다. **왜 필요한가**: 복잡한 도메인에서 쓰기 모델은 비즈니스 로직에 최적화되고, 읽기 모델은 조회 성능에 최적화될 수 있습니다. **구현**: 쓰기는 정규화된 RDBMS에 저장, 읽기는 비정규화된 읽기 전용 DB(또는 ElasticSearch)에서 처리. 쓰기 후 이벤트를 발행하여 읽기 모델을 동기화합니다(메시지 큐 활용). **장점**: 읽기/쓰기 독립적 확장, 복잡한 쿼리 성능 향상. **단점**: 복잡성 증가, 읽기 모델 동기화 지연(Eventually Consistent). **모바일 앱 관점**: CQRS 환경에서는 데이터 저장 직후 조회 시 최신 데이터가 안 보일 수 있습니다. "글 작성 → 목록 새로고침"에서 방금 쓴 글이 안 보이는 이유가 이것일 수 있습니다. Optimistic UI로 대응하세요.

### 운영 관련
- [ ] Kafka Consumer Lag는 무엇이고 어떻게 모니터링할까?

> **답변**: **Consumer Lag**은 Producer가 발행한 메시지 offset과 Consumer가 처리한 offset의 차이입니다. Lag이 증가하면 메시지 처리가 밀리고 있다는 의미입니다. **모니터링 방법**: (1) `kafka-consumer-groups.sh --describe` 명령어로 확인 (2) Kafka Manager, AKHQ 같은 GUI 도구 사용 (3) Prometheus + Grafana로 메트릭 수집 (4) AWS MSK는 CloudWatch 메트릭 제공. **알림 설정**: Lag이 특정 값(예: 10000) 초과 시 알림. **대응 방법**: (1) Consumer 인스턴스 수 증가 (파티션 수까지 가능) (2) Consumer 처리 로직 최적화 (3) 배치 처리 크기 조정. **모바일 앱 관점**: Consumer Lag이 크면 푸시 알림이 늦게 도착하거나, 이메일이 몇 시간 후에 발송될 수 있습니다. "주문 완료했는데 알림이 30분 후에 왔어요" 같은 문의가 들어오면 Consumer Lag을 확인하세요.

- [ ] 메시지 재처리가 필요할 때 어떻게 해야 할까?

> **답변**: 버그 수정 후 과거 메시지를 다시 처리하거나, 장애로 유실된 메시지를 복구해야 할 때 필요합니다. **Kafka 재처리**: (1) **Offset Reset**: Consumer Group의 offset을 과거로 되돌림. `kafka-consumer-groups.sh --reset-offsets --to-datetime` (2) **새 Consumer Group**: 새 그룹 ID로 처음부터 읽기 (3) **Kafka Connect**: 특정 기간 메시지를 다른 저장소에서 재발행. **SQS 재처리**: DLQ에 쌓인 메시지를 다시 원래 큐로 이동하거나, Lambda로 재처리. **주의점**: 재처리 시 Consumer가 **멱등성**을 가져야 합니다. 같은 주문 이벤트를 두 번 처리해도 주문이 두 개 생기면 안 됩니다. **모바일 앱 관점**: 메시지 재처리로 동일한 푸시 알림이 여러 번 갈 수 있습니다. 앱에서 알림 ID로 중복을 필터링하거나, 서버에서 발송 기록을 확인하여 중복 방지해야 합니다.

```python
# 멱등성 있는 Consumer 예시
class OrderEventConsumer:
    def process(self, event):
        # 처리 기록 확인 (멱등성 보장)
        if self.redis.sismember('processed_events', event['event_id']):
            logger.info(f"Already processed: {event['event_id']}")
            return

        # 실제 처리
        self._handle_event(event)

        # 처리 완료 기록 (TTL 설정으로 메모리 관리)
        self.redis.sadd('processed_events', event['event_id'])
        self.redis.expire('processed_events', 86400 * 7)  # 7일
```

- [ ] 파티션 수는 어떻게 결정해야 할까?

> **답변**: Kafka 파티션 수는 **병렬 처리 능력**을 결정합니다. 파티션 수 = 최대 Consumer 수입니다. **결정 기준**: (1) **처리량 계산**: 초당 메시지 수 / Consumer 하나의 처리량 = 최소 파티션 수. 예: 10,000 msg/s ÷ 1,000 msg/s = 10개 (2) **여유분 확보**: 향후 트래픽 증가 고려하여 2-3배 설정 (3) **키 분포**: Partition Key의 카디널리티보다 많으면 일부 파티션이 비어있음. **권장 수**: 소규모 서비스 3-6개, 중규모 12-24개, 대규모 50개 이상. **주의점**: 파티션 수는 증가만 가능하고 줄일 수 없습니다. 파티션이 너무 많으면 오버헤드가 증가하고, 너무 적으면 확장이 제한됩니다. **모바일 앱 관점**: 파티션 수가 부족하면 이벤트 처리 지연으로 앱 사용자 경험이 나빠집니다. 특히 실시간 알림, 채팅 기능에서 체감됩니다.

### 심화
- [ ] Kafka Streams와 ksqlDB는 무엇인가?

> **답변**: **Kafka Streams**는 Kafka 토픽의 데이터를 실시간으로 처리하는 스트림 처리 라이브러리입니다. 별도 클러스터 없이 Java 앱에 포함하여 사용합니다. 필터링, 집계, 조인, 윈도우 연산 등을 지원합니다. **ksqlDB**는 SQL로 스트림 처리를 할 수 있게 해주는 도구입니다. `SELECT * FROM orders WHERE amount > 100`처럼 SQL 문법으로 실시간 데이터를 처리합니다. **사용 예시**: (1) 실시간 집계: 최근 5분간 주문 금액 합계 (2) 데이터 변환: 이벤트 포맷 변경 후 다른 토픽으로 (3) 실시간 알림: 특정 조건 감지 시 알림 토픽으로 발행. **모바일 앱 관점**: 앱에서 실시간 대시보드(현재 접속자 수, 실시간 랭킹 등)를 구현할 때 백엔드에서 Kafka Streams로 집계된 데이터를 제공받습니다.

- [ ] Event Sourcing과 CQRS를 함께 사용하는 이유는?

> **답변**: **Event Sourcing**은 상태 대신 **이벤트의 시퀀스**를 저장합니다. 계좌 잔액 "10,000원"을 저장하는 대신 "입금 5,000원", "입금 3,000원", "출금 -2,000원", "입금 4,000원" 이벤트를 저장합니다. 현재 상태는 이벤트를 재생하여 계산합니다. **CQRS와의 시너지**: Event Sourcing만 쓰면 현재 상태 조회가 느립니다(모든 이벤트 재생 필요). CQRS로 읽기 모델을 분리하면 조회 성능 문제를 해결합니다. 이벤트 발행 → 읽기 모델 업데이트. **장점**: 완벽한 감사 로그, 과거 어느 시점으로든 상태 복원 가능, 버그 수정 후 이벤트 재생으로 데이터 보정. **단점**: 구현 복잡, 스토리지 증가, Eventually Consistent. **적합한 도메인**: 금융(거래 내역), 의료(진료 기록), 법무(문서 변경 이력). **모바일 앱 관점**: "거래 내역" 화면이 Event Sourcing 기반이면 매우 빠르게 조회되고 필터링됩니다. 다만 잔액 조회와 내역의 합이 일시적으로 안 맞을 수 있습니다(Eventually Consistent).

- [ ] AWS EventBridge vs SNS+SQS 차이점은?

> **답변**: **SNS+SQS**는 단순한 Pub/Sub 모델입니다. SNS가 메시지를 브로드캐스트하고 SQS가 구독하여 큐에 저장합니다. 직접적인 메시지 전달에 적합합니다. **EventBridge**는 이벤트 기반 아키텍처를 위한 서버리스 이벤트 버스입니다. 이벤트 내용 기반 라우팅(Rule), 스키마 레지스트리, 다양한 AWS 서비스 통합이 특징입니다. **선택 기준**: (1) **SNS+SQS**: 단순한 메시지 전달, 직접적인 제어 필요, 비용 민감 (2) **EventBridge**: 이벤트 기반 아키텍처, 복잡한 라우팅 규칙, SaaS 연동, 서버리스 환경. **비용**: EventBridge가 메시지당 비용이 더 높지만, 관리 오버헤드가 적습니다. **모바일 앱 관점**: 앱에서 직접 접하지는 않지만, EventBridge를 사용하면 "특정 지역 사용자에게만 알림", "VIP 사용자에게 우선 처리" 같은 복잡한 라우팅이 백엔드에서 쉽게 구현됩니다.

---

## 8. 모바일 앱에서 비동기 처리 결과 받기

서버가 비동기로 처리하는 작업의 결과를 모바일 앱에서 받는 방법들입니다.

### 방법 1: Polling (가장 단순)

```swift
// iOS - 주문 상태 폴링
class OrderStatusPoller {
    private var timer: Timer?
    private let pollingInterval: TimeInterval = 2.0
    private let maxAttempts = 30  // 최대 1분

    func startPolling(orderId: String, completion: @escaping (OrderStatus) -> Void) {
        var attempts = 0

        timer = Timer.scheduledTimer(withTimeInterval: pollingInterval, repeats: true) { [weak self] _ in
            attempts += 1

            Task {
                let status = try await API.getOrderStatus(orderId)

                if status.isFinal || attempts >= self?.maxAttempts ?? 0 {
                    self?.timer?.invalidate()
                    completion(status)
                }
            }
        }
    }
}
```

### 방법 2: WebSocket (실시간)

```kotlin
// Android - WebSocket으로 실시간 업데이트
class OrderWebSocket(private val orderId: String) {
    private var webSocket: WebSocket? = null

    fun connect(onStatusUpdate: (OrderStatus) -> Unit) {
        val client = OkHttpClient()
        val request = Request.Builder()
            .url("wss://api.example.com/orders/$orderId/status")
            .build()

        webSocket = client.newWebSocket(request, object : WebSocketListener() {
            override fun onMessage(webSocket: WebSocket, text: String) {
                val status = Json.decodeFromString<OrderStatus>(text)
                onStatusUpdate(status)

                if (status.isFinal) {
                    webSocket.close(1000, "Completed")
                }
            }

            override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
                // 재연결 로직
                reconnect()
            }
        })
    }

    fun disconnect() {
        webSocket?.close(1000, "User cancelled")
    }
}
```

### 방법 3: Push Notification (백그라운드 대응)

```
비동기 처리 결과 전달 패턴
───────────────────────────────────────────────────────

┌──────────┐     ┌──────────┐     ┌───────────┐     ┌──────────┐
│ Mobile   │────>│  API     │────>│  Kafka    │────>│ Consumer │
│ App      │     │  Server  │     │           │     │          │
└──────────┘     └──────────┘     └───────────┘     └────┬─────┘
     ▲                                                    │
     │                                                    │
     │ Push Notification                                  │
     │ (FCM/APNS)                                        │
     │                                                    ▼
     │           ┌───────────────────────────────────────────┐
     └───────────│ Push Service                             │
                 │ - 처리 완료 알림                          │
                 │ - 상태 변경 알림                          │
                 └───────────────────────────────────────────┘

앱 상태별 전달 방식:
- Foreground: WebSocket 또는 Polling
- Background: Push Notification (Silent/Visible)
- Killed: Push Notification → 앱 실행 시 최신 상태 조회
```

---

## 9. 실무에서 자주 겪는 문제와 해결책

### 문제 1: 순서가 중요한 이벤트 처리

```
문제: 같은 사용자의 이벤트가 다른 순서로 처리됨
─────────────────────────────────────────────

발생 순서: 주문생성 → 결제완료 → 주문취소
처리 순서: 주문생성 → 주문취소 → 결제완료 (!)
결과: 이미 취소된 주문에 결제가 완료된 것으로 처리

해결 방법:
1. Partition Key 사용
   - 같은 user_id의 이벤트는 같은 파티션으로
   - 파티션 내에서는 순서 보장

2. 이벤트에 시퀀스 번호 포함
   - {seq: 1, event: "CREATED"}
   - {seq: 2, event: "PAID"}
   - Consumer가 순서대로 처리

3. 이전 상태 검증
   - "결제완료" 처리 전 현재 상태가 "주문생성"인지 확인
   - 아니면 재시도 또는 DLQ로 이동
```

### 문제 2: 푸시 알림 중복 발송

```swift
// iOS - 푸시 알림 중복 방지
class PushNotificationHandler {
    private var processedNotificationIds = Set<String>()

    func handle(_ notification: UNNotification) {
        let notificationId = notification.request.identifier

        // 중복 체크
        guard !processedNotificationIds.contains(notificationId) else {
            return
        }

        processedNotificationIds.insert(notificationId)

        // 처리
        processNotification(notification)

        // 메모리 관리 (오래된 ID 제거)
        if processedNotificationIds.count > 100 {
            processedNotificationIds.removeFirst()
        }
    }
}
```

### 문제 3: 이벤트 처리 실패 시 사용자 대응

```kotlin
// Android - 처리 상태에 따른 UI 표시
sealed class OrderProcessingState {
    object Processing : OrderProcessingState()
    data class Completed(val order: Order) : OrderProcessingState()
    data class Failed(val reason: String, val canRetry: Boolean) : OrderProcessingState()
    object Timeout : OrderProcessingState()
}

@Composable
fun OrderStatusScreen(state: OrderProcessingState) {
    when (state) {
        is OrderProcessingState.Processing -> {
            CircularProgressIndicator()
            Text("주문을 처리하고 있습니다...")
        }
        is OrderProcessingState.Completed -> {
            Icon(Icons.Default.CheckCircle, tint = Color.Green)
            Text("주문이 완료되었습니다!")
        }
        is OrderProcessingState.Failed -> {
            Icon(Icons.Default.Error, tint = Color.Red)
            Text(state.reason)
            if (state.canRetry) {
                Button(onClick = { /* 재시도 */ }) {
                    Text("다시 시도")
                }
            }
        }
        is OrderProcessingState.Timeout -> {
            Text("처리 시간이 초과되었습니다.")
            Text("주문 내역에서 상태를 확인해주세요.")
        }
    }
}
```

---

## 참고 자료

- [Apache Kafka 공식 문서](https://kafka.apache.org/documentation/)
- [AWS SQS 개발자 가이드](https://docs.aws.amazon.com/sqs/)
- [Confluent Kafka 튜토리얼](https://developer.confluent.io/)
- [Martin Fowler - Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
- [Microservices Patterns - Chris Richardson](https://microservices.io/patterns/)
- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
