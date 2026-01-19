# Trino

## 1. 한 줄 요약

여러 데이터 소스(S3, MySQL, Kafka 등)에 **하나의 SQL로 쿼리**할 수 있게 해주는 분산 쿼리 엔진입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점의 비유

**통합 검색 엔진**을 생각해보세요.

구글에서 검색하면 웹사이트, 이미지, 뉴스, 지도 등 다양한 소스의 결과를 한 번에 볼 수 있습니다. Trino도 마찬가지로, 여러 데이터 저장소의 데이터를 하나의 SQL로 조회할 수 있게 해줍니다.

```
┌───────────────────────────────────────────────────────────────┐
│  통합 검색 (Google)              │  Trino                     │
├──────────────────────────────────┼────────────────────────────┤
│  검색어 입력                     │  SQL 작성                   │
│  "맛집 서울"                     │  SELECT * FROM ...          │
│                                  │                              │
│  결과:                           │  결과:                       │
│  - 웹사이트 (Web)                │  - S3 (Parquet)             │
│  - 지도 (Maps)                   │  - MySQL (RDS)              │
│  - 이미지 (Images)               │  - Kafka (Stream)           │
│  - 뉴스 (News)                   │  - Elasticsearch            │
└──────────────────────────────────┴────────────────────────────┘
```

### Trino vs Spark 차이점

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      TRINO vs SPARK                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Trino: "빠른 대화형 쿼리"                                               │
│  ─────────────────────────                                               │
│  • 목적: 분석가의 Ad-hoc 쿼리                                            │
│  • 특징: 결과를 빠르게 반환 (초~분)                                      │
│  • 비유: 식당에서 메뉴 바로 주문 → 빠른 서빙                             │
│                                                                          │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                           │
│  │ Analyst │ ──▶ │  Trino  │ ──▶ │ Result  │  (초 단위)                │
│  └─────────┘     └─────────┘     └─────────┘                           │
│                                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  Spark: "대규모 배치 처리"                                               │
│  ────────────────────────                                                │
│  • 목적: 대용량 ETL, 복잡한 파이프라인                                   │
│  • 특징: 전체 처리 후 저장 (분~시간)                                     │
│  • 비유: 대형 물류센터에서 전체 분류 후 배송                             │
│                                                                          │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                           │
│  │  ETL    │ ──▶ │  Spark  │ ──▶ │ Storage │  (분~시간)                │
│  │  Job    │     │ Cluster │     │  (S3)   │                           │
│  └─────────┘     └─────────┘     └─────────┘                           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 언제 Trino를 사용하는가?

| 시나리오 | 도구 | 이유 |
|----------|------|------|
| "오늘 가입자 수 알려줘" | **Trino** | 빠른 Ad-hoc 쿼리 |
| "어제 구매 데이터 집계해서 저장해" | **Spark** | ETL 배치 처리 |
| "S3와 MySQL 데이터 조인해줘" | **Trino** | 다중 소스 쿼리 |
| "1년치 로그 전체 분석해줘" | **Spark** | 대용량 처리 |

---

## 3. 구조 다이어그램

### Trino 아키텍처

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       TRINO ARCHITECTURE                                 │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │         Client              │
                    │  (SQL Workbench, DBeaver)   │
                    └─────────────┬───────────────┘
                                  │ SQL Query
                                  ▼
                    ┌─────────────────────────────┐
                    │       Coordinator           │
                    │  ┌───────────────────────┐  │
                    │  │   Query Planner       │  │
                    │  │   - 파싱              │  │
                    │  │   - 최적화            │  │
                    │  │   - 실행 계획 생성     │  │
                    │  └───────────────────────┘  │
                    └─────────────┬───────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
          ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Worker 1    │    │     Worker 2    │    │     Worker 3    │
│  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │ Executor  │  │    │  │ Executor  │  │    │  │ Executor  │  │
│  └───────────┘  │    │  └───────────┘  │    │  └───────────┘  │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │      Connectors       │
                    └───────────┬───────────┘
                                │
    ┌───────────────┬───────────┼───────────┬───────────────┐
    │               │           │           │               │
    ▼               ▼           ▼           ▼               ▼
