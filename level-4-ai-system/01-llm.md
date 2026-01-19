# LLM (Large Language Model) 구조

## 1. 한 줄 요약

대량의 텍스트 데이터로 학습된 신경망 모델로, 다음 단어를 예측하는 방식으로 자연어를 이해하고 생성합니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

LLM을 이해하는 가장 쉬운 비유는 **스마트폰 키보드의 자동완성 기능**입니다.

```
일반 자동완성                    LLM
┌─────────────────┐            ┌─────────────────────────────┐
│ "안녕하" → "세요" │            │ "프로젝트 일정이 촉박한데"  │
│                 │            │         ↓                   │
│ 단순 패턴 매칭   │            │ "우선순위를 정하고 핵심     │
│                 │            │  기능에 집중하는 것이       │
│                 │            │  좋겠습니다. 구체적으로..." │
└─────────────────┘            └─────────────────────────────┘
```

차이점:
- **자동완성**: 자주 사용하는 패턴 기억 (수천 개 패턴)
- **LLM**: 언어의 구조와 의미 이해 (수조 개 파라미터)

### 핵심 개념

1. **토큰(Token)**: 텍스트를 작은 단위로 쪼갠 것
   ```
   "Hello, World!" → ["Hello", ",", " World", "!"]
   "안녕하세요" → ["안녕", "하", "세요"]
   ```

2. **컨텍스트 윈도우**: 한 번에 처리할 수 있는 토큰 수
   - GPT-4o: 128K 토큰 (책 한 권 분량)
   - Claude 3.5 Sonnet: 200K 토큰 (여러 권의 문서)
   - Gemini 1.5 Pro: 2M 토큰 (대규모 코드베이스)

3. **파라미터**: 모델이 학습한 지식의 양
   - GPT-3: 1,750억 개
   - GPT-4: 약 1.7조 개 (추정, MoE 구조)
   - Llama 3.1: 4050억 개

---

## 3. 구조 다이어그램

### Transformer 아키텍처 (간략화)

```
                    입력: "오늘 날씨가"
                           │
                           ▼
┌──────────────────────────────────────────────┐
│              토큰화 (Tokenization)            │
│       ["오늘", "날씨", "가"]                  │
└──────────────────────┬───────────────────────┘
                       ▼
┌──────────────────────────────────────────────┐
│            임베딩 (Embedding)                 │
│    각 토큰을 고차원 벡터로 변환               │
│    [0.2, -0.5, ...] [0.8, 0.1, ...] [...]   │
└──────────────────────┬───────────────────────┘
                       ▼
┌──────────────────────────────────────────────┐
│        Self-Attention (핵심!)                │
│    "오늘"과 "날씨"의 관계를 파악             │
│    단어들 간의 문맥적 연결 학습              │
└──────────────────────┬───────────────────────┘
                       ▼
┌──────────────────────────────────────────────┐
│           Feed Forward Network               │
│    비선형 변환으로 패턴 추출                 │
└──────────────────────┬───────────────────────┘
                       ▼
                 (여러 층 반복)
                       │
                       ▼
┌──────────────────────────────────────────────┐
│              출력 확률 분포                   │
│    "좋습니다": 0.3, "맑습니다": 0.25, ...    │
└──────────────────────┬───────────────────────┘
                       ▼
                출력: "좋습니다"
```

### Self-Attention 시각화

```
        "The cat sat on the mat"

         The  cat  sat  on  the  mat
    The  [0.1][0.3][0.1][0.1][0.2][0.2]
    cat  [0.2][0.1][0.4][0.1][0.1][0.1]  ← cat은 sat과 강하게 연결
    sat  [0.1][0.3][0.1][0.2][0.1][0.2]
    on   [0.1][0.1][0.2][0.1][0.1][0.4]  ← on은 mat과 강하게 연결
    the  [0.1][0.2][0.1][0.1][0.1][0.4]
    mat  [0.1][0.1][0.2][0.3][0.2][0.1]

    → 단어들 간의 관계를 학습
```

