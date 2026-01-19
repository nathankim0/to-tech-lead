# Database 구조

## 1. 한 줄 요약

데이터베이스는 애플리케이션의 **영속적인 데이터 저장소**로, 관계형(RDBMS)과 비관계형(NoSQL)으로 나뉘며 각각의 특성에 맞게 선택한다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

#### 관계형 데이터베이스 = 엑셀 스프레드시트

| 엑셀 | RDBMS |
|:---|:---|
| 시트 | 테이블 (Table) |
| 행 (Row) | 레코드 (Record) |
| 열 (Column) | 필드 (Field) |
| 시트 간 참조 | 외래 키 (Foreign Key) |
| 필터/정렬 | SQL 쿼리 |

엑셀처럼 정형화된 데이터를 행과 열로 구조화하여 저장합니다. "사용자" 시트와 "주문" 시트를 사용자 ID로 연결하는 것처럼요.

#### NoSQL = 서류 폴더 캐비닛

| 서류 캐비닛 | NoSQL |
|:---|:---|
| 폴더 | 컬렉션 (Collection) |
| 서류 | 문서 (Document) |
| 서류 형식 다양 | 스키마 유연함 |
| 폴더별 정리 | 컬렉션별 데이터 |

서류 폴더에는 다양한 형식의 문서를 넣을 수 있습니다. 어떤 서류는 2장, 어떤 서류는 10장일 수 있죠. NoSQL도 각 문서가 다른 구조를 가질 수 있습니다.

### 왜 알아야 할까?

모바일 앱의 데이터는 결국 서버 DB에 저장됩니다.
- API 응답 속도는 **DB 쿼리 성능**에 직결됩니다
- 데이터 모델을 이해하면 **효율적인 API 요청**을 설계할 수 있습니다
- CoreData/Room을 쓴다면 **로컬 DB 설계**에도 활용됩니다

---

## 3. 구조 다이어그램

### RDBMS 테이블 관계

```
┌────────────────────────────────────────────────────────────────────┐
│                        E-Commerce Database                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│   users                          orders                            │
│   ┌─────────────────┐           ┌─────────────────────┐            │
│   │ id (PK)         │     ┌────>│ id (PK)             │            │
│   │ email           │     │     │ user_id (FK) ───────┼───┐        │
│   │ name            │     │     │ status              │   │        │
│   │ created_at      │<────┘     │ total_price         │   │        │
│   └─────────────────┘           │ created_at          │   │        │
│           │                     └─────────────────────┘   │        │
│           │                              │                │        │
│           │                              │ 1:N            │        │
│           │                              ▼                │        │
│           │                     ┌─────────────────────┐   │        │
│           │                     │ order_items         │   │        │
│           │                     │ ───────────────────  │   │        │
│           │                     │ id (PK)             │   │        │
│           │                     │ order_id (FK)       │   │        │
│           │                     │ product_id (FK) ────┼───┼──┐     │
│           │                     │ quantity            │   │  │     │
│           │                     │ price               │   │  │     │
│           │                     └─────────────────────┘   │  │     │
│           │                                               │  │     │
│           │ 1:N                                           │  │     │
│           ▼                                               │  │     │
│   ┌─────────────────┐           ┌─────────────────────┐   │  │     │
│   │ addresses       │           │ products            │<──┼──┘     │
│   │ ─────────────── │           │ ───────────────────  │   │        │
│   │ id (PK)         │           │ id (PK)             │   │        │
│   │ user_id (FK) ───┼───────────│ name                │   │        │
│   │ city            │           │ price               │   │        │
│   │ street          │           │ category_id (FK)    │   │        │
│   │ is_default      │           │ stock               │   │        │
│   └─────────────────┘           └─────────────────────┘   │        │
│                                                           │        │
│   관계 표시:  ───>  1:N (One to Many)                       │        │
│              <──>  N:M (Many to Many, 중간 테이블 필요)      │        │
│                                                           │        │
└───────────────────────────────────────────────────────────┴────────┘
```

### 인덱스 동작 원리

