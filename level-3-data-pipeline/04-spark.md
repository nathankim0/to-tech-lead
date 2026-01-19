# Spark

## 1. 한 줄 요약

대규모 데이터를 **여러 서버에 분산**하여 **병렬로 처리**하는 통합 분석 엔진입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점의 비유

**대형 마트 계산대**를 생각해보세요.

```
❌ 단일 서버 처리 (1개 계산대)
┌─────────────────────────────────────────────────────┐
│  고객 100명 → [계산대 1개] → 1명씩 순서대로        │
│                                                      │
│  처리 시간: 100명 × 3분 = 300분                     │
└─────────────────────────────────────────────────────┘

✅ Spark 분산 처리 (10개 계산대)
┌─────────────────────────────────────────────────────┐
│  고객 100명 → [계산대 10개] → 10명씩 병렬 처리      │
│                                                      │
│  처리 시간: (100명 ÷ 10) × 3분 = 30분              │
└─────────────────────────────────────────────────────┘
```

### 왜 Spark인가?

모바일 앱의 일일 이벤트 5천만 건을 처리한다고 가정해봅시다.

| 방식 | 처리 시간 | 비용 |
|------|----------|------|
| **단일 Python 스크립트** | 10시간+ | 서버 1대 |
| **Spark (10 노드)** | 30분 | 서버 10대 × 30분 |
| **Spark (100 노드)** | 5분 | 서버 100대 × 5분 |

Spark는 **수평 확장(Scale-out)**이 가능하기 때문에, 노드를 추가하면 처리 시간이 줄어듭니다.

### Spark의 핵심 개념

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPARK CORE CONCEPTS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. RDD (Resilient Distributed Dataset)                         │
│     "복구 가능한 분산 데이터셋"                                   │
│     • 불변(Immutable)                                            │
│     • 파티션으로 분할되어 분산 저장                               │
│     • 실패 시 재계산으로 복구                                     │
│                                                                  │
│  2. DataFrame                                                    │
│     "테이블처럼 사용하는 분산 데이터"                             │
│     • SQL처럼 컬럼 기반 조작                                     │
│     • 최적화된 실행 계획 (Catalyst)                              │
│     • Pandas와 유사한 API                                        │
│                                                                  │
│  3. Lazy Evaluation                                              │
│     "실제 필요할 때까지 실행하지 않음"                            │
│     • Transformation: 실행 계획만 기록                           │
│     • Action: 실제 실행 트리거                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 구조 다이어그램

### Spark 클러스터 아키텍처

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      SPARK CLUSTER ARCHITECTURE                          │
└─────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │      Driver Program         │
                    │  ┌───────────────────────┐  │
                    │  │    SparkContext       │  │
                    │  │  (작업 조율, DAG 생성)  │  │
                    │  └───────────────────────┘  │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │     Cluster Manager         │
                    │  (YARN / Mesos / K8s)       │
                    │  - 리소스 할당               │
                    │  - 워커 관리                 │
                    └─────────────┬───────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
          ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Worker Node   │    │   Worker Node   │    │   Worker Node   │
│  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │ Executor  │  │    │  │ Executor  │  │    │  │ Executor  │  │
│  │┌───┬───┐  │  │    │  │┌───┬───┐  │  │    │  │┌───┬───┐  │  │
│  ││T1 │T2 │  │  │    │  ││T3 │T4 │  │  │    │  ││T5 │T6 │  │  │
│  │└───┴───┘  │  │    │  │└───┴───┘  │  │    │  │└───┴───┘  │  │
│  │ Cache     │  │    │  │ Cache     │  │    │  │ Cache     │  │
│  └───────────┘  │    │  └───────────┘  │    │  └───────────┘  │
│  ┌───────────┐  │    │  ┌───────────┐  │    │  ┌───────────┐  │
│  │ Partition │  │    │  │ Partition │  │    │  │ Partition │  │
│  │ [P1] [P2] │  │    │  │ [P3] [P4] │  │    │  │ [P5] [P6] │  │
│  └───────────┘  │    │  └───────────┘  │    │  └───────────┘  │
└─────────────────┘    └─────────────────┘    └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  용어 설명:                                                              │
│  • Driver: 작업을 조율하는 마스터 프로세스                               │
│  • Executor: 실제 작업을 수행하는 워커 프로세스                          │
│  • Task (T1~T6): 파티션 하나를 처리하는 작업 단위                        │
│  • Partition (P1~P6): 데이터를 분할한 단위                               │
└─────────────────────────────────────────────────────────────────────────┘
```

### Spark 작업 실행 흐름

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     SPARK JOB EXECUTION FLOW                             │
└─────────────────────────────────────────────────────────────────────────┘

  코드 작성                        DAG 생성                    실행
  ─────────                        ────────                    ────

  df.filter(...)  ──┐
  df.select(...)  ──┼──▶  Stage 1 ──▶ Stage 2 ──▶ Stage 3  ──▶ 결과
  df.groupBy(...) ──┘        │           │           │
                             ▼           ▼           ▼
                          [T1,T2]    [T3,T4]     [T5,T6]


  ┌───────────────────────────────────────────────────────────────────┐
  │  Transformation (Lazy)              │  Action (Trigger)           │
  │  ─────────────────                  │  ──────────────             │
  │  • filter()                         │  • count()                  │
  │  • select()                         │  • collect()                │
  │  • groupBy()                        │  • write()                  │
  │  • join()                           │  • show()                   │
  │                                     │                              │
  │  → 실행 계획만 기록                   │  → 실제 실행 시작            │
  └───────────────────────────────────────────────────────────────────┘
```

