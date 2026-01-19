# Redis

## 1. 한 줄 요약

Redis는 **인메모리 키-값 저장소**로, 캐시, 세션 관리, 실시간 데이터 처리에 사용되는 초고속 데이터베이스이다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

#### Redis = 책상 위 포스트잇

| 포스트잇 | Redis |
|:---|:---|
| 책상 위 (빠르게 접근) | 메모리 (RAM) |
| 자주 보는 정보 메모 | 캐시 데이터 |
| 시간 지나면 떼어냄 | TTL (만료 시간) |
| 새 정보로 교체 | 캐시 갱신 |

책상 위 포스트잇에 자주 찾는 전화번호를 적어두면 전화번호부를 뒤지는 것보다 빠릅니다. Redis도 마찬가지로 자주 조회하는 데이터를 메모리에 저장해 빠르게 응답합니다.

#### 캐시의 원리

```
캐시가 없는 경우                    캐시가 있는 경우
──────────────────────────────────────────────────────────────

    App                                App
     │                                  │
     │ 매번 DB 조회                      │ 캐시 확인
     │ (100ms)                          │ (1ms)
     ▼                                  ▼
  ┌──────┐                          ┌───────┐  히트! → 바로 응답
  │  DB  │                          │ Redis │
  └──────┘                          └───┬───┘
                                        │ 미스 → DB 조회 후 캐시 저장
                                        ▼
                                    ┌──────┐
                                    │  DB  │
                                    └──────┘

100번 요청 시:                       100번 요청 시:
100 × 100ms = 10,000ms              1 × 100ms + 99 × 1ms = 199ms
                                    (50배 빠름!)
```

### 왜 알아야 할까?

모바일 앱이 빠르게 느껴지는 이유 중 하나는 서버의 캐싱 전략입니다.
- API 응답 시간이 **10ms vs 100ms**인 것은 UX에 큰 차이
- **실시간 기능** (채팅, 알림)에 Redis가 자주 사용됨
- Rate Limiting, 랭킹, 세션 관리 등 **다양한 용도**로 활용

---

## 3. 구조 다이어그램