### 2024-2025년 주요 LLM 아키텍처 트렌드

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    최신 LLM 아키텍처 발전 동향                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. Mixture of Experts (MoE)                                           │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  입력 → Router → Expert 1 (활성화)                          │   │
│     │                 → Expert 2 (비활성화)                        │   │
│     │                 → Expert 3 (활성화)                          │   │
│     │                 → Expert N (비활성화)                        │   │
│     │  • GPT-4, Mixtral, Gemini 등이 사용                         │   │
│     │  • 전체 파라미터 중 일부만 활성화 → 비용 효율적              │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. Multi-Modal LLM                                                    │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  텍스트 ──┐                                                  │   │
│     │  이미지 ──┼──→ 통합 인코더 → LLM → 응답                     │   │
│     │  오디오 ──┘                                                  │   │
│     │  • GPT-4o, Claude 3.5, Gemini 1.5 등                        │   │
│     │  • 모바일 앱에서 사진+텍스트 동시 입력 가능                 │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. Small Language Models (SLM)                                        │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • Phi-3 (3.8B), Gemma 2 (2B~27B), Llama 3.2 (1B~3B)       │   │
│     │  • 온디바이스 실행 가능 (iOS, Android)                      │   │
│     │  • 프라이버시 보장, 오프라인 동작                           │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 모바일 앱에서의 LLM 활용

#### 1. 챗봇 / 고객 지원

```swift
// iOS에서 LLM API 호출 예시
struct ChatMessage: Codable {
    let role: String  // "user" or "assistant"
    let content: String
}

class ChatService {
    private let apiClient: APIClient
    private var conversationHistory: [ChatMessage] = []

    // 시스템 프롬프트 상수화
    private let systemPrompt = "당신은 친절한 고객 지원 전문가입니다."

    func sendMessage(_ userMessage: String) async throws -> String {
        conversationHistory.append(
            ChatMessage(role: "user", content: userMessage)
        )

        let requestBody: [String: Any] = [
            "model": "gpt-4o",
            "messages": conversationHistory.map { ["role": $0.role, "content": $0.content] },
            "max_tokens": 1000,
            "temperature": 0.7
        ]

        let response = try await apiClient.post(
            "/chat/completions",
            body: requestBody
        )

        let assistantMessage = response.choices[0].message.content
        conversationHistory.append(
            ChatMessage(role: "assistant", content: assistantMessage)
        )

        return assistantMessage
    }

    // 스트리밍 응답 (UX 개선)
    func sendMessageStreaming(_ userMessage: String) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                let requestBody: [String: Any] = [
                    "model": "gpt-4o",
                    "messages": [["role": "user", "content": userMessage]],
                    "stream": true
                ]

                for try await chunk in apiClient.streamPost("/chat/completions", body: requestBody) {
                    continuation.yield(chunk.choices[0].delta.content ?? "")
                }
                continuation.finish()
            }
        }
    }
}
```

#### 2. 콘텐츠 요약

```kotlin
// Android에서 긴 글 요약
class SummarizationService(private val llmClient: LLMClient) {

    companion object {
        private const val DEFAULT_MAX_LENGTH = 100
        private const val TOKEN_MULTIPLIER = 2
    }

    suspend fun summarize(longText: String, maxLength: Int = DEFAULT_MAX_LENGTH): String {
        val prompt = """
            다음 글을 ${maxLength}자 이내로 요약해주세요.
            핵심 내용만 포함하고, 한국어로 작성해주세요.

            글:
            $longText

            요약:
        """.trimIndent()

        return llmClient.complete(
            prompt = prompt,
            maxTokens = maxLength * TOKEN_MULTIPLIER,  // 토큰은 글자보다 적음
            temperature = 0.3f  // 창의성 낮게 (사실적 요약)
        )
    }

    // 다단계 요약 (긴 문서용)
    suspend fun summarizeLongDocument(document: String, chunkSize: Int = 4000): String {
        val chunks = document.chunked(chunkSize)
        val summaries = chunks.map { chunk -> summarize(chunk, maxLength = 200) }
        return summarize(summaries.joinToString("\n\n"), maxLength = 300)
    }
}
```

#### 3. 스마트 검색 (의미 기반)

