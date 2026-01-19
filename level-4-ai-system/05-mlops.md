# MLOps (Machine Learning Operations)

## 1. 한 줄 요약

ML 모델의 개발, 배포, 모니터링, 재학습을 자동화하여 프로덕션 환경에서 안정적으로 운영하는 실천 방법론입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

MLOps는 **모바일 앱의 CI/CD 파이프라인**과 같은 개념입니다.

```
모바일 DevOps                       MLOps
┌─────────────────────┐            ┌────────────────────────────┐
│                     │            │                            │
│  코드 작성          │            │  모델 학습                 │
│     ↓               │            │     ↓                      │
│  테스트             │            │  모델 평가                 │
│     ↓               │            │     ↓                      │
│  빌드               │            │  모델 패키징               │
│     ↓               │            │     ↓                      │
│  배포 (App Store)   │            │  배포 (API 엔드포인트)    │
│     ↓               │            │     ↓                      │
│  모니터링 (Crashlytics) │        │  모니터링 (정확도, 지연)  │
│     ↓               │            │     ↓                      │
│  버그 수정 → 재배포 │            │  모델 드리프트 → 재학습   │
│                     │            │                            │
└─────────────────────┘            └────────────────────────────┘
```

### MLOps가 필요한 이유

```
┌─────────────────────────────────────────────────────────────────┐
│               ML 프로젝트가 실패하는 이유                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  연구/실험 단계                     프로덕션                   │
│  ┌─────────────────┐              ┌─────────────────┐          │
│  │ Jupyter 노트북  │    Gap!      │ 안정적인 서비스 │          │
│  │ "잘 되네!"      │────────────▶│ "왜 안 돼?!"    │          │
│  │ 로컬 환경       │   ?????      │ 실제 트래픽     │          │
│  └─────────────────┘              └─────────────────┘          │
│                                                                 │
│  문제:                                                         │
│  - "내 노트북에서는 됐는데..."                                 │
│  - 데이터가 바뀌면 성능 저하 (Data Drift)                      │
│  - 모델 버전 관리 안 됨                                        │
│  - 재학습 파이프라인 없음                                       │
│  - 장애 시 롤백 불가                                           │
│                                                                 │
│  해결: MLOps                                                   │
│  - 자동화된 파이프라인                                         │
│  - 모델 버전 관리 (Model Registry)                             │
│  - 지속적 모니터링                                              │
│  - 자동 재학습 트리거                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 구조 다이어그램

### MLOps 전체 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                      MLOps 전체 파이프라인                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    데이터 파이프라인                      │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐     │   │
│  │  │데이터   │─▶│ ETL/    │─▶│Feature  │─▶│Feature  │     │   │
│  │  │소스     │  │전처리   │  │Engineering│ │Store    │     │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └────┬────┘     │   │
│  └─────────────────────────────────────────────┼───────────┘   │
│                                                │               │
│  ┌─────────────────────────────────────────────┼───────────┐   │
│  │                    학습 파이프라인           │           │   │
│  │                                             ▼           │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │   │
│  │  │학습     │─▶│모델     │─▶│모델     │─▶│Model    │    │   │
│  │  │Job      │  │학습     │  │평가     │  │Registry │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └────┬────┘    │   │
│  └─────────────────────────────────────────────┼──────────┘   │
│                                                │               │
│  ┌─────────────────────────────────────────────┼───────────┐   │
│  │                    서빙 파이프라인           │           │   │
│  │                                             ▼           │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │   │
│  │  │배포     │─▶│모델     │─▶│API      │─▶│로드     │    │   │
│  │  │승인     │  │서빙     │  │Gateway  │  │밸런서   │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └────┬────┘    │   │
│  └─────────────────────────────────────────────┼──────────┘   │
│                                                │               │
│  ┌─────────────────────────────────────────────┼───────────┐   │
│  │                    모니터링                  ▼           │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │   │
│  │  │성능     │  │데이터   │  │모델     │  │알림/    │    │   │
│  │  │메트릭   │  │드리프트 │  │드리프트 │  │재학습   │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### LLMOps 특화 아키텍처 (2024-2025)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       LLMOps 아키텍처 (2024-2025)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                   프롬프트 관리 (Prompt Engineering)           │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │     │
│  │  │ 프롬프트  │  │ 버전     │  │ A/B      │  │ 프롬프트   │  │     │
│  │  │ 템플릿    │  │ 관리     │  │ 테스트   │  │ 최적화    │  │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│                              ▼                                          │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                    LLM Gateway (통합 관리)                     │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │     │
│  │  │ Rate      │  │ 모델     │  │ 비용     │  │ Semantic  │  │     │
│  │  │ Limiting  │  │ 라우팅   │  │ 추적     │  │ Caching   │  │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │     │
│  │                                                               │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │     │
│  │  │ Fallback  │  │ 로깅/    │  │ Guardrails│  │ Token     │  │     │
│  │  │ 처리      │  │ 추적     │  │ (안전)   │  │ Budgeting │  │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│                              ▼                                          │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                    LLM Providers (멀티 프로바이더)             │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │     │
│  │  │ OpenAI    │  │ Anthropic │  │ Google    │  │ Self-     │  │     │
│  │  │ GPT-4o    │  │ Claude 3.5│  │ Gemini 1.5│  │ hosted    │  │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│                              ▼                                          │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                    평가 & 모니터링 (LLM Observability)         │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐  │     │
│  │  │ 응답     │  │ LLM 품질 │  │ 비용     │  │ 환각     │  │     │
│  │  │ 지연시간 │  │ 평가     │  │ 분석     │  │ 감지     │  │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘  │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### CI/CD for ML

```
┌─────────────────────────────────────────────────────────────────┐
│                      ML CI/CD 파이프라인                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  코드 푸시                                                      │
│      │                                                          │
│      ▼                                                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    CI (Continuous Integration)          │   │
│  │                                                         │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │   │
│  │  │ Lint/   │─▶│ Unit    │─▶│ 데이터  │─▶│ 모델    │   │   │
│  │  │ Format  │  │ Test    │  │ 검증    │  │ 테스트  │   │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │   │
│  └───────────────────────────────────────────────────────────┘ │
│      │                                                          │
│      ▼                                                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    CT (Continuous Training)             │   │
│  │                                                         │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │   │
│  │  │ 데이터  │─▶│ 모델    │─▶│ 평가    │─▶│Registry │   │   │
│  │  │ 준비    │  │ 학습    │  │         │  │ 등록    │   │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │   │
│  └───────────────────────────────────────────────────────────┘ │
│      │                                                          │
│      ▼                                                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    CD (Continuous Deployment)           │   │
│  │                                                         │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │   │
│  │  │ Staging │─▶│ 카나리  │─▶│ 점진적  │─▶│ 프로덕  │   │   │
│  │  │ 배포    │  │ 배포    │  │ 롤아웃  │  │ 션      │   │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │   │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 1. 모델 서빙 (vLLM)

```python
# vLLM으로 LLM 서빙
from vllm import LLM, SamplingParams
from vllm.entrypoints.openai.api_server import app
import uvicorn

# 모델 로드 (최적화된 추론)
llm = LLM(
    model="meta-llama/Llama-3.2-8B-Instruct",
    tensor_parallel_size=2,  # GPU 2개 사용
    gpu_memory_utilization=0.9,
    max_model_len=4096,
)

# FastAPI 엔드포인트
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    messages: list[dict]
    max_tokens: int = 512
    temperature: float = 0.7

@app.post("/v1/chat/completions")
async def chat(request: ChatRequest):
    prompt = format_messages(request.messages)

    sampling_params = SamplingParams(
        temperature=request.temperature,
        max_tokens=request.max_tokens,
    )

    outputs = llm.generate([prompt], sampling_params)

    return {
        "choices": [{
            "message": {
                "role": "assistant",
                "content": outputs[0].outputs[0].text
            }
        }]
    }

# 실행
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 2. 모델 레지스트리 (MLflow)

```python
# MLflow로 모델 버전 관리
import mlflow
from mlflow.tracking import MlflowClient

mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("llm-chatbot")