### Redis 데이터 타입

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Redis Data Types                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. String (문자열) - 가장 기본                                       │
│  ─────────────────────────────────────                              │
│     SET user:123:name "홍길동"                                       │
│     GET user:123:name  →  "홍길동"                                   │
│                                                                     │
│     용도: 캐시, 카운터, 세션 토큰                                     │
│                                                                     │
│  2. Hash (해시) - 객체 저장                                          │
│  ─────────────────────────────────────                              │
│     HSET user:123 name "홍길동" email "hong@example.com"             │
│     HGET user:123 name  →  "홍길동"                                  │
│     HGETALL user:123   →  {"name": "홍길동", "email": "..."}         │
│                                                                     │
│     용도: 사용자 프로필, 상품 정보 캐시                               │
│                                                                     │
│  3. List (리스트) - 순서가 있는 목록                                  │
│  ─────────────────────────────────────                              │
│     LPUSH queue:notification "알림1"                                │
│     LPUSH queue:notification "알림2"                                │
│     RPOP queue:notification  →  "알림1"                             │
│                                                                     │
│     용도: 메시지 큐, 최근 활동 로그                                   │
│                                                                     │
│  4. Set (집합) - 중복 없는 집합                                       │
│  ─────────────────────────────────────                              │
│     SADD article:123:likes "user:1" "user:2" "user:3"               │
│     SISMEMBER article:123:likes "user:1"  →  1 (존재함)             │
│     SCARD article:123:likes  →  3 (총 개수)                          │
│                                                                     │
│     용도: 좋아요, 태그, 팔로잉/팔로워                                 │
│                                                                     │
│  5. Sorted Set (정렬된 집합) - 점수로 정렬                            │
│  ─────────────────────────────────────                              │
│     ZADD leaderboard 100 "user:1" 200 "user:2" 150 "user:3"         │
│     ZREVRANGE leaderboard 0 2  →  ["user:2", "user:3", "user:1"]    │
│     ZRANK leaderboard "user:2"  →  0 (1등)                          │
│                                                                     │
│     용도: 랭킹, 타임라인, 우선순위 큐                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 캐시 전략 패턴

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Cache Patterns                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Cache-Aside (Lazy Loading)                                      │
│  ─────────────────────────────────────                              │
│                                                                     │
│      ┌─────┐                                                        │
│      │ App │                                                        │
│      └──┬──┘                                                        │
│         │ ① 캐시 조회                                                │
│         ▼                                                           │
│      ┌─────────┐                                                    │
│      │  Redis  │──── 히트 → 바로 반환                                │
│      └────┬────┘                                                    │
│           │ 미스                                                     │
│           ▼                                                         │
│      ┌─────────┐                                                    │
│      │   DB    │ ② DB 조회                                          │
│      └────┬────┘                                                    │
│           │                                                         │
│           ▼                                                         │
│      ┌─────────┐                                                    │
│      │  Redis  │ ③ 캐시 저장                                        │
│      └─────────┘                                                    │
│                                                                     │
│  장점: 실제 요청된 데이터만 캐싱                                      │
│  단점: 첫 요청은 느림 (Cache Miss)                                   │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  2. Write-Through                                                   │
│  ─────────────────────────────────────                              │
│                                                                     │
│      ┌─────┐                                                        │
│      │ App │                                                        │
│      └──┬──┘                                                        │
│         │ 쓰기 요청                                                  │
│         ▼                                                           │
│      ┌─────────┐     동시에     ┌─────────┐                         │
│      │  Redis  │ ◄───────────► │   DB    │                         │
│      └─────────┘    저장        └─────────┘                         │
│                                                                     │
│  장점: 캐시와 DB 항상 동기화                                          │
│  단점: 쓰기 지연 발생                                                │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  3. Write-Behind (Write-Back)                                       │
│  ─────────────────────────────────────                              │
│                                                                     │
│      ┌─────┐                                                        │
│      │ App │                                                        │
│      └──┬──┘                                                        │
│         │ ① 쓰기 요청                                                │
│         ▼                                                           │
│      ┌─────────┐                                                    │
│      │  Redis  │ ← 즉시 저장                                        │
│      └────┬────┘                                                    │
│           │ ② 비동기로 나중에                                        │
│           ▼                                                         │
│      ┌─────────┐                                                    │
│      │   DB    │ ← 배치 저장                                        │
│      └─────────┘                                                    │
│                                                                     │
│  장점: 쓰기 성능 최고                                                │
│  단점: 데이터 유실 위험                                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Redis Pub/Sub

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Redis Pub/Sub                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Publisher                                                         │
│   (서버)                                                            │
│      │                                                              │
│      │ PUBLISH chat:room:123 "안녕하세요!"                           │
│      ▼                                                              │
│   ┌──────────────────────────────────────────┐                      │
│   │                  Redis                    │                      │
│   │                                          │                      │
│   │    Channel: chat:room:123                │                      │
│   │                                          │                      │
│   └────────────┬────────────┬────────────────┘                      │
│                │            │                                        │
│       ┌────────┘            └────────┐                              │
│       ▼                              ▼                              │
│   ┌────────┐                    ┌────────┐                          │
│   │ User A │                    │ User B │                          │
│   │ (구독) │                    │ (구독) │                          │
│   └────────┘                    └────────┘                          │
│                                                                     │
│   SUBSCRIBE chat:room:123      SUBSCRIBE chat:room:123              │
│   → "안녕하세요!" 수신          → "안녕하세요!" 수신                  │
│                                                                     │
│   용도: 실시간 채팅, 알림, 이벤트 브로드캐스트                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 예시 1: API 응답 캐싱

```python
# Python + Redis 캐시 예시
import redis
import json
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

CACHE_TTL_SECONDS = 300  # 5분

def cache_response(key_prefix: str, ttl: int = CACHE_TTL_SECONDS):
    """API 응답 캐시 데코레이터"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 캐시 키 생성
            cache_key = f"{key_prefix}:{':'.join(str(a) for a in args)}"

            # 1. 캐시 확인
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # 2. 캐시 미스 - 실제 함수 실행
            result = await func(*args, **kwargs)

            # 3. 결과 캐싱
            redis_client.setex(
                cache_key,
                ttl,
                json.dumps(result, default=str)
            )

            return result
        return wrapper
    return decorator


# 사용 예시
@cache_response("product", ttl=600)
async def get_product(product_id: int):
    """상품 정보 조회 - 10분 캐시"""
    return await db.query(Product).filter(Product.id == product_id).first()


# 캐시 무효화
def invalidate_product_cache(product_id: int):
    """상품 정보 수정 시 캐시 삭제"""
    redis_client.delete(f"product:{product_id}")
```