┌────────┐    ┌────────┐   ┌────────┐   ┌────────┐    ┌────────┐
│   S3   │    │  MySQL │   │  Kafka │   │  Redis │    │  ES    │
│(Hive)  │    │        │   │        │   │        │    │        │
└────────┘    └────────┘   └────────┘   └────────┘    └────────┘
```

### Connector 개념

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       TRINO CONNECTORS                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Connector = 데이터 소스와의 연결 드라이버                                │
│                                                                          │
│  ┌─────────────────┐                                                    │
│  │ Hive Connector  │ ──▶ S3 (Parquet, ORC, JSON)                       │
│  │                 │     HDFS                                            │
│  └─────────────────┘                                                    │
│                                                                          │
│  ┌─────────────────┐                                                    │
│  │ MySQL Connector │ ──▶ Amazon RDS, Aurora                             │
│  └─────────────────┘     Cloud SQL                                       │
│                                                                          │
│  ┌─────────────────┐                                                    │
│  │ Kafka Connector │ ──▶ Real-time 이벤트 스트림                        │
│  └─────────────────┘                                                    │
│                                                                          │
│  ┌─────────────────┐                                                    │
│  │ Redis Connector │ ──▶ 캐시 데이터                                    │
│  └─────────────────┘                                                    │
│                                                                          │
│  ┌─────────────────┐                                                    │
│  │ Iceberg         │ ──▶ 최신 테이블 포맷                               │
│  │ Connector       │     (ACID, Time Travel)                            │
│  └─────────────────┘                                                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 쿼리 실행 과정

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    QUERY EXECUTION FLOW                                  │
└─────────────────────────────────────────────────────────────────────────┘

  SQL Query:
  SELECT u.name, COUNT(*) as purchases
  FROM s3.events e
  JOIN mysql.users u ON e.user_id = u.id
  WHERE e.event_date = '2024-01-15'
  GROUP BY u.name

  실행 과정:

  1. PARSING
     └── SQL을 AST(Abstract Syntax Tree)로 변환

  2. ANALYSIS
     └── 테이블, 컬럼 존재 확인
     └── 타입 검증

  3. PLANNING
     └── 논리적 실행 계획 생성
     └── 최적화 (Predicate Pushdown 등)

  4. SCHEDULING
     └── Worker에 Task 분배

  5. EXECUTION
     ┌─────────────────────────────────────────────────────────┐
     │                                                          │
     │  Worker 1              Worker 2              Worker 3   │
     │  ┌─────────────┐      ┌─────────────┐      ┌─────────┐ │
     │  │ Scan S3     │      │ Scan S3     │      │ Scan    │ │
     │  │ Partition 1 │      │ Partition 2 │      │ MySQL   │ │
     │  └──────┬──────┘      └──────┬──────┘      └────┬────┘ │
     │         │                    │                   │      │
     │         └────────────────────┼───────────────────┘      │
     │                              │                          │
     │                     ┌────────▼────────┐                 │
     │                     │      JOIN       │                 │
     │                     └────────┬────────┘                 │
     │                              │                          │
     │                     ┌────────▼────────┐                 │
     │                     │    GROUP BY     │                 │
     │                     └────────┬────────┘                 │
     │                              │                          │
     │                     ┌────────▼────────┐                 │
     │                     │     Result      │                 │
     │                     └─────────────────┘                 │
     │                                                          │
     └─────────────────────────────────────────────────────────┘

  6. RESULT
     └── Client에 결과 반환
```

---

## 4. 실무 적용 예시

### 4.1 Trino 설정 예시

```properties
# config.properties (Coordinator)
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=50GB
query.max-memory-per-node=10GB
discovery-server.enabled=true
discovery.uri=http://coordinator:8080

# config.properties (Worker)
coordinator=false
http-server.http.port=8080
query.max-memory=50GB
query.max-memory-per-node=10GB
discovery.uri=http://coordinator:8080
```

### 4.2 Catalog 설정 (Connector)

```properties
# hive.properties - S3 데이터 조회용
connector.name=hive
hive.metastore.uri=thrift://metastore:9083
hive.s3.path-style-access=true
hive.s3.endpoint=s3.ap-northeast-2.amazonaws.com
hive.s3.aws-access-key=${ENV:AWS_ACCESS_KEY}
hive.s3.aws-secret-key=${ENV:AWS_SECRET_KEY}

# mysql.properties - RDS 조회용
connector.name=mysql
connection-url=jdbc:mysql://rds-endpoint:3306
connection-user=${ENV:MYSQL_USER}
connection-password=${ENV:MYSQL_PASSWORD}
```

### 4.3 실무 쿼리 예시

