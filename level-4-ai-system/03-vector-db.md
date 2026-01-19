# Vector DB (벡터 데이터베이스)

## 1. 한 줄 요약

텍스트, 이미지 등을 숫자 벡터로 변환하여 저장하고, 의미적 유사도를 기반으로 빠르게 검색하는 특수한 데이터베이스입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

Vector DB는 **음악 추천 앱의 "비슷한 곡 찾기"** 기능과 같습니다.

```
전통적 DB (SQL)                    Vector DB
┌─────────────────────┐            ┌────────────────────────────┐
│ SELECT * FROM songs │            │ "이 노래와 비슷한 곡은?"   │
│ WHERE genre = 'pop' │            │         ↓                  │
│ AND year = 2024;    │            │  노래를 벡터로 변환        │
│         ↓           │            │  [0.2, 0.8, -0.1, ...]    │
│ 정확히 일치하는 것만│            │         ↓                  │
│ 찾음                │            │  비슷한 벡터 검색          │
│                     │            │  (템포, 분위기, 장르 유사) │
│                     │            │         ↓                  │
│                     │            │  "분위기가 비슷한" 곡 반환 │
└─────────────────────┘            └────────────────────────────┘
```

### 임베딩(Embedding)이란?

텍스트나 이미지를 숫자 배열(벡터)로 변환하는 것:

```
텍스트                              벡터 (간략화)
┌──────────────────┐               ┌────────────────────────────┐
│ "맛있는 피자"     │  ──────────▶  │ [0.8, 0.2, 0.5, -0.3, ...]│
│ "pizza is yummy" │  ──────────▶  │ [0.75, 0.25, 0.48, -0.28]  │ ← 비슷!
│ "날씨가 좋다"    │  ──────────▶  │ [-0.1, 0.9, -0.4, 0.6, ...] │ ← 다름
└──────────────────┘               └────────────────────────────┘
```

의미가 비슷한 문장은 비슷한 벡터가 됨 (언어가 달라도!)

### 유사도 검색 원리

```
       벡터 공간 (2차원으로 단순화)

        ▲
        │    ★ 검색 쿼리: "맛있는 음식"
        │   ·
        │  ·  · 피자 레시피 (가까움 = 유사)
        │ ·
        │·      · 파스타 맛집 (가까움)
        │
        │              · 축구 뉴스 (멀음 = 다름)
        │
        │                    · 정치 기사 (멀음)
        └───────────────────────────────────▶

        가까운 거리 = 높은 유사도
```

---

## 3. 구조 다이어그램

### Vector DB 내부 구조

```
┌─────────────────────────────────────────────────────────────────┐
│                        Vector Database                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    인덱스 계층                            │  │
│  │  ┌─────────────────────────────────────────────────────┐ │  │
│  │  │              HNSW (Hierarchical NSW)                │ │  │
│  │  │                                                     │ │  │
│  │  │    Layer 2:  ○────────○────────○                   │ │  │
│  │  │              │        │        │                   │ │  │
│  │  │    Layer 1:  ○──○──○──○──○──○──○──○               │ │  │
│  │  │              │  │  │  │  │  │  │  │               │ │  │
│  │  │    Layer 0:  ○○○○○○○○○○○○○○○○○○○○○ (모든 벡터)    │ │  │
│  │  └─────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    스토리지 계층                          │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │  │
│  │  │ 벡터 데이터 │  │  메타데이터  │  │  원본 텍스트 │      │  │
│  │  │ [0.1, 0.2]  │  │ {id, title} │  │ "원본 문장"  │      │  │
│  │  │ [0.3, -0.1] │  │ {category}  │  │             │      │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 검색 알고리즘 비교

```
1. Brute Force (정확하지만 느림)
   ┌────────────────────────────────────────┐
   │ 모든 벡터와 거리 계산                  │
   │ O(n) - 1억개면 1억번 계산              │
   │                                        │
   │ 쿼리 ──▶ ○ ○ ○ ○ ○ ○ ○ ... 모두 비교  │
   └────────────────────────────────────────┘

2. IVF (Inverted File Index)
   ┌────────────────────────────────────────┐
   │ 클러스터로 그룹화 후 검색              │
   │ O(n/k) - k개 클러스터                  │
   │                                        │
   │       ┌───○○○ Cluster A               │
   │ 쿼리 ─┼───○○○ Cluster B ← 여기만 검색 │
   │       └───○○○ Cluster C               │
   └────────────────────────────────────────┘

3. HNSW (Hierarchical Navigable Small World)
   ┌────────────────────────────────────────┐
   │ 그래프 기반 탐색 (가장 효율적)         │
   │ O(log n)                               │
   │                                        │
   │ 쿼리 → L2 → L1 → L0 → 결과            │
   │        (넓게) (좁혀감) (정밀)          │
   └────────────────────────────────────────┘