### 예시 2: 세션 관리

```python
# 세션 기반 인증 with Redis
import uuid
from datetime import timedelta

SESSION_TTL = timedelta(days=7)

class SessionManager:
    def __init__(self):
        self.redis = redis.Redis(host='localhost', port=6379, db=1)

    def create_session(self, user_id: int, device_info: dict) -> str:
        """새 세션 생성"""
        session_id = str(uuid.uuid4())

        session_data = {
            'user_id': user_id,
            'device': device_info.get('device_name'),
            'ip': device_info.get('ip'),
            'created_at': datetime.now().isoformat()
        }

        # 세션 저장 (7일 TTL)
        self.redis.setex(
            f"session:{session_id}",
            SESSION_TTL,
            json.dumps(session_data)
        )

        # 사용자의 세션 목록에 추가 (다중 디바이스 로그인)
        self.redis.sadd(f"user:{user_id}:sessions", session_id)

        return session_id

    def get_session(self, session_id: str) -> dict | None:
        """세션 조회"""
        data = self.redis.get(f"session:{session_id}")
        return json.loads(data) if data else None

    def delete_session(self, session_id: str):
        """로그아웃 - 단일 세션 삭제"""
        session = self.get_session(session_id)
        if session:
            user_id = session['user_id']
            self.redis.delete(f"session:{session_id}")
            self.redis.srem(f"user:{user_id}:sessions", session_id)

    def delete_all_sessions(self, user_id: int):
        """모든 기기에서 로그아웃"""
        session_ids = self.redis.smembers(f"user:{user_id}:sessions")
        for session_id in session_ids:
            self.redis.delete(f"session:{session_id.decode()}")
        self.redis.delete(f"user:{user_id}:sessions")
```

### 예시 3: Rate Limiting

```python
# API Rate Limiting with Redis
from fastapi import HTTPException

RATE_LIMIT_REQUESTS = 100  # 요청 수
RATE_LIMIT_WINDOW = 60     # 초 (1분)

class RateLimiter:
    def __init__(self):
        self.redis = redis.Redis(host='localhost', port=6379, db=2)

    def is_allowed(self, identifier: str) -> tuple[bool, int]:
        """
        Rate limit 체크
        Returns: (허용 여부, 남은 요청 수)
        """
        key = f"rate_limit:{identifier}"
        current = self.redis.get(key)

        if current is None:
            # 첫 요청 - 카운터 시작
            self.redis.setex(key, RATE_LIMIT_WINDOW, 1)
            return True, RATE_LIMIT_REQUESTS - 1

        current_count = int(current)

        if current_count >= RATE_LIMIT_REQUESTS:
            # 제한 초과
            ttl = self.redis.ttl(key)
            return False, 0

        # 카운터 증가
        self.redis.incr(key)
        return True, RATE_LIMIT_REQUESTS - current_count - 1


# FastAPI 미들웨어로 적용
rate_limiter = RateLimiter()

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    client_ip = request.client.host
    allowed, remaining = rate_limiter.is_allowed(client_ip)

    if not allowed:
        raise HTTPException(
            status_code=429,
            detail="Too Many Requests",
            headers={"Retry-After": "60"}
        )

    response = await call_next(request)
    response.headers["X-RateLimit-Remaining"] = str(remaining)
    return response
```

### 예시 4: 실시간 랭킹