```
인덱스가 없는 경우 (Full Table Scan)
══════════════════════════════════════════════════════════════

SELECT * FROM users WHERE email = 'hong@example.com'

    ┌─────────────────────────────────────────────┐
    │                 users 테이블                 │
    │  ┌────┬─────────────────────┬───────────┐   │
    │  │ id │ email               │ name      │   │
    │  ├────┼─────────────────────┼───────────┤   │
    │  │ 1  │ kim@example.com     │ 김철수    │ ← 검색
    │  │ 2  │ lee@example.com     │ 이영희    │ ← 검색
    │  │ 3  │ hong@example.com    │ 홍길동    │ ← ✅ 찾음!
    │  │ 4  │ park@example.com    │ 박민수    │ ← 검색 (계속)
    │  │ .. │ ...                 │ ...       │ ← 검색 (끝까지)
    │  └────┴─────────────────────┴───────────┘   │
    │                                             │
    │  → 100만 행이면 최악의 경우 100만 번 비교!    │
    └─────────────────────────────────────────────┘


인덱스가 있는 경우 (B-Tree Index)
══════════════════════════════════════════════════════════════

CREATE INDEX idx_email ON users(email);

         ┌─────────────────────────────┐
         │      B-Tree Index Root      │
         │    [kim@... | park@...]     │
         └─────────────┬───────────────┘
                       │
         ┌─────────────┴───────────────┐
         ▼                             ▼
    ┌─────────────┐              ┌─────────────┐
    │ [hong@...]  │              │ [lee@...]   │
    │ [kim@...]   │              │ [park@...]  │
    └──────┬──────┘              └─────────────┘
           │
           ▼
    ┌─────────────────────────┐
    │ hong@example.com → id:3 │  ← 바로 찾음!
    └─────────────────────────┘

    → 100만 행이어도 약 20번의 비교로 찾음 (log₂ 1,000,000 ≈ 20)
```

### NoSQL (MongoDB) 문서 구조

```json
// MongoDB Document 예시
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "email": "hong@example.com",
  "name": "홍길동",
  "profile": {
    "avatar": "https://...",
    "bio": "안녕하세요"
  },
  "orders": [
    {
      "orderId": "ORD-001",
      "items": [
        { "productId": "P001", "name": "맥북", "quantity": 1 }
      ],
      "totalPrice": 2500000,
      "status": "delivered"
    },
    {
      "orderId": "ORD-002",
      "items": [
        { "productId": "P002", "name": "아이패드", "quantity": 2 }
      ],
      "totalPrice": 1800000,
      "status": "shipping"
    }
  ],
  "createdAt": ISODate("2024-01-15T10:30:00Z")
}
```

### 데이터베이스 확장 전략

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Scale Up vs Scale Out                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Scale Up (수직 확장)              Scale Out (수평 확장)             │
│  ─────────────────────            ─────────────────────             │
│                                                                     │
│       ┌───────────┐                  ┌─────┐ ┌─────┐ ┌─────┐        │
│       │           │                  │ DB1 │ │ DB2 │ │ DB3 │        │
│       │    DB     │      →          └──┬──┘ └──┬──┘ └──┬──┘        │
│       │  (더 큰   │                     │       │       │           │
│       │   서버)   │                     └───────┼───────┘           │
│       │           │                             │                   │
│       └───────────┘                        샤딩/레플리카             │
│                                                                     │
│  - CPU, RAM, SSD 업그레이드       - 여러 서버에 데이터 분산           │
│  - 간단하지만 한계가 있음          - 복잡하지만 무한 확장 가능         │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Replication (복제)                Sharding (샤딩)                   │
│  ─────────────────────            ─────────────────────             │
│                                                                     │
│      ┌──────────┐                 ┌──────────┐                      │
│      │  Master  │                 │  Shard 1 │ users A-M            │
│      │  (쓰기)  │                 └──────────┘                      │
│      └────┬─────┘                 ┌──────────┐                      │
│           │                       │  Shard 2 │ users N-Z            │
│     ┌─────┴─────┐                 └──────────┘                      │
│     ▼           ▼                                                   │
│ ┌────────┐ ┌────────┐             - 데이터를 분할하여 저장            │
│ │ Replica│ │ Replica│             - 각 샤드는 일부 데이터만 담당       │
│ │ (읽기) │ │ (읽기) │             - 특정 키(예: user_id)로 분배       │
│ └────────┘ └────────┘                                               │
│                                                                     │
│ - 동일 데이터를 여러 서버에 복사   - 대용량 데이터 처리에 필수         │
│ - 읽기 성능 향상, 장애 대응        - 설계와 운영이 복잡함              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 예시 1: SQL vs NoSQL 선택 기준