### Shuffle 이해하기

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SHUFFLE OPERATION                                 │
└─────────────────────────────────────────────────────────────────────────┘

  groupBy('user_id') 실행 시:

  Before Shuffle                          After Shuffle
  ──────────────                          ─────────────

  ┌─────────────┐                         ┌─────────────┐
  │ Partition 1 │                         │ Partition 1 │
  │ user_a: 10  │     ───────────────▶    │ user_a: 10  │
  │ user_b: 20  │       네트워크 전송       │ user_a: 15  │
  │ user_c: 30  │                         │ user_a: 25  │
  └─────────────┘                         └─────────────┘

  ┌─────────────┐                         ┌─────────────┐
  │ Partition 2 │                         │ Partition 2 │
  │ user_a: 15  │     ───────────────▶    │ user_b: 20  │
  │ user_d: 40  │       (Shuffle)         │ user_b: 35  │
  │ user_b: 35  │                         └─────────────┘
  └─────────────┘
                                          ┌─────────────┐
  ┌─────────────┐                         │ Partition 3 │
  │ Partition 3 │     ───────────────▶    │ user_c: 30  │
  │ user_a: 25  │                         │ user_d: 40  │
  │ user_c: 30  │                         └─────────────┘
  └─────────────┘


  ⚠️  Shuffle은 비용이 매우 큼!
  • 디스크 I/O + 네트워크 I/O 발생
  • 성능 병목의 주요 원인
  • 최소화하는 것이 최적화의 핵심
```

---

## 4. 실무 적용 예시

### 4.1 PySpark 기본 예제

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, sum, avg, to_date, hour

# SparkSession 생성
spark = SparkSession.builder \
    .appName("MobileAppLogAnalysis") \
    .config("spark.sql.shuffle.partitions", "200") \
    .getOrCreate()

# S3에서 데이터 읽기
df = spark.read.parquet("s3://data-lake/raw/app-logs/dt=2024-01-15/")

# 스키마 확인
df.printSchema()
# root
#  |-- event_name: string
#  |-- event_timestamp: timestamp
#  |-- user_id: string
#  |-- device_type: string
#  |-- params: map<string, string>


# 기본 Transformation
events_df = df \
    .filter(col("event_name").isNotNull()) \
    .withColumn("event_date", to_date("event_timestamp")) \
    .withColumn("event_hour", hour("event_timestamp"))

# 집계 쿼리
daily_summary = events_df \
    .groupBy("event_date", "event_name") \
    .agg(
        count("*").alias("event_count"),
        countDistinct("user_id").alias("unique_users")
    ) \
    .orderBy("event_date", col("event_count").desc())

# 결과 확인 (Action - 여기서 실제 실행됨)
daily_summary.show(10)

# 결과 저장
daily_summary.write \
    .mode("overwrite") \
    .partitionBy("event_date") \
    .parquet("s3://data-lake/processed/daily_summary/")
```

### 4.2 복잡한 분석 예제: 유저 리텐션 계산