```python
# 게임 리더보드 with Redis Sorted Set

class Leaderboard:
    def __init__(self, game_id: str):
        self.redis = redis.Redis(host='localhost', port=6379, db=3)
        self.key = f"leaderboard:{game_id}"

    def update_score(self, user_id: str, score: int):
        """점수 업데이트 (더 높은 점수로만 갱신)"""
        current = self.redis.zscore(self.key, user_id)
        if current is None or score > current:
            self.redis.zadd(self.key, {user_id: score})

    def get_rank(self, user_id: str) -> int | None:
        """사용자 순위 조회 (0부터 시작)"""
        rank = self.redis.zrevrank(self.key, user_id)
        return rank + 1 if rank is not None else None

    def get_top(self, count: int = 10) -> list:
        """상위 N명 조회"""
        results = self.redis.zrevrange(
            self.key, 0, count - 1, withscores=True
        )
        return [
            {"rank": i + 1, "user_id": uid.decode(), "score": int(score)}
            for i, (uid, score) in enumerate(results)
        ]

    def get_around_me(self, user_id: str, count: int = 5) -> list:
        """내 주변 순위 조회"""
        my_rank = self.redis.zrevrank(self.key, user_id)
        if my_rank is None:
            return []

        start = max(0, my_rank - count)
        end = my_rank + count

        results = self.redis.zrevrange(
            self.key, start, end, withscores=True
        )
        return [
            {
                "rank": start + i + 1,
                "user_id": uid.decode(),
                "score": int(score),
                "is_me": uid.decode() == user_id
            }
            for i, (uid, score) in enumerate(results)
        ]


# 사용 예시
leaderboard = Leaderboard("game:weekly")
leaderboard.update_score("user:123", 15000)

top10 = leaderboard.get_top(10)
my_rank = leaderboard.get_rank("user:123")
around_me = leaderboard.get_around_me("user:123")
```

---

## 5. 장단점

### Redis 장점

| 장점 | 설명 |
|:---|:---|
| **초고속** | 메모리 기반으로 1ms 이하 응답 |
| **다양한 자료구조** | String, Hash, List, Set, Sorted Set 등 |
| **Atomic 연산** | INCR, DECR 등 원자적 연산 지원 |
| **TTL 지원** | 자동 만료로 캐시 관리 용이 |
| **Pub/Sub** | 실시간 메시징 지원 |
| **클러스터** | 수평 확장 가능 |

### Redis 단점

| 단점 | 설명 |
|:---|:---|
| **메모리 비용** | RAM은 SSD보다 비쌈 |
| **휘발성** | 기본적으로 재시작 시 데이터 유실 (RDB/AOF로 영속화 가능) |
| **단일 스레드** | CPU 바운드 작업에 제한 |
| **복잡한 쿼리 불가** | SQL처럼 복잡한 조회 불가능 |
| **메모리 한계** | 물리 메모리 이상 저장 불가 |

### 캐시 관련 주의사항

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cache 관련 주의사항                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. Cache Stampede (캐시 폭주)                                       │
│  ─────────────────────────────────────                              │
│  문제: 캐시 만료 순간 다수 요청이 동시에 DB 조회                       │
│                                                                     │
│       캐시 만료!                                                     │
│           │                                                         │
│    ┌──────┼──────┐                                                  │
│    ▼      ▼      ▼                                                  │
│   Req1  Req2   Req3  → 모두 DB 조회 → DB 과부하!                     │
│                                                                     │
│  해결:                                                               │
│  - Lock을 이용한 단일 갱신                                           │
│  - 캐시 만료 전 미리 갱신 (Background Refresh)                        │
│  - TTL에 랜덤 값 추가 (분산)                                         │
│                                                                     │
│  2. Cache Penetration (캐시 관통)                                    │
│  ─────────────────────────────────────                              │
│  문제: 존재하지 않는 데이터 반복 요청 → 매번 DB 조회                   │
│                                                                     │
│  해결:                                                               │
│  - Null 값도 캐싱 (짧은 TTL)                                         │
│  - Bloom Filter로 존재 여부 사전 체크                                 │
│                                                                     │
│  3. Cache Inconsistency (캐시 불일치)                                │
│  ─────────────────────────────────────                              │
│  문제: DB 데이터 변경 시 캐시와 불일치                                │
│                                                                     │
│  해결:                                                               │
│  - 데이터 변경 시 즉시 캐시 무효화                                    │
│  - 적절한 TTL 설정                                                   │
│  - Write-Through 패턴 사용                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. 내 생각

> 이 섹션은 학습 후 본인의 생각을 정리하는 공간입니다.

```
Q1. 현재 서비스에서 캐싱이 필요한 데이터는 무엇인가?


Q2. 모바일 앱에서 서버 캐시 외에 로컬 캐시는 어떻게 활용하고 있는가?


Q3. 캐시 무효화는 어떤 시점에 해야 할까?


```

---

## 7. 추가 질문

더 깊이 학습하기 위한 질문들입니다.

### 캐싱 전략
- [ ] Read-Through와 Cache-Aside의 차이점은?