```sql
-- 1. S3 데이터 직접 조회 (Hive Connector)
SELECT
    event_date,
    event_name,
    COUNT(*) as event_count,
    COUNT(DISTINCT user_id) as unique_users
FROM hive.datalake.app_events
WHERE event_date = DATE '2024-01-15'
GROUP BY event_date, event_name
ORDER BY event_count DESC;


-- 2. S3와 MySQL 조인 (Cross-catalog 쿼리)
SELECT
    u.user_name,
    u.created_at as signup_date,
    COUNT(e.event_id) as total_events,
    SUM(CASE WHEN e.event_name = 'purchase' THEN 1 ELSE 0 END) as purchases
FROM hive.datalake.app_events e
JOIN mysql.production.users u
    ON e.user_id = CAST(u.id AS VARCHAR)
WHERE e.event_date >= DATE '2024-01-01'
GROUP BY u.user_name, u.created_at
ORDER BY purchases DESC
LIMIT 100;


-- 3. 퍼널 분석 쿼리
WITH funnel AS (
    SELECT
        user_id,
        MAX(CASE WHEN event_name = 'app_open' THEN 1 ELSE 0 END) as step1_open,
        MAX(CASE WHEN event_name = 'product_view' THEN 1 ELSE 0 END) as step2_view,
        MAX(CASE WHEN event_name = 'add_to_cart' THEN 1 ELSE 0 END) as step3_cart,
        MAX(CASE WHEN event_name = 'purchase' THEN 1 ELSE 0 END) as step4_purchase
    FROM hive.datalake.app_events
    WHERE event_date = DATE '2024-01-15'
    GROUP BY user_id
)
SELECT
    'App Open' as step,
    1 as step_number,
    COUNT(*) as users,
    100.0 as conversion_rate
FROM funnel WHERE step1_open = 1

UNION ALL

SELECT
    'Product View' as step,
    2 as step_number,
    SUM(step2_view) as users,
    ROUND(SUM(step2_view) * 100.0 / SUM(step1_open), 2) as conversion_rate
FROM funnel WHERE step1_open = 1

UNION ALL

SELECT
    'Add to Cart' as step,
    3 as step_number,
    SUM(step3_cart) as users,
    ROUND(SUM(step3_cart) * 100.0 / SUM(step1_open), 2) as conversion_rate
FROM funnel WHERE step1_open = 1

UNION ALL

SELECT
    'Purchase' as step,
    4 as step_number,
    SUM(step4_purchase) as users,
    ROUND(SUM(step4_purchase) * 100.0 / SUM(step1_open), 2) as conversion_rate
FROM funnel WHERE step1_open = 1

ORDER BY step_number;


-- 4. 윈도우 함수를 활용한 세션 분석
WITH session_events AS (
    SELECT
        user_id,
        event_timestamp,
        event_name,
        LAG(event_timestamp) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp
        ) as prev_event_time
    FROM hive.datalake.app_events
    WHERE event_date = DATE '2024-01-15'
),
sessions AS (
    SELECT
        user_id,
        event_timestamp,
        event_name,
        SUM(CASE
            WHEN prev_event_time IS NULL
                OR event_timestamp - prev_event_time > INTERVAL '30' MINUTE
            THEN 1
            ELSE 0
        END) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp
        ) as session_id
    FROM session_events
)
SELECT
    user_id,
    session_id,
    MIN(event_timestamp) as session_start,
    MAX(event_timestamp) as session_end,
    COUNT(*) as events_in_session,
    DATE_DIFF('second', MIN(event_timestamp), MAX(event_timestamp)) as session_duration_sec
FROM sessions
GROUP BY user_id, session_id
ORDER BY session_duration_sec DESC
LIMIT 100;
```

### 4.4 쿼리 최적화 팁

```sql
-- 1. 파티션 프루닝 활용 (필수!)
-- ❌ Bad: 전체 스캔
SELECT * FROM hive.datalake.app_events
WHERE YEAR(event_timestamp) = 2024;

-- ✅ Good: 파티션 컬럼으로 필터
SELECT * FROM hive.datalake.app_events
WHERE event_date >= DATE '2024-01-01'
  AND event_date < DATE '2024-02-01';


-- 2. 컬럼 선택 최소화
-- ❌ Bad: 모든 컬럼
SELECT * FROM hive.datalake.app_events;

-- ✅ Good: 필요한 컬럼만
SELECT event_name, user_id, event_timestamp
FROM hive.datalake.app_events;


-- 3. LIMIT 조기 사용
-- ❌ Bad: 전체 정렬 후 LIMIT
SELECT * FROM hive.datalake.app_events
ORDER BY event_timestamp DESC
LIMIT 100;

-- ✅ Good: 필터 후 정렬
SELECT * FROM hive.datalake.app_events
WHERE event_date = CURRENT_DATE
ORDER BY event_timestamp DESC
LIMIT 100;


-- 4. EXPLAIN으로 실행 계획 확인
EXPLAIN (TYPE DISTRIBUTED)
SELECT event_name, COUNT(*)
FROM hive.datalake.app_events
WHERE event_date = DATE '2024-01-15'
GROUP BY event_name;
```

---

## 5. 장단점

### Trino의 장점

