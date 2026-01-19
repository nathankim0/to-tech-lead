# ETL (Extract, Transform, Load)

## 1. 한 줄 요약

원시(raw) 데이터를 **추출(Extract)**하고, **변환(Transform)**하여, 분석 가능한 형태로 **적재(Load)**하는 데이터 처리 프로세스입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점의 비유

**요리 과정**을 생각해보세요.

| ETL 단계 | 요리 과정 | 모바일 데이터 예시 |
|----------|----------|-------------------|
| **Extract** | 재료 구매 | S3에서 JSON 로그 읽기 |
| **Transform** | 손질 & 조리 | 파싱, 정제, 집계 |
| **Load** | 그릇에 담기 | 웨어하우스에 저장 |

마트에서 산 재료(원시 데이터)를 그대로 먹지 않듯이, 로그 데이터도 가공해야 의미 있는 분석이 가능합니다.

### 모바일 앱 로그의 ETL 예시

```
[Extract] 원시 로그
{
  "e": "btn_clk",
  "ts": 1705312200000,
  "p": {"btn": "buy", "pid": "SKU123"}
}

     ↓ Transform (정규화, 타입 변환, 보강)

[Transform] 정제된 데이터
{
  "event_name": "button_click",
  "event_date": "2024-01-15",
  "event_timestamp": "2024-01-15T10:30:00Z",
  "button_id": "buy",
  "product_id": "SKU123",
  "product_name": "무선 이어폰",        ← 상품 테이블 Join
  "product_category": "electronics"    ← 상품 테이블 Join
}

     ↓ Load (웨어하우스 적재)

[Load] 분석 테이블
┌─────────────┬────────────┬─────────────┬──────────────┐
│ event_date  │ event_name │ product_id  │ click_count  │
├─────────────┼────────────┼─────────────┼──────────────┤
│ 2024-01-15  │ btn_click  │ SKU123      │ 1,234        │
└─────────────┴────────────┴─────────────┴──────────────┘
```

---

## 3. 구조 다이어그램

### ETL 파이프라인 전체 구조

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ETL PIPELINE ARCHITECTURE                        │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  DATA SOURCES                                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐                │
│  │ App Logs │  │ DB Dump  │  │ API Data │  │ 3rd Party│                │
│  │  (S3)    │  │  (RDS)   │  │ (REST)   │  │(GA, FB)  │                │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘                │
│       │             │             │             │                        │
└───────┼─────────────┼─────────────┼─────────────┼────────────────────────┘
        │             │             │             │
        └─────────────┴──────┬──────┴─────────────┘
                             │
                    ┌────────▼────────┐
                    │    EXTRACT      │
                    │   (데이터 추출)   │
                    │  ┌────────────┐ │
                    │  │ Connectors │ │
                    │  │ - S3 Reader│ │
                    │  │ - JDBC     │ │
                    │  │ - REST API │ │
                    │  └────────────┘ │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   TRANSFORM     │
                    │   (데이터 변환)   │
                    │                 │
                    │  ┌────────────┐ │
                    │  │ - 파싱      │ │
                    │  │ - 정제      │ │
                    │  │ - 검증      │ │
                    │  │ - 집계      │ │
                    │  │ - Join      │ │
                    │  │ - 익명화    │ │
                    │  └────────────┘ │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │      LOAD       │
                    │   (데이터 적재)   │
                    │                 │
                    │  ┌────────────┐ │
                    │  │ - Insert   │ │
                    │  │ - Upsert   │ │
                    │  │ - Append   │ │
                    │  └────────────┘ │
                    └────────┬────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────────┐
│  DATA TARGETS                                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│  │ Data Lake    │  │ Data         │  │ Feature      │                  │
│  │ (S3 Parquet) │  │ Warehouse    │  │ Store        │                  │
│  │              │  │ (Redshift)   │  │ (Redis)      │                  │
│  └──────────────┘  └──────────────┘  └──────────────┘                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### ETL vs ELT 비교