```python
from pyspark.sql import Window
from pyspark.sql.functions import (
    datediff, min as spark_min, max as spark_max,
    when, lit
)

def calculate_retention(spark, start_date, end_date):
    """D1, D7, D30 리텐션 계산"""

    # 이벤트 데이터 로드
    events = spark.read.parquet(
        f"s3://data-lake/processed/events/"
    ).filter(
        (col("event_date") >= start_date) &
        (col("event_date") <= end_date)
    )

    # 유저별 첫 방문일, 마지막 방문일 계산
    user_activity = events.groupBy("user_id").agg(
        spark_min("event_date").alias("first_visit"),
        spark_max("event_date").alias("last_visit")
    )

    # 코호트별 리텐션 계산
    cohort_retention = user_activity \
        .withColumn(
            "days_retained",
            datediff("last_visit", "first_visit")
        ) \
        .withColumn(
            "d1_retained",
            when(col("days_retained") >= 1, 1).otherwise(0)
        ) \
        .withColumn(
            "d7_retained",
            when(col("days_retained") >= 7, 1).otherwise(0)
        ) \
        .withColumn(
            "d30_retained",
            when(col("days_retained") >= 30, 1).otherwise(0)
        )

    # 코호트별 집계
    result = cohort_retention.groupBy("first_visit").agg(
        count("*").alias("cohort_size"),
        sum("d1_retained").alias("d1_retained"),
        sum("d7_retained").alias("d7_retained"),
        sum("d30_retained").alias("d30_retained")
    ).withColumn(
        "d1_retention_rate",
        col("d1_retained") / col("cohort_size") * 100
    ).withColumn(
        "d7_retention_rate",
        col("d7_retained") / col("cohort_size") * 100
    ).withColumn(
        "d30_retention_rate",
        col("d30_retained") / col("cohort_size") * 100
    )

    return result

# 실행
retention_df = calculate_retention(spark, "2024-01-01", "2024-01-31")
retention_df.show()
```

### 4.3 Spark 최적화 팁

```python
# 1. 파티션 수 최적화
# 너무 적으면: 병렬성 부족
# 너무 많으면: 오버헤드 증가
# 권장: 코어 수의 2~4배

df = df.repartition(200)  # 명시적 파티션 수 지정


# 2. Broadcast Join (작은 테이블 조인 시)
from pyspark.sql.functions import broadcast

# 작은 테이블 (< 10MB)은 브로드캐스트
large_df.join(broadcast(small_df), "join_key")


# 3. 캐싱 (반복 사용 데이터)
# 여러 번 사용되는 DataFrame은 캐시
frequently_used_df = df.filter(...).cache()
frequently_used_df.count()  # 캐시 실행


# 4. 불필요한 컬럼 제거
# 필요한 컬럼만 선택하여 메모리 절약
df.select("col1", "col2", "col3")  # 전체 컬럼 대신


# 5. 필터 조기 적용 (Predicate Pushdown)
# 필터를 가능한 빨리 적용
df.filter(col("dt") == "2024-01-15") \
  .join(other_df, "key")  # 필터 후 조인


# 6. 파티션 키로 필터링
# 파티션 프루닝 활용
spark.read.parquet("s3://bucket/data/") \
    .filter(col("dt") == "2024-01-15")  # 해당 파티션만 읽음
```

### 4.4 AWS EMR에서 Spark 실행

```bash
# EMR 클러스터 생성
aws emr create-cluster \
    --name "spark-etl-cluster" \
    --release-label emr-6.10.0 \
    --applications Name=Spark \
    --instance-type m5.xlarge \
    --instance-count 10 \
    --use-default-roles

# Spark 작업 제출
aws emr add-steps \
    --cluster-id j-XXXXXXXXXXXXX \
    --steps Type=Spark,Name="DailyETL",\
ActionOnFailure=CONTINUE,\
Args=[--deploy-mode,cluster,\
--master,yarn,\
s3://scripts/daily_etl.py]
```

---

## 5. 장단점

### Spark vs 다른 도구 비교

| 항목 | Spark | Pandas | Hadoop MR |
|------|-------|--------|-----------|
| **데이터 크기** | TB~PB | GB | TB~PB |
| **처리 속도** | 빠름 (인메모리) | 빠름 (단일노드) | 느림 (디스크) |
| **러닝커브** | 중간 | 낮음 | 높음 |
| **실시간 처리** | 지원 (Streaming) | 미지원 | 미지원 |
| **ML 지원** | MLlib | Scikit-learn | Mahout |