# 학습 및 모델 등록
with mlflow.start_run():
    # 파라미터 로깅
    mlflow.log_params({
        "base_model": "llama-3.2-8b",
        "lora_rank": 16,
        "learning_rate": 2e-4,
        "epochs": 3
    })

    # 학습 실행
    model = train_model(config)

    # 메트릭 로깅
    mlflow.log_metrics({
        "eval_loss": 0.15,
        "perplexity": 8.5,
        "accuracy": 0.92
    })

    # 모델 저장
    mlflow.pyfunc.log_model(
        artifact_path="model",
        python_model=model,
        registered_model_name="chatbot-model"
    )

# 모델 버전 관리
client = MlflowClient()

# Staging으로 승격
client.transition_model_version_stage(
    name="chatbot-model",
    version=3,
    stage="Staging"
)

# Production으로 승격 (테스트 후)
client.transition_model_version_stage(
    name="chatbot-model",
    version=3,
    stage="Production"
)

# 모델 로드
model = mlflow.pyfunc.load_model(
    "models:/chatbot-model/Production"
)
```

### 3. 모니터링 대시보드

```python
# Prometheus + Grafana 연동
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# 메트릭 정의
REQUEST_COUNT = Counter(
    'llm_requests_total',
    'Total LLM API requests',
    ['model', 'status']
)

LATENCY = Histogram(
    'llm_request_latency_seconds',
    'Request latency in seconds',
    ['model'],
    buckets=[0.1, 0.5, 1, 2, 5, 10, 30]
)

TOKEN_USAGE = Counter(
    'llm_tokens_total',
    'Total tokens used',
    ['model', 'type']  # type: input/output
)

COST_GAUGE = Gauge(
    'llm_cost_dollars',
    'Estimated cost in USD',
    ['model']
)

# 미들웨어에서 메트릭 수집
class MetricsMiddleware:
    async def __call__(self, request, call_next):
        start_time = time.time()

        response = await call_next(request)

        latency = time.time() - start_time
        model = request.json().get('model', 'unknown')

        # 메트릭 기록
        REQUEST_COUNT.labels(model=model, status=response.status_code).inc()
        LATENCY.labels(model=model).observe(latency)

        return response

# 토큰 사용량 추적
def track_tokens(model: str, input_tokens: int, output_tokens: int):
    TOKEN_USAGE.labels(model=model, type='input').inc(input_tokens)
    TOKEN_USAGE.labels(model=model, type='output').inc(output_tokens)

    # 비용 계산 (예: GPT-4 기준)
    cost = (input_tokens * 0.00003) + (output_tokens * 0.00006)
    COST_GAUGE.labels(model=model).set(cost)

# Prometheus 엔드포인트 시작
start_http_server(9090)
```

### 4. LLM Gateway 구현 (Enhanced)

```python
# LLM Gateway - 여러 LLM 프로바이더 통합 (2024-2025 Best Practice)
from abc import ABC, abstractmethod
import asyncio
from typing import Optional
import hashlib
import json
from datetime import datetime
from dataclasses import dataclass
from enum import Enum

class ModelTier(Enum):
    PREMIUM = "premium"      # GPT-4o, Claude 3.5 Opus
    STANDARD = "standard"    # GPT-4o-mini, Claude 3.5 Sonnet
    BUDGET = "budget"        # Llama 3.2, Mistral

@dataclass
class RoutingConfig:
    """요청 라우팅 설정"""
    model_tier: ModelTier = ModelTier.STANDARD
    max_tokens: int = 4096
    timeout_seconds: float = 30.0
    enable_caching: bool = True
    enable_guardrails: bool = True

class LLMGateway:
    """엔터프라이즈급 LLM Gateway"""

    def __init__(self):
        self.providers = {
            "gpt-4o": OpenAIProvider(model="gpt-4o"),
            "gpt-4o-mini": OpenAIProvider(model="gpt-4o-mini"),
            "claude-3.5-sonnet": AnthropicProvider(model="claude-3-5-sonnet-20241022"),
            "claude-3.5-haiku": AnthropicProvider(model="claude-3-5-haiku-20241022"),
            "gemini-1.5-pro": GoogleProvider(model="gemini-1.5-pro"),
            "llama-3.2-8b": SelfHostedProvider(model="llama-3.2-8b"),
        }
        self.cache = SemanticCache()  # 의미 기반 캐싱
        self.rate_limiter = TokenBucketRateLimiter()
        self.cost_tracker = CostTracker()
        self.guardrails = Guardrails()

    async def chat(
        self,
        messages: list[dict],
        model: str = "gpt-4o-mini",
        user_id: str = None,
        config: RoutingConfig = None,
        **kwargs
    ) -> dict:
        config = config or RoutingConfig()

        # 1. Guardrails 체크 (입력 검증)
        if config.enable_guardrails:
            safety_result = await self.guardrails.check_input(messages)
            if not safety_result.is_safe:
                return self._safety_response(safety_result)

        # 2. Rate Limiting (토큰 버킷)
        if not await self.rate_limiter.allow(user_id, estimated_tokens=500):
            raise RateLimitExceeded(retry_after=self.rate_limiter.retry_after(user_id))

        # 3. Semantic Cache 확인
        if config.enable_caching:
            cached = await self.cache.get_similar(messages, threshold=0.95)
            if cached:
                return self._add_metadata(cached, from_cache=True)

        # 4. 모델 선택 및 호출
        provider = self.providers.get(model)
        if not provider:
            model = self._select_fallback_model(config.model_tier)
            provider = self.providers[model]

        start_time = datetime.now()
        try:
            response = await asyncio.wait_for(
                provider.chat(messages, **kwargs),
                timeout=config.timeout_seconds
            )
        except (asyncio.TimeoutError, ProviderError) as e:
            # 5. Fallback 처리
            response = await self._fallback(messages, model, config, **kwargs)

        latency = (datetime.now() - start_time).total_seconds()

        # 6. Guardrails 체크 (출력 검증)
        if config.enable_guardrails:
            output_result = await self.guardrails.check_output(response)
            if not output_result.is_safe:
                response = self._filter_response(response, output_result)

        # 7. 캐시 저장 및 비용 추적
        if config.enable_caching:
            await self.cache.set(messages, response)

        await self.cost_tracker.record(
            model=model,
            input_tokens=response.get("usage", {}).get("prompt_tokens", 0),
            output_tokens=response.get("usage", {}).get("completion_tokens", 0),
            user_id=user_id
        )

        return self._add_metadata(response, model=model, latency=latency)

    async def _fallback(self, messages, failed_model, config, **kwargs):
        """지능형 Fallback 처리"""
        fallback_chains = {
            ModelTier.PREMIUM: ["gpt-4o", "claude-3.5-sonnet", "gemini-1.5-pro"],
            ModelTier.STANDARD: ["gpt-4o-mini", "claude-3.5-haiku", "llama-3.2-8b"],
            ModelTier.BUDGET: ["llama-3.2-8b", "gpt-4o-mini"],
        }

        chain = fallback_chains.get(config.model_tier, [])
        for backup_model in chain:
            if backup_model == failed_model:
                continue
            try:
                return await self.providers[backup_model].chat(messages, **kwargs)
            except Exception:
                continue

        raise AllProvidersFailedError("All fallback models failed")

    def _select_fallback_model(self, tier: ModelTier) -> str:
        """티어에 맞는 기본 모델 선택"""
        defaults = {
            ModelTier.PREMIUM: "gpt-4o",
            ModelTier.STANDARD: "gpt-4o-mini",
            ModelTier.BUDGET: "llama-3.2-8b",
        }
        return defaults[tier]


# Semantic Cache 구현
class SemanticCache:
    """의미 기반 캐싱 - 유사한 쿼리도 캐시 히트"""

    def __init__(self):
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        self.vector_store = chromadb.Client()
        self.collection = self.vector_store.create_collection("llm_cache")

    async def get_similar(self, messages: list[dict], threshold: float = 0.95):
        query_text = self._messages_to_text(messages)
        query_embedding = self.embedding_model.encode(query_text)

        results = self.collection.query(
            query_embeddings=[query_embedding.tolist()],
            n_results=1
        )

        if results['distances'] and results['distances'][0]:
            similarity = 1 - results['distances'][0][0]
            if similarity >= threshold:
                return json.loads(results['documents'][0][0])
        return None

    async def set(self, messages: list[dict], response: dict, ttl: int = 3600):
        query_text = self._messages_to_text(messages)
        query_embedding = self.embedding_model.encode(query_text)

        self.collection.add(
            embeddings=[query_embedding.tolist()],
            documents=[json.dumps(response)],
            ids=[hashlib.md5(query_text.encode()).hexdigest()]
        )

    def _messages_to_text(self, messages: list[dict]) -> str:
        return " ".join([m.get("content", "") for m in messages])