```

### 데이터 파이프라인

```
┌─────────────────────────────────────────────────────────────────┐
│                        데이터 인덱싱                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  원본 문서         청킹            임베딩          저장         │
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │  PDF    │    │ Chunk1  │    │ [0.1,..]│    │ Vector  │     │
│  │  Word   │───▶│ Chunk2  │───▶│ [0.3,..]│───▶│   DB    │     │
│  │  HTML   │    │ Chunk3  │    │ [-0.2,.]│    │         │     │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│                                      │                         │
│                           ┌──────────┘                         │
│                           ▼                                    │
│                    ┌─────────────┐                             │
│                    │Embedding API│                             │
│                    │ (OpenAI,    │                             │
│                    │  Cohere)    │                             │
│                    └─────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

### 2024-2025년 Vector DB 생태계

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Vector DB 생태계 (2024-2025)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  클라우드 관리형 (Managed)                                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Pinecone: 가장 성숙한 관리형 서비스, 빠른 시작                  │   │
│  │  Weaviate Cloud: 하이브리드 검색, GraphQL API                    │   │
│  │  Zilliz Cloud: Milvus 기반 관리형, 대규모 확장성                 │   │
│  │  MongoDB Atlas Vector: 기존 MongoDB에 벡터 검색 추가             │   │
│  │  Supabase pgvector: PostgreSQL 기반, 풀스택 개발에 적합          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  셀프 호스팅 (Self-hosted)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Milvus: 대규모 분산 처리, 쿠버네티스 네이티브                    │   │
│  │  Qdrant: Rust 기반 고성능, 메모리 효율적                         │   │
│  │  Weaviate: 모듈형, 다양한 벡터라이저 지원                        │   │
│  │  Chroma: 경량, 개발/프로토타입용, LangChain 통합 우수             │   │
│  │  pgvector: PostgreSQL 확장, 기존 인프라 활용                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  임베딩 모델 (2024-2025 추천)                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  OpenAI text-embedding-3-small: 저비용, 좋은 성능                │   │
│  │  OpenAI text-embedding-3-large: 최고 성능, 고비용                │   │
│  │  Cohere embed-v3: 다국어 우수, 검색 최적화                       │   │
│  │  Voyage AI: 코드/법률 도메인 특화                                │   │
│  │  BGE-M3: 오픈소스 최고 성능, 다국어 지원                         │   │
│  │  Jina Embeddings v2: 8K 토큰 긴 문맥 지원                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 1. 시맨틱 검색 구현

```python
# Python - Pinecone 사용 예시
from pinecone import Pinecone, ServerlessSpec
from openai import OpenAI

# 상수 정의
EMBEDDING_MODEL = "text-embedding-3-small"
EMBEDDING_DIMENSION = 1536
DEFAULT_TOP_K = 5

# 초기화
pc = Pinecone(api_key="your-api-key")
client = OpenAI()

# 인덱스 생성 (최초 1회)
if "product-search" not in pc.list_indexes().names():
    pc.create_index(
        name="product-search",
        dimension=EMBEDDING_DIMENSION,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1")
    )

index = pc.Index("product-search")

class SemanticSearch:
    def __init__(self):
        self.embedding_model = EMBEDDING_MODEL

    def embed(self, text: str) -> list[float]:
        """텍스트를 벡터로 변환"""
        response = client.embeddings.create(
            model=self.embedding_model,
            input=text
        )
        return response.data[0].embedding

    def index_product(self, product: dict):
        """상품 정보를 벡터 DB에 저장"""
        text = f"{product['name']} {product['description']}"
        vector = self.embed(text)

        index.upsert(vectors=[{
            "id": product["id"],
            "values": vector,
            "metadata": {
                "name": product["name"],
                "price": product["price"],
                "category": product["category"],
                "image_url": product.get("image_url", "")
            }
        }])

    def batch_index(self, products: list[dict], batch_size: int = 100):
        """대량 인덱싱 (배치 처리로 효율화)"""
        for i in range(0, len(products), batch_size):
            batch = products[i:i + batch_size]
            texts = [f"{p['name']} {p['description']}" for p in batch]

            # 배치 임베딩
            response = client.embeddings.create(
                model=self.embedding_model,
                input=texts
            )

            vectors = []
            for j, p in enumerate(batch):
                vectors.append({
                    "id": p["id"],
                    "values": response.data[j].embedding,
                    "metadata": {
                        "name": p["name"],
                        "price": p["price"],
                        "category": p["category"]
                    }
                })

            index.upsert(vectors=vectors)
            print(f"Indexed {i + len(batch)} / {len(products)}")

    def search(self, query: str, top_k: int = DEFAULT_TOP_K,
               filters: dict = None) -> list[dict]:
        """자연어로 상품 검색"""
        query_vector = self.embed(query)

        results = index.query(
            vector=query_vector,
            top_k=top_k,
            filter=filters,
            include_metadata=True
        )

        return [{
            "id": match["id"],
            "score": match["score"],
            **match["metadata"]
        } for match in results["matches"]]

# 사용 예시
search = SemanticSearch()

# 검색
results = search.search(
    query="비오는 날 입기 좋은 따뜻한 옷",
    filters={"category": {"$in": ["의류", "아우터"]}}
)
# 결과: 방수 자켓, 후드티, 레인코트 등
```