```typescript
// React Native - 자연어 검색
interface Product {
  id: string;
  name: string;
  description: string;
  category: string;
}

const semanticSearch = async (query: string, products: Product[]): Promise<Product[]> => {
  const prompt = `
    사용자 검색어: "${query}"

    다음 상품 중 관련된 것을 찾아주세요:
    ${products.map((p, i) => `${i}. ${p.name}: ${p.description}`).join('\n')}

    관련 상품 인덱스를 JSON 배열로 반환: [0, 3, 5]
  `;

  const response = await llmClient.complete(prompt);
  const indices: number[] = JSON.parse(response);
  return indices.map(i => products[i]);
};

// 사용 예시
// 검색어: "비 오는 날 신을 신발"
// 결과: 방수 운동화, 레인부츠, 고어텍스 하이킹화
```

#### 4. 온디바이스 LLM (프라이버시 보호)

```swift
// iOS - Core ML로 온디바이스 LLM 실행
import CoreML

class OnDeviceLLM {
    private var model: MLModel?

    func loadModel() async throws {
        // Apple의 MLX 또는 변환된 GGUF 모델 사용
        let config = MLModelConfiguration()
        config.computeUnits = .cpuAndNeuralEngine

        model = try await MLModel.load(
            contentsOf: Bundle.main.url(forResource: "phi-3-mini", withExtension: "mlmodelc")!,
            configuration: config
        )
    }

    func generate(prompt: String) async throws -> String {
        guard let model = model else { throw LLMError.modelNotLoaded }

        // 온디바이스에서 추론 실행
        // 네트워크 불필요, 개인정보 보호
        let input = try MLDictionaryFeatureProvider(dictionary: ["prompt": prompt])
        let output = try await model.prediction(from: input)

        return output.featureValue(for: "response")?.stringValue ?? ""
    }
}
```

### 주요 LLM API 비교 (2024-2025 기준)

| 모델 | 제공사 | 강점 | 컨텍스트 | 가격 (1M 토큰 Input/Output) |
|------|--------|------|----------|------------------------------|
| GPT-4o | OpenAI | 범용성, 멀티모달, 속도 | 128K | $2.5 / $10 |
| GPT-4o-mini | OpenAI | 비용 효율, 빠른 응답 | 128K | $0.15 / $0.60 |
| Claude 3.5 Sonnet | Anthropic | 코딩, 긴 문맥, 안전성 | 200K | $3 / $15 |
| Claude 3.5 Haiku | Anthropic | 빠른 응답, 저비용 | 200K | $0.25 / $1.25 |
| Gemini 1.5 Pro | Google | 초장문 컨텍스트, 멀티모달 | 2M | $1.25 / $5 |
| Gemini 1.5 Flash | Google | 속도, 비용 효율 | 1M | $0.075 / $0.30 |
| Llama 3.1 405B | Meta | 오픈소스, 커스텀 가능 | 128K | 셀프호스팅 |
| Mistral Large | Mistral | 유럽 데이터 규정 준수 | 128K | $2 / $6 |

### 모바일 앱에서 LLM 비용 최적화 전략

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       LLM 비용 최적화 전략                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 모델 티어링 (Model Tiering)                                        │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  간단한 질문 ────→ GPT-4o-mini / Gemini Flash (저비용)      │   │
│     │  복잡한 분석 ────→ GPT-4o / Claude Sonnet (고성능)          │   │
│     │  라우터가 질문 복잡도 판단 후 적절한 모델 선택               │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 캐싱 전략                                                          │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 자주 묻는 질문 → Redis/메모리 캐시                       │   │
│     │  • 시맨틱 캐싱: 비슷한 질문도 캐시 히트                      │   │
│     │  • 비용 절감 60-80% 가능                                     │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. 프롬프트 최적화                                                    │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 불필요한 컨텍스트 제거                                    │   │
│     │  • 출력 길이 제한 (max_tokens)                               │   │
│     │  • 효율적인 시스템 프롬프트 설계                             │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. 장단점

### 장점

| 항목 | 설명 |
|------|------|
| **범용성** | 하나의 모델로 다양한 작업 수행 (요약, 번역, 코딩, 분석) |
| **자연어 인터페이스** | 복잡한 쿼리 없이 자연어로 요청 가능 |
| **빠른 프로토타이핑** | API 호출만으로 AI 기능 구현 |
| **지속적 개선** | 모델 업그레이드 시 자동으로 성능 향상 |
| **멀티모달 지원** | 텍스트, 이미지, 오디오 통합 처리 (GPT-4o, Gemini) |