| 장점 | 설명 |
|------|------|
| **다중 소스 쿼리** | S3, MySQL, Kafka 등을 하나의 SQL로 |
| **빠른 쿼리** | 대화형 분석에 적합한 속도 |
| **표준 SQL** | ANSI SQL 호환, 러닝커브 낮음 |
| **확장성** | 노드 추가로 성능 확장 |
| **무료/오픈소스** | 라이선스 비용 없음 |

### Trino의 단점

| 단점 | 설명 |
|------|------|
| **메모리 의존** | 대용량 조인 시 메모리 부족 가능 |
| **ETL 부적합** | 대규모 데이터 변환에는 Spark 권장 |
| **운영 복잡** | 클러스터 관리 필요 |
| **장애 복구** | 쿼리 중간 실패 시 재시작 필요 |

### Amazon Athena vs Trino 직접 운영

| 항목 | Amazon Athena | Trino (Self-managed) |
|------|--------------|----------------------|
| **관리** | 서버리스 (무관리) | 직접 운영 |
| **비용** | 스캔 데이터 기반 | 인프라 비용 |
| **성능** | 공유 리소스 | 전용 클러스터 |
| **연결** | AWS 서비스 중심 | 다양한 커넥터 |
| **적합** | 가벼운 Ad-hoc | 대규모 분석 팀 |

---

## 6. 내 생각

```
(학습 후 자신의 생각을 정리해보세요)

- Trino가 우리 팀에 필요한 상황은?

- Athena로 충분한지, Trino가 필요한지?

- 데이터 분석가와 어떻게 협업할 수 있을까?

```

---

## 7. 추가 질문

### 기초 레벨
1. Trino와 Spark SQL의 차이점은 무엇인가요?

> **답변**: 둘 다 분산 SQL 엔진이지만 설계 목적과 특성이 다릅니다. **Trino**: **(1) 목적**: 빠른 대화형(Interactive) 쿼리. 분석가가 즉시 결과를 확인하는 Ad-hoc 분석에 적합. **(2) 아키텍처**: 메모리 기반 파이프라인. 중간 결과를 디스크에 저장하지 않음. 실패 시 처음부터 재시작. **(3) 특징**: 다양한 데이터 소스(S3, MySQL, Kafka 등)를 하나의 SQL로 조인 가능. **(4) 적합 용도**: 대시보드 쿼리, 탐색적 분석, 실시간 보고서. **Spark SQL**: **(1) 목적**: 대규모 배치 처리 및 ETL. 수 테라바이트 데이터 변환에 적합. **(2) 아키텍처**: 디스크 기반 Shuffle. 실패 시 해당 Stage만 재실행(Fault Tolerance). **(3) 특징**: ML(MLlib), 스트리밍(Structured Streaming) 등 다양한 기능 통합. **(4) 적합 용도**: 일일 ETL 배치, 대용량 집계, ML 파이프라인. **실무 사용 패턴**: Spark로 ETL 처리 → S3에 Parquet 저장 → Trino로 Ad-hoc 쿼리. 모바일 개발자 비유로 Spark는 백그라운드 배치 작업, Trino는 사용자 요청 즉시 응답하는 API 서버입니다.

2. Connector의 역할은 무엇인가요?

> **답변**: **Connector**는 Trino가 다양한 데이터 소스에 접근할 수 있게 해주는 플러그인입니다. 각 Connector는 특정 데이터 소스와 통신하는 방법을 정의합니다. **역할**: (1) **메타데이터 제공**: 테이블 스키마, 파티션 정보를 Trino에 전달. (2) **데이터 읽기**: 데이터 소스에서 실제 데이터를 읽어 Trino Worker에 전달. (3) **Predicate Pushdown**: WHERE 조건을 데이터 소스로 내려보내 스캔량 감소. (4) **데이터 쓰기** (일부): INSERT, CREATE TABLE AS SELECT 지원. **주요 Connector**: (1) **Hive Connector**: S3, HDFS의 Parquet/ORC 파일 접근. 가장 많이 사용. (2) **MySQL/PostgreSQL Connector**: 관계형 DB 직접 조회. (3) **Iceberg/Delta Connector**: 최신 테이블 포맷 지원. (4) **Kafka Connector**: 실시간 스트림 데이터 조회. **설정 예시**:
> ```properties
> # catalog/hive.properties
> connector.name=hive
> hive.metastore.uri=thrift://metastore:9083
> hive.s3.path-style-access=true
> ```
> Connector 덕분에 `SELECT * FROM hive.datalake.events e JOIN mysql.prod.users u ON e.user_id = u.id`처럼 다른 소스 간 조인이 가능합니다.

3. Coordinator와 Worker의 역할 차이는?