```
┌──────────────────────────────────────────────────────────────────┐
│                      ETL vs ELT 비교                              │
├───────────────────────────┬──────────────────────────────────────┤
│          ETL              │              ELT                      │
├───────────────────────────┼──────────────────────────────────────┤
│                           │                                       │
│  Source ──▶ [Transform]   │  Source ──▶ Storage ──▶ [Transform] │
│             ──▶ Storage   │                                       │
│                           │                                       │
│  • 변환 후 저장            │  • 저장 후 변환                        │
│  • 스토리지 비용 절약       │  • 원본 데이터 보존                    │
│  • 전통적 방식             │  • 현대적 방식 (클라우드)              │
│  • Spark, Airflow         │  • dbt, Redshift                     │
│                           │                                       │
└───────────────────────────┴──────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 4.1 Airflow DAG 예시 (ETL 워크플로우)

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# DAG 기본 설정
default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

# DAG 정의
dag = DAG(
    'mobile_app_logs_etl',
    default_args=default_args,
    description='모바일 앱 로그 ETL 파이프라인',
    schedule_interval='0 2 * * *',  # 매일 새벽 2시
    start_date=datetime(2024, 1, 1),
    catchup=False,
)

# Task 정의
def extract_logs(**context):
    """S3에서 전날 로그 추출"""
    execution_date = context['ds']
    s3_path = f"s3://logs-bucket/raw/dt={execution_date}/"
    # 로그 파일 목록 조회 및 다운로드
    return s3_path

def transform_logs(**context):
    """로그 정제 및 변환"""
    # 1. JSON 파싱
    # 2. 스키마 검증
    # 3. 타임스탬프 정규화
    # 4. 상품 정보 Join
    # 5. PII 마스킹
    pass

def load_to_warehouse(**context):
    """Redshift에 적재"""
    # COPY 명령으로 벌크 적재
    pass

# Task 연결
extract = PythonOperator(
    task_id='extract_logs',
    python_callable=extract_logs,
    dag=dag,
)

transform = PythonOperator(
    task_id='transform_logs',
    python_callable=transform_logs,
    dag=dag,
)

load = PythonOperator(
    task_id='load_to_warehouse',
    python_callable=load_to_warehouse,
    dag=dag,
)

# 의존성 설정
extract >> transform >> load
```

### 4.2 Transform 로직 상세 예시

```python
import pandas as pd
from datetime import datetime

def transform_mobile_logs(raw_df: pd.DataFrame) -> pd.DataFrame:
    """모바일 앱 로그 변환 함수"""

    # 1. 컬럼명 정규화
    column_mapping = {
        'e': 'event_name',
        'ts': 'event_timestamp',
        'uid': 'user_id',
        'did': 'device_id',
        'p': 'params'
    }
    df = raw_df.rename(columns=column_mapping)

    # 2. 타임스탬프 변환 (밀리초 → datetime)
    df['event_timestamp'] = pd.to_datetime(
        df['event_timestamp'],
        unit='ms',
        utc=True
    )
    df['event_date'] = df['event_timestamp'].dt.date

    # 3. 이벤트명 매핑
    event_mapping = {
        'btn_clk': 'button_click',
        'scr_view': 'screen_view',
        'purch': 'purchase',
    }
    df['event_name'] = df['event_name'].map(event_mapping)

    # 4. 파라미터 추출 (JSON → 컬럼)
    df['screen_name'] = df['params'].apply(
        lambda x: x.get('screen') if isinstance(x, dict) else None
    )
    df['product_id'] = df['params'].apply(
        lambda x: x.get('pid') if isinstance(x, dict) else None
    )

    # 5. 데이터 품질 검증
    df = df[df['event_name'].notna()]  # 유효한 이벤트만
    df = df[df['user_id'].notna()]     # user_id 필수

    # 6. 중복 제거
    df = df.drop_duplicates(
        subset=['user_id', 'event_timestamp', 'event_name']
    )

    return df
```

### 4.3 일반적인 Transform 패턴