### Spark의 장점

- **속도**: 인메모리 처리로 Hadoop MR 대비 100배 빠름
- **통합**: 배치, 스트리밍, SQL, ML을 하나의 엔진에서
- **확장성**: 수천 대 노드까지 확장 가능
- **다양한 언어**: Python, Scala, Java, R 지원

### Spark의 단점

- **메모리 의존**: 메모리 부족 시 성능 저하
- **복잡한 튜닝**: 최적 성능을 위한 파라미터 튜닝 필요
- **작은 데이터에 비효율**: GB 이하에서는 Pandas가 효율적
- **디버깅 어려움**: 분산 환경 디버깅이 복잡

---

## 6. 내 생각

```
(학습 후 자신의 생각을 정리해보세요)

- 현재 우리 팀에서 Spark를 사용하는 용도는?

- 데이터 규모가 커지면 어떤 최적화가 필요할까?

- Spark Streaming의 필요성은?

```

---

## 7. 추가 질문

### 기초 레벨
1. RDD와 DataFrame의 차이점은 무엇인가요?

> **답변**: **RDD(Resilient Distributed Dataset)**는 Spark의 가장 기본적인 데이터 구조로, 불변의 분산 객체 컬렉션입니다. 타입 안전성이 있고 저수준 제어가 가능하지만 최적화가 어렵습니다. **DataFrame**은 RDD 위에 구축된 고수준 API로, 테이블처럼 행과 열을 가지며 SQL과 유사하게 사용합니다. **주요 차이점**: (1) **최적화**: DataFrame은 Catalyst 옵티마이저가 쿼리를 최적화하지만 RDD는 개발자가 직접 최적화해야 합니다. (2) **성능**: DataFrame이 일반적으로 2~10배 빠릅니다. (3) **사용 편의성**: DataFrame은 SQL, Python pandas처럼 직관적. RDD는 map, reduce 같은 함수형 프로그래밍 스타일. (4) **스키마**: DataFrame은 스키마(컬럼명, 타입)가 명시적. RDD는 스키마 없음. **실무 권장**: 99%의 경우 DataFrame 사용. RDD는 비정형 데이터나 저수준 제어가 필요할 때만 사용합니다. 모바일 개발자 비유로 DataFrame은 Core Data, RDD는 직접 SQLite 쿼리 작성과 비슷합니다.

2. Lazy Evaluation이 왜 중요한가요?

> **답변**: **Lazy Evaluation(지연 평가)**은 Spark에서 Transformation(filter, select, join 등)을 호출해도 즉시 실행하지 않고, Action(count, collect, write 등)이 호출될 때 한꺼번에 실행하는 방식입니다. **중요한 이유**: (1) **최적화 기회**: 전체 작업을 미리 파악하여 불필요한 연산 제거, 연산 순서 재배치, 술어 푸시다운(Predicate Pushdown) 적용. (2) **효율적인 실행 계획**: 여러 Transformation을 하나의 Stage로 합쳐서 중간 결과 저장 없이 파이프라이닝. (3) **리소스 절약**: 실제 필요한 데이터만 읽고 처리. 예를 들어 `df.filter().select().filter()`에서 두 filter를 합치고, select 전에 filter를 적용합니다.
> ```python
> # Lazy - 아직 실행 안 됨
> df2 = df.filter(col("country") == "KR")
> df3 = df2.select("user_id", "event")
>
> # Action - 이때 실행됨
> result = df3.count()  # filter + select가 최적화되어 한 번에 실행
> ```
> 모바일 개발자에게 비유하면, Combine이나 RxSwift에서 subscribe 전까지 실행되지 않는 것과 유사합니다.

3. Partition 수는 어떻게 결정하나요?

> **답변**: Partition 수는 병렬 처리 단위로, **너무 적으면 병렬성 부족, 너무 많으면 오버헤드 증가**입니다. **결정 기준**: (1) **기본 규칙**: `파티션 수 = 클러스터 총 코어 수 × 2~4`. 예: 10노드 × 4코어 = 40코어 → 80~160 파티션. (2) **데이터 크기 기준**: 파티션당 128MB~1GB가 적정. 10GB 데이터 → 10~80 파티션. (3) **Shuffle 파티션**: `spark.sql.shuffle.partitions` 기본값 200. 데이터 크기에 맞게 조정. **실무 가이드라인**:
> - 작은 데이터(< 1GB): 기본값 사용 또는 `coalesce()`로 줄이기
> - 중간 데이터(1~100GB): 코어 수 × 2~4
> - 큰 데이터(> 100GB): 데이터 크기 / 128MB
> ```python
> # 파티션 수 확인
> df.rdd.getNumPartitions()
>
> # 파티션 조정
> df.repartition(100)  # Shuffle 발생, 파티션 증가/감소 모두 가능
> df.coalesce(10)      # Shuffle 없이 파티션 감소만 가능
> ```
> 파티션이 너무 작으면 메모리 부족(OOM), 너무 많으면 작업 스케줄링 오버헤드가 발생합니다.