### 단점

| 항목 | 설명 |
|------|------|
| **비용** | 대량 호출 시 비용 급증 (토큰 단위 과금) |
| **지연시간** | 응답 생성에 수 초 소요 (스트리밍으로 완화 가능) |
| **환각(Hallucination)** | 그럴듯하지만 틀린 정보 생성 가능 |
| **데이터 프라이버시** | 민감한 데이터를 외부 API로 전송해야 함 |
| **일관성** | 같은 입력에도 다른 출력 가능 (temperature 조절로 완화) |

### 한계와 해결책

```
문제: 환각 (Hallucination)
┌─────────────────────────────────────────────────────────────┐
│ Q: "삼성전자의 2024년 3분기 매출은?"                        │
│ A: "삼성전자의 2024년 3분기 매출은 79조원입니다." (❌ 거짓) │
└─────────────────────────────────────────────────────────────┘

해결책: RAG (Retrieval-Augmented Generation)
┌─────────────────────────────────────────────────────────────┐
│ 1. 실제 재무 데이터베이스에서 검색                          │
│ 2. 검색된 데이터를 LLM에게 제공                            │
│ 3. LLM이 데이터 기반으로 답변                              │
│ A: "삼성전자의 2024년 3분기 매출은 79조원입니다.           │
│     출처: 삼성전자 분기보고서 (2024.10.30)"                │
└─────────────────────────────────────────────────────────────┘

문제: 높은 비용
┌─────────────────────────────────────────────────────────────┐
│ 해결책:                                                     │
│ 1. 캐싱: 반복 질문 캐시 (60-80% 비용 절감)                 │
│ 2. 모델 티어링: 간단한 질문은 저비용 모델 사용              │
│ 3. 프롬프트 압축: 불필요한 컨텍스트 제거                    │
│ 4. 배치 처리: 여러 요청 묶어서 처리                        │
└─────────────────────────────────────────────────────────────┘

문제: 지연시간
┌─────────────────────────────────────────────────────────────┐
│ 해결책:                                                     │
│ 1. 스트리밍: 토큰 단위로 실시간 응답 표시                   │
│ 2. 낙관적 UI: 로딩 중에도 인터랙션 허용                    │
│ 3. 프리페칭: 예상되는 다음 질문 미리 처리                   │
│ 4. Edge 배포: 사용자와 가까운 서버에서 처리                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. 내 생각

```
이 섹션은 학습 후 직접 작성해보세요.