> **답변**: **Cache-Aside(Lazy Loading)**는 애플리케이션이 직접 캐시를 관리합니다. 캐시 미스 시 앱이 DB에서 조회하고 캐시에 저장합니다. 대부분의 웹 서비스에서 사용하는 일반적인 패턴입니다. **Read-Through**는 캐시 계층이 자동으로 DB 조회를 처리합니다. 앱은 항상 캐시에만 요청하고, 캐시 미스 시 캐시가 알아서 DB에서 가져옵니다. **차이점**: Cache-Aside는 구현이 단순하고 제어가 쉬우나 코드 중복이 발생합니다. Read-Through는 코드가 깔끔하지만 캐시 계층 설정이 필요합니다(AWS ElastiCache의 DAX가 Read-Through 방식). **모바일 관점**: 앱 개발자 입장에서는 어떤 패턴이든 API 응답만 받으면 됩니다. 다만 Cache-Aside의 경우 첫 요청(cold cache)이 느릴 수 있고, Read-Through는 일정한 응답 시간을 기대할 수 있습니다.

- [ ] 캐시 워밍(Cache Warming)은 언제, 어떻게 해야 할까?

> **답변**: **캐시 워밍**은 서비스 시작 전에 미리 캐시를 채워두는 작업입니다. **필요한 시점**: (1) 서버 재시작 후 첫 요청들이 모두 캐시 미스로 느려지는 것 방지 (2) 새벽에 캐시가 만료되어 오전 피크 시간에 DB 부하 급증 방지 (3) 신규 피처 배포 후 관련 데이터 프리로딩. **워밍 방법**: (1) **Startup Hook**: 서버 시작 시 인기 상품, 자주 조회되는 설정 등을 캐싱 (2) **Scheduled Job**: 새벽 5시에 오늘의 추천 상품, 홈 배너 등을 미리 캐싱 (3) **이벤트 기반**: 상품 정보 변경 시 관련 캐시를 미리 갱신. **모바일 앱 관점**: 서버 배포 직후나 장애 복구 직후 앱이 갑자기 느려진다면 캐시 워밍이 안 된 것일 수 있습니다. 이런 상황을 대비해 앱에서 적절한 로딩 표시와 타임아웃 처리가 필요합니다.

```python
# 캐시 워밍 예시
class CacheWarmer:
    def __init__(self, redis: Redis, db: Database):
        self.redis = redis
        self.db = db

    async def warm_on_startup(self):
        """서버 시작 시 캐시 워밍"""
        # 1. 인기 상품 Top 100
        popular_products = await self.db.get_popular_products(limit=100)
        for product in popular_products:
            self.redis.setex(f"product:{product.id}", 3600, product.to_json())

        # 2. 카테고리 목록 (거의 안 변함)
        categories = await self.db.get_all_categories()
        self.redis.set("categories:all", json.dumps(categories))

        # 3. 앱 설정값
        config = await self.db.get_app_config()
        self.redis.set("app:config", json.dumps(config))
```

- [ ] Multi-tier 캐싱 (L1: 로컬, L2: Redis)은 어떻게 구현할까?

> **답변**: **Multi-tier 캐싱**은 여러 계층의 캐시를 두어 성능을 극대화합니다. **L1 (로컬 메모리)**: 각 서버 인스턴스의 메모리. 네트워크 호출 없이 가장 빠름. 서버별로 다른 값을 가질 수 있어 일관성 문제 가능. **L2 (Redis)**: 네트워크 호출 필요하지만 모든 서버가 공유하는 중앙 캐시. **조회 흐름**: L1 확인 → 미스 시 L2 확인 → 미스 시 DB 조회 → L2에 저장 → L1에 저장. **무효화 흐름**: 데이터 변경 시 L2 삭제 + Redis Pub/Sub으로 모든 서버의 L1 무효화 알림. **적합한 데이터**: 자주 조회되지만 거의 안 변하는 데이터 (앱 설정, 코드 테이블, 환율 정보). **모바일 앱에서의 유사 패턴**: 앱 내 메모리 캐시(L1)와 서버 캐시(L2)의 조합. URLSession의 URLCache, Coil/Glide의 메모리+디스크 캐시가 비슷한 개념입니다.

### 운영 관련
- [ ] Redis 메모리 사용량 모니터링은 어떻게 할까?