### 심화 레벨
4. Shuffle이 발생하는 연산과 최소화 방법은?

> **답변**: **Shuffle**은 파티션 간 데이터 재분배로, 네트워크 I/O와 디스크 I/O가 발생하여 성능에 큰 영향을 줍니다. **Shuffle이 발생하는 연산**: (1) `groupBy().agg()` - 같은 키를 한 파티션으로 모음, (2) `join()` - 양쪽 데이터를 키로 재분배, (3) `repartition()` - 명시적 재파티셔닝, (4) `distinct()` - 중복 제거를 위해 재분배, (5) `orderBy()` - 전역 정렬. **최소화 방법**: (1) **Broadcast Join**: 작은 테이블(< 10MB)을 모든 워커에 복제하여 Shuffle 회피.
> ```python
> from pyspark.sql.functions import broadcast
> large_df.join(broadcast(small_df), "key")  # Shuffle 없음
> ```
> (2) **파티션 키 일치**: 이미 같은 키로 파티셔닝된 데이터끼리 join하면 Shuffle 감소. (3) **집계 전 필터링**: `WHERE` 조건을 먼저 적용하여 Shuffle할 데이터 양 감소. (4) **Bucket Join**: 테이블을 같은 키로 미리 버켓팅. (5) **coalesce() 사용**: `repartition()` 대신 Shuffle 없는 `coalesce()`로 파티션 감소.

5. Spark의 메모리 관리 구조(Execution Memory vs Storage Memory)는?

> **답변**: Spark는 JVM Heap 메모리를 **Execution Memory**와 **Storage Memory**로 나눕니다. **Execution Memory**: Shuffle, Join, Sort 같은 연산의 중간 결과를 저장하는 공간. 부족하면 디스크로 Spill 발생. **Storage Memory**: cache(), persist()로 저장된 RDD/DataFrame을 보관하는 공간. **메모리 구조** (Spark 2.0+ Unified Memory):
> ```
> JVM Heap (예: 10GB)
> ├── Reserved Memory: 300MB (시스템 예약)
> ├── User Memory: (1 - spark.memory.fraction) × (Heap - 300MB)
> │   └── 사용자 데이터 구조, UDF 변수
> └── Spark Memory: spark.memory.fraction × (Heap - 300MB)
>     ├── Execution Memory: 동적 할당
>     └── Storage Memory: 동적 할당 (spark.memory.storageFraction)
> ```
> **핵심 포인트**: (1) Unified Memory에서는 Execution과 Storage가 서로의 빈 공간을 빌려 사용 가능. (2) Execution이 부족하면 Storage를 강제로 evict 가능. (3) `spark.memory.fraction` 기본값 0.6 (60%가 Spark Memory). OOM 발생 시 이 비율을 조정하거나 클러스터 메모리를 증가시킵니다.

6. Adaptive Query Execution(AQE)의 장점은?

> **답변**: **AQE(Adaptive Query Execution)**는 Spark 3.0+에서 도입된 기능으로, **런타임에 실행 계획을 동적으로 최적화**합니다. 기존에는 통계 기반 정적 최적화만 가능했지만, AQE는 실제 데이터를 보고 조정합니다. **주요 기능과 장점**: (1) **동적 파티션 병합(Coalescing Post-Shuffle Partitions)**: Shuffle 후 너무 작은 파티션들을 자동으로 합침. `spark.sql.shuffle.partitions=200`으로 시작해도 실제 데이터에 맞게 조정. (2) **동적 Join 전략 변경**: 한쪽 테이블이 예상보다 작으면 Sort-Merge Join → Broadcast Join으로 자동 변경. (3) **Skew Join 최적화**: 특정 키에 데이터가 몰린 경우(Data Skew) 자동 감지 후 해당 파티션을 분할. **설정 방법**:
> ```python
> spark.conf.set("spark.sql.adaptive.enabled", "true")  # 기본 활성화 (Spark 3.2+)
> spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
> spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
> ```
> AQE 덕분에 파티션 수 튜닝이나 Join 전략 선택에 덜 신경 써도 됩니다.