```
┌─────────────────────────────────────────────────────────────────┐
│                   COMMON TRANSFORM PATTERNS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. 정규화 (Normalization)                                       │
│     ┌──────────────────┐      ┌──────────────────────┐         │
│     │ "btn_clk"        │ ──▶  │ "button_click"       │         │
│     │ "BTN_CLK"        │ ──▶  │ "button_click"       │         │
│     └──────────────────┘      └──────────────────────┘         │
│                                                                  │
│  2. 타입 변환 (Type Casting)                                     │
│     ┌──────────────────┐      ┌──────────────────────┐         │
│     │ "1705312200000"  │ ──▶  │ 2024-01-15 10:30:00  │         │
│     │ (String/Long)    │      │ (Timestamp)          │         │
│     └──────────────────┘      └──────────────────────┘         │
│                                                                  │
│  3. 데이터 보강 (Enrichment)                                     │
│     ┌──────────────────┐      ┌──────────────────────┐         │
│     │ product_id:      │      │ product_id: "SKU123" │         │
│     │ "SKU123"         │ ──▶  │ product_name: "이어폰"│         │
│     │                  │ JOIN │ category: "전자제품"  │         │
│     └──────────────────┘      └──────────────────────┘         │
│                                                                  │
│  4. 집계 (Aggregation)                                           │
│     ┌──────────────────┐      ┌──────────────────────┐         │
│     │ event1, event2,  │      │ date: 2024-01-15     │         │
│     │ event3, event4,  │ ──▶  │ event_count: 1000    │         │
│     │ ...              │ SUM  │ unique_users: 500    │         │
│     └──────────────────┘      └──────────────────────┘         │
│                                                                  │
│  5. 익명화 (Anonymization)                                       │
│     ┌──────────────────┐      ┌──────────────────────┐         │
│     │ email:           │      │ email_hash:          │         │
│     │ "user@mail.com"  │ ──▶  │ "a1b2c3d4..."        │         │
│     │ phone: "010-..." │ HASH │ phone: null          │         │
│     └──────────────────┘      └──────────────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. 장단점

### 배치 ETL vs 실시간 ETL

| 항목 | 배치 ETL | 실시간 ETL (Streaming) |
|------|----------|----------------------|
| **지연** | 시간~일 단위 | 초~분 단위 |
| **복잡도** | 낮음 | 높음 |
| **비용** | 저렴 | 비쌈 |
| **사용 사례** | 일일 리포트 | 실시간 대시보드 |
| **도구** | Airflow, dbt | Spark Streaming, Flink |
| **오류 처리** | 재실행 용이 | 복잡 |

### ETL 도구 비교

| 도구 | 장점 | 단점 | 적합한 경우 |
|------|------|------|------------|
| **Airflow** | 유연, 커스텀 가능 | 러닝커브 높음 | 복잡한 워크플로우 |
| **dbt** | SQL 기반, 버전관리 | Transform만 지원 | ELT 환경 |
| **Glue** | AWS 통합, 서버리스 | 벤더 종속 | AWS 환경 |
| **Fivetran** | 노코드, 빠른 구축 | 비용 높음 | 빠른 시작 |

---

## 6. 내 생각

```
(학습 후 자신의 생각을 정리해보세요)

- 현재 우리 팀의 ETL 파이프라인 구조는?

- 가장 시간이 오래 걸리는 Transform 작업은?

- 데이터 품질 이슈가 자주 발생하는 지점은?