```

### 5. 데이터 드리프트 감지

```python
# 데이터 드리프트 감지 시스템
from scipy import stats
import numpy as np
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime, timedelta

@dataclass
class DriftReport:
    feature_name: str
    drift_score: float
    is_drifted: bool
    drift_type: str  # "gradual", "sudden", "seasonal"
    timestamp: datetime
    recommendation: str

class DriftDetector:
    """실시간 데이터 드리프트 감지"""

    DRIFT_THRESHOLD = 0.05  # p-value 임계값

    def __init__(self):
        self.baseline_stats = {}
        self.window_size = 1000
        self.alert_handler = AlertHandler()

    async def detect_drift(
        self,
        feature_name: str,
        current_data: np.ndarray,
        baseline_data: np.ndarray = None
    ) -> DriftReport:
        """KS Test를 사용한 드리프트 감지"""

        if baseline_data is None:
            baseline_data = self.baseline_stats.get(feature_name)
            if baseline_data is None:
                # 첫 데이터는 baseline으로 저장
                self.baseline_stats[feature_name] = current_data
                return DriftReport(
                    feature_name=feature_name,
                    drift_score=0.0,
                    is_drifted=False,
                    drift_type="none",
                    timestamp=datetime.now(),
                    recommendation="Baseline established"
                )

        # Kolmogorov-Smirnov 테스트
        statistic, p_value = stats.ks_2samp(baseline_data, current_data)

        is_drifted = p_value < self.DRIFT_THRESHOLD
        drift_type = self._classify_drift_type(baseline_data, current_data)

        report = DriftReport(
            feature_name=feature_name,
            drift_score=statistic,
            is_drifted=is_drifted,
            drift_type=drift_type,
            timestamp=datetime.now(),
            recommendation=self._get_recommendation(is_drifted, drift_type)
        )

        if is_drifted:
            await self.alert_handler.send_alert(report)

        return report

    def _classify_drift_type(
        self,
        baseline: np.ndarray,
        current: np.ndarray
    ) -> str:
        """드리프트 유형 분류"""
        mean_shift = abs(np.mean(current) - np.mean(baseline))
        std_shift = abs(np.std(current) - np.std(baseline))

        baseline_std = np.std(baseline)
        if mean_shift > 2 * baseline_std:
            return "sudden"
        elif mean_shift > 0.5 * baseline_std:
            return "gradual"
        elif std_shift > baseline_std:
            return "variance_change"
        return "none"

    def _get_recommendation(self, is_drifted: bool, drift_type: str) -> str:
        if not is_drifted:
            return "No action required"

        recommendations = {
            "sudden": "즉시 모델 재학습 권장. 데이터 소스 변경 확인 필요.",
            "gradual": "1-2주 내 재학습 스케줄링 권장. 트렌드 모니터링 지속.",
            "variance_change": "특이값 분석 필요. 데이터 품질 점검 권장.",
        }
        return recommendations.get(drift_type, "수동 검토 필요")


# LLM 특화 드리프트 감지
class LLMDriftDetector:
    """LLM 응답 품질 드리프트 감지"""

    def __init__(self):
        self.metrics_history = []
        self.baseline_metrics = None

    async def track_response(self, response: dict, ground_truth: str = None):
        """응답 품질 메트릭 추적"""
        metrics = {
            "response_length": len(response.get("content", "")),
            "latency": response.get("latency", 0),
            "token_count": response.get("usage", {}).get("total_tokens", 0),
            "timestamp": datetime.now()
        }

        # 품질 점수 계산 (ground truth 있을 경우)
        if ground_truth:
            metrics["similarity_score"] = self._calculate_similarity(
                response.get("content", ""),
                ground_truth
            )

        self.metrics_history.append(metrics)

        # 최근 100개 응답 기준으로 드리프트 체크
        if len(self.metrics_history) >= 100:
            await self._check_quality_drift()

    async def _check_quality_drift(self):
        """품질 드리프트 감지"""
        recent = self.metrics_history[-100:]

        if self.baseline_metrics is None:
            self.baseline_metrics = self._calculate_baseline(recent)
            return

        current_avg_length = np.mean([m["response_length"] for m in recent])
        current_avg_latency = np.mean([m["latency"] for m in recent])

        # 응답 길이 급변 감지 (20% 이상 변화)
        if abs(current_avg_length - self.baseline_metrics["avg_length"]) / self.baseline_metrics["avg_length"] > 0.2:
            await self._alert("Response length drift detected")

        # 지연시간 증가 감지 (50% 이상 증가)
        if current_avg_latency > self.baseline_metrics["avg_latency"] * 1.5:
            await self._alert("Latency degradation detected")
```

### 6. 모바일 앱 연동 (iOS/Android)

```swift
// iOS - MLOps 친화적 AI Service Client
import Foundation

// MARK: - Configuration
struct AIServiceConfig {
    let baseURL: URL
    let apiKey: String
    var modelVersion: String = "v1"
    var experimentId: String?
    var timeout: TimeInterval = 30.0
    var retryCount: Int = 3

    // Feature Flags
    var enableCaching: Bool = true
    var enableFallback: Bool = true
}

// MARK: - Request/Response Models
struct ChatMessage: Codable {
    let role: String
    let content: String
}

struct ChatRequest: Codable {
    let messages: [ChatMessage]
    let maxTokens: Int
    let temperature: Double
    let stream: Bool

    enum CodingKeys: String, CodingKey {
        case messages
        case maxTokens = "max_tokens"
        case temperature
        case stream
    }
}

struct ChatResponse: Codable {
    let id: String
    let choices: [Choice]
    let usage: Usage?
    let metadata: ResponseMetadata?

    struct Choice: Codable {
        let message: ChatMessage
        let finishReason: String?

        enum CodingKeys: String, CodingKey {
            case message
            case finishReason = "finish_reason"
        }
    }

    struct Usage: Codable {
        let promptTokens: Int
        let completionTokens: Int
        let totalTokens: Int

        enum CodingKeys: String, CodingKey {
            case promptTokens = "prompt_tokens"
            case completionTokens = "completion_tokens"
            case totalTokens = "total_tokens"
        }
    }

    struct ResponseMetadata: Codable {
        let modelVersion: String?
        let experimentGroup: String?
        let fromCache: Bool?
        let latencyMs: Int?

        enum CodingKeys: String, CodingKey {
            case modelVersion = "model_version"
            case experimentGroup = "experiment_group"
            case fromCache = "from_cache"
            case latencyMs = "latency_ms"
        }
    }
}