- LLM을 처음 사용해본 경험은?
- 가장 인상적이었던 기능은?
- 실제 프로젝트에 적용한다면 어떤 기능?
- 우려되는 점은?
```

---

## 7. 추가 질문

### 초급

1. 토큰이란 무엇이고, 왜 글자 수와 다른가요?

> **답변**: 토큰은 LLM이 텍스트를 처리하는 기본 단위입니다. 영어에서는 대략 하나의 단어가 1-2개 토큰이고, 한국어에서는 한 글자가 1-2개 토큰에 해당합니다. 글자 수와 다른 이유는 LLM이 텍스트를 통계적으로 자주 등장하는 패턴(서브워드)으로 분할하는 BPE(Byte Pair Encoding) 알고리즘을 사용하기 때문입니다. 예를 들어 "unhappiness"는 "un", "happiness"로 분할될 수 있고, 자주 쓰이지 않는 단어는 더 많은 토큰으로 분할됩니다. 실무에서는 OpenAI의 tiktoken 라이브러리로 정확한 토큰 수를 계산할 수 있으며, 대략적으로 영어는 4글자당 1토큰, 한국어는 1-2글자당 1토큰으로 추정합니다.

2. Temperature 파라미터는 어떤 역할을 하나요?

> **답변**: Temperature는 LLM 응답의 "창의성" 또는 "무작위성"을 조절하는 파라미터입니다. 0에 가까울수록 가장 확률 높은 다음 토큰을 선택하여 결정론적이고 일관된 응답을 생성하며, 1에 가까울수록 다양한 토큰을 선택할 확률이 높아져 창의적이고 다양한 응답을 생성합니다. 모바일 앱에서 실무적으로 사용할 때, 고객 지원 챗봇처럼 정확한 정보가 필요한 경우 0.1-0.3을, 창작 글쓰기나 브레인스토밍에는 0.7-1.0을 사용합니다. 코드 생성에는 0.2-0.4가 적합하며, 팩트 기반 요약에는 0에 가까운 값을 권장합니다.

3. 컨텍스트 윈도우가 꽉 차면 어떻게 되나요?

> **답변**: 컨텍스트 윈도우가 꽉 차면 API 호출이 실패하거나, 초과된 부분이 잘려서 처리됩니다. 이는 모바일 챗봇에서 긴 대화를 나눌 때 흔히 발생하는 문제입니다. 해결 방법으로는 (1) 오래된 메시지부터 제거하는 슬라이딩 윈도우 방식, (2) 이전 대화 내용을 LLM으로 요약하여 압축하는 방식, (3) RAG를 사용해 필요한 정보만 검색하여 컨텍스트에 포함하는 방식이 있습니다. 실무에서는 시스템 프롬프트 + 최근 N개 메시지만 유지하고, 중요한 정보는 별도로 저장했다가 필요할 때 검색하는 하이브리드 접근법을 많이 사용합니다.

### 중급

4. Self-Attention이 RNN보다 좋은 이유는 무엇인가요?

> **답변**: Self-Attention이 RNN보다 우수한 이유는 크게 세 가지입니다. 첫째, **병렬 처리**가 가능합니다. RNN은 순차적으로 토큰을 처리해야 하지만, Self-Attention은 모든 토큰을 동시에 처리할 수 있어 GPU 활용도가 높고 학습 속도가 빠릅니다. 둘째, **장거리 의존성**을 잘 포착합니다. RNN은 문장이 길어질수록 앞부분의 정보가 희석되는 vanishing gradient 문제가 있지만, Self-Attention은 모든 토큰 쌍 간의 관계를 직접 계산하므로 문장 처음과 끝의 관계도 동등하게 학습합니다. 셋째, **해석 가능성**이 높습니다. Attention 가중치를 시각화하여 모델이 어떤 단어에 집중했는지 확인할 수 있습니다. 실제로 이런 특성 덕분에 GPT, Claude, Gemini 등 모든 최신 LLM이 Transformer 기반입니다.

5. Few-shot learning과 Zero-shot learning의 차이는?

> **답변**: Zero-shot learning은 예시 없이 작업 지시만으로 모델이 작업을 수행하는 것이고, Few-shot learning은 몇 개의 예시를 프롬프트에 포함하여 모델이 패턴을 학습하게 하는 것입니다. 모바일 앱 개발 실무에서 예를 들면, 상품 리뷰 감성 분석을 할 때 Zero-shot은 "다음 리뷰가 긍정인지 부정인지 판단해주세요: {리뷰}"처럼 지시만 하고, Few-shot은 "리뷰: '배송이 빨라요' → 긍정, 리뷰: '품질이 나빠요' → 부정, 리뷰: {새 리뷰} → ?"처럼 예시를 포함합니다. Few-shot은 복잡하거나 도메인 특화 작업에서 정확도가 높지만 토큰 소비가 많고, Zero-shot은 간단한 작업에서 비용 효율적입니다. 일반적으로 먼저 Zero-shot으로 시도하고, 결과가 불만족스러우면 Few-shot으로 전환합니다.

6. LLM의 응답 시간을 줄이려면 어떤 방법이 있나요?

> **답변**: LLM 응답 시간을 줄이는 방법은 다음과 같습니다. (1) **스트리밍 응답**: 완성된 응답을 기다리지 않고 토큰 단위로 실시간 표시하여 체감 지연시간을 크게 줄입니다. 모바일 앱에서는 ChatGPT처럼 타이핑 효과를 주면 사용자 경험이 향상됩니다. (2) **더 작은 모델 사용**: GPT-4o 대신 GPT-4o-mini나 Claude Haiku를 사용하면 2-3배 빨라집니다. (3) **프롬프트 최적화**: 불필요한 지시문과 컨텍스트를 줄여 입력 토큰을 감소시킵니다. (4) **max_tokens 제한**: 출력 길이를 적절히 제한합니다. (5) **캐싱**: 동일하거나 유사한 질문은 캐시된 응답을 반환합니다. (6) **Edge 배포**: Cloudflare Workers AI 같은 엣지 서비스를 활용하면 네트워크 지연을 줄일 수 있습니다.

### 심화

7. KV Cache는 무엇이고 왜 중요한가요?

> **답변**: KV Cache(Key-Value Cache)는 Transformer의 Self-Attention 계산에서 이전에 계산된 Key와 Value 벡터를 캐싱하여 재사용하는 최적화 기법입니다. LLM이 토큰을 하나씩 생성할 때, 매번 전체 시퀀스의 Attention을 다시 계산하면 O(n²) 복잡도로 매우 비효율적입니다. KV Cache를 사용하면 이전 토큰들의 K, V값을 저장해두고 새 토큰만 추가로 계산하면 되어 O(n)으로 줄어듭니다. 실무에서 중요한 이유는, KV Cache 크기가 GPU 메모리를 많이 차지하여 동시 처리 가능한 요청 수(throughput)에 직접 영향을 주기 때문입니다. vLLM 같은 서빙 프레임워크는 PagedAttention으로 KV Cache를 효율적으로 관리하여 2-4배 더 많은 요청을 처리할 수 있게 합니다. 긴 컨텍스트(128K+ 토큰)를 사용할 때 특히 중요합니다.

8. Mixture of Experts(MoE) 아키텍처의 장점은?

> **답변**: MoE(Mixture of Experts)는 모델을 여러 개의 "전문가" 서브네트워크로 나누고, 입력에 따라 일부 전문가만 활성화하는 아키텍처입니다. 장점으로는 (1) **비용 효율성**: GPT-4가 1.7조 파라미터지만 추론 시 실제 활성화되는 것은 훨씬 적어, 작은 모델 수준의 연산 비용으로 큰 모델의 성능을 얻습니다. (2) **전문화**: 각 전문가가 특정 도메인(코드, 수학, 언어 등)에 특화되어 전체적인 성능이 향상됩니다. (3) **확장성**: 전문가 수를 늘려 모델 용량을 키우면서도 추론 비용은 선형적으로만 증가합니다. Mixtral 8x7B는 실제로 7B 모델 8개를 사용하지만 추론 시에는 2개만 활성화되어 14B 수준의 연산으로 56B급 성능을 냅니다. 모바일 앱 개발자 입장에서는 MoE 기반 모델(GPT-4, Gemini)이 비용 대비 성능이 좋다는 점을 인지하면 됩니다.

9. Quantization이 모델 성능에 미치는 영향은?

> **답변**: Quantization(양자화)는 모델 가중치의 정밀도를 줄이는 기법으로, 예를 들어 32비트 부동소수점(FP32)을 8비트 정수(INT8)나 4비트(INT4)로 변환합니다. 영향으로는 (1) **메모리 감소**: 4비트 양자화 시 모델 크기가 1/8로 줄어들어, 7B 모델이 약 4GB로 모바일 기기에서도 실행 가능해집니다. (2) **속도 향상**: 메모리 대역폭 병목이 줄어 추론 속도가 1.5-2배 빨라집니다. (3) **품질 저하**: 일반적으로 8비트는 거의 품질 손실이 없고, 4비트는 1-3% 정도 성능 하락이 있습니다. 그러나 GPTQ, AWQ, GGUF 같은 최신 양자화 기법은 품질 손실을 최소화합니다. 실무에서 온디바이스 LLM(Llama.cpp, MLX)을 사용할 때 4비트 양자화 모델을 주로 사용하며, 서버에서는 8비트나 FP16으로 품질과 성능의 균형을 맞춥니다.

### 실습 과제

```
과제 1: 토큰 카운터 만들기
- tiktoken 라이브러리로 텍스트의 토큰 수 계산
- 비용 예측 기능 추가