> **답변**: Trino 클러스터는 **Coordinator 1대 + Worker N대**로 구성됩니다. **Coordinator**: (1) 클라이언트 쿼리를 받아 파싱, 분석, 최적화. (2) 논리적 실행 계획 → 물리적 실행 계획 변환. (3) 작업을 Worker에 분배하고 진행 상황 모니터링. (4) 최종 결과를 클라이언트에 반환. (5) **리소스**: CPU/메모리 요구량이 Worker보다 적음. 보통 1대로 충분. **Worker**: (1) Coordinator가 할당한 Task 실행. (2) 데이터 소스(S3, MySQL 등)에서 데이터 읽기. (3) 필터링, 집계, 조인 등 실제 연산 수행. (4) 중간 결과를 다른 Worker로 전송(Shuffle). (5) **리소스**: 메모리와 CPU를 많이 사용. 노드 수로 성능 확장. **비유**: Coordinator는 식당 매니저(주문 받고 배분), Worker는 요리사(실제 요리 담당). Coordinator가 병목이 되면 쿼리 대기열이 쌓이고, Worker가 부족하면 쿼리 실행이 느려집니다.

### 심화 레벨
4. Predicate Pushdown이란 무엇이고, 왜 중요한가요?

> **답변**: **Predicate Pushdown**은 WHERE 조건(Predicate)을 데이터 소스 레벨로 내려보내서 스캔할 데이터 양을 줄이는 최적화 기법입니다. **작동 방식**:
> ```sql
> SELECT * FROM s3_events WHERE event_date = '2024-01-15' AND country = 'KR'
> ```
> Pushdown 없이: S3에서 전체 데이터 읽기 → Trino에서 필터링 (느림)
> Pushdown 있으면: S3에서 해당 파티션/행만 읽기 (빠름)
>
> **중요한 이유**: (1) **I/O 감소**: 불필요한 데이터를 읽지 않음. (2) **비용 절감**: S3 GET 요청 수, 스캔 데이터 양 감소 (Athena 비용 직결). (3) **성능 향상**: 네트워크 전송량과 처리 시간 대폭 감소. **Pushdown이 적용되는 경우**: (1) 파티션 컬럼 조건 (가장 효과적), (2) Parquet/ORC의 Row Group 통계 활용, (3) MySQL 등 RDBMS로 조건 전달. **확인 방법**:
> ```sql
> EXPLAIN (TYPE DISTRIBUTED) SELECT ... FROM ... WHERE ...
> -- ScanFilterProject에서 pushdown된 조건 확인
> ```
> 파티션 컬럼(`event_date`)으로 필터링하면 수백 배 성능 차이가 날 수 있습니다.

5. Trino에서 메모리 부족 에러가 발생할 때 대처 방법은?

> **답변**: Trino는 메모리 기반 엔진이라 대용량 쿼리에서 메모리 부족이 자주 발생합니다. **에러 유형과 대처**: **(1) Query exceeded per-node memory limit**: 단일 노드에서 쿼리가 사용할 수 있는 메모리 초과. → 해결: `query.max-memory-per-node` 증가, 또는 데이터 필터링 강화.
> ```properties
> query.max-memory-per-node=8GB
> query.max-total-memory-per-node=10GB
> ```
> **(2) Query exceeded distributed memory limit**: 전체 클러스터의 쿼리 메모리 제한 초과. → 해결: `query.max-memory` 증가, 또는 쿼리 분할.
> **(3) 특정 연산이 메모리 많이 사용**: ORDER BY (전역 정렬), 대규모 JOIN, COUNT(DISTINCT). → 해결: LIMIT 추가, 근사 함수 사용 (`approx_distinct()`), Broadcast Join 대신 Hash Join.
> **쿼리 수정 방법**:
> ```sql
> -- 잘못된 예: 전체 정렬
> SELECT * FROM events ORDER BY event_timestamp;
>
> -- 개선: 파티션 제한 + LIMIT
> SELECT * FROM events
> WHERE event_date = '2024-01-15'
> ORDER BY event_timestamp
> LIMIT 10000;
> ```
> 실무에서는 무거운 쿼리를 실행하기 전에 `EXPLAIN`으로 실행 계획을 확인하고, 예상 스캔 크기를 파악하는 습관이 중요합니다.

6. 쿼리 성능을 모니터링하는 방법은?