```
┌─────────────────────────────────────────────────────────────────────┐
│                    언제 무엇을 선택할까?                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  RDBMS (MySQL, PostgreSQL)를 선택해야 할 때:                         │
│  ────────────────────────────────────────────                       │
│  ✅ 데이터 간 관계가 복잡할 때 (주문-상품-사용자-결제)                 │
│  ✅ 트랜잭션이 중요할 때 (금융, 결제)                                 │
│  ✅ 데이터 정합성이 중요할 때 (재고 관리)                             │
│  ✅ 복잡한 쿼리/집계가 필요할 때 (리포트, 분석)                       │
│                                                                     │
│  NoSQL (MongoDB, DynamoDB)를 선택해야 할 때:                         │
│  ────────────────────────────────────────────                       │
│  ✅ 스키마가 자주 변경될 때 (초기 스타트업)                           │
│  ✅ 대용량 비정형 데이터 (로그, 소셜 피드)                            │
│  ✅ 빠른 읽기/쓰기가 필요할 때 (실시간 채팅)                          │
│  ✅ 수평 확장이 필수일 때 (글로벌 서비스)                             │
│                                                                     │
│  실제 서비스 예시:                                                   │
│  ────────────────                                                   │
│  • 카카오톡: 메시지 저장 → NoSQL (대용량, 고속)                       │
│  • 쿠팡: 주문/결제 → RDBMS (트랜잭션)                                 │
│  • 넷플릭스: 사용자 시청 기록 → NoSQL (대용량 로그)                    │
│  • 토스: 송금 내역 → RDBMS (금융 정합성)                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 예시 2: N+1 문제와 해결

```python
# N+1 문제 발생 예시 (Django ORM)
# ─────────────────────────────────────────────────

# 주문 목록을 가져옴 (1번의 쿼리)
orders = Order.objects.all()

# 각 주문에 대해 사용자 정보를 가져옴 (N번의 쿼리)
for order in orders:
    print(order.user.name)  # 매번 새로운 쿼리 발생!

# 실행되는 쿼리:
# SELECT * FROM orders;                    -- 1번
# SELECT * FROM users WHERE id = 1;        -- 2번
# SELECT * FROM users WHERE id = 2;        -- 3번
# ... (주문 수만큼 반복)                    -- N번

# 총 N+1번의 쿼리 발생!


# 해결 방법 1: select_related (JOIN)
# ─────────────────────────────────────────────────

orders = Order.objects.select_related('user').all()

for order in orders:
    print(order.user.name)  # 추가 쿼리 없음

# 실행되는 쿼리:
# SELECT orders.*, users.*
# FROM orders
# JOIN users ON orders.user_id = users.id;  -- 딱 1번!


# 해결 방법 2: prefetch_related (1:N 관계)
# ─────────────────────────────────────────────────

orders = Order.objects.prefetch_related('items').all()

for order in orders:
    for item in order.items.all():  # 추가 쿼리 없음
        print(item.product_name)

# 실행되는 쿼리:
# SELECT * FROM orders;
# SELECT * FROM order_items WHERE order_id IN (1, 2, 3, ...);
# 총 2번의 쿼리!
```

### 예시 3: 인덱스 설계

```sql
-- 자주 사용되는 쿼리 분석
-- ─────────────────────────────────────────────────

-- 쿼리 1: 이메일로 사용자 조회 (로그인)
SELECT * FROM users WHERE email = 'hong@example.com';
--> email 컬럼에 UNIQUE 인덱스 필요

-- 쿼리 2: 특정 사용자의 주문 목록 (최신순)
SELECT * FROM orders
WHERE user_id = 123
ORDER BY created_at DESC;
--> (user_id, created_at) 복합 인덱스 필요

-- 쿼리 3: 상품 검색 (카테고리 + 가격 범위)
SELECT * FROM products
WHERE category_id = 5
  AND price BETWEEN 10000 AND 50000
ORDER BY created_at DESC;
--> (category_id, price, created_at) 복합 인덱스 고려

-- 인덱스 생성
-- ─────────────────────────────────────────────────
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);
CREATE INDEX idx_products_category_price ON products(category_id, price);


-- 인덱스 사용 확인 (EXPLAIN)
-- ─────────────────────────────────────────────────
EXPLAIN SELECT * FROM orders WHERE user_id = 123;