// MARK: - AI Service Client
actor AIServiceClient {
    private let config: AIServiceConfig
    private let urlSession: URLSession
    private var currentModelVersion: String

    // 로컬 캐시
    private var responseCache: [String: (response: ChatResponse, timestamp: Date)] = [:]
    private let cacheMaxAge: TimeInterval = 300 // 5분

    init(config: AIServiceConfig) {
        self.config = config
        self.currentModelVersion = config.modelVersion

        let sessionConfig = URLSessionConfiguration.default
        sessionConfig.timeoutIntervalForRequest = config.timeout
        self.urlSession = URLSession(configuration: sessionConfig)
    }

    func chat(messages: [ChatMessage], options: ChatOptions = .default) async throws -> ChatResponse {
        // 1. 로컬 캐시 확인
        if config.enableCaching {
            let cacheKey = generateCacheKey(messages: messages)
            if let cached = responseCache[cacheKey],
               Date().timeIntervalSince(cached.timestamp) < cacheMaxAge {
                return cached.response
            }
        }

        // 2. 요청 생성
        var request = try createRequest(messages: messages, options: options)

        // 3. 재시도 로직과 함께 요청 수행
        var lastError: Error?
        for attempt in 0..<config.retryCount {
            do {
                let response = try await performRequest(request)

                // 캐시 저장
                if config.enableCaching {
                    let cacheKey = generateCacheKey(messages: messages)
                    responseCache[cacheKey] = (response, Date())
                }

                // 모델 버전 업데이트
                if let newVersion = response.metadata?.modelVersion {
                    currentModelVersion = newVersion
                }

                return response

            } catch let error as AIServiceError where error.isRetryable {
                lastError = error
                // 지수 백오프
                try await Task.sleep(nanoseconds: UInt64(pow(2.0, Double(attempt)) * 1_000_000_000))
            } catch {
                throw error
            }
        }

        throw lastError ?? AIServiceError.unknown
    }

    // 스트리밍 응답 지원
    func chatStream(messages: [ChatMessage]) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                do {
                    var request = try createRequest(messages: messages, options: .init(stream: true))

                    let (bytes, response) = try await urlSession.bytes(for: request)

                    guard let httpResponse = response as? HTTPURLResponse,
                          httpResponse.statusCode == 200 else {
                        throw AIServiceError.serverError
                    }

                    for try await line in bytes.lines {
                        if line.hasPrefix("data: "),
                           let chunk = parseStreamChunk(line) {
                            continuation.yield(chunk)
                        }
                    }

                    continuation.finish()
                } catch {
                    continuation.finish(throwing: error)
                }
            }
        }
    }

    private func createRequest(messages: [ChatMessage], options: ChatOptions) throws -> URLRequest {
        var request = URLRequest(url: config.baseURL.appendingPathComponent("/v1/chat/completions"))
        request.httpMethod = "POST"
        request.addValue("Bearer \(config.apiKey)", forHTTPHeaderField: "Authorization")
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.addValue(currentModelVersion, forHTTPHeaderField: "X-Model-Version")
        request.addValue(UUID().uuidString, forHTTPHeaderField: "X-Request-ID")

        // A/B 테스트 지원
        if let experimentId = config.experimentId {
            request.addValue(experimentId, forHTTPHeaderField: "X-Experiment-ID")
        }

        let body = ChatRequest(
            messages: messages,
            maxTokens: options.maxTokens,
            temperature: options.temperature,
            stream: options.stream
        )
        request.httpBody = try JSONEncoder().encode(body)

        return request
    }

    private func generateCacheKey(messages: [ChatMessage]) -> String {
        let content = messages.map { "\($0.role):\($0.content)" }.joined()
        return content.data(using: .utf8)?.base64EncodedString() ?? ""
    }
}

// MARK: - Chat Options
struct ChatOptions {
    var maxTokens: Int = 1024
    var temperature: Double = 0.7
    var stream: Bool = false

    static let `default` = ChatOptions()
}

// MARK: - Error Types
enum AIServiceError: Error {
    case networkError
    case serverError
    case rateLimited(retryAfter: Int)
    case invalidResponse
    case unknown

    var isRetryable: Bool {
        switch self {
        case .networkError, .serverError, .rateLimited:
            return true
        default:
            return false
        }
    }
}
```

```kotlin
// Android - MLOps 친화적 AI Service Client
package com.example.ai.service

import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import okhttp3.*
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.RequestBody.Companion.toRequestBody
import java.io.IOException
import java.util.UUID
import java.util.concurrent.TimeUnit
import kotlin.math.pow

// Configuration
data class AIServiceConfig(
    val baseUrl: String,
    val apiKey: String,
    var modelVersion: String = "v1",
    var experimentId: String? = null,
    val timeout: Long = 30L,
    val retryCount: Int = 3,
    val enableCaching: Boolean = true,
    val enableFallback: Boolean = true
)

// Request/Response Models
@Serializable
data class ChatMessage(
    val role: String,
    val content: String
)

@Serializable
data class ChatRequest(
    val messages: List<ChatMessage>,
    val max_tokens: Int = 1024,
    val temperature: Double = 0.7,
    val stream: Boolean = false
)

@Serializable
data class ChatResponse(
    val id: String,
    val choices: List<Choice>,
    val usage: Usage? = null,
    val metadata: ResponseMetadata? = null
) {
    @Serializable
    data class Choice(
        val message: ChatMessage,
        val finish_reason: String? = null
    )

    @Serializable
    data class Usage(
        val prompt_tokens: Int,
        val completion_tokens: Int,
        val total_tokens: Int
    )

    @Serializable
    data class ResponseMetadata(
        val model_version: String? = null,
        val experiment_group: String? = null,
        val from_cache: Boolean? = null,
        val latency_ms: Int? = null
    )
}

// AI Service Client
class AIServiceClient(
    private val config: AIServiceConfig
) {
    private val json = Json { ignoreUnknownKeys = true }

    private val client = OkHttpClient.Builder()
        .connectTimeout(config.timeout, TimeUnit.SECONDS)
        .readTimeout(config.timeout, TimeUnit.SECONDS)
        .addInterceptor(RetryInterceptor(config.retryCount))
        .addInterceptor(LoggingInterceptor())
        .build()

    // 로컬 캐시 (LRU Cache)
    private val responseCache = object : LinkedHashMap<String, CacheEntry>(100, 0.75f, true) {
        override fun removeEldestEntry(eldest: Map.Entry<String, CacheEntry>): Boolean {
            return size > 100
        }
    }

    private data class CacheEntry(
        val response: ChatResponse,
        val timestamp: Long
    )

    private val cacheMaxAge = 5 * 60 * 1000L // 5분

    suspend fun chat(
        messages: List<ChatMessage>,
        options: ChatOptions = ChatOptions()
    ): Result<ChatResponse> {
        // 1. 캐시 확인
        if (config.enableCaching) {
            val cacheKey = generateCacheKey(messages)
            responseCache[cacheKey]?.let { cached ->
                if (System.currentTimeMillis() - cached.timestamp < cacheMaxAge) {
                    return Result.success(cached.response)
                }
            }
        }

        // 2. 요청 생성 및 수행
        return try {
            val request = createRequest(messages, options)
            val response = executeRequest(request)

            // 캐시 저장
            if (config.enableCaching) {
                val cacheKey = generateCacheKey(messages)
                responseCache[cacheKey] = CacheEntry(response, System.currentTimeMillis())
            }

            Result.success(response)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    // 스트리밍 응답
    fun chatStream(messages: List<ChatMessage>): Flow<String> = flow {
        val request = createRequest(messages, ChatOptions(stream = true))

        client.newCall(request).execute().use { response ->
            if (!response.isSuccessful) {
                throw IOException("Server error: ${response.code}")
            }

            response.body?.source()?.let { source ->
                while (!source.exhausted()) {
                    val line = source.readUtf8Line() ?: break
                    if (line.startsWith("data: ")) {
                        val chunk = parseStreamChunk(line.removePrefix("data: "))
                        if (chunk != null) emit(chunk)
                    }
                }
            }
        }
    }

    private fun createRequest(messages: List<ChatMessage>, options: ChatOptions): Request {
        val body = json.encodeToString(
            ChatRequest.serializer(),
            ChatRequest(
                messages = messages,
                max_tokens = options.maxTokens,
                temperature = options.temperature,
                stream = options.stream
            )
        )

        return Request.Builder()
            .url("${config.baseUrl}/v1/chat/completions")
            .post(body.toRequestBody("application/json".toMediaType()))
            .addHeader("Authorization", "Bearer ${config.apiKey}")
            .addHeader("X-Model-Version", config.modelVersion)
            .addHeader("X-Request-ID", UUID.randomUUID().toString())
            .apply {
                config.experimentId?.let { addHeader("X-Experiment-ID", it) }
            }
            .build()
    }

    private fun generateCacheKey(messages: List<ChatMessage>): String {
        return messages.joinToString("") { "${it.role}:${it.content}" }
            .hashCode().toString()
    }

    private fun executeRequest(request: Request): ChatResponse {
        client.newCall(request).execute().use { response ->
            if (!response.isSuccessful) {
                when (response.code) {
                    429 -> {
                        val retryAfter = response.header("Retry-After")?.toIntOrNull() ?: 60
                        throw RateLimitException(retryAfter)
                    }
                    else -> throw IOException("Server error: ${response.code}")
                }
            }

            val body = response.body?.string() ?: throw IOException("Empty response")
            return json.decodeFromString(ChatResponse.serializer(), body)
        }
    }
}

// Chat Options
data class ChatOptions(
    val maxTokens: Int = 1024,
    val temperature: Double = 0.7,
    val stream: Boolean = false
)

// Retry Interceptor
class RetryInterceptor(private val maxRetries: Int) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        var lastException: IOException? = null

        repeat(maxRetries) { attempt ->
            try {
                val response = chain.proceed(chain.request())
                if (response.isSuccessful || response.code !in listOf(502, 503, 504)) {
                    return response
                }
                response.close()
            } catch (e: IOException) {
                lastException = e
            }

            // 지수 백오프
            Thread.sleep((2.0.pow(attempt) * 1000).toLong())
        }

        throw lastException ?: IOException("Request failed after $maxRetries retries")
    }
}