과제 2: 스트리밍 응답 구현
- LLM API의 스트리밍 기능 사용
- 실시간으로 응답을 화면에 표시

과제 3: 프롬프트 최적화
- 같은 결과를 더 적은 토큰으로 얻기
- 응답 품질 유지하면서 비용 50% 절감
```

### 실습 과제 예시 코드

```python
# 과제 1: 토큰 카운터 및 비용 계산기
import tiktoken

class TokenCounter:
    # 모델별 가격 (1M 토큰당 USD)
    PRICING = {
        "gpt-4o": {"input": 2.5, "output": 10},
        "gpt-4o-mini": {"input": 0.15, "output": 0.60},
        "claude-3-5-sonnet": {"input": 3, "output": 15},
    }

    def __init__(self, model: str = "gpt-4o"):
        self.model = model
        self.encoding = tiktoken.encoding_for_model("gpt-4o")

    def count_tokens(self, text: str) -> int:
        """텍스트의 토큰 수 계산"""
        return len(self.encoding.encode(text))

    def estimate_cost(self, input_text: str, expected_output_tokens: int = 500) -> dict:
        """비용 예측"""
        input_tokens = self.count_tokens(input_text)
        pricing = self.PRICING.get(self.model, self.PRICING["gpt-4o"])

        input_cost = (input_tokens / 1_000_000) * pricing["input"]
        output_cost = (expected_output_tokens / 1_000_000) * pricing["output"]

        return {
            "input_tokens": input_tokens,
            "estimated_output_tokens": expected_output_tokens,
            "input_cost_usd": round(input_cost, 6),
            "output_cost_usd": round(output_cost, 6),
            "total_cost_usd": round(input_cost + output_cost, 6)
        }