> **답변**: Trino 쿼리 성능 모니터링은 **Web UI, 시스템 테이블, 외부 도구**를 활용합니다. **(1) Trino Web UI (기본 포트 8080)**: 실행 중/완료된 쿼리 목록, 쿼리별 실행 시간, CPU/메모리 사용량, Stage별 진행 상황과 병목 지점, 실시간 실행 계획 시각화. **(2) 시스템 테이블 쿼리**:
> ```sql
> -- 느린 쿼리 찾기
> SELECT query_id, query, execution_time, total_cpu_time
> FROM system.runtime.queries
> WHERE state = 'FINISHED'
> ORDER BY execution_time DESC
> LIMIT 20;
>
> -- 현재 실행 중인 쿼리
> SELECT * FROM system.runtime.queries WHERE state = 'RUNNING';
> ```
> **(3) JMX 메트릭**: Prometheus로 수집하여 Grafana 대시보드 구성. 주요 지표: 쿼리 실행 수, 평균 실행 시간, 메모리 사용률, Worker 상태. **(4) Query History**: `etc/event-listener.properties`로 쿼리 이력을 외부 저장소에 기록. **실무 팁**: 분석팀에서 사용하는 대시보드 쿼리를 정기적으로 리뷰하고, 느린 쿼리의 실행 계획을 분석하여 최적화 가이드를 제공합니다.

### 실무 시나리오
7. 분석가가 느린 쿼리를 실행할 때 어떻게 대응하나요?

> **답변**: 분석가의 느린 쿼리 대응은 **즉각 대응, 근본 해결, 예방**의 3단계로 접근합니다. **(1) 즉각 대응**: 쿼리가 클러스터를 독점하면 Web UI에서 해당 쿼리 Kill. 긴급하지 않은 쿼리는 우선순위 낮은 Resource Group으로 이동.
> ```sql
> -- 쿼리 종료
> CALL system.runtime.kill_query(query_id => 'xxx', message => 'Resource limit exceeded');
> ```
> **(2) 근본 해결**: 쿼리 분석 후 최적화 제안: 파티션 컬럼 필터 추가, 불필요한 컬럼 제거(`SELECT *` 지양), 서브쿼리를 WITH절로 변환, 큰 테이블끼리 JOIN 시 Spark ETL로 미리 집계.
> ```sql
> -- 최적화 전
> SELECT * FROM events WHERE YEAR(event_timestamp) = 2024;
>
> -- 최적화 후
> SELECT user_id, event_name, event_timestamp
> FROM events
> WHERE event_date >= '2024-01-01' AND event_date < '2025-01-01';
> ```
> **(3) 예방**: 느린 쿼리 패턴을 문서화하여 분석팀과 공유, 자주 사용되는 무거운 쿼리는 Materialized View 또는 사전 집계 테이블로 대체, Resource Group으로 사용자별/팀별 리소스 제한 설정.

8. S3 데이터와 RDS 데이터를 조인할 때 주의사항은?

> **답변**: S3(대용량)와 RDS(트랜잭션 DB)의 Cross-catalog 조인은 강력하지만 주의가 필요합니다. **주의사항**: **(1) RDS 부하**: Trino가 RDS의 전체 테이블을 스캔할 수 있어 프로덕션 DB에 부하. → 해결: Read Replica 사용, 필요한 데이터만 뽑아 S3에 미리 덤프.
> ```sql
> -- 잘못된 예: RDS 전체 스캔
> SELECT * FROM s3.events e JOIN mysql.prod.users u ON e.user_id = u.id;
>
> -- 개선: RDS 데이터 먼저 필터링
> SELECT * FROM s3.events e
> JOIN (SELECT id, name FROM mysql.prod.users WHERE created_at > '2024-01-01') u
> ON e.user_id = u.id;
> ```
> **(2) 타입 불일치**: S3의 user_id(STRING)와 RDS의 id(INT) 조인 시 암묵적 형변환으로 성능 저하. → 해결: CAST로 명시적 변환.
> ```sql
> ON CAST(e.user_id AS INTEGER) = u.id
> ```
> **(3) 데이터 위치**: 큰 S3 데이터를 RDS 쪽으로 Shuffle하면 네트워크 병목. → 해결: 항상 작은 테이블(RDS)이 Broadcast되도록 쿼리 순서 조정.
> **(4) 실시간성 차이**: S3는 ETL 주기에 따라 지연, RDS는 실시간. 데이터 시점 불일치 주의. → 해결: 같은 시점 기준으로 필터링.

9. Trino 클러스터의 적정 크기는 어떻게 결정하나요?

> **답변**: Trino 클러스터 크기는 **동시 쿼리 수, 쿼리 복잡도, 데이터 크기**를 기반으로 결정합니다. **(1) Worker 노드 수**: 경험적으로 동시 실행 쿼리 × 2~4 노드. 10명이 동시에 쿼리하면 20~40 노드. 데이터 크기가 크면 더 필요. **(2) Worker 사양**: 메모리가 핵심. 노드당 64~128GB RAM 권장. CPU는 8~16 코어면 충분. 인스턴스 타입: AWS r5/r6i (메모리 최적화).
> ```
> 예시 클러스터 (중간 규모):
> - Coordinator: 1 × r5.2xlarge (8 vCPU, 64GB)
> - Worker: 10 × r5.4xlarge (16 vCPU, 128GB)
> - 총 메모리: ~1.3TB
> ```
> **(3) 설정 가이드라인**:
> ```properties
> # 노드당 쿼리 메모리 (Worker RAM의 70~80%)
> query.max-memory-per-node=100GB
> query.max-total-memory-per-node=110GB
>
> # 전체 쿼리 메모리
> query.max-memory=800GB  # Worker 수 × 노드당 메모리
> ```
> **(4) 스케일링 전략**: 사용량이 일정하지 않으면 Kubernetes + Trino Operator로 오토스케일링. 피크 시간에만 노드 증가. 처음에는 작게 시작하고, 모니터링하면서 점진적으로 확장하는 것이 비용 효율적입니다.