-- 결과 예시:
-- +----+-------------+--------+------+------------------------+
-- | id | select_type | table  | type | possible_keys          |
-- +----+-------------+--------+------+------------------------+
-- |  1 | SIMPLE      | orders | ref  | idx_orders_user_created|
-- +----+-------------+--------+------+------------------------+
-- type이 'ref'면 인덱스 사용 중, 'ALL'이면 풀 스캔
```

### 예시 4: 트랜잭션 처리

```python
# 주문 생성 트랜잭션 예시 (Python + SQLAlchemy)

from sqlalchemy.orm import Session
from sqlalchemy.exc import IntegrityError

def create_order(user_id: int, items: list) -> Order:
    """
    주문 생성 - ACID 보장이 필요한 작업
    1. 재고 확인 및 차감
    2. 주문 생성
    3. 결제 처리

    하나라도 실패하면 모두 롤백
    """
    with Session(engine) as session:
        try:
            # 트랜잭션 시작
            session.begin()

            # 1. 재고 확인 및 차감
            for item in items:
                product = session.query(Product).filter(
                    Product.id == item['product_id']
                ).with_for_update().first()  # 행 레벨 락

                if product.stock < item['quantity']:
                    raise InsufficientStockError(
                        f"재고 부족: {product.name}"
                    )

                product.stock -= item['quantity']

            # 2. 주문 생성
            order = Order(
                user_id=user_id,
                total_price=calculate_total(items),
                status='pending'
            )
            session.add(order)
            session.flush()  # order.id 생성

            # 3. 주문 항목 생성
            for item in items:
                order_item = OrderItem(
                    order_id=order.id,
                    product_id=item['product_id'],
                    quantity=item['quantity'],
                    price=item['price']
                )
                session.add(order_item)

            # 모든 작업 성공 - 커밋
            session.commit()
            return order

        except Exception as e:
            # 하나라도 실패 - 롤백
            session.rollback()
            raise e
```

---

## 5. 장단점

### RDBMS vs NoSQL

| 구분 | RDBMS | NoSQL |
|:---|:---|:---|
| **데이터 모델** | 테이블 (정형) | 문서/키-값/그래프 (유연) |
| **스키마** | 고정 스키마 | 유연한 스키마 |
| **확장성** | 수직 확장 (Scale Up) | 수평 확장 (Scale Out) |
| **트랜잭션** | ACID 완벽 지원 | 제한적 지원 (BASE) |
| **쿼리** | SQL (표준화) | 각 DB별 문법 |
| **적합한 경우** | 복잡한 관계, 정합성 중요 | 대용량, 빠른 개발 |

### ACID vs BASE

```
ACID (관계형 DB)                    BASE (NoSQL)
─────────────────────────────────────────────────────────────
A - Atomicity (원자성)              BA - Basically Available
    트랜잭션은 전부 성공 또는 전부 실패       기본적으로 가용성 보장

C - Consistency (일관성)            S - Soft State
    트랜잭션 전후 데이터 정합성 유지          상태가 일시적으로 불일치할 수 있음

I - Isolation (격리성)              E - Eventually Consistent
    동시 트랜잭션 간 간섭 없음              결국에는 일관성 도달

D - Durability (지속성)
    커밋된 데이터는 영구 저장

사용 예시:
─────────────────────────────────────────────────────────────
ACID: 계좌 이체 - A에서 B로 100만원 이체
      → A 출금과 B 입금이 반드시 함께 성공/실패해야 함

BASE: 좋아요 카운트 - 게시물 좋아요 수
      → 잠시 동안 일부 서버에서 999, 일부에서 1000이어도 괜찮음
```

---

## 6. 내 생각

> 이 섹션은 학습 후 본인의 생각을 정리하는 공간입니다.

```
Q1. 현재 서비스에서 DB 쿼리 성능 이슈를 경험한 적이 있는가?


Q2. 모바일 앱에서 로컬 DB (CoreData/Room)를 사용할 때 서버 DB와 동기화는 어떻게 하는가?


Q3. SQL과 NoSQL을 함께 사용하는 Polyglot Persistence 전략은 언제 필요할까?