### 2. 모바일 앱 연동

```kotlin
// Android - 시맨틱 검색 서비스
data class SearchResult(
    val id: String,
    val score: Float,
    val name: String,
    val description: String,
    val imageUrl: String,
    val price: Int
)

data class SearchRequest(
    val query: String,
    val filter: Map<String, Any>? = null,
    val topK: Int = 10
)

class SemanticSearchService(
    private val apiClient: ApiClient
) {
    companion object {
        private const val DEFAULT_TOP_K = 10
        private const val SEARCH_ENDPOINT = "/api/search/semantic"
        private const val SIMILAR_ENDPOINT = "/api/products/%s/similar"
    }

    suspend fun search(
        query: String,
        category: String? = null,
        limit: Int = DEFAULT_TOP_K
    ): List<SearchResult> {
        val request = SearchRequest(
            query = query,
            filter = category?.let { mapOf("category" to it) },
            topK = limit
        )

        return apiClient.post<SearchResponse>(
            endpoint = SEARCH_ENDPOINT,
            body = request
        ).results
    }

    // 유사 상품 추천
    suspend fun findSimilar(productId: String, limit: Int = 5): List<SearchResult> {
        return apiClient.get<SearchResponse>(
            endpoint = SIMILAR_ENDPOINT.format(productId),
            params = mapOf("limit" to limit)
        ).results
    }

    // 이미지 기반 검색 (멀티모달)
    suspend fun searchByImage(imageBase64: String): List<SearchResult> {
        return apiClient.post<SearchResponse>(
            endpoint = "/api/search/image",
            body = mapOf("image" to imageBase64)
        ).results
    }
}

// ViewModel에서 사용
class SearchViewModel(
    private val searchService: SemanticSearchService
) : ViewModel() {

    private val _results = MutableStateFlow<List<SearchResult>>(emptyList())
    val results: StateFlow<List<SearchResult>> = _results

    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading

    fun search(query: String) {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                _results.value = searchService.search(query)
            } catch (e: Exception) {
                // 에러 처리
            } finally {
                _isLoading.value = false
            }
        }
    }
}
```

```swift
// iOS - 시맨틱 검색 서비스
struct SearchResult: Decodable {
    let id: String
    let score: Float
    let name: String
    let description: String
    let imageUrl: String
    let price: Int
}

actor SemanticSearchService {
    private let apiClient: APIClient

    // 상수
    private static let defaultTopK = 10
    private static let searchEndpoint = "/api/search/semantic"

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func search(query: String, category: String? = nil) async throws -> [SearchResult] {
        var body: [String: Any] = [
            "query": query,
            "topK": Self.defaultTopK
        ]

        if let category = category {
            body["filter"] = ["category": category]
        }

        return try await apiClient.post(Self.searchEndpoint, body: body)
    }

    // 유사 아이템 추천
    func findSimilar(to itemId: String) async throws -> [SearchResult] {
        return try await apiClient.get("/api/products/\(itemId)/similar")
    }
}

// SwiftUI View
struct SearchView: View {
    @StateObject private var viewModel = SearchViewModel()
    @State private var searchText = ""

    var body: some View {
        NavigationView {
            VStack {
                // 검색 바
                TextField("상품 검색...", text: $searchText)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .padding()
                    .onSubmit {
                        Task {
                            await viewModel.search(query: searchText)
                        }
                    }

                // 검색 결과
                if viewModel.isLoading {
                    ProgressView()
                } else {
                    List(viewModel.results, id: \.id) { result in
                        SearchResultRow(result: result)
                    }
                }
            }
            .navigationTitle("검색")
        }
    }
}
```

### 3. 주요 Vector DB 비교 (2024-2025)