### 실무 시나리오
7. Spark 작업이 OOM(Out of Memory)으로 실패할 때 대처 방법은?

> **답변**: OOM은 Spark에서 가장 흔한 문제 중 하나입니다. **원인별 대처 방법**: (1) **Driver OOM**: collect(), toPandas() 등으로 대량 데이터를 Driver로 가져올 때 발생. → `take(n)`, `limit(n)` 사용, 또는 결과를 S3에 저장.
> ```python
> # 잘못된 예
> all_data = df.collect()  # 모든 데이터를 Driver로
>
> # 올바른 예
> df.write.parquet("s3://output/")  # 결과를 S3에 저장
> ```
> (2) **Executor OOM**: 파티션 데이터가 너무 크거나 Shuffle/Join이 큰 경우. → 파티션 수 증가 (`repartition(500)`), Executor 메모리 증가 (`--executor-memory 8g`). (3) **GC Overhead**: 작은 객체가 너무 많아 GC에 시간 소모. → 직렬화 방식 변경 (Kryo), `spark.memory.fraction` 조정. (4) **Data Skew**: 특정 키에 데이터 편중. → salting(키에 랜덤 값 추가), AQE 활성화. **디버깅 방법**: Spark UI에서 Stage별 메모리 사용량, Spill 크기, Task 실행 시간 분포 확인.

8. 스키마 진화(Schema Evolution)를 Spark에서 어떻게 처리하나요?

> **답변**: **Schema Evolution**은 시간이 지나면서 데이터 스키마가 변경되는 것을 처리하는 방법입니다. 모바일 앱 로그는 앱 버전마다 스키마가 다를 수 있어 특히 중요합니다. **Spark에서의 처리 방법**: (1) **mergeSchema 옵션**: Parquet 파일들의 스키마를 자동으로 병합.
> ```python
> df = spark.read.option("mergeSchema", "true").parquet("s3://bucket/data/")
> ```
> (2) **Schema 명시**: 읽을 때 명시적으로 스키마 지정, 없는 필드는 null.
> ```python
> from pyspark.sql.types import StructType, StructField, StringType
> schema = StructType([
>     StructField("user_id", StringType(), True),
>     StructField("event_name", StringType(), True),
>     StructField("new_field", StringType(), True)  # 구버전에는 없음
> ])
> df = spark.read.schema(schema).parquet("s3://...")
> ```
> (3) **Delta Lake/Iceberg 사용**: 테이블 레벨에서 스키마 진화를 자동 관리.
> ```python
> # Delta Lake
> df.write.format("delta").option("mergeSchema", "true").mode("append").save(path)
> ```
> (4) **전처리에서 통일**: ETL 단계에서 버전별 분기 처리 후 통일된 스키마로 저장.

9. Spark 작업의 성능을 모니터링하는 방법은?

> **답변**: Spark 성능 모니터링은 **Spark UI, 로그, 외부 도구**를 활용합니다. **(1) Spark UI (기본 포트 4040)**: Job/Stage/Task 단위 실행 시간, DAG 시각화, Shuffle 읽기/쓰기 크기, Executor별 메모리/GC 시간, SQL 탭에서 쿼리 실행 계획 확인. **주요 확인 포인트**:
> - Stage별 소요 시간: 병목 Stage 식별
> - Shuffle Read/Write: 크면 최적화 필요
> - Spill (Memory/Disk): 발생하면 메모리 부족
> - Task 시간 분포: 일부만 느리면 Data Skew
> **(2) Spark History Server**: 완료된 작업의 UI를 나중에 볼 수 있음. 프로덕션 환경 필수. **(3) 지표 수집**:
> ```
> spark.metrics.conf.*.sink.ganglia.class=org.apache.spark.metrics.sink.GangliaSink
> spark.metrics.conf.*.sink.prometheus.class=org.apache.spark.metrics.sink.PrometheusServlet
> ```
> **(4) AWS EMR 연동**: CloudWatch Metrics로 클러스터 상태, Ganglia로 상세 지표 확인. **(5) Datadog/Grafana 연동**: Spark Metrics를 시각화하여 대시보드 구성. 실무에서는 일일 ETL 작업의 실행 시간 추이를 모니터링하고, 급격한 증가 시 알림을 설정합니다.