```

---

## 7. 추가 질문

더 깊이 학습하기 위한 질문들입니다.

### 설계 관련
- [ ] 정규화 vs 비정규화의 트레이드오프는?

> **답변**: **정규화(Normalization)**는 데이터 중복을 제거하고 무결성을 보장하지만, 조회 시 JOIN이 많아집니다. **비정규화(Denormalization)**는 읽기 성능을 위해 의도적으로 중복을 허용합니다. **트레이드오프**: 정규화는 쓰기 작업이 많고 데이터 일관성이 중요한 시스템(결제, 재고)에 적합합니다. 비정규화는 읽기가 압도적으로 많은 시스템(상품 목록, 게시판)에 적합합니다. **모바일 앱 관점**: API 응답에 필요한 데이터가 여러 테이블에 분산되면 JOIN이 많아져 느려집니다. 이럴 때 비정규화로 `상품 테이블`에 `카테고리_이름`을 중복 저장하면 한 번의 조회로 가능합니다. 실무에서는 "먼저 정규화하고, 성능 문제가 생기면 선택적으로 비정규화"하는 전략을 권장합니다. 예: 주문 테이블에 주문 당시 상품명/가격을 저장 (상품 정보 변경에 영향받지 않도록).

- [ ] 소프트 삭제(Soft Delete)와 하드 삭제(Hard Delete) 중 무엇을 선택해야 할까?

> **답변**: **소프트 삭제**는 실제로 데이터를 삭제하지 않고 `deleted_at` 컬럼에 삭제 시간을 기록합니다. **하드 삭제**는 DB에서 완전히 제거합니다. **소프트 삭제 선택 시**: (1) 법적 요구로 데이터 보관이 필요할 때 (금융, 의료) (2) 실수로 삭제한 데이터 복구가 필요할 때 (3) 삭제된 데이터도 통계/분석에 사용할 때. **하드 삭제 선택 시**: (1) 개인정보 완전 삭제가 법적으로 요구될 때 (GDPR) (2) 저장 공간 절약이 필요할 때 (3) 삭제된 데이터 참조로 인한 버그 위험을 없애고 싶을 때. **모바일 앱 고려**: 소프트 삭제 시 API에서 항상 `WHERE deleted_at IS NULL`을 추가해야 합니다. 인덱스도 `WHERE deleted_at IS NULL`을 포함하는 부분 인덱스를 사용하면 성능이 좋습니다. 회원 탈퇴 시에는 개인정보(이름, 연락처)는 하드 삭제하고, 주문 이력 같은 비식별 데이터는 소프트 삭제하는 혼합 전략도 가능합니다.

- [ ] 테이블 파티셔닝은 언제 필요하고 어떻게 적용할까?

> **답변**: **파티셔닝**은 하나의 큰 테이블을 여러 작은 물리적 조각으로 나누는 기법입니다. **필요한 시점**: (1) 테이블 크기가 수억 건 이상일 때 (2) 오래된 데이터 삭제/아카이빙이 빈번할 때 (3) 특정 범위 데이터만 자주 조회할 때. **파티셔닝 전략**: Range(날짜별), List(지역별), Hash(ID 기반) 방식이 있습니다. **실무 예시**: 주문 테이블을 월별로 Range 파티셔닝하면, "이번 달 주문 조회"는 하나의 파티션만 스캔합니다. 2년 지난 데이터 삭제도 파티션 통째로 DROP하면 빠릅니다. **모바일 앱 관점**: 파티셔닝은 서버 내부 최적화라 앱에서 직접 신경 쓸 일은 없지만, "최근 3개월 주문만 조회 가능" 같은 제약이 생길 수 있습니다. 날짜 범위 필터를 필수로 요구하는 API가 있다면 파티셔닝 최적화 때문일 수 있습니다.

```sql
-- PostgreSQL 월별 파티셔닝 예시
CREATE TABLE orders (
    id SERIAL,
    user_id INT,
    created_at TIMESTAMP,
    total_price DECIMAL
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### 성능 관련
- [ ] 슬로우 쿼리를 어떻게 찾고 최적화할까?

> **답변**: **슬로우 쿼리 발견**: (1) MySQL은 `slow_query_log` 활성화, (2) PostgreSQL은 `log_min_duration_statement` 설정, (3) AWS RDS는 Performance Insights 사용, (4) APM 도구(Datadog, New Relic)에서 DB 탭 확인. **최적화 과정**: 1단계로 `EXPLAIN ANALYZE`로 실행 계획을 확인합니다. `Seq Scan`(풀 스캔)이 보이면 인덱스 부재, `Nested Loop`이 많으면 JOIN 최적화 필요입니다. 2단계로 적절한 인덱스를 추가합니다 (WHERE, ORDER BY, JOIN 조건 컬럼). 3단계로 쿼리 자체를 개선합니다 (SELECT * 대신 필요한 컬럼만, 불필요한 서브쿼리 제거). **모바일 앱에서의 영향**: API 응답 시간이 갑자기 느려졌다면 백엔드에 슬로우 쿼리가 생겼을 가능성이 높습니다. 특정 조건(대량 데이터 사용자)에서만 느리다면 인덱스 문제일 수 있습니다. 백엔드 팀에 Slow Query 로그 확인을 요청하세요.

- [ ] 커넥션 풀링은 왜 필요하고 어떻게 설정할까?

> **답변**: **커넥션 풀링**은 DB 연결을 미리 만들어두고 재사용하는 기법입니다. DB 연결 생성은 TCP 핸드셰이크, 인증, 세션 초기화 등 비용이 큽니다(50-200ms). **풀링 없이**: 매 요청마다 연결 생성/종료 → 느리고 DB 연결 수 폭증. **풀링 사용 시**: 미리 만든 연결을 재사용 → 빠르고 연결 수 제한. **설정 가이드**: (1) 최소 연결 수: 평상시 동시 요청 수 (예: 10) (2) 최대 연결 수: 피크 시 동시 요청 수 + 여유 (예: 50) (3) 연결 타임아웃: 5-10초 (너무 길면 풀 고갈 시 요청이 쌓임). **모바일 앱 관점**: 커넥션 풀이 고갈되면 API가 타임아웃됩니다. "서버 점검 중"이 아닌데 갑자기 모든 API가 느려지면 DB 커넥션 풀 문제일 수 있습니다. 이때 앱에서는 Retry 로직과 적절한 타임아웃 설정이 중요합니다.

- [ ] Read Replica를 사용할 때 Replication Lag은 어떻게 처리할까?

> **답변**: **Replication Lag**은 Master에 쓴 데이터가 Replica에 복제되기까지의 지연 시간입니다. 보통 수십 밀리초지만, 부하가 심하면 수 초까지 발생합니다. **문제 시나리오**: 사용자가 게시글 작성(Master 쓰기) 후 바로 목록 조회(Replica 읽기)하면 방금 쓴 글이 안 보입니다! **해결 전략**: (1) **Read-your-writes 보장**: 쓰기 직후 일정 시간은 Master에서 읽기. 쿠키나 헤더로 "방금 쓴 사용자" 표시. (2) **Versioning**: 응답에 버전 번호 포함, 클라이언트가 기대 버전과 비교. (3) **Sticky Session**: 같은 사용자는 같은 Replica로 라우팅. (4) **동기식 복제**: Lag 없지만 쓰기 성능 저하. **모바일 앱 대응**: "작성 완료" 후 목록 화면으로 돌아갈 때 1-2초 딜레이를 주거나, Optimistic UI로 로컬에서 먼저 표시합니다.

```kotlin
// Android - Optimistic UI로 Replication Lag 우회
class PostViewModel : ViewModel() {
    private val _posts = MutableStateFlow<List<Post>>(emptyList())

    fun createPost(content: String) {
        val optimisticPost = Post(
            id = UUID.randomUUID().toString(),
            content = content,
            createdAt = System.currentTimeMillis(),
            isLocal = true  // 아직 서버 확정 전
        )

        // 1. 로컬에 먼저 표시
        _posts.value = listOf(optimisticPost) + _posts.value

        // 2. 서버에 저장
        viewModelScope.launch {
            try {
                val serverPost = api.createPost(content)
                // 3. 서버 응답으로 교체
                _posts.value = _posts.value.map {
                    if (it.id == optimisticPost.id) serverPost else it
                }
            } catch (e: Exception) {
                // 4. 실패 시 롤백
                _posts.value = _posts.value.filter { it.id != optimisticPost.id }
            }
        }
    }
}
```

### 심화
- [ ] CAP 정리란 무엇이고, 실제 DB들은 어떤 선택을 했을까?

> **답변**: **CAP 정리**는 분산 시스템에서 Consistency(일관성), Availability(가용성), Partition Tolerance(분단 내성) 세 가지를 동시에 만족할 수 없다는 이론입니다. 네트워크 분단(P)은 분산 시스템에서 불가피하므로, 실제로는 **CP vs AP** 선택입니다. **CP 선택 (일관성 우선)**: MongoDB(default), HBase, Redis Cluster. 네트워크 분단 시 일부 요청을 거부하더라도 데이터 일관성 보장. 금융, 재고 관리에 적합. **AP 선택 (가용성 우선)**: Cassandra, DynamoDB, CouchDB. 네트워크 분단 시에도 응답하지만 데이터가 일시적으로 불일치할 수 있음. 소셜 피드, 채팅에 적합. **모바일 앱 관점**: 결제 관련 기능은 CP DB를 사용하는 API로, 좋아요 카운트 같은 것은 AP DB로 처리됩니다. "좋아요 수가 새로고침마다 다르다"는 것은 AP DB의 Eventually Consistent 특성 때문입니다.

- [ ] NewSQL (CockroachDB, TiDB)은 무엇이고 언제 사용할까?

> **답변**: **NewSQL**은 전통적인 RDBMS의 ACID 트랜잭션과 SQL 호환성을 유지하면서, NoSQL처럼 수평 확장이 가능한 데이터베이스입니다. **대표적인 NewSQL**: CockroachDB(분산 PostgreSQL 호환), TiDB(분산 MySQL 호환), Google Spanner(Cloud). **선택 시점**: (1) 기존 MySQL/PostgreSQL 앱을 수정 없이 확장하고 싶을 때 (2) 글로벌 서비스로 여러 리전에 DB를 분산해야 할 때 (3) 샤딩의 복잡성 없이 자동 분산을 원할 때. **주의점**: 단일 노드 성능은 전통 RDBMS보다 낮을 수 있고, 학습 곡선과 운영 복잡도가 있습니다. 비용도 높습니다. **모바일 앱 관점**: 앱 개발자가 직접 선택할 일은 드물지만, 글로벌 서비스에서 "어느 나라에서든 빠른 DB 응답"이 필요하다면 NewSQL이 답일 수 있습니다. TiDB나 CockroachDB를 도입하면 기존 코드 변경 없이 확장 가능합니다.

- [ ] 데이터베이스 마이그레이션은 어떻게 안전하게 진행할까?

> **답변**: **DB 마이그레이션**은 스키마나 데이터를 변경하는 작업으로, 잘못하면 서비스 장애로 이어집니다. **안전한 마이그레이션 원칙**: (1) **하위 호환성 유지**: 새 컬럼 추가 시 기본값 설정 또는 nullable로. 구버전 앱이 동작해야 합니다. (2) **점진적 변경**: 컬럼명 변경 시 → 새 컬럼 추가 → 데이터 복사 → 앱 수정 배포 → 구 컬럼 삭제 (3단계에 걸쳐). (3) **무중단 배포**: pt-online-schema-change(MySQL), pg_repack(PostgreSQL) 등 온라인 마이그레이션 도구 사용. (4) **롤백 계획**: 항상 되돌릴 수 있는 스크립트 준비. **모바일 앱 배포와 연계**: DB 마이그레이션과 앱 업데이트 순서가 중요합니다. 새 컬럼이 필요한 앱 버전 배포 → 사용자 업데이트 대기 → 구 컬럼 삭제. 강제 업데이트 없이는 구버전 앱이 계속 동작해야 합니다.

```
안전한 컬럼명 변경 과정 (old_name → new_name)
───────────────────────────────────────────────────────

1단계: 새 컬럼 추가 (서비스 영향 없음)
ALTER TABLE users ADD COLUMN new_name VARCHAR(255);

2단계: 양쪽에 쓰기 (앱 배포)
- 앱: old_name, new_name 둘 다 저장
- 기존 데이터 마이그레이션: UPDATE users SET new_name = old_name WHERE new_name IS NULL;

3단계: 새 컬럼으로 읽기 전환 (앱 배포)
- 앱: new_name에서 읽기, old_name은 폴백용

4단계: 구 컬럼 삭제 (구버전 앱 없을 때)
ALTER TABLE users DROP COLUMN old_name;
```

---

## 8. 모바일 앱과 서버 DB의 동기화 패턴

모바일 앱에서 로컬 DB(CoreData, Room)와 서버 DB 간의 동기화는 중요한 주제입니다.

### 패턴 1: Pull 기반 동기화

```swift
// iOS - Pull 기반 동기화 (가장 일반적)
class SyncManager {
    private let api: API
    private let coreData: CoreDataManager

    func syncOrders() async throws {
        // 1. 마지막 동기화 시간 확인
        let lastSyncTime = UserDefaults.standard.object(forKey: "lastOrderSync") as? Date ?? .distantPast

        // 2. 변경된 데이터만 서버에서 가져오기
        let updatedOrders = try await api.getOrders(modifiedSince: lastSyncTime)

        // 3. 로컬 DB 업데이트
        for order in updatedOrders {
            coreData.upsertOrder(order)  // INSERT or UPDATE
        }

        // 4. 동기화 시간 저장
        UserDefaults.standard.set(Date(), forKey: "lastOrderSync")
    }
}
```

### 패턴 2: Push + Pull 양방향 동기화

```kotlin
// Android - 오프라인 작업 동기화
@Entity(tableName = "pending_actions")
data class PendingAction(
    @PrimaryKey val id: String = UUID.randomUUID().toString(),
    val type: String,  // "CREATE", "UPDATE", "DELETE"
    val entityType: String,  // "order", "review"
    val payload: String,  // JSON
    val createdAt: Long = System.currentTimeMillis()
)

class SyncService(
    private val db: AppDatabase,
    private val api: Api
) {
    suspend fun syncPendingActions() {
        val pendingActions = db.pendingActionDao().getAll()

        for (action in pendingActions) {
            try {
                when (action.type) {
                    "CREATE" -> api.createEntity(action.entityType, action.payload)
                    "UPDATE" -> api.updateEntity(action.entityType, action.payload)
                    "DELETE" -> api.deleteEntity(action.entityType, action.payload)
                }
                // 성공 시 삭제
                db.pendingActionDao().delete(action)
            } catch (e: Exception) {
                // 실패 시 다음 동기화에 재시도
                Log.e("Sync", "Failed to sync action: ${action.id}")
            }
        }
    }
}
```

### 패턴 3: Conflict Resolution (충돌 해결)

```
동기화 충돌 해결 전략
───────────────────────────────────────────────────────

1. Last Write Wins (마지막 쓰기 승리)
   - 가장 단순, 타임스탬프가 늦은 쪽이 승리
   - 데이터 유실 가능성 있음

2. Server Wins (서버 우선)
   - 항상 서버 데이터로 덮어쓰기
   - 오프라인 작업 결과가 사라질 수 있음

3. Client Wins (클라이언트 우선)
   - 항상 클라이언트 데이터로 덮어쓰기
   - 다른 기기의 변경사항을 덮어쓸 수 있음

4. Merge (병합)
   - 필드별로 변경 감지 후 병합
   - 복잡하지만 가장 정확

5. User Decision (사용자 선택)
   - 충돌 시 사용자에게 선택권 부여
   - UX가 복잡해질 수 있음
```

---

## 9. 모바일 개발자가 알아야 할 DB 용어

| 용어 | 설명 | 모바일 앱에서의 영향 |
|:---|:---|:---|
| **인덱스** | 검색 속도를 높이는 자료구조 | 인덱스 없으면 API가 느려짐 |
| **트랜잭션** | 여러 작업을 하나로 묶어 원자적 실행 | 결제 중 일부만 처리되는 것 방지 |
| **락(Lock)** | 동시 접근 제어 | 동시 구매 시 재고 정합성 |
| **데드락** | 서로 락을 기다려 멈추는 상태 | API 타임아웃 발생 |
| **풀 스캔** | 테이블 전체를 읽는 것 | 데이터 많으면 API 매우 느림 |
| **커서** | 페이지네이션용 포인터 | 무한 스크롤 구현 시 사용 |
| **샤딩** | 데이터를 여러 DB로 분산 | 사용자 ID로 라우팅될 수 있음 |
| **레플리카** | 읽기 전용 복제본 | 쓰기 직후 조회 시 지연 |

---

## 참고 자료

- [Use The Index, Luke - SQL 인덱싱 가이드](https://use-the-index-luke.com/)
- [MySQL 공식 문서](https://dev.mysql.com/doc/)
- [MongoDB University](https://university.mongodb.com/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Android Room Database](https://developer.android.com/training/data-storage/room)
- [Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/)