```
┌──────────────┬───────────────┬───────────────┬───────────────┬───────────────┐
│              │   Pinecone    │   Weaviate    │    Milvus     │   pgvector    │
├──────────────┼───────────────┼───────────────┼───────────────┼───────────────┤
│ 배포 방식    │ 완전 관리형   │ 관리형/셀프  │ 셀프 호스팅   │ PostgreSQL 확장│
│ 가격         │ 사용량 기반   │ 무료~기업용   │ 무료(오픈소스)│ DB 비용만     │
│ 확장성       │ 자동 확장     │ 좋음          │ 매우 좋음     │ 제한적        │
│ 인덱스       │ 자체 개발     │ HNSW         │ 다양한 지원   │ IVFFlat,HNSW │
│ 필터링       │ 메타데이터    │ GraphQL      │ 스칼라 필터   │ SQL WHERE    │
│ 지연시간     │ <50ms         │ <100ms        │ <20ms         │ DB 의존       │
│ 적합한 경우  │ 빠른 시작     │ 그래프 검색   │ 대규모 온프렘 │ 기존 PG 사용  │
│ 한국어 지원  │ 좋음          │ 좋음          │ 좋음          │ 모델 의존     │
└──────────────┴───────────────┴───────────────┴───────────────┴───────────────┘

추가 옵션 (2024-2025):
- Chroma: 로컬 개발용, 가볍고 빠름, LangChain 기본 통합
- Qdrant: 러스트 기반, 고성능, 메모리 효율
- LanceDB: 서버리스, 로컬 파일 기반, 임베디드 사용
- Turbopuffer: 초저비용, S3 기반 저장소
- MongoDB Atlas Vector: 기존 MongoDB 사용자에게 적합
```

### 4. 하이브리드 검색

```python
# 키워드 검색 + 벡터 검색 조합
from rank_bm25 import BM25Okapi
import numpy as np

class HybridSearch:
    def __init__(self, vector_store, documents: list[dict]):
        self.vector_store = vector_store

        # BM25 인덱스 초기화
        tokenized_docs = [doc["text"].split() for doc in documents]
        self.bm25 = BM25Okapi(tokenized_docs)
        self.documents = documents

    def search(self, query: str, alpha: float = 0.5, top_k: int = 10) -> list[dict]:
        """
        alpha: 0 = 키워드만, 1 = 벡터만, 0.5 = 균등
        """
        # 벡터 검색 (의미 기반)
        vector_results = self.vector_store.similarity_search_with_score(query, k=top_k * 2)

        # BM25 검색 (키워드 기반)
        tokenized_query = query.split()
        bm25_scores = self.bm25.get_scores(tokenized_query)
        bm25_top_indices = np.argsort(bm25_scores)[::-1][:top_k * 2]
        keyword_results = [(self.documents[i], bm25_scores[i]) for i in bm25_top_indices]

        # 점수 결합 (Reciprocal Rank Fusion)
        combined = self._reciprocal_rank_fusion(
            vector_results,
            keyword_results,
            alpha
        )

        return combined[:top_k]

    def _reciprocal_rank_fusion(self, vec_results, kw_results, alpha):
        """RRF 알고리즘으로 결과 융합"""
        k = 60  # RRF 상수
        scores = {}
        doc_map = {}

        for rank, (doc, score) in enumerate(vec_results):
            doc_id = doc.metadata.get("id", id(doc))
            scores[doc_id] = scores.get(doc_id, 0) + alpha / (k + rank + 1)
            doc_map[doc_id] = doc

        for rank, (doc, score) in enumerate(kw_results):
            doc_id = doc.get("id", id(doc))
            scores[doc_id] = scores.get(doc_id, 0) + (1 - alpha) / (k + rank + 1)
            if doc_id not in doc_map:
                doc_map[doc_id] = doc

        # 점수 기준 정렬
        sorted_ids = sorted(scores.keys(), key=lambda x: scores[x], reverse=True)
        return [{"doc": doc_map[doc_id], "score": scores[doc_id]} for doc_id in sorted_ids]

# 사용 예시
hybrid = HybridSearch(pinecone_index, documents)

# "삼성 갤럭시" → 키워드도 중요, 의미도 중요
results = hybrid.search("삼성 갤럭시 최신 폰", alpha=0.5)
```

### 5. 멀티모달 검색 (이미지 + 텍스트)

```python
# CLIP 모델로 이미지-텍스트 통합 검색
from transformers import CLIPProcessor, CLIPModel
import torch
from PIL import Image

class MultiModalSearch:
    def __init__(self, vector_store):
        self.model = CLIPModel.from_pretrained("openai/clip-vit-base-patch32")
        self.processor = CLIPProcessor.from_pretrained("openai/clip-vit-base-patch32")
        self.vector_store = vector_store

    def embed_image(self, image_path: str) -> list[float]:
        """이미지를 벡터로 변환"""
        image = Image.open(image_path)
        inputs = self.processor(images=image, return_tensors="pt")

        with torch.no_grad():
            image_features = self.model.get_image_features(**inputs)

        return image_features[0].tolist()

    def embed_text(self, text: str) -> list[float]:
        """텍스트를 벡터로 변환"""
        inputs = self.processor(text=[text], return_tensors="pt", padding=True)

        with torch.no_grad():
            text_features = self.model.get_text_features(**inputs)

        return text_features[0].tolist()

    def search_by_image(self, image_path: str, top_k: int = 5) -> list:
        """이미지로 유사한 상품 검색"""
        image_vector = self.embed_image(image_path)
        return self.vector_store.query(vector=image_vector, top_k=top_k)

    def search_by_text(self, text: str, top_k: int = 5) -> list:
        """텍스트로 이미지 검색"""
        text_vector = self.embed_text(text)
        return self.vector_store.query(vector=text_vector, top_k=top_k)

# 모바일 앱에서 카메라로 찍은 사진으로 검색
# "이 옷과 비슷한 상품 찾기"
```