---

## 8. 모바일 앱 로그 처리를 위한 Spark 활용

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    모바일 앱 로그의 Spark 처리 파이프라인                          │
└─────────────────────────────────────────────────────────────────────────────────┘

  S3 (Raw JSON)                      Spark Cluster                    S3 (Parquet)
       │                                  │                                │
       │   ┌───────────────────────────────────────────────────────┐      │
       │   │                 EMR Cluster (10 nodes)                │      │
       │   │                                                       │      │
       ▼   │   ┌─────────────────────────────────────────────┐    │      ▼
  ┌────────┴───┤              Driver Node                    │────┴────────┐
  │            │   • SparkContext 관리                       │             │
  │ dt=01-15/  │   • DAG 스케줄링                            │   dt=01-15/ │
  │ ├─ h=00/   │   • 결과 수집                               │   └─ *.parquet│
  │ ├─ h=01/   │   └─────────────────────────────────────────┘             │
  │ └─ h=23/   │                      │                                    │
  │            │                      ▼                                    │
  │            │   ┌─────────────────────────────────────────────┐        │
  │            │   │              Worker Nodes                    │        │
  │            │   │   ┌──────┐ ┌──────┐ ┌──────┐ ... ┌──────┐  │        │
  │            │   │   │ P1   │ │ P2   │ │ P3   │     │ P200 │  │        │
  │            │   │   │Filter│ │Filter│ │Filter│     │Filter│  │        │
  │            │   │   │Parse │ │Parse │ │Parse │     │Parse │  │        │
  │            │   │   │Enrich│ │Enrich│ │Enrich│     │Enrich│  │        │
  │            │   │   └──────┘ └──────┘ └──────┘     └──────┘  │        │
  │            │   │                                             │        │
  │            │   │   각 파티션이 병렬로 처리됨                   │        │
  │            │   └─────────────────────────────────────────────┘        │
  │            │                                                          │
  │            └──────────────────────────────────────────────────────────┘
  │                                                                        │
  └────────────────────────────────────────────────────────────────────────┘

  처리량: 50GB 원시 로그 → 10분 처리 → 15GB Parquet (70% 압축)
```

### 모바일 앱 DAU/MAU 계산 Spark 코드 예시

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import (
    col, countDistinct, date_trunc, when, sum as spark_sum,
    datediff, current_date, collect_set, size, array_intersect
)

# SparkSession 생성
spark = SparkSession.builder \
    .appName("MobileAppMetrics") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.shuffle.partitions", "200") \
    .getOrCreate()

def calculate_dau_mau(start_date: str, end_date: str):
    """일별 DAU와 MAU 계산"""

    # 1. 이벤트 데이터 로드 (파티션 프루닝)
    events_df = spark.read.parquet("s3://data-lake/processed/events/") \
        .filter(
            (col("event_date") >= start_date) &
            (col("event_date") <= end_date)
        )

    # 2. DAU 계산 (일별 고유 사용자 수)
    dau_df = events_df \
        .groupBy("event_date") \
        .agg(countDistinct("user_id").alias("dau")) \
        .orderBy("event_date")

    # 3. MAU 계산 (월별 고유 사용자 수)
    mau_df = events_df \
        .withColumn("month", date_trunc("month", col("event_date"))) \
        .groupBy("month") \
        .agg(countDistinct("user_id").alias("mau")) \
        .orderBy("month")

    # 4. 스티키니스 (DAU/MAU 비율)
    daily_metrics = dau_df \
        .withColumn("month", date_trunc("month", col("event_date"))) \
        .join(mau_df, "month") \
        .withColumn("stickiness", col("dau") / col("mau") * 100) \
        .select("event_date", "dau", "mau", "stickiness")

    return daily_metrics

def calculate_retention_cohort(cohort_date: str, days: int = 7):
    """코호트 기반 리텐션 계산"""

    events_df = spark.read.parquet("s3://data-lake/processed/events/")

    # 코호트 정의 (특정 날짜에 첫 방문한 사용자)
    cohort_users = events_df \
        .groupBy("user_id") \
        .agg(spark.sql.functions.min("event_date").alias("first_visit")) \
        .filter(col("first_visit") == cohort_date) \
        .select("user_id")

    cohort_size = cohort_users.count()

    # D1~D7 리텐션 계산
    retention_results = []

    for day in range(1, days + 1):
        target_date = f"DATE_ADD('{cohort_date}', {day})"

        retained_users = events_df \
            .filter(col("event_date") == eval(target_date)) \
            .join(cohort_users, "user_id", "inner") \
            .select("user_id").distinct().count()

        retention_rate = (retained_users / cohort_size) * 100
        retention_results.append({
            "day": f"D{day}",
            "retained_users": retained_users,
            "retention_rate": round(retention_rate, 2)
        })

    return spark.createDataFrame(retention_results)

# 실행
dau_mau = calculate_dau_mau("2024-01-01", "2024-01-31")
dau_mau.show(10)
dau_mau.write.mode("overwrite").parquet("s3://data-lake/curated/daily_metrics/")
```