```

---

## 7. 추가 질문

### 기초 레벨
1. ETL과 ELT의 차이점은 무엇이고, 언제 각각을 사용하나요?

> **답변**: **ETL(Extract-Transform-Load)**은 데이터를 추출한 후 별도의 처리 서버(Spark 등)에서 변환하고 최종 저장소에 적재하는 전통적인 방식입니다. 반면 **ELT(Extract-Load-Transform)**는 먼저 원본 데이터를 데이터 레이크/웨어하우스에 적재한 후, 그 안에서 SQL(dbt 등)로 변환하는 현대적인 방식입니다. **ETL 사용 시기**: (1) 데이터 정제가 복잡하여 전용 처리 엔진이 필요할 때, (2) 원본 데이터에 민감 정보가 있어 저장 전 제거해야 할 때, (3) 저장 공간을 절약해야 할 때. **ELT 사용 시기**: (1) 클라우드 웨어하우스(BigQuery, Snowflake)의 강력한 연산 능력을 활용할 때, (2) 원본 데이터를 보존하며 다양한 분석 요구에 대응해야 할 때, (3) dbt 같은 SQL 기반 도구로 비즈니스 로직을 관리할 때. 최근 추세는 클라우드 스토리지 비용 하락과 웨어하우스 성능 향상으로 ELT가 더 많이 사용됩니다.

2. 배치 처리와 실시간 처리의 적절한 사용 사례는?

> **답변**: **배치 처리**는 대량의 데이터를 정해진 주기(시간/일/주)에 한꺼번에 처리하는 방식이고, **실시간(스트리밍) 처리**는 데이터가 발생하는 즉시 처리하는 방식입니다. **배치 처리 적합 사례**: (1) 일일 매출 리포트, 주간 KPI 대시보드 같은 정기 보고서, (2) 사용자 세그먼트 계산, 추천 모델 학습 같이 전체 데이터가 필요한 작업, (3) 비용 최적화가 중요한 경우 (배치가 저렴). **실시간 처리 적합 사례**: (1) 실시간 이상 탐지(결제 사기, 서버 장애), (2) 라이브 대시보드(현재 접속자 수, 실시간 매출), (3) 이벤트 기반 트리거(장바구니 이탈 알림, 실시간 개인화). 모바일 앱에서는 대부분의 분석은 배치로 충분하지만, 결제 사기 탐지나 실시간 푸시 발송 같은 기능은 실시간 처리가 필요합니다. Lambda Architecture(배치 + 실시간 병행)나 Kappa Architecture(실시간 단일)를 고려할 수 있습니다.

3. Transform 단계에서 가장 흔한 작업은 무엇인가요?

> **답변**: Transform 단계에서 자주 수행하는 작업들은 다음과 같습니다. **(1) 데이터 정제(Cleansing)**: null 값 처리, 중복 제거, 이상치 필터링. 예: `WHERE event_name IS NOT NULL`. **(2) 타입 변환(Type Casting)**: 문자열 타임스탬프를 datetime으로, 문자열 숫자를 numeric으로 변환. 예: 밀리초 `1705312200000`을 `2024-01-15 10:30:00`으로. **(3) 필드 정규화(Normalization)**: 이벤트명 통일(`btn_clk` → `button_click`), 대소문자 표준화. **(4) 데이터 보강(Enrichment)**: 다른 테이블과 JOIN하여 정보 추가. 예: product_id로 상품명, 카테고리 조인. **(5) 집계(Aggregation)**: 일별/사용자별 이벤트 카운트, 합계, 평균 계산. **(6) 파생 필드 생성**: 기존 필드로 새 필드 계산. 예: `event_timestamp`에서 `event_hour`, `day_of_week` 추출. **(7) 익명화/마스킹**: 개인정보 해싱, IP 대역 마스킹.

### 심화 레벨
4. 멱등성(Idempotency)이 ETL에서 왜 중요한가요?

> **답변**: **멱등성**은 동일한 작업을 여러 번 실행해도 결과가 같은 성질입니다. ETL에서 중요한 이유는 크게 세 가지입니다. **(1) 재실행 안전성**: ETL 작업이 중간에 실패했을 때 처음부터 다시 실행해도 데이터가 중복되거나 손상되지 않습니다. **(2) 백필(Backfill) 용이성**: 과거 데이터를 재처리해야 할 때 기존 데이터를 덮어쓰기만 하면 됩니다. **(3) 운영 단순화**: 데이터 정합성 문제 발생 시 "일단 재실행"이 가능해집니다. **멱등성 구현 방법**: (1) `INSERT` 대신 `MERGE/UPSERT` 사용: 키가 있으면 업데이트, 없으면 삽입. (2) **파티션 덮어쓰기**: 특정 날짜 파티션 전체를 삭제 후 재생성. (3) `DELETE` + `INSERT` 조합: 같은 트랜잭션에서 기존 데이터 삭제 후 삽입. 예를 들어 일별 ETL에서 `DELETE FROM table WHERE dt = '2024-01-15'; INSERT INTO table SELECT ... WHERE dt = '2024-01-15';` 패턴을 사용합니다.

5. Late arriving data는 어떻게 처리해야 하나요?

> **답변**: **Late arriving data(지연 도착 데이터)**는 이벤트 발생 시간과 서버 수신 시간에 차이가 있는 데이터입니다. 모바일 앱에서 특히 흔한데, 오프라인 상태에서 발생한 이벤트가 나중에 전송되는 경우입니다. **처리 방법**: **(1) 이벤트 시간 vs 처리 시간 구분**: 파티션 키로 `event_timestamp`(이벤트 발생 시간)와 `processed_date`(처리 시간) 둘 다 저장. **(2) 허용 윈도우 설정**: 예를 들어 7일 이내 지연 데이터는 수용, 그 이후는 별도 처리. **(3) 재처리 파이프라인**: Late data가 도착하면 해당 날짜의 집계 테이블을 재계산. **(4) Incremental 업데이트**: dbt의 incremental 모델처럼 새 데이터만 추가하고 정기적으로 전체 재계산. **실무 팁**: 스트리밍 시스템(Flink, Spark Streaming)에서는 Watermark를 설정하여 얼마나 늦은 데이터까지 기다릴지 정의합니다. 보통 모바일 앱은 24~48시간의 late 데이터 허용 윈도우를 설정합니다.

6. Schema evolution은 어떻게 관리하나요?

> **답변**: **Schema evolution**은 데이터 스키마가 시간에 따라 변경되는 것을 관리하는 방법입니다. **관리 전략**: **(1) 버전 관리**: 스키마 버전 필드(`schema_version`)를 데이터에 포함시켜 Consumer가 버전별로 다르게 파싱. **(2) Schema Registry 사용**: Confluent Schema Registry나 AWS Glue Schema Registry로 스키마 버전을 중앙 관리하고 호환성 검사 자동화. **(3) 호환성 규칙 적용**: Avro/Protobuf의 forward/backward compatibility 규칙 준수. 새 필드 추가는 허용, 기존 필드 삭제/타입 변경은 금지. **(4) ETL에서 유연한 파싱**: JSON 파싱 시 존재하지 않는 필드는 null로 처리, 새 필드는 optional로 처리. **dbt에서의 처리**:
> ```sql
> -- 버전별 분기 처리
> CASE
>   WHEN schema_version >= '2.0' THEN payment_method
>   ELSE 'unknown'
> END AS payment_method
> ```
> 앱 버전이 다양하게 공존하므로 최소 1년간 이전 스키마를 지원하는 것이 안전합니다.

### 실무 시나리오
7. ETL 작업이 실패했을 때 알림을 어떻게 설정하나요?

> **답변**: ETL 실패 알림은 **모니터링 시스템과 알림 채널 통합**으로 구현합니다. **(1) Airflow**: `on_failure_callback`에 Slack/PagerDuty 알림 함수 등록, 이메일 알림은 `email_on_failure=True` 설정. **(2) AWS Step Functions/Glue**: CloudWatch Alarms를 설정하고 SNS로 알림 전송. **(3) dbt Cloud**: 내장 알림 기능으로 Slack/Email 연동. **알림에 포함할 정보**: (1) 실패한 DAG/Job 이름, (2) 실패 시간과 에러 메시지, (3) 실행 로그 링크, (4) 영향받는 테이블/파티션. **실무 팁**: 모든 실패에 알림을 보내면 알림 피로(Alert Fatigue)가 발생합니다. 심각도별로 분류하여 Critical은 즉시 알림(PagerDuty), Warning은 Slack 채널, Info는 이메일로 분리합니다. 또한 실패 후 자동 재시도(3회)를 설정하고, 모든 재시도 실패 시에만 알림을 보내면 일시적 오류로 인한 불필요한 알림을 줄일 수 있습니다.

8. 원본 데이터가 변경되면 이미 처리된 데이터는 어떻게 하나요?

> **답변**: 원본 데이터 변경 시 **Change Data Capture(CDC)**와 **재처리 전략**을 사용합니다. **(1) CDC 방식**: Debezium, AWS DMS 등으로 원본 DB의 변경 사항(INSERT/UPDATE/DELETE)을 실시간으로 캡처하여 파이프라인에 전달. 변경된 레코드만 처리하므로 효율적. **(2) Full Refresh**: 매일 전체 테이블을 덮어쓰기. 단순하지만 데이터가 크면 비효율적. **(3) Incremental + Merge**: 변경된 레코드를 `updated_at` 타임스탬프로 감지하고 MERGE/UPSERT로 처리. **(4) Soft Delete 처리**: 원본에서 삭제된 데이터는 웨어하우스에서 `is_deleted=true`로 마킹. **실무에서 주의할 점**: (1) 원본이 변경되면 하위 테이블(downstream)도 재계산이 필요할 수 있음, (2) 변경 이력이 중요하면 SCD Type 2(히스토리 보존)로 설계, (3) 일일 스냅샷을 별도로 유지하면 특정 시점 데이터 조회가 가능합니다.

9. GDPR 삭제 요청이 들어오면 ETL 파이프라인에서 어떻게 처리하나요?

> **답변**: GDPR의 "잊혀질 권리(Right to be Forgotten)" 요청은 **모든 데이터 저장소에서 해당 사용자 데이터 삭제**를 의미합니다. **처리 프로세스**: **(1) 삭제 요청 수신**: 사용자 식별자(user_id, email 등)를 담은 삭제 요청을 별도 큐/테이블에 저장. **(2) 데이터 레이크(S3) 처리**: Parquet 파일은 수정 불가하므로, 해당 파티션 전체를 다시 쓰거나 Delta Lake/Iceberg의 DELETE 기능 사용. **(3) 데이터 웨어하우스 처리**: `DELETE FROM table WHERE user_id = 'xxx'` 실행. **(4) 백업/아카이브 처리**: Glacier 등 아카이브에서도 삭제 필요. **(5) 감사 로그**: 삭제 완료 시간과 범위를 기록하여 컴플라이언스 증빙. **아키텍처 고려사항**: 처음부터 user_id를 중앙화하고, 개인정보는 별도 테이블에 분리(Pseudonymization)하면 삭제가 용이합니다. 예: 분석 테이블에는 해시된 user_key만 저장하고, user_key ↔ 실제 개인정보 매핑 테이블만 삭제하면 나머지 데이터는 익명화됩니다.

---

## 8. 모바일 앱 로그의 ETL 파이프라인 상세

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│               모바일 앱 로그 ETL: S3 → 웨어하우스 상세 흐름                        │
└─────────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────────────────────┐
  │  S3 Raw Layer (Bronze)                                                       │
  │  s3://data-lake/raw/app-logs/dt=2024-01-15/hour=10/                        │
  │                                                                              │
  │  원시 JSON 예시:                                                             │
  │  {"e":"btn_clk","ts":1705312200000,"uid":"u123","p":{"btn":"buy"}}          │
  │  {"e":"scr_view","ts":1705312205000,"uid":"u456","p":{"scr":"home"}}        │
  └─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ EXTRACT (매시간 Airflow DAG)
                                        ▼
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │  Spark Job: Extract                                                          │
  │                                                                              │
  │  df = spark.read.json("s3://data-lake/raw/app-logs/dt=2024-01-15/")        │
  │                                                                              │
  │  • S3 Select로 필요한 파티션만 읽기                                          │
  │  • 압축 해제 (gzip → raw)                                                    │
  │  • 기본 스키마 추론                                                          │
  └─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ TRANSFORM
                                        ▼
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │  Spark Job: Transform                                                        │
  │                                                                              │
  │  Step 1: 스키마 정규화                                                       │
  │  ┌─────────────────────────────┐    ┌─────────────────────────────┐        │
  │  │ {"e":"btn_clk",...}         │ -> │ {"event_name":"button_click" │        │
  │  │                             │    │  "event_timestamp":"2024-.."}│        │
  │  └─────────────────────────────┘    └─────────────────────────────┘        │
  │                                                                              │
  │  Step 2: 타임스탬프 변환                                                     │
  │  • 밀리초 → ISO 8601 datetime                                               │
  │  • event_date, event_hour 파생 필드 생성                                    │
  │                                                                              │
  │  Step 3: 데이터 품질 검사                                                    │
  │  • user_id IS NOT NULL                                                      │
  │  • event_timestamp 범위 검증 (미래 날짜 제외)                                │
  │  • 중복 제거 (event_id 기준)                                                 │
  │                                                                              │
  │  Step 4: 데이터 보강 (Enrichment)                                            │
  │  • product_id → 상품 마스터 테이블 JOIN → 상품명, 카테고리 추가             │
  │  • user_id → 사용자 테이블 JOIN → 사용자 세그먼트 추가                      │
  │                                                                              │
  │  Step 5: 익명화/마스킹                                                       │
  │  • email → SHA256 해시                                                       │
  │  • IP → /24 대역만 저장 (예: 192.168.1.xxx)                                 │
  └─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ LOAD
                                        ▼
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │  S3 Processed Layer (Silver)                                                 │
  │  s3://data-lake/processed/events/dt=2024-01-15/                            │
  │                                                                              │
  │  Parquet 형식으로 저장:                                                      │
  │  • 컬럼 기반 압축으로 70% 용량 절감                                          │
  │  • 스키마 내장으로 타입 안정성                                               │
  │  • 파티션 프루닝으로 쿼리 최적화                                             │
  └─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ dbt (일일 배치)
                                        ▼
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │  Data Warehouse - Mart Layer (Gold)                                          │
  │                                                                              │
  │  fact_events: 정제된 이벤트 팩트 테이블                                      │
  │  ┌──────────┬──────────┬──────────┬──────────┬──────────┐                  │
  │  │event_id  │event_date│user_id   │event_name│product_id│                  │
  │  ├──────────┼──────────┼──────────┼──────────┼──────────┤                  │
  │  │e_001     │2024-01-15│u_123     │purchase  │SKU001    │                  │
  │  └──────────┴──────────┴──────────┴──────────┴──────────┘                  │
  │                                                                              │
  │  agg_daily_events: 일별 집계 테이블                                         │
  │  ┌──────────┬──────────┬─────────────┬────────────┐                        │
  │  │event_date│event_name│event_count  │unique_users│                        │
  │  ├──────────┼──────────┼─────────────┼────────────┤                        │
  │  │2024-01-15│purchase  │12,345       │8,901       │                        │
  │  └──────────┴──────────┴─────────────┴────────────┘                        │
  └─────────────────────────────────────────────────────────────────────────────┘
```