---

## 5. 장단점

### 장점

| 항목 | 설명 |
|------|------|
| **의미 기반 검색** | 키워드 불일치 문제 해결 (동의어, 다국어 지원) |
| **빠른 검색** | 수억 개 벡터에서 밀리초 단위 검색 |
| **멀티모달** | 텍스트, 이미지, 오디오 등 다양한 데이터 검색 |
| **추천 시스템** | 유사 아이템 추천에 활용 |
| **RAG 핵심** | LLM과 결합하여 지식 검색 |

### 단점

| 항목 | 설명 |
|------|------|
| **정확한 매칭 약함** | "SKU-12345" 같은 정확한 검색은 어려움 |
| **인덱싱 비용** | 대량 데이터 임베딩 시 비용과 시간 소요 |
| **차원의 저주** | 고차원 벡터는 검색 효율 저하 |
| **블랙박스** | 왜 유사하다고 판단했는지 설명 어려움 |
| **업데이트 오버헤드** | 문서 변경 시 재임베딩 필요 |

### 선택 가이드

```
┌─────────────────────────────────────────────────────────────────┐
│                    Vector DB 선택 기준                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  프로토타입/PoC                                                │
│  └──▶ Chroma (로컬, 무료, 간편)                                │
│       또는 LanceDB (서버리스, 로컬 파일)                        │
│                                                                 │
│  스타트업/중소 규모                                            │
│  └──▶ Pinecone (관리형, 빠른 시작)                             │
│       또는 Supabase pgvector (PostgreSQL 사용 시)               │
│                                                                 │
│  대규모/온프레미스                                              │
│  └──▶ Milvus (오픈소스, 확장성)                                │
│       또는 Qdrant (고성능, 메모리 효율)                         │
│                                                                 │
│  기존 PostgreSQL 사용 중                                       │
│  └──▶ pgvector (별도 인프라 불필요)                            │
│                                                                 │
│  기존 MongoDB 사용 중                                          │
│  └──▶ MongoDB Atlas Vector Search                              │
│                                                                 │
│  비용 최적화 중요                                               │
│  └──▶ Turbopuffer (S3 기반, 초저비용)                          │
│       또는 셀프호스팅 Qdrant/Milvus                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 내 생각

```
이 섹션은 학습 후 직접 작성해보세요.