---

## 8. 모바일 앱 데이터 분석을 위한 Trino 활용

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    모바일 앱 분석 시나리오에서의 Trino 활용                         │
└─────────────────────────────────────────────────────────────────────────────────┘

  분석가가 자주 하는 질문들:

  "오늘 DAU가 얼마야?"
  "지난주 대비 구매 전환율 변화는?"
  "iOS vs Android 사용 패턴 차이는?"
  "특정 캠페인의 ROI는?"

                │
                ▼
  ┌─────────────────────────────────────────────────────────────────────────┐
  │  Trino Cluster                                                          │
  │                                                                          │
  │  ┌─────────────────────────────────────────────────────────────────┐   │
  │  │  SELECT                                                          │   │
  │  │    DATE(event_timestamp) AS date,                               │   │
  │  │    COUNT(DISTINCT user_id) AS dau,                              │   │
  │  │    SUM(CASE WHEN event_name = 'purchase' THEN 1 ELSE 0 END)    │   │
  │  │  FROM hive.datalake.app_events                                  │   │
  │  │  WHERE event_date >= DATE '2024-01-01'                          │   │
  │  │  GROUP BY DATE(event_timestamp)                                 │   │
  │  └─────────────────────────────────────────────────────────────────┘   │
  │                                                                          │
  └─────────────────────────────────────────────────────────────────────────┘
                │
        ┌───────┴───────┬───────────────┐
        ▼               ▼               ▼
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ S3       │   │ MySQL    │   │ Redis    │
  │ (Events) │   │ (Users)  │   │ (Cache)  │
  └──────────┘   └──────────┘   └──────────┘

  실행 시간: 5~30초 (데이터 크기에 따라)
```

### 분석가를 위한 Trino 쿼리 템플릿 (모바일 앱)

```sql
-- ============================================================
-- 1. 일별 핵심 지표 (DAU, 세션, 이벤트)
-- ============================================================
SELECT
    event_date,
    COUNT(DISTINCT user_id) AS dau,
    COUNT(DISTINCT session_id) AS total_sessions,
    COUNT(*) AS total_events,
    ROUND(COUNT(*) * 1.0 / COUNT(DISTINCT user_id), 2) AS events_per_user
FROM hive.datalake.app_events
WHERE event_date >= CURRENT_DATE - INTERVAL '30' DAY
GROUP BY event_date
ORDER BY event_date DESC;


-- ============================================================
-- 2. 구매 퍼널 분석
-- ============================================================
WITH funnel AS (
    SELECT
        user_id,
        MAX(CASE WHEN event_name = 'app_open' THEN 1 ELSE 0 END) AS step1_open,
        MAX(CASE WHEN event_name = 'product_view' THEN 1 ELSE 0 END) AS step2_view,
        MAX(CASE WHEN event_name = 'add_to_cart' THEN 1 ELSE 0 END) AS step3_cart,
        MAX(CASE WHEN event_name = 'purchase' THEN 1 ELSE 0 END) AS step4_purchase
    FROM hive.datalake.app_events
    WHERE event_date = CURRENT_DATE - INTERVAL '1' DAY
    GROUP BY user_id
)
SELECT
    'App Open' AS step_name,
    COUNT(*) AS users,
    100.0 AS conversion_rate
FROM funnel WHERE step1_open = 1
UNION ALL
SELECT
    'Product View',
    SUM(step2_view),
    ROUND(SUM(step2_view) * 100.0 / NULLIF(SUM(step1_open), 0), 2)
FROM funnel WHERE step1_open = 1
UNION ALL
SELECT
    'Add to Cart',
    SUM(step3_cart),
    ROUND(SUM(step3_cart) * 100.0 / NULLIF(SUM(step1_open), 0), 2)
FROM funnel WHERE step1_open = 1
UNION ALL
SELECT
    'Purchase',
    SUM(step4_purchase),
    ROUND(SUM(step4_purchase) * 100.0 / NULLIF(SUM(step1_open), 0), 2)
FROM funnel WHERE step1_open = 1;