> **답변**: Redis는 메모리 기반이라 모니터링이 매우 중요합니다. **모니터링 방법**: (1) `INFO memory` 명령어로 used_memory, maxmemory 확인 (2) `MEMORY DOCTOR`로 메모리 문제 진단 (3) AWS CloudWatch, Datadog Redis Integration 사용 (4) Redis Insight GUI 도구 활용. **주요 지표**: `used_memory` (현재 사용량), `maxmemory` (최대 허용량), `mem_fragmentation_ratio` (조각화율, 1.5 이상이면 문제), `evicted_keys` (메모리 부족으로 삭제된 키 수). **알림 설정**: 메모리 사용률 80% 도달 시 알림, evicted_keys 급증 시 알림. **모바일 앱 관점**: Redis 메모리가 꽉 차면 캐시 키가 삭제되어 API 응답이 느려지거나, 세션이 사라져 강제 로그아웃될 수 있습니다. "갑자기 로그아웃되었어요" 문의가 급증하면 Redis 세션 eviction을 의심해보세요.

- [ ] maxmemory-policy 설정은 어떤 것을 선택해야 할까?

> **답변**: `maxmemory-policy`는 메모리가 가득 찼을 때 어떤 키를 삭제할지 결정합니다. **정책 종류**: (1) **noeviction**: 삭제 안 함, 메모리 부족 시 에러 반환 (세션/락 용도에 적합) (2) **allkeys-lru**: 모든 키 중 가장 오래 사용 안 된 키 삭제 (일반적인 캐시에 권장) (3) **volatile-lru**: TTL이 설정된 키 중 LRU 삭제 (4) **allkeys-random**: 랜덤 삭제 (5) **volatile-ttl**: TTL이 가장 짧은 키부터 삭제. **권장 설정**: 캐시 용도는 `allkeys-lru`, 세션 저장소는 `noeviction` (메모리를 충분히 확보), 랭킹/리더보드는 `noeviction` (데이터 유실 방지). **모바일 앱 관점**: `noeviction` 설정인데 메모리가 부족하면 API가 에러를 반환합니다. `allkeys-lru`면 인기 있는 데이터는 남고 덜 쓰이는 데이터가 삭제되어 캐시 히트율이 유지됩니다.

- [ ] Redis Sentinel과 Cluster의 차이점은?

> **답변**: **Redis Sentinel**은 **고가용성(HA)** 솔루션입니다. Master-Slave 구조에서 Master가 죽으면 Slave를 새 Master로 승격시킵니다. 데이터는 분산하지 않고 복제만 합니다. 소규모 서비스, 읽기 확장이 필요한 경우 적합합니다. **Redis Cluster**는 **수평 확장** 솔루션입니다. 데이터를 여러 노드에 샤딩하여 분산 저장합니다. 각 노드가 전체 데이터의 일부만 담당하고, 노드 추가로 용량과 처리량을 늘릴 수 있습니다. 대용량 데이터, 높은 처리량이 필요한 경우 적합합니다. **실무 선택 기준**: 데이터 10GB 이하 + 초당 10만 요청 이하 = Sentinel로 충분. 그 이상이면 Cluster 고려. AWS ElastiCache는 둘 다 지원합니다. **모바일 앱 관점**: 클라이언트 입장에서는 차이가 없습니다. 다만 Cluster 환경에서는 MULTI/EXEC 트랜잭션이 같은 슬롯의 키에서만 동작하는 제약이 있어 백엔드 설계에 영향을 줍니다.

### 심화
- [ ] Redis Stream은 무엇이고 Pub/Sub과 어떻게 다른가?

> **답변**: **Pub/Sub**은 실시간 메시지 브로드캐스팅입니다. 구독자가 없으면 메시지는 사라지고, 과거 메시지 조회가 불가능합니다. 실시간 채팅, 알림 브로드캐스트에 적합합니다. **Redis Stream**은 영속적인 메시지 로그입니다. 메시지가 저장되어 나중에 조회 가능하고, Consumer Group으로 메시지를 분배 처리할 수 있습니다. Kafka의 간소화 버전이라고 생각하면 됩니다. **차이점 요약**: Pub/Sub은 "지금 듣고 있는 사람에게 전달", Stream은 "기록 남기고 순서대로 처리". **사용 예시**: 채팅 = Pub/Sub (실시간 전달), 주문 이벤트 처리 = Stream (누락 방지, 재처리 가능). **모바일 앱 관점**: 실시간 채팅에서 앱이 백그라운드로 갔다가 돌아왔을 때, Pub/Sub만 쓰면 그 사이 메시지를 놓칩니다. Stream을 병행하면 마지막 읽은 위치부터 가져올 수 있습니다.