- Vector DB가 기존 검색과 다른 점은?
- 우리 서비스에서 벡터 검색이 유용한 기능은?
- 어떤 Vector DB를 선택하고 그 이유는?
- 예상되는 기술적 챌린지는?
```

---

## 7. 추가 질문

### 초급

1. 임베딩의 차원(dimension)은 무엇을 의미하나요?

> **답변**: 임베딩 차원은 텍스트나 이미지를 표현하는 벡터의 길이(숫자의 개수)입니다. 예를 들어 OpenAI의 text-embedding-3-small은 1536차원으로, 하나의 텍스트가 1536개의 숫자 배열로 표현됩니다. 차원이 높을수록 더 많은 의미적 뉘앙스를 포착할 수 있지만, 저장 공간과 검색 시간이 증가합니다. 실무에서 text-embedding-3-small(1536차원)은 대부분의 용도에 충분하며, text-embedding-3-large(3072차원)는 높은 정확도가 필요한 경우에 사용합니다. 모바일 앱 관점에서는 차원이 높을수록 API 응답 크기가 커지므로 네트워크 대역폭을 고려해야 합니다. 일반적으로 256~1536차원이 실무에서 많이 사용됩니다.

2. 코사인 유사도(Cosine Similarity)는 어떻게 계산하나요?

> **답변**: 코사인 유사도는 두 벡터가 이루는 각도의 코사인 값으로, 1에 가까울수록 유사하고 0에 가까울수록 다릅니다. 수학적으로는 `cos(θ) = (A·B) / (||A|| × ||B||)`로, 두 벡터의 내적을 각 벡터 크기의 곱으로 나눕니다. 예를 들어, "맛있는 피자"와 "pizza is delicious"의 임베딩 벡터가 0.92의 코사인 유사도를 가지면 매우 유사하다는 의미입니다. Vector DB는 이 계산을 최적화하여 수억 개 벡터 중에서 밀리초 내에 가장 유사한 벡터를 찾습니다. 모바일 앱에서 직접 계산할 일은 드물지만, 검색 결과의 score 값을 해석할 때 유용합니다. 0.8 이상이면 높은 유사도, 0.5 이하는 관련성이 낮다고 볼 수 있습니다.

3. 메타데이터 필터링이 왜 필요한가요?

> **답변**: 메타데이터 필터링은 벡터 유사도 검색에 추가 조건을 적용하는 기능입니다. 예를 들어 "따뜻한 겨울 옷"을 검색할 때, 벡터 검색만으로는 여름 옷도 결과에 포함될 수 있습니다. 메타데이터 필터로 `{"season": "winter", "category": "outerwear"}`를 추가하면 정확한 결과를 얻습니다. 실무에서 자주 사용하는 필터: (1) **카테고리/태그**: 특정 상품 카테고리만 검색 (2) **날짜 범위**: 최근 문서만 검색 (3) **접근 권한**: 사용자가 볼 수 있는 문서만 검색 (4) **가격 범위**: 예산 내 상품만 검색. Pinecone, Weaviate 등 대부분의 Vector DB가 메타데이터 필터링을 지원하며, SQL의 WHERE 절처럼 동작합니다.

### 중급

4. HNSW 알고리즘의 원리를 설명해보세요.

> **답변**: HNSW(Hierarchical Navigable Small World)는 계층적 그래프 구조로 빠른 근사 최근접 이웃 검색을 가능하게 하는 알고리즘입니다. **작동 원리**: (1) 벡터들을 여러 층(layer)의 그래프로 구성합니다. 상위 층은 벡터가 적고 연결이 넓어 "고속도로" 역할을 하고, 하위 층은 벡터가 많고 세밀합니다. (2) 검색 시 최상위 층에서 시작하여 쿼리와 가장 가까운 노드로 이동합니다. (3) 더 가까운 노드가 없으면 한 층 내려가 반복합니다. (4) 최하위 층에서 최종 이웃을 찾습니다. **장점**: O(log n) 복잡도로 매우 빠르고, 정확도와 속도의 균형이 좋습니다. **파라미터 튜닝**: M(노드당 연결 수)이 크면 정확도는 높지만 메모리를 많이 사용하고, efConstruction/efSearch가 크면 더 정확하지만 느려집니다. 대부분의 최신 Vector DB가 HNSW를 기본으로 사용합니다.

5. ANN(Approximate Nearest Neighbor)과 KNN의 차이는?

> **답변**: KNN(K-Nearest Neighbors)은 모든 데이터를 비교하여 정확한 K개의 최근접 이웃을 찾는 방법으로, O(n) 복잡도로 대규모 데이터에서는 매우 느립니다. ANN(Approximate Nearest Neighbor)은 100% 정확하지 않지만 훨씬 빠르게 "거의 정확한" 결과를 반환합니다. **실무적 의미**: 1억 개 벡터에서 KNN은 수 초~분이 걸리지만, ANN(HNSW)은 밀리초 내에 95-99% 정확도로 결과를 반환합니다. 대부분의 실제 애플리케이션에서 이 정도 정확도면 충분합니다. "비슷한 상품 추천"에서 정확히 1등이 아닌 2등 상품이 나와도 사용자 경험에 큰 영향이 없습니다. Vector DB의 recall 파라미터로 정확도와 속도의 균형을 조절할 수 있습니다. 프로덕션에서는 거의 항상 ANN을 사용합니다.

6. 청크 크기가 검색 품질에 미치는 영향은?

> **답변**: 청크 크기는 Vector DB 검색 품질에 큰 영향을 미칩니다. **작은 청크 (100-300 토큰)**: 장점으로 정밀한 매칭이 가능하고 관련 없는 정보가 적게 포함됩니다. 단점으로 문맥이 부족하여 의미 파악이 어렵고, 중요 정보가 여러 청크에 분산될 수 있습니다. FAQ, 정의 검색에 적합합니다. **큰 청크 (800-1500 토큰)**: 장점으로 풍부한 문맥을 제공하고 완전한 개념을 담습니다. 단점으로 불필요한 정보가 포함되고 검색 비용이 증가합니다. 기술 문서, 긴 설명에 적합합니다. **실무 권장**: 500-1000 토큰으로 시작하여 테스트 후 조정합니다. 오버랩(50-200 토큰)을 두어 문맥 연결을 유지합니다. 문서 유형에 따라 다른 청크 크기를 적용하는 것도 방법입니다.

### 심화

7. 벡터 양자화(Quantization)란 무엇이고 왜 사용하나요?

> **답변**: 벡터 양자화는 벡터의 정밀도를 줄여 메모리 사용량과 검색 속도를 개선하는 기법입니다. **종류**: (1) **Scalar Quantization (SQ)**: float32를 int8로 변환, 4배 메모리 절감. (2) **Product Quantization (PQ)**: 벡터를 하위 공간으로 분할 후 코드북으로 압축, 10-100배 절감. (3) **Binary Quantization**: 0/1로만 표현, 최대 압축. **장단점**: 장점으로 메모리 사용량 크게 감소(수십 배), 검색 속도 향상, 비용 절감이 있습니다. 단점으로 검색 정확도가 약간 감소(1-5%)합니다. **실무 적용**: 수천만~수억 벡터를 다룰 때 필수입니다. Pinecone, Milvus는 자동 양자화를 지원합니다. 모바일 앱의 온디바이스 검색에서도 메모리 제한 때문에 양자화가 중요합니다. 대부분의 경우 SQ로 4배 압축해도 사용자가 체감하는 품질 차이는 미미합니다.

8. Multi-tenancy를 Vector DB에서 어떻게 구현하나요?

> **답변**: Multi-tenancy는 여러 고객(tenant)이 하나의 Vector DB 인프라를 공유하면서 데이터를 격리하는 방식입니다. **구현 방법**: (1) **Namespace/Collection 분리**: 각 테넌트에게 별도 namespace 할당. Pinecone, Milvus가 지원. 장점으로 완전한 격리와 간편한 삭제가 가능합니다. 단점으로 테넌트 수가 많으면 관리 복잡도가 증가합니다. (2) **메타데이터 필터링**: 모든 벡터에 tenant_id 메타데이터 추가, 검색 시 `filter={"tenant_id": "customer_123"}`. 장점으로 구현이 간단합니다. 단점으로 대규모 테넌트에서 성능 저하가 가능합니다. (3) **하이브리드**: 대형 테넌트는 별도 namespace, 소형 테넌트는 공유 namespace + 메타데이터 필터. **보안 고려**: API 레벨에서 tenant_id 검증 필수, 메타데이터 필터 우회 방지, 테넌트 간 데이터 유출 방지 감사 로깅이 필요합니다.

9. Vector DB의 확장성 한계와 해결 방법은?

> **답변**: Vector DB의 확장성 한계와 해결 방법입니다. **한계**: (1) **메모리**: HNSW 인덱스는 모든 벡터를 메모리에 유지해야 빠른 검색이 가능. 10억 개 * 1536차원 * 4바이트 = ~6TB 메모리. (2) **인덱싱 시간**: 대량 벡터 추가 시 인덱스 재구축에 시간 소요. (3) **단일 노드 한계**: 메모리와 CPU 병목. **해결 방법**: (1) **샤딩**: 데이터를 여러 노드에 분산. Milvus, Weaviate가 자동 샤딩 지원. (2) **양자화**: 메모리 사용량 4-10배 감소. (3) **티어드 스토리지**: 핫 데이터는 메모리, 콜드 데이터는 디스크/S3. (4) **관리형 서비스**: Pinecone, Zilliz Cloud는 자동 확장 지원. **실무 권장**: 1000만 벡터 이하는 단일 인스턴스로 충분. 그 이상은 샤딩 + 양자화 조합. 수십억 이상은 Milvus 클러스터나 관리형 서비스 검토. 비용 최적화를 위해 자주 검색되는 데이터와 아카이브 데이터를 분리합니다.

### 실습 과제

```
과제 1: 로컬 Vector DB 구축
- Chroma로 문서 검색 시스템 구현
- 100개 문서 인덱싱 및 검색 테스트