-- ============================================================
-- 3. 플랫폼별 사용 패턴 비교 (iOS vs Android)
-- ============================================================
SELECT
    platform,
    COUNT(DISTINCT user_id) AS users,
    COUNT(DISTINCT session_id) AS sessions,
    ROUND(AVG(session_duration_sec), 0) AS avg_session_duration,
    ROUND(COUNT(*) * 1.0 / COUNT(DISTINCT session_id), 2) AS events_per_session
FROM hive.datalake.app_events
WHERE event_date >= CURRENT_DATE - INTERVAL '7' DAY
GROUP BY platform;


-- ============================================================
-- 4. S3 이벤트 + MySQL 사용자 정보 조인
-- ============================================================
SELECT
    u.user_segment,
    COUNT(DISTINCT e.user_id) AS users,
    SUM(CASE WHEN e.event_name = 'purchase' THEN 1 ELSE 0 END) AS purchases,
    SUM(e.event_value) AS total_revenue
FROM hive.datalake.app_events e
JOIN mysql.production.users u
    ON CAST(e.user_id AS INTEGER) = u.id
WHERE e.event_date = CURRENT_DATE - INTERVAL '1' DAY
  AND u.is_active = true
GROUP BY u.user_segment
ORDER BY total_revenue DESC;
```

---

## 9. 실무에서 자주 겪는 Trino 문제와 해결책

### 문제 1: 쿼리가 너무 느림

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 쿼리가 수 분 이상 걸림 | 파티션 프루닝 미적용 | 파티션 컬럼(event_date)으로 WHERE 조건 추가 |
| SELECT * 사용 | 불필요한 컬럼 스캔 | 필요한 컬럼만 명시 |
| 대규모 JOIN | 양쪽 테이블 모두 큼 | 작은 테이블 먼저 필터링, 또는 Spark로 사전 집계 |

```sql
-- 느린 쿼리 (파티션 프루닝 안 됨)
SELECT * FROM events WHERE YEAR(event_timestamp) = 2024;

-- 빠른 쿼리 (파티션 프루닝 적용)
SELECT user_id, event_name, event_timestamp
FROM events
WHERE event_date >= DATE '2024-01-01' AND event_date < DATE '2024-02-01';
```

### 문제 2: 분석가마다 다른 결과

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 같은 지표인데 수치가 다름 | 필터 조건 차이 | 표준 쿼리 템플릿 제공 |
| DAU 계산 방식 불일치 | user_id vs device_id | 지표 정의 문서화 |
| 타임존 차이 | UTC vs KST | 표준 타임존 정의 |

```sql
-- 표준 DAU 계산 (팀 내 합의된 정의)
-- 정의: 해당 날짜에 앱을 1회 이상 사용한 로그인 사용자 수
-- 타임존: KST (UTC+9)
SELECT
    DATE(event_timestamp AT TIME ZONE 'Asia/Seoul') AS kst_date,
    COUNT(DISTINCT CASE WHEN user_id IS NOT NULL THEN user_id END) AS dau
FROM hive.datalake.app_events
WHERE event_date >= CURRENT_DATE - INTERVAL '7' DAY
  AND event_name != 'background_ping'  -- 백그라운드 이벤트 제외
GROUP BY DATE(event_timestamp AT TIME ZONE 'Asia/Seoul');
```

### 문제 3: 클러스터 리소스 부족

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 쿼리 큐에 대기 | 동시 쿼리 과다 | Resource Group으로 우선순위 관리 |
| 특정 쿼리가 리소스 독점 | 대용량 쿼리 | 쿼리 타임아웃 설정, 쿼리 분할 |
| 피크 시간 성능 저하 | 업무 시간 집중 | 오토스케일링, 쿼리 캐싱 |

```properties
# resource-groups.json - 팀별 리소스 할당
{
  "rootGroups": [
    {
      "name": "analytics_team",
      "softMemoryLimit": "70%",
      "maxQueued": 100,
      "hardConcurrencyLimit": 20
    },
    {
      "name": "etl_jobs",
      "softMemoryLimit": "20%",
      "maxQueued": 10,
      "hardConcurrencyLimit": 5
    },
    {
      "name": "ad_hoc",
      "softMemoryLimit": "10%",
      "maxQueued": 50,
      "hardConcurrencyLimit": 10
    }
  ]
}
```

---

## 참고 자료

- [Trino 공식 문서](https://trino.io/docs/current/)
- [Trino: The Definitive Guide (Book)](https://www.oreilly.com/library/view/trino-the-definitive/9781098137229/)
- [AWS Athena 개발자 가이드](https://docs.aws.amazon.com/athena/)
- [Trino SQL 함수 레퍼런스](https://trino.io/docs/current/functions.html)
- [Starburst (Enterprise Trino)](https://www.starburst.io/)