- [ ] Lua Script를 Redis에서 사용하는 이유는?

> **답변**: **Lua Script**는 여러 Redis 명령을 **원자적으로 실행**할 수 있게 합니다. Redis는 단일 스레드이므로 Lua 스크립트 실행 중에는 다른 명령이 끼어들지 않습니다. **필요한 경우**: (1) 재고 차감: "재고 확인 → 부족하면 실패, 충분하면 차감"을 원자적으로 (2) Rate Limiting: "현재 카운트 확인 → 증가 → TTL 설정"을 한 번에 (3) 분산 락: "키가 없으면 설정, 있으면 실패"를 원자적으로. **예시**: 재고가 1개 남았을 때 두 사용자가 동시에 구매하면? Lua 없이는 둘 다 재고 확인(1) → 둘 다 차감 시도 → 재고 -1이 될 수 있습니다. Lua로는 한 번에 하나만 성공합니다.

```lua
-- 재고 차감 Lua 스크립트
local stock = tonumber(redis.call('GET', KEYS[1]) or '0')
local requested = tonumber(ARGV[1])

if stock >= requested then
    redis.call('DECRBY', KEYS[1], requested)
    return 1  -- 성공
else
    return 0  -- 재고 부족
end
```

- [ ] Redis와 Memcached의 차이점은?

> **답변**: 둘 다 인메모리 캐시지만 중요한 차이가 있습니다. **데이터 구조**: Redis는 String, Hash, List, Set, Sorted Set 등 다양한 자료구조 지원. Memcached는 String(Key-Value)만 지원. **영속성**: Redis는 RDB/AOF로 디스크 저장 가능. Memcached는 순수 인메모리만 (재시작 시 모든 데이터 유실). **클러스터링**: Redis Cluster는 자동 샤딩+복제. Memcached는 클라이언트 샤딩만 (서버끼리 통신 없음). **기능**: Redis는 Pub/Sub, 트랜잭션, Lua Script, Stream 등 풍부한 기능. Memcached는 단순 캐시 기능만. **선택 기준**: 단순 캐시만 필요 + 극한의 성능 = Memcached. 다양한 자료구조 + 영속성 + 부가 기능 = Redis. **실무**: 대부분 Redis를 선택합니다. Memcached는 레거시 시스템이나 특수한 성능 요구사항에서 사용됩니다.

---

## 8. 모바일 앱에서의 캐싱 전략

서버의 Redis 캐시와 모바일 앱의 로컬 캐시는 함께 고려해야 합니다.

### 계층별 캐시 전략