### 실제 dbt 모델 예시

```sql
-- models/staging/stg_app_events.sql
-- Staging: 원시 데이터 정제

{{ config(materialized='view') }}

WITH source AS (
    SELECT * FROM {{ source('raw', 'app_logs') }}
    WHERE dt = '{{ var("execution_date") }}'
),

renamed AS (
    SELECT
        -- 고유 식별자 생성
        {{ dbt_utils.generate_surrogate_key(['user_id', 'event_timestamp', 'event_name']) }} AS event_id,

        -- 이벤트 정보
        CASE event_name
            WHEN 'btn_clk' THEN 'button_click'
            WHEN 'scr_view' THEN 'screen_view'
            WHEN 'purch' THEN 'purchase'
            ELSE event_name
        END AS event_name,

        -- 타임스탬프 변환
        TIMESTAMP_MILLIS(event_timestamp) AS event_timestamp,
        DATE(TIMESTAMP_MILLIS(event_timestamp)) AS event_date,

        -- 사용자/디바이스 정보
        user_id,
        device_id,

        -- 파라미터 추출
        JSON_EXTRACT_SCALAR(params, '$.screen_name') AS screen_name,
        JSON_EXTRACT_SCALAR(params, '$.product_id') AS product_id,
        SAFE_CAST(JSON_EXTRACT_SCALAR(params, '$.amount') AS FLOAT64) AS amount,

        -- 메타데이터
        dt AS partition_date,
        CURRENT_TIMESTAMP() AS _loaded_at

    FROM source
    WHERE user_id IS NOT NULL
      AND event_timestamp > 0
)

SELECT * FROM renamed
```