과제 2: 임베딩 모델 비교
- OpenAI vs Cohere vs 오픈소스 임베딩 모델
- 한국어 검색 품질 비교

과제 3: 하이브리드 검색
- BM25 + Vector 검색 조합 구현
- 검색 품질 측정 (MRR, NDCG)
```

### 실습 과제 예시 코드

```python
# 과제 1: Chroma로 로컬 문서 검색
import chromadb
from chromadb.utils import embedding_functions

# Chroma 클라이언트 생성
chroma_client = chromadb.PersistentClient(path="./chroma_db")

# OpenAI 임베딩 함수 설정
openai_ef = embedding_functions.OpenAIEmbeddingFunction(
    api_key="your-api-key",
    model_name="text-embedding-3-small"
)

# 컬렉션 생성
collection = chroma_client.get_or_create_collection(
    name="documents",
    embedding_function=openai_ef,
    metadata={"hnsw:space": "cosine"}  # 코사인 유사도 사용
)

# 문서 추가
documents = [
    {"id": "1", "text": "파이썬은 배우기 쉬운 프로그래밍 언어입니다.", "category": "programming"},
    {"id": "2", "text": "자바스크립트는 웹 개발에 필수적입니다.", "category": "programming"},
    {"id": "3", "text": "맛있는 파스타 레시피를 소개합니다.", "category": "cooking"},
    # ... 더 많은 문서
]

collection.add(
    ids=[d["id"] for d in documents],
    documents=[d["text"] for d in documents],
    metadatas=[{"category": d["category"]} for d in documents]
)