```
┌─────────────────────────────────────────────────────────────────────┐
│                     3-Tier Caching Strategy                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   L1: 앱 메모리 캐시 (가장 빠름, 앱 종료 시 삭제)                      │
│   ─────────────────────────────────────────────                     │
│   • 현재 화면 데이터                                                 │
│   • 자주 조회되는 작은 데이터 (유저 프로필, 설정)                      │
│   • iOS: NSCache, Android: LruCache                                 │
│   • TTL: 5분~1시간                                                  │
│                                                                     │
│   L2: 앱 디스크 캐시 (앱 재시작 후에도 유지)                          │
│   ─────────────────────────────────────────────                     │
│   • 이미지, 썸네일                                                   │
│   • 오프라인 사용 데이터                                             │
│   • iOS: FileManager, Android: DiskLruCache                        │
│   • TTL: 1일~1주일                                                  │
│                                                                     │
│   L3: 서버 캐시 (Redis - 모든 사용자 공유)                            │
│   ─────────────────────────────────────────────                     │
│   • 인기 상품 목록, 카테고리                                         │
│   • API 응답 캐시                                                   │
│   • TTL: 설정에 따라 다름                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### iOS 캐시 구현 예시

```swift
// iOS - 간단한 메모리 + 디스크 캐시
actor CacheManager {
    private let memoryCache = NSCache<NSString, NSData>()
    private let fileManager = FileManager.default
    private let cacheDirectory: URL

    static let shared = CacheManager()

    init() {
        let paths = fileManager.urls(for: .cachesDirectory, in: .userDomainMask)
        cacheDirectory = paths[0].appendingPathComponent("AppCache")
        try? fileManager.createDirectory(at: cacheDirectory, withIntermediateDirectories: true)

        // 메모리 캐시 제한
        memoryCache.countLimit = 100
        memoryCache.totalCostLimit = 50 * 1024 * 1024  // 50MB
    }

    func get<T: Codable>(_ key: String, type: T.Type) -> T? {
        // 1. 메모리 캐시 확인
        if let data = memoryCache.object(forKey: key as NSString) {
            return try? JSONDecoder().decode(T.self, from: data as Data)
        }

        // 2. 디스크 캐시 확인
        let fileURL = cacheDirectory.appendingPathComponent(key.md5Hash)
        guard let data = try? Data(contentsOf: fileURL) else { return nil }

        // 만료 확인
        let attributes = try? fileManager.attributesOfItem(atPath: fileURL.path)
        if let modDate = attributes?[.modificationDate] as? Date,
           Date().timeIntervalSince(modDate) > 3600 {  // 1시간 지남
            try? fileManager.removeItem(at: fileURL)
            return nil
        }

        // 메모리에 올리기
        memoryCache.setObject(data as NSData, forKey: key as NSString)
        return try? JSONDecoder().decode(T.self, from: data)
    }

    func set<T: Codable>(_ value: T, forKey key: String) {
        guard let data = try? JSONEncoder().encode(value) else { return }

        // 메모리에 저장
        memoryCache.setObject(data as NSData, forKey: key as NSString)

        // 디스크에 저장
        let fileURL = cacheDirectory.appendingPathComponent(key.md5Hash)
        try? data.write(to: fileURL)
    }
}
```

### API 응답 캐시 with ETag

```kotlin
// Android - ETag를 활용한 조건부 요청
class ApiCacheInterceptor(
    private val cacheStore: CacheStore
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()

        // GET 요청만 캐싱
        if (request.method != "GET") {
            return chain.proceed(request)
        }

        val cacheKey = request.url.toString()
        val cachedResponse = cacheStore.get(cacheKey)

        // 캐시된 ETag가 있으면 조건부 요청
        val modifiedRequest = cachedResponse?.etag?.let { etag ->
            request.newBuilder()
                .header("If-None-Match", etag)
                .build()
        } ?: request

        val response = chain.proceed(modifiedRequest)

        return when (response.code) {
            304 -> {
                // 서버 데이터가 변경되지 않음 - 캐시 사용
                response.close()
                cachedResponse!!.toResponse(request)
            }
            200 -> {
                // 새 데이터 - 캐시 갱신
                val etag = response.header("ETag")
                val body = response.body?.string()
                if (etag != null && body != null) {
                    cacheStore.put(cacheKey, CachedData(etag, body))
                }
                response.newBuilder()
                    .body(body?.toResponseBody(response.body?.contentType()))
                    .build()
            }
            else -> response
        }
    }
}
```

---

## 9. 캐시 관련 API 응답 헤더

모바일 앱에서 서버 캐시를 효과적으로 활용하려면 HTTP 캐시 헤더를 이해해야 합니다.

| 헤더 | 설명 | 예시 |
|:---|:---|:---|
| **Cache-Control** | 캐시 정책 지정 | `max-age=3600` (1시간 캐시) |
| **ETag** | 리소스 버전 식별자 | `"abc123"` |
| **Last-Modified** | 마지막 수정 시간 | `Wed, 15 Jan 2024 10:00:00 GMT` |
| **Expires** | 만료 시간 (구식) | 특정 날짜/시간 |
| **Vary** | 캐시 키에 포함할 헤더 | `Accept-Language` |

```
Cache-Control 주요 값
─────────────────────────────────────────────
max-age=N        : N초 동안 캐시 사용
no-cache         : 매번 서버에 검증 필요 (ETag 확인)
no-store         : 캐시 저장 금지 (민감한 데이터)
private          : 개인 캐시만 허용 (CDN 캐시 X)
public           : 공유 캐시 허용
immutable        : 절대 변하지 않음 (버전된 파일)
```

---

## 참고 자료

- [Redis 공식 문서](https://redis.io/documentation)
- [Redis University](https://university.redis.com/)
- [Redis Best Practices](https://redis.io/docs/manual/patterns/)
- [HTTP Caching - MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/Caching)
- [iOS NSCache](https://developer.apple.com/documentation/foundation/nscache)
- [Android LruCache](https://developer.android.com/reference/android/util/LruCache)