// Custom Exceptions
class RateLimitException(val retryAfter: Int) : Exception("Rate limited. Retry after $retryAfter seconds")
```

### 7. GitHub Actions CI/CD for ML

```yaml
# .github/workflows/ml-cicd.yml
name: ML CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    paths:
      - 'models/**'
      - 'training/**'
      - 'data/**'
  pull_request:
    branches: [main]
  schedule:
    # 매주 월요일 새벽 2시에 재학습 트리거
    - cron: '0 2 * * 1'

env:
  PYTHON_VERSION: '3.11'
  MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
  AWS_REGION: ap-northeast-2

jobs:
  # 1. 코드 품질 검사
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt

      - name: Run linting
        run: |
          ruff check .
          mypy models/ training/

      - name: Run unit tests
        run: pytest tests/unit/ -v --cov=models --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: coverage.xml

  # 2. 데이터 검증
  data-validation:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Validate training data
        run: |
          python scripts/validate_data.py \
            --data-path data/training/ \
            --schema-path schemas/training_data.json

      - name: Check data drift
        run: |
          python scripts/check_drift.py \
            --baseline data/baseline/ \
            --current data/training/ \
            --threshold 0.05

  # 3. 모델 학습
  train-model:
    runs-on: ubuntu-latest
    needs: data-validation
    if: github.event_name == 'push' || github.event_name == 'schedule'

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Train model
        run: |
          python training/train.py \
            --config configs/training_config.yaml \
            --experiment-name ${{ github.ref_name }}-${{ github.sha }}
        env:
          MLFLOW_TRACKING_URI: ${{ env.MLFLOW_TRACKING_URI }}

      - name: Evaluate model
        run: |
          python training/evaluate.py \
            --model-path outputs/model \
            --test-data data/test/

      - name: Upload model artifacts
        uses: actions/upload-artifact@v4
        with:
          name: model-artifacts
          path: outputs/

  # 4. 모델 등록
  register-model:
    runs-on: ubuntu-latest
    needs: train-model
    if: github.ref == 'refs/heads/main'

    outputs:
      model_version: ${{ steps.register.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Download model artifacts
        uses: actions/download-artifact@v4
        with:
          name: model-artifacts
          path: outputs/

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Register model to MLflow
        id: register
        run: |
          VERSION=$(python scripts/register_model.py \
            --model-path outputs/model \
            --model-name chatbot-model \
            --stage Staging)
          echo "version=$VERSION" >> $GITHUB_OUTPUT
        env:
          MLFLOW_TRACKING_URI: ${{ env.MLFLOW_TRACKING_URI }}

  # 5. Staging 배포
  deploy-staging:
    runs-on: ubuntu-latest
    needs: register-model
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging
        run: |
          kubectl apply -f k8s/staging/deployment.yaml
          kubectl set image deployment/llm-server \
            llm-server=registry.example.com/llm-server:${{ needs.register-model.outputs.model_version }}
        env:
          KUBECONFIG: ${{ secrets.STAGING_KUBECONFIG }}

      - name: Run smoke tests
        run: |
          python tests/integration/smoke_tests.py \
            --endpoint https://staging-api.example.com \
            --timeout 300

  # 6. Production 배포 (수동 승인 필요)
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Canary deployment
        run: |
          # 10% 트래픽으로 시작
          kubectl apply -f k8s/production/canary.yaml

          # 성능 모니터링
          sleep 300
          python scripts/check_canary_metrics.py \
            --threshold-latency 2000 \
            --threshold-error-rate 0.01
        env:
          KUBECONFIG: ${{ secrets.PRODUCTION_KUBECONFIG }}

      - name: Full rollout
        if: success()
        run: |
          kubectl apply -f k8s/production/deployment.yaml
          kubectl rollout status deployment/llm-server --timeout=600s
        env:
          KUBECONFIG: ${{ secrets.PRODUCTION_KUBECONFIG }}

      - name: Promote model to Production
        run: |
          python scripts/promote_model.py \
            --model-name chatbot-model \
            --version ${{ needs.register-model.outputs.model_version }} \
            --stage Production
        env:
          MLFLOW_TRACKING_URI: ${{ env.MLFLOW_TRACKING_URI }}

      - name: Notify deployment
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Model deployed to production",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Model Deployment Complete*\n• Version: ${{ needs.register-model.outputs.model_version }}\n• Environment: Production\n• Commit: ${{ github.sha }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 5. 장단점

### 장점

| 항목 | 설명 |
|------|------|
| **자동화** | 수동 작업 최소화, 인적 오류 감소 |
| **재현성** | 동일한 결과를 언제든 재현 가능 |
| **확장성** | 모델과 트래픽 증가에 대응 |
| **안정성** | 모니터링, 롤백으로 서비스 안정화 |
| **협업** | 팀 간 명확한 인터페이스 |

### 단점

| 항목 | 설명 |
|------|------|
| **초기 비용** | 인프라 구축에 시간과 비용 필요 |
| **복잡성** | 다양한 도구 학습 필요 |
| **오버엔지니어링** | 작은 프로젝트에는 과할 수 있음 |
| **전문 인력** | MLOps 경험자 채용 어려움 |

### MLOps 성숙도 단계

```
Level 0: 수동                    Level 1: 자동화 시작
┌─────────────────────┐         ┌─────────────────────┐
│ - Jupyter 노트북    │         │ - 파이프라인 도입   │
│ - 수동 배포         │   ───▶  │ - 기본 모니터링     │
│ - 버전 관리 없음    │         │ - 수동 재학습       │
└─────────────────────┘         └─────────────────────┘
                                         │
                                         ▼
Level 3: 완전 자동화             Level 2: 자동 재학습
┌─────────────────────┐         ┌─────────────────────┐
│ - CI/CD/CT 완전 자동│         │ - 자동 재학습       │
│ - 자동 A/B 테스트   │   ◀───  │ - 모델 레지스트리   │
│ - 피처 스토어       │         │ - 드리프트 감지     │
│ - 자동 롤백         │         │                     │
└─────────────────────┘         └─────────────────────┘
```

---

## 6. 내 생각

```
이 섹션은 학습 후 직접 작성해보세요.

- MLOps의 핵심 가치는 무엇인가?
- 우리 팀에서 가장 먼저 자동화해야 할 부분은?
- 현재 ML 운영에서 가장 큰 페인포인트는?
- 어떤 MLOps 도구를 먼저 도입할까?
```

---

## 7. 추가 질문

### 초급

1. MLOps와 DevOps의 차이점은 무엇인가요?

> **답변**: MLOps는 DevOps의 원칙을 기반으로 하되, 머신러닝 고유의 특성을 추가로 다룹니다. **핵심 차이점**은 다음과 같습니다. 첫째, **코드 + 데이터 + 모델**의 3중 버전 관리가 필요합니다. DevOps는 코드만 관리하면 되지만, MLOps는 데이터 버전과 모델 아티팩트까지 추적해야 합니다. 둘째, **실험 추적**이 필수입니다. 하이퍼파라미터, 메트릭, 학습 곡선 등 수백 번의 실험을 체계적으로 관리해야 합니다. 셋째, **지속적 학습(CT: Continuous Training)**이 추가됩니다. 데이터 드리프트 발생 시 자동 재학습이 필요합니다. 넷째, **모델 성능 모니터링**이 다릅니다. 앱은 크래시/에러율을 보지만, ML은 정확도, 편향, 환각 등을 감시합니다. 모바일 개발자 관점에서 보면, DevOps가 "앱이 정상 동작하는가?"를 확인한다면, MLOps는 "모델이 정확한 예측을 하는가?"까지 확인하는 확장된 개념입니다.

2. Model Registry가 필요한 이유는?

> **답변**: Model Registry는 ML 모델의 **버전 관리 및 배포 상태를 중앙에서 관리**하는 시스템입니다. 필요한 이유는 다음과 같습니다. 첫째, **버전 추적**: "지난주에 잘 동작하던 모델로 롤백해야 해요"라는 요청에 즉시 대응 가능합니다. 둘째, **배포 단계 관리**: Development → Staging → Production 단계를 명확히 구분하고, 각 단계에서 어떤 모델이 서빙 중인지 한눈에 파악됩니다. 셋째, **메타데이터 저장**: 학습에 사용된 데이터셋, 하이퍼파라미터, 성능 메트릭을 모델과 함께 기록하여 재현성을 보장합니다. 넷째, **거버넌스**: 누가 언제 모델을 승인했는지 감사 로그를 남깁니다. 모바일 앱에서 TestFlight/Google Play 내부 테스트가 하는 역할과 유사하게, Model Registry는 ML 모델의 생명주기를 체계적으로 관리합니다. 대표적인 도구로는 MLflow Model Registry, SageMaker Model Registry, Vertex AI Model Registry가 있습니다.

3. 데이터 드리프트(Data Drift)란 무엇인가요?

> **답변**: 데이터 드리프트는 **프로덕션 데이터의 통계적 특성이 학습 데이터와 달라지는 현상**입니다. 모바일 앱에 비유하면, 앱은 한국 사용자 기준으로 개발했는데 갑자기 미국 사용자가 몰려와서 예상치 못한 버그가 발생하는 상황과 유사합니다. **드리프트 유형**은 크게 세 가지입니다. (1) **데이터 드리프트**: 입력 데이터 분포 변화 (예: 평균 구매 금액이 5만원 → 15만원), (2) **개념 드리프트**: 입력과 출력의 관계 변화 (예: 코로나 이후 "재택" 검색의 의미 변화), (3) **레이블 드리프트**: 정답 레이블의 분포 변화. **실제 사례**로, 이커머스 추천 모델이 블랙프라이데이 기간에 평소와 다른 구매 패턴으로 성능 저하를 겪는 경우가 있습니다. **감지 방법**으로는 KS Test, PSI(Population Stability Index), 분포 시각화 등이 사용됩니다. 드리프트가 감지되면 재학습 파이프라인을 트리거하여 모델을 최신 데이터에 적응시킵니다.

### 중급

4. A/B 테스트와 카나리 배포의 차이는?

> **답변**: 둘 다 새로운 버전을 안전하게 배포하는 방법이지만 **목적과 방식이 다릅니다**. **A/B 테스트**는 "어떤 모델이 더 좋은가?"를 검증하는 **실험**입니다. 사용자를 무작위로 그룹 분할하여 각각 다른 모델을 경험하게 하고, 통계적으로 유의미한 차이를 측정합니다. 예: "새 추천 모델이 CTR을 5% 향상시키는가?" 실험 기간이 정해져 있고, 충분한 샘플 수가 필요합니다. **카나리 배포**는 "새 모델이 안전한가?"를 확인하는 **점진적 롤아웃**입니다. 소수(1-5%) 트래픽으로 시작해서 문제가 없으면 점진적으로 확대합니다. 에러율, 지연시간 등 시스템 안정성에 초점을 맞춥니다. 문제 발생 시 즉시 롤백합니다. **실무에서는 둘을 조합**합니다. 카나리로 안전성 확인 → A/B로 효과 검증 → 전체 배포. 모바일 앱의 단계적 출시(Staged Rollout)와 Feature Flag를 생각하면 이해가 쉽습니다.

5. Feature Store의 역할과 이점은?

> **답변**: Feature Store는 **ML 피처를 중앙에서 관리하고 재사용할 수 있게 하는 데이터 플랫폼**입니다. **역할**은 세 가지입니다. (1) **피처 계산 및 저장**: 원본 데이터에서 피처를 추출하고, 학습용(배치)과 서빙용(실시간) 둘 다 제공합니다. (2) **피처 카탈로그**: 어떤 피처가 있고, 어떻게 계산되며, 누가 사용하는지 문서화합니다. (3) **학습-서빙 일관성 보장**: 학습 때 사용한 피처와 동일한 로직으로 추론 시에도 피처를 생성합니다. **이점**은 다음과 같습니다. 첫째, **재사용성**: "사용자 최근 7일 구매 횟수" 피처를 한 번 만들면 여러 모델에서 재사용합니다. 둘째, **일관성**: 학습과 서빙의 피처 불일치(Training-Serving Skew) 문제를 방지합니다. 셋째, **개발 속도 향상**: 새 모델 개발 시 기존 피처 조합으로 빠르게 프로토타이핑 가능합니다. 대표적인 도구로 Feast(오픈소스), Tecton, Databricks Feature Store가 있습니다. 모바일에서 공통 유틸리티 라이브러리를 만들어 여러 앱에서 재사용하는 것과 같은 개념입니다.

6. 모델 성능 저하를 어떻게 감지하나요?

> **답변**: **다층적 모니터링 전략**이 필요합니다. **1. 실시간 메트릭 모니터링**: 지연시간(P50, P95, P99), 에러율, 처리량(QPS)을 대시보드로 시각화하고 임계값 초과 시 알림을 발송합니다. **2. 입력 드리프트 감지**: 입력 데이터의 분포 변화를 통계적으로 감지합니다(KS Test, PSI). 입력이 달라지면 곧 출력 품질도 저하될 신호입니다. **3. 출력 품질 모니터링**: 정답 레이블이 있다면 정확도/F1 추적, 없다면 대리 지표(응답 길이, confidence score 분포)를 모니터링합니다. LLM의 경우 환각률, 거부율 등을 추적합니다. **4. 비즈니스 메트릭 연동**: 추천 모델이라면 CTR, 구매 전환율 등 비즈니스 KPI와 연결하여 실질적 성능을 측정합니다. **5. 사용자 피드백**: 좋아요/싫어요, 재생성 요청 비율, 고객 문의 증가 등 정성적 신호도 수집합니다. **실무 팁**: 단일 지표에 의존하지 말고, 여러 신호의 종합적 패턴을 파악하세요. 지연시간만 보다가 품질 저하를 놓치거나, 정확도만 보다가 지연 문제를 놓칠 수 있습니다.

### 심화

7. Shadow Deployment란 무엇이고 언제 사용하나요?

> **답변**: Shadow Deployment(또는 Shadow Mode, Traffic Mirroring)는 **새 모델에 실제 트래픽을 복사해서 보내되, 응답은 사용자에게 반환하지 않는 배포 방식**입니다. 기존 모델(Primary)이 실제 응답을 처리하고, 새 모델(Shadow)은 동일한 요청을 받아 처리하지만 결과는 로깅만 합니다. **사용 시점**: (1) 대규모 아키텍처 변경(예: 모델 서빙 프레임워크 교체)의 안전성 검증, (2) 실제 트래픽 패턴에서 새 모델의 지연시간/리소스 사용량 측정, (3) A/B 테스트 전 사전 검증(정성적 출력 품질 리뷰). **장점**: 사용자에게 영향 없이 실제 환경 테스트 가능, 장애 위험 제로. **단점**: 인프라 비용 2배, 부작용(side effect) 있는 API는 주의 필요(예: 외부 결제 API 호출). **구현 방법**: Istio, Envoy 같은 서비스 메시에서 트래픽 미러링 기능 사용, 또는 LLM Gateway에서 직접 구현. 모바일 개발의 "기능은 구현했지만 Feature Flag로 숨겨두고 내부 테스트"하는 것과 유사한 개념입니다.

8. Multi-armed Bandit과 A/B 테스트의 차이는?

> **답변**: **A/B 테스트**는 실험 기간 동안 트래픽을 **고정 비율**로 분배하고, 실험 종료 후 승자를 결정합니다. 예: 50:50으로 2주간 진행 후 통계 분석. **Multi-armed Bandit(MAB)**은 실험 중에 **동적으로 트래픽을 조정**하여, 좋은 성과를 보이는 쪽에 점점 더 많은 트래픽을 할당합니다. **핵심 차이**는 탐색(Exploration)과 활용(Exploitation)의 균형입니다. A/B는 탐색에 집중(정보 수집), MAB는 활용도 동시에 진행(수익 최적화). **MAB의 장점**: (1) 열등한 변형에 낭비되는 트래픽 최소화, (2) 더 빠르게 최적 변형 발견, (3) 계절성/트렌드 변화에 적응. **MAB의 단점**: (1) 통계적 유의성 해석이 복잡, (2) 장기적 효과 측정 어려움, (3) 구현 복잡도 높음. **실무 권장**: 단기 최적화(배너 문구, 버튼 색상)는 MAB, 중요한 제품 결정(새 기능 출시)은 A/B 테스트. 대표 알고리즘으로 Epsilon-Greedy, UCB(Upper Confidence Bound), Thompson Sampling이 있습니다.

9. LLM 특화 평가 지표(BLEU, ROUGE, 등)는 어떤 것이 있나요?

> **답변**: LLM 평가 지표는 **태스크 유형**에 따라 선택합니다. **1. 텍스트 생성 품질 (참조 기반)**: - **BLEU**: N-gram 정밀도 기반, 번역/요약 평가. 짧은 출력에 불리함. - **ROUGE**: 재현율 기반, 요약 평가에 적합. ROUGE-1(유니그램), ROUGE-L(최장 공통 부분 수열) - **BERTScore**: 임베딩 유사도 기반, 의미적 유사성 측정. 동의어 처리 가능. **2. 태스크 특화 지표**: - **정확도/F1**: 분류, QA의 정답 일치율 - **Exact Match(EM)**: QA에서 정확히 일치하는 비율 - **Pass@k**: 코드 생성에서 k번 시도 내 정답률 (HumanEval 벤치마크) **3. LLM 특화 평가**: - **Perplexity**: 모델의 예측 불확실성, 낮을수록 좋음 - **Toxicity Score**: 유해 콘텐츠 비율 (Perspective API) - **Hallucination Rate**: 환각(사실과 다른 생성) 비율 - **Faithfulness**: RAG에서 검색 문서 기반 응답 비율 **4. Human Evaluation**: - LLM-as-Judge: GPT-4로 응답 품질 평가 (비용 효율적) - Elo Rating: 두 모델 응답을 비교하여 상대 순위 결정 (Chatbot Arena 방식) **실무 팁**: 자동 지표만으로는 한계가 있으므로, 정기적인 사람 평가와 사용자 피드백을 병행하세요. 도메인 특화 평가셋을 직접 구축하는 것도 중요합니다.

### 실습 과제

```
과제 1: 기본 파이프라인 구축
- MLflow로 실험 추적 설정
- 간단한 모델 버전 관리 구현

과제 2: 모니터링 대시보드
- Prometheus + Grafana로 메트릭 시각화
- 지연시간, 에러율, 비용 대시보드

과제 3: CI/CD 파이프라인
- GitHub Actions로 모델 테스트 자동화
- 모델 배포 자동화 (Staging → Production)
```

---

## 주요 MLOps 도구 비교 (2024-2025)

```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│     영역        │    오픈소스     │     관리형      │     용도        │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ 실험 추적       │ MLflow          │ Weights&Biases  │ 학습 기록       │
│ 파이프라인      │ Kubeflow        │ SageMaker       │ 워크플로우      │
│ 피처 스토어     │ Feast           │ Tecton          │ 피처 관리       │
│ 모델 서빙       │ vLLM, TGI       │ SageMaker EP    │ 추론 서비스     │
│ 모니터링        │ Prometheus      │ DataDog         │ 성능 추적       │
│ LLM Gateway     │ LiteLLM         │ Portkey         │ LLM 통합 관리   │
│ LLM 평가        │ RAGAS, DeepEval │ Langsmith       │ 품질 평가       │
│ Guardrails     │ Guardrails AI   │ AWS Bedrock     │ 안전성 보장     │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

---

## 2024-2025 MLOps/LLMOps 트렌드

### 1. LLMOps의 부상

```
Traditional MLOps                          LLMOps (2024-2025)
┌────────────────────────┐                ┌────────────────────────┐
│ • 모델 학습 중심       │                │ • 프롬프트 엔지니어링  │
│ • 피처 엔지니어링      │  ───────────▶  │ • RAG 파이프라인       │
│ • 배치 예측           │                │ • Fine-tuning/PEFT    │
│ • A/B 테스트          │                │ • 멀티 프로바이더 관리 │
│ • 데이터 드리프트     │                │ • 비용/토큰 최적화     │
│                        │                │ • 환각/안전성 모니터링 │
└────────────────────────┘                └────────────────────────┘
```

### 2. 핵심 트렌드

| 트렌드 | 설명 | 주요 도구 |
|--------|------|-----------|
| **LLM Gateway** | 멀티 프로바이더 통합, 라우팅, 캐싱 | LiteLLM, Portkey, OpenRouter |
| **Prompt Management** | 프롬프트 버전 관리, A/B 테스트 | Langsmith, PromptLayer |
| **Guardrails** | 입출력 안전성, 환각 방지 | Guardrails AI, NeMo Guardrails |
| **LLM Observability** | 트레이싱, 비용 분석, 품질 모니터링 | Langfuse, Helicone, Phoenix |
| **Semantic Caching** | 의미 기반 응답 캐싱으로 비용 절감 | GPTCache, Redis + Vector |
| **Edge AI** | 모바일/엣지 디바이스 모델 배포 | ONNX, Core ML, TensorFlow Lite |

### 3. 비용 최적화 전략

```python
# LLM 비용 최적화 예시
class CostOptimizer:
    """LLM 비용 최적화 전략"""

    # 모델별 비용 (1K 토큰 기준, USD)
    PRICING = {
        "gpt-4o": {"input": 0.0025, "output": 0.01},
        "gpt-4o-mini": {"input": 0.00015, "output": 0.0006},
        "claude-3.5-sonnet": {"input": 0.003, "output": 0.015},
        "claude-3.5-haiku": {"input": 0.0008, "output": 0.004},
        "llama-3.2-8b": {"input": 0.0001, "output": 0.0001},  # Self-hosted 추정
    }

    def __init__(self):
        self.semantic_cache = SemanticCache()
        self.prompt_compressor = PromptCompressor()

    async def optimize_request(
        self,
        messages: list[dict],
        required_quality: str = "standard"  # "premium", "standard", "budget"
    ) -> tuple[str, list[dict]]:
        """요청 최적화: 모델 선택 + 프롬프트 압축"""

        # 1. 캐시 확인 (비용 0)
        cached = await self.semantic_cache.get(messages)
        if cached:
            return "cache_hit", messages

        # 2. 프롬프트 압축 (토큰 수 감소)
        compressed = await self.prompt_compressor.compress(messages)

        # 3. 복잡도 분석으로 모델 선택
        complexity = self._analyze_complexity(compressed)

        model = self._select_model(complexity, required_quality)

        return model, compressed

    def _analyze_complexity(self, messages: list[dict]) -> str:
        """태스크 복잡도 분석"""
        total_length = sum(len(m.get("content", "")) for m in messages)

        # 복잡한 지시사항 패턴
        complex_patterns = ["분석해", "비교해", "추론해", "step by step", "자세히"]
        has_complex = any(p in str(messages) for p in complex_patterns)

        if total_length > 3000 or has_complex:
            return "high"
        elif total_length > 500:
            return "medium"
        return "low"

    def _select_model(self, complexity: str, quality: str) -> str:
        """복잡도와 품질 요구사항에 따른 모델 선택"""
        selection_matrix = {
            ("high", "premium"): "gpt-4o",
            ("high", "standard"): "claude-3.5-sonnet",
            ("high", "budget"): "gpt-4o-mini",
            ("medium", "premium"): "claude-3.5-sonnet",
            ("medium", "standard"): "gpt-4o-mini",
            ("medium", "budget"): "claude-3.5-haiku",
            ("low", "premium"): "gpt-4o-mini",
            ("low", "standard"): "claude-3.5-haiku",
            ("low", "budget"): "llama-3.2-8b",
        }
        return selection_matrix.get((complexity, quality), "gpt-4o-mini")

    def calculate_cost(self, model: str, input_tokens: int, output_tokens: int) -> float:
        """비용 계산"""
        pricing = self.PRICING.get(model, self.PRICING["gpt-4o-mini"])
        return (input_tokens / 1000 * pricing["input"] +
                output_tokens / 1000 * pricing["output"])

    def estimate_monthly_savings(
        self,
        daily_requests: int,
        avg_tokens: int = 1000,
        cache_hit_rate: float = 0.3,
        compression_rate: float = 0.2
    ) -> dict:
        """월간 비용 절감 예측"""
        base_cost = daily_requests * 30 * self.calculate_cost("gpt-4o", avg_tokens, avg_tokens)

        # 캐싱으로 인한 절감
        cache_savings = base_cost * cache_hit_rate

        # 압축으로 인한 절감
        remaining_requests = daily_requests * (1 - cache_hit_rate)
        compression_savings = remaining_requests * 30 * self.calculate_cost(
            "gpt-4o",
            int(avg_tokens * compression_rate),
            avg_tokens
        )

        # 모델 다운그레이드로 인한 절감 (50%가 저렴한 모델 사용 가정)
        downgrade_savings = base_cost * 0.5 * 0.7

        return {
            "base_monthly_cost": base_cost,
            "cache_savings": cache_savings,
            "compression_savings": compression_savings,
            "downgrade_savings": downgrade_savings,
            "total_savings": cache_savings + compression_savings + downgrade_savings,
            "optimized_cost": base_cost - (cache_savings + compression_savings + downgrade_savings)
        }
```

---

## 실무 문제 해결 가이드

### 문제 1: 모델 배포 후 성능 저하

```
증상: 테스트에서는 좋았는데 프로덕션에서 성능 급락

원인 분석 체크리스트:
┌────────────────────────────────────────────────────────────┐
│ 1. 데이터 드리프트 확인                                    │
│    → 프로덕션 데이터 분포와 학습 데이터 비교              │
│                                                            │
│ 2. Feature Skew 확인                                       │
│    → 학습 때와 서빙 때 피처 계산 로직 일치 여부           │
│                                                            │
│ 3. 전처리 불일치                                           │
│    → 정규화, 토큰화 등 전처리 파이프라인 일관성           │
│                                                            │
│ 4. 환경 차이                                               │
│    → 라이브러리 버전, GPU/CPU 차이, 양자화 영향           │
│                                                            │
│ 5. 트래픽 패턴                                             │
│    → 테스트와 다른 쿼리 유형, 언어, 길이                  │
└────────────────────────────────────────────────────────────┘

해결 방안:
- Shadow Deployment로 사전 검증
- Feature Store로 학습-서빙 일관성 보장
- 프로덕션 트래픽 샘플로 오프라인 평가 추가
```

### 문제 2: LLM 비용 폭발

```
증상: 월 LLM API 비용이 예산의 3배 초과

분석 및 해결:
┌────────────────────────────────────────────────────────────┐
│ 1. 비용 분석                                               │
│    → 모델별, 사용자별, 기능별 비용 breakdown              │
│    → 상위 10% 요청이 전체 비용의 몇 %인지 파악            │
│                                                            │
│ 2. 프롬프트 최적화                                         │
│    → 불필요한 시스템 프롬프트 제거                        │
│    → Few-shot 예제 수 최적화                              │
│    → 프롬프트 압축 기법 적용                              │
│                                                            │
│ 3. 캐싱 전략                                               │
│    → Semantic Cache로 유사 쿼리 재사용                    │
│    → FAQ 등 정형화된 질문 사전 캐싱                       │
│                                                            │
│ 4. 모델 티어링                                             │
│    → 단순 태스크는 저렴한 모델 (Haiku, GPT-4o-mini)       │
│    → 복잡한 태스크만 프리미엄 모델 사용                   │
│                                                            │
│ 5. 요청 제한                                               │
│    → 사용자별 일일 토큰 한도                              │
│    → 최대 출력 토큰 제한                                  │
└────────────────────────────────────────────────────────────┘
```

### 문제 3: 환각(Hallucination) 발생

```
증상: LLM이 사실과 다른 정보를 자신있게 생성

대응 전략:
┌────────────────────────────────────────────────────────────┐
│ 1. RAG 도입                                                │
│    → 신뢰할 수 있는 소스에서 정보 검색 후 생성            │
│    → "검색된 문서만을 기반으로 답변하세요" 지시           │
│                                                            │
│ 2. 출력 검증                                               │
│    → 별도 모델로 사실 확인 (Fact-checking)                │
│    → 출처 명시 요구 및 검증                               │
│                                                            │
│ 3. Guardrails 적용                                         │
│    → 특정 패턴(날짜, 수치, 이름) 검증 규칙                │
│    → "모르겠습니다" 응답 허용 프롬프트                    │
│                                                            │
│ 4. Confidence Score 활용                                   │
│    → 낮은 확신도 응답은 검토 후 반환                      │
│                                                            │
│ 5. 모니터링                                                │
│    → 환각률 메트릭 추적                                   │
│    → 사용자 피드백으로 환각 사례 수집                     │
└────────────────────────────────────────────────────────────┘
```

---

## 모바일 AI 통합 Best Practices

### 1. 오프라인 대응

```swift
// iOS: 오프라인 모드 대응
class OfflineAwareAIService {
    private let remoteService: AIServiceClient
    private let localModel: CoreMLModel?
    private let networkMonitor = NWPathMonitor()

    private var isOnline: Bool = true

    func chat(messages: [ChatMessage]) async throws -> ChatResponse {
        if isOnline {
            do {
                return try await remoteService.chat(messages: messages)
            } catch {
                // 네트워크 실패 시 로컬 폴백
                return try await fallbackToLocal(messages: messages)
            }
        } else {
            return try await fallbackToLocal(messages: messages)
        }
    }

    private func fallbackToLocal(messages: [ChatMessage]) async throws -> ChatResponse {
        // 1. 로컬 CoreML 모델 사용 (제한된 기능)
        if let localModel = localModel {
            let result = try await localModel.predict(messages.last?.content ?? "")
            return ChatResponse(/* ... */)
        }

        // 2. 캐시된 응답 검색
        if let cached = searchCache(query: messages.last?.content ?? "") {
            return cached
        }

        // 3. 사전 정의된 응답
        return ChatResponse(/* "오프라인 모드입니다. 인터넷 연결 후 다시 시도해주세요." */)
    }
}
```

### 2. 에러 처리 및 사용자 경험

```kotlin
// Android: 견고한 에러 처리
sealed class AIResult<out T> {
    data class Success<T>(val data: T) : AIResult<T>()
    data class Error(val error: AIError) : AIResult<Nothing>()
    object Loading : AIResult<Nothing>()
}

sealed class AIError {
    object NetworkError : AIError()
    data class RateLimited(val retryAfter: Int) : AIError()
    object ServerError : AIError()
    data class ContentFiltered(val reason: String) : AIError()
    object Timeout : AIError()
}

class AIViewModel(
    private val aiService: AIServiceClient
) : ViewModel() {

    private val _state = MutableStateFlow<AIResult<ChatResponse>>(AIResult.Loading)
    val state: StateFlow<AIResult<ChatResponse>> = _state

    fun sendMessage(message: String) {
        viewModelScope.launch {
            _state.value = AIResult.Loading

            aiService.chat(listOf(ChatMessage("user", message)))
                .onSuccess { response ->
                    _state.value = AIResult.Success(response)
                }
                .onFailure { error ->
                    _state.value = AIResult.Error(mapError(error))
                    handleError(error)
                }
        }
    }

    private fun handleError(error: Throwable) {
        when (error) {
            is RateLimitException -> {
                // 재시도 타이머 표시
                scheduleRetry(error.retryAfter)
            }
            is IOException -> {
                // 오프라인 모드 전환
                enableOfflineMode()
            }
        }
    }

    private fun mapError(error: Throwable): AIError = when (error) {
        is RateLimitException -> AIError.RateLimited(error.retryAfter)
        is IOException -> AIError.NetworkError
        else -> AIError.ServerError
    }
}
```

### 3. 성능 최적화

```
모바일 AI 통합 성능 체크리스트:
┌────────────────────────────────────────────────────────────┐
│ [ ] 스트리밍 응답으로 체감 지연 감소                       │
│ [ ] 로컬 캐시로 반복 요청 최적화                           │
│ [ ] 백그라운드 프리페칭 (예측 가능한 쿼리)                 │
│ [ ] 요청 압축 (gzip)                                       │
│ [ ] 연결 재사용 (HTTP/2, Keep-Alive)                       │
│ [ ] 토큰 제한으로 응답 크기 관리                           │
│ [ ] 낙관적 UI 업데이트                                     │
│ [ ] 타임아웃 및 재시도 전략                                │
└────────────────────────────────────────────────────────────┘
```