---

## 9. 실무에서 자주 겪는 Spark 문제와 해결책

### 문제 1: Data Skew (데이터 편향)

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 대부분 Task는 빨리 끝나는데 일부만 오래 걸림 | 특정 키에 데이터 집중 | Salting, AQE 활성화 |
| Spark UI에서 Task 시간 편차가 큼 | Join/GroupBy 키 불균형 | Broadcast Join, 키 분리 |

```python
# Salting 기법으로 Data Skew 해결
from pyspark.sql.functions import concat, lit, rand, floor

SALT_BUCKETS = 10

# 원래 코드 (Skew 발생 가능)
# df.groupBy("country").count()

# Salting 적용
df_salted = df.withColumn(
    "salted_key",
    concat(col("country"), lit("_"), floor(rand() * SALT_BUCKETS))
)

# Salted key로 집계 후 원래 키로 재집계
result = df_salted \
    .groupBy("salted_key") \
    .count() \
    .withColumn("country", split(col("salted_key"), "_")[0]) \
    .groupBy("country") \
    .agg(spark_sum("count").alias("total_count"))
```

### 문제 2: Spark 작업이 느림

| 증상 | 원인 | 해결책 |
|------|------|--------|
| Shuffle 크기가 큼 | 불필요한 컬럼 포함 | select()로 필요한 컬럼만 |
| Full Scan 발생 | 파티션 필터 누락 | 파티션 컬럼으로 WHERE 조건 |
| 반복 연산 | 캐시 미사용 | cache()/persist() 활용 |

```python
# 최적화 전
df = spark.read.parquet("s3://data-lake/events/")  # 전체 파티션 스캔
result = df.filter(col("event_date") == "2024-01-15").groupBy(...).agg(...)

# 최적화 후
df = spark.read.parquet("s3://data-lake/events/") \
    .filter(col("event_date") == "2024-01-15")  # 파티션 프루닝

# 여러 번 사용되는 DataFrame은 캐시
df_filtered = df.select("user_id", "event_name", "event_date").cache()
df_filtered.count()  # 캐시 실행

# 이후 여러 연산에서 재사용
result1 = df_filtered.groupBy("event_name").count()
result2 = df_filtered.groupBy("user_id").count()
```

### 문제 3: 클러스터 리소스 낭비

| 증상 | 원인 | 해결책 |
|------|------|--------|
| Executor가 대기 상태 | 파티션 수 < Executor 수 | 파티션 수 증가 |
| 메모리는 남는데 느림 | 코어 활용 부족 | executor-cores 조정 |
| 작업 간 리소스 경합 | 단일 큰 클러스터 공유 | 작업별 클러스터 분리 |

```bash
# EMR 클러스터 설정 최적화
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --num-executors 20 \
  --executor-cores 4 \
  --executor-memory 8g \
  --driver-memory 4g \
  --conf spark.dynamicAllocation.enabled=true \
  --conf spark.dynamicAllocation.maxExecutors=50 \
  --conf spark.sql.adaptive.enabled=true \
  my_etl_job.py
```

---

## 참고 자료

- [Apache Spark 공식 문서](https://spark.apache.org/docs/latest/)
- [PySpark API 레퍼런스](https://spark.apache.org/docs/latest/api/python/)
- [Spark The Definitive Guide (Book)](https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/)
- [Delta Lake 공식 문서](https://docs.delta.io/latest/index.html)
- [AWS EMR 모범 사례](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-ha.html)