# 사용 예시
counter = TokenCounter("gpt-4o-mini")
cost = counter.estimate_cost("안녕하세요, 오늘 날씨가 어떤가요?", expected_output_tokens=100)
print(f"예상 비용: ${cost['total_cost_usd']}")
```

---

## 8. 실무 운영 시 주의사항

### 자주 겪는 문제와 해결책

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      실무 운영 시 주의사항                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. Rate Limiting 처리                                                 │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • OpenAI: TPM(분당 토큰), RPM(분당 요청) 제한               │   │
│     │  • 429 에러 시 지수 백오프로 재시도                          │   │
│     │  • 여러 API 키 로드밸런싱 또는 예비 모델 준비                │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 비용 폭증 방지                                                     │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 일일/월간 비용 한도 설정 (OpenAI Usage Limits)            │   │
│     │  • 사용자별 요청 제한                                        │   │
│     │  • 비정상 사용 패턴 모니터링 및 알림                         │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. 프롬프트 인젝션 방어                                               │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 사용자 입력 검증 및 이스케이프                            │   │
│     │  • 시스템 프롬프트 노출 방지                                 │   │
│     │  • 역할 분리: 시스템/사용자/어시스턴트 메시지 구분           │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  4. 장애 대응                                                          │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 멀티 프로바이더 전략: OpenAI 장애 시 Claude로 폴백        │   │
│     │  • 우아한 성능 저하: AI 실패 시 기본 응답 또는 캐시 반환    │   │
│     │  • 서킷 브레이커 패턴 적용                                   │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 프롬프트 인젝션 방어 예시

```python
import re

class PromptSanitizer:
    # 위험한 패턴들
    DANGEROUS_PATTERNS = [
        r"ignore.*previous.*instructions",
        r"forget.*system.*prompt",
        r"you are now",
        r"act as",
        r"pretend to be",
    ]

    def sanitize(self, user_input: str) -> str:
        """사용자 입력 검증 및 정화"""
        # 위험 패턴 검사
        for pattern in self.DANGEROUS_PATTERNS:
            if re.search(pattern, user_input, re.IGNORECASE):
                raise ValueError("잠재적으로 위험한 입력이 감지되었습니다.")

        # 길이 제한
        max_length = 4000
        if len(user_input) > max_length:
            user_input = user_input[:max_length]

        return user_input

    def build_safe_prompt(self, system_prompt: str, user_input: str) -> list:
        """안전한 메시지 구조 생성"""
        sanitized_input = self.sanitize(user_input)

        return [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": f"<user_query>{sanitized_input}</user_query>"}
        ]
```