---

## 9. 실무에서 자주 겪는 ETL 문제와 해결책

### 문제 1: ETL 작업이 너무 오래 걸림

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 일일 ETL이 24시간 초과 | 데이터 양 증가, 비효율적 쿼리 | 증분 처리(Incremental)로 전환 |
| Spark 작업 지연 | Shuffle이 많은 연산 | Broadcast Join, 파티션 최적화 |
| 특정 시간대에 병목 | 리소스 경합 | 작업 스케줄 분산, 클러스터 확장 |

```python
# 비효율적 (Full Refresh)
df = spark.read.parquet("s3://data-lake/all-data/")  # 전체 읽기
result = df.filter(col("dt") == "2024-01-15")

# 효율적 (Partition Pruning)
df = spark.read.parquet("s3://data-lake/all-data/dt=2024-01-15/")  # 파티션만 읽기
```

### 문제 2: 데이터 품질 이슈

| 증상 | 원인 | 해결책 |
|------|------|--------|
| null 값이 예상보다 많음 | 앱 버전별 스키마 차이 | COALESCE로 기본값 설정 |
| 집계 수치가 맞지 않음 | 중복 데이터 | event_id 기반 중복 제거 |
| 미래 날짜 데이터 존재 | 클라이언트 시간 조작 | 서버 수신 시간으로 필터링 |