# 검색
results = collection.query(
    query_texts=["프로그래밍 언어 배우기"],
    n_results=3,
    where={"category": "programming"}  # 메타데이터 필터
)

print(results)
```

```python
# 과제 2: 임베딩 모델 비교
from sentence_transformers import SentenceTransformer
import numpy as np
from openai import OpenAI

# 테스트 쿼리와 문서
queries = ["맛있는 한국 음식", "프로그래밍 배우기", "서울 여행 추천"]
documents = [
    "김치찌개는 한국의 대표적인 음식입니다.",
    "비빔밥은 건강에 좋은 한국 요리입니다.",
    "파이썬 기초 강좌를 시작해보세요.",
    "서울 명동은 쇼핑하기 좋은 곳입니다.",
]

def compare_embeddings():
    results = {}

    # 1. OpenAI
    client = OpenAI()
    openai_embeddings = []
    for text in documents:
        response = client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        openai_embeddings.append(response.data[0].embedding)
    results["OpenAI"] = openai_embeddings

    # 2. BGE-M3 (오픈소스, 한국어 우수)
    bge_model = SentenceTransformer("BAAI/bge-m3")
    bge_embeddings = bge_model.encode(documents)
    results["BGE-M3"] = bge_embeddings

    # 3. 다국어 MiniLM
    minilm_model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")
    minilm_embeddings = minilm_model.encode(documents)
    results["MiniLM"] = minilm_embeddings

    # 유사도 비교
    for model_name, embeddings in results.items():
        print(f"\n{model_name} 모델:")
        query_emb = client.embeddings.create(
            model="text-embedding-3-small",
            input=queries[0]
        ).data[0].embedding if model_name == "OpenAI" else bge_model.encode([queries[0]])[0]

        for i, doc in enumerate(documents):
            similarity = np.dot(query_emb, embeddings[i]) / (
                np.linalg.norm(query_emb) * np.linalg.norm(embeddings[i])
            )
            print(f"  {doc[:30]}... : {similarity:.3f}")

compare_embeddings()
```

---

## 8. 실무 운영 시 주의사항

### Vector DB 운영 체크리스트

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Vector DB 운영 체크리스트                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 인덱스 관리                                                         │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 정기적 인덱스 최적화 (리빌드)                              │   │
│     │  • 삭제된 벡터 정리 (vacuum)                                  │   │
│     │  • 인덱스 파라미터 튜닝 (M, efConstruction)                   │   │
│     │  • 백업 및 복구 전략 수립                                     │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 성능 모니터링                                                       │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 검색 지연시간 (p50, p95, p99)                              │   │
│     │  • 메모리 사용량                                              │   │
│     │  • QPS (Queries Per Second)                                   │   │
│     │  • 인덱스 크기 추이                                           │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. 비용 최적화                                                         │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 임베딩 배치 처리로 API 호출 최소화                        │   │
│     │  • 양자화로 메모리 비용 절감                                  │   │
│     │  • 사용량 낮은 시간대 인덱싱 스케줄링                         │   │
│     │  • 관리형 vs 셀프호스팅 비용 비교                             │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  4. 데이터 품질                                                         │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 중복 벡터 탐지 및 제거                                     │   │
│     │  • 임베딩 드리프트 모니터링 (모델 변경 시)                    │   │
│     │  • 메타데이터 일관성 검증                                     │   │
│     │  • 정기적 검색 품질 테스트                                    │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 임베딩 비용 최적화

```python
# 임베딩 비용 계산기
class EmbeddingCostCalculator:
    # 가격 (1M 토큰당, 2024-2025 기준)
    PRICING = {
        "text-embedding-3-small": 0.02,  # $0.02/1M tokens
        "text-embedding-3-large": 0.13,  # $0.13/1M tokens
        "text-embedding-ada-002": 0.10,  # Legacy
    }

    def estimate_cost(self, texts: list[str], model: str = "text-embedding-3-small") -> dict:
        """임베딩 비용 추정"""
        # 대략적인 토큰 수 계산 (한국어: 글자당 ~1.5토큰)
        total_chars = sum(len(t) for t in texts)
        estimated_tokens = total_chars * 1.5  # 한국어 기준

        price_per_token = self.PRICING.get(model, 0.02) / 1_000_000
        estimated_cost = estimated_tokens * price_per_token

        return {
            "texts_count": len(texts),
            "estimated_tokens": int(estimated_tokens),
            "model": model,
            "estimated_cost_usd": round(estimated_cost, 4),
            "cost_per_1000_docs": round(estimated_cost / len(texts) * 1000, 4)
        }

# 사용 예시
calculator = EmbeddingCostCalculator()
cost = calculator.estimate_cost(documents)
print(f"예상 비용: ${cost['estimated_cost_usd']}")
print(f"1000개 문서당: ${cost['cost_per_1000_docs']}")
```