```sql
-- 데이터 품질 검사 쿼리 (dbt test)
-- tests/assert_no_future_events.sql
SELECT *
FROM {{ ref('stg_app_events') }}
WHERE event_date > CURRENT_DATE()
-- 결과가 0건이어야 테스트 통과
```

### 문제 3: 스키마 변경으로 인한 파이프라인 중단

| 증상 | 원인 | 해결책 |
|------|------|--------|
| JSON 파싱 에러 | 필드명 변경 | 여러 필드명을 COALESCE로 처리 |
| 타입 에러 | 숫자→문자열 변경 | SAFE_CAST로 안전하게 변환 |
| 새 필드 누락 | 구버전 앱 데이터 | optional 필드로 처리, 기본값 설정 |

```sql
-- 스키마 변경에 안전한 파싱
SELECT
    -- 필드명이 변경된 경우
    COALESCE(
        JSON_EXTRACT_SCALAR(params, '$.product_id'),
        JSON_EXTRACT_SCALAR(params, '$.pid'),
        JSON_EXTRACT_SCALAR(params, '$.item_id')
    ) AS product_id,

    -- 타입이 변경된 경우
    SAFE_CAST(
        COALESCE(amount, CAST(amount_string AS FLOAT64))
        AS FLOAT64
    ) AS amount

FROM raw_events
```

---

## 참고 자료

- [Apache Airflow 공식 문서](https://airflow.apache.org/docs/)
- [dbt 공식 문서](https://docs.getdbt.com/)
- [데이터 품질 관리 가이드](https://www.montecarlodata.com/blog-data-quality/)
- [Great Expectations - 데이터 검증 도구](https://greatexpectations.io/)
- [Apache Iceberg - 테이블 포맷](https://iceberg.apache.org/)
