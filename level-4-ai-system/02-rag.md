# RAG (Retrieval-Augmented Generation)

## 1. 한 줄 요약

외부 지식을 검색하여 LLM에게 제공함으로써, 최신 정보와 정확한 답변을 생성하는 기술입니다.

---

## 2. 쉽게 설명

### 모바일 개발자 관점에서의 비유

RAG는 **오픈북 시험**과 같습니다.

```
일반 LLM (Closed Book)              RAG (Open Book)
┌─────────────────────┐            ┌────────────────────────────┐
│                     │            │                            │
│  "2024년 매출은?"   │            │  "2024년 매출은?"          │
│        ↓            │            │        ↓                   │
│   [머릿속 기억만    │            │   [참고자료 검색]          │
│    으로 답변]       │            │        ↓                   │
│        ↓            │            │   [관련 문서 찾음]         │
│  "약 70조쯤...?"    │            │        ↓                   │
│  (불확실, 오류 가능)│            │   [문서 기반 답변]         │
│                     │            │        ↓                   │
│                     │            │  "79조원입니다 (출처:...)" │
└─────────────────────┘            └────────────────────────────┘
```

### 왜 RAG가 필요한가?

LLM의 한계:
1. **학습 데이터 마감일**: 특정 시점까지의 데이터만 학습
2. **환각(Hallucination)**: 모르는 것도 자신있게 답변
3. **도메인 지식 부족**: 회사 내부 문서, 최신 정보 모름

RAG의 해결:
```
┌─────────────────────────────────────────────────────────────┐
│  사용자: "우리 회사 휴가 정책이 어떻게 되나요?"             │
├─────────────────────────────────────────────────────────────┤
│  일반 LLM: "일반적으로 연차 15일이 주어집니다..."          │
│            (회사 정책 모름, 틀릴 수 있음)                   │
├─────────────────────────────────────────────────────────────┤
│  RAG:      [사내 문서 검색] → 인사규정.pdf 발견             │
│            "당사는 연차 20일 + 리프레시 휴가 5일이며,       │
│             입사 3년차부터 추가 휴가가 제공됩니다.          │
│             (출처: 인사규정 v2.3, 2024.01.01)"              │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 구조 다이어그램

### RAG 전체 파이프라인

```
┌─────────────────────────────────────────────────────────────────┐
│                    [오프라인] 데이터 준비 단계                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│   │ 원본    │    │ 전처리  │    │ 청킹    │    │ 임베딩  │    │
│   │ 문서    │───▶│         │───▶│(Chunk)  │───▶│         │    │
│   │ PDF,MD  │    │ 텍스트  │    │ 분할    │    │ 벡터화  │    │
│   └─────────┘    └─────────┘    └─────────┘    └────┬────┘    │
│                                                      │         │
│                                                      ▼         │
│                                                ┌─────────┐     │
│                                                │Vector DB│     │
│                                                │ 저장    │     │
│                                                └─────────┘     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    [온라인] 질의 응답 단계                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│   │ 사용자  │    │ 쿼리    │    │ 벡터    │    │ 관련    │    │
│   │ 질문    │───▶│ 임베딩  │───▶│ 검색    │───▶│ 문서    │    │
│   └─────────┘    └─────────┘    └─────────┘    └────┬────┘    │
│                                                      │         │
│                                                      ▼         │
│   ┌─────────┐    ┌─────────────────────────────────────┐       │
│   │  최종   │◀───│         프롬프트 생성               │       │
│   │  응답   │    │  "다음 문서를 참고하여 답변하세요:  │       │
│   └─────────┘    │   [검색된 문서 내용]                │       │
│        ▲         │   질문: {사용자 질문}"              │       │
│        │         └─────────────────┬───────────────────┘       │
│        │                           │                           │
│        │                           ▼                           │
│        │                     ┌─────────┐                       │
│        └─────────────────────│   LLM   │                       │
│                              └─────────┘                       │
└─────────────────────────────────────────────────────────────────┘
```

### 청킹(Chunking) 전략

```
원본 문서 (10,000 토큰)
┌────────────────────────────────────────────────────────────┐
│ 제1장 서론                                                 │
│ 본 문서는 AI 시스템 아키텍처에 대해 설명합니다...         │
│                                                            │
│ 제2장 LLM 기초                                            │
│ Large Language Model은...                                  │
│ ...                                                        │
│ 제3장 RAG 구현                                            │
│ RAG는 검색과 생성을 결합한...                             │
│ ...                                                        │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
           청킹 (Chunk Size: 500 토큰, Overlap: 50)
                              │
    ┌────────────┬────────────┼────────────┬────────────┐
    ▼            ▼            ▼            ▼            ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Chunk 1 │ │Chunk 2 │ │Chunk 3 │ │Chunk 4 │ │Chunk 5 │
│제1장...│ │...기초 │ │LLM은...│ │...RAG  │ │...결합 │
│(500)   │ │(500)   │ │(500)   │ │(500)   │ │(500)   │
└────────┘ └────────┘ └────────┘ └────────┘ └────────┘
     ↑         ↑
     └──50토큰 겹침──┘ (문맥 유지)
```

### 2024-2025년 RAG 아키텍처 발전

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      RAG 아키텍처 발전 단계                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Naive RAG (기본)                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  질문 → 임베딩 → 검색 → 컨텍스트 + 질문 → LLM → 응답           │   │
│  │  • 단순하지만 검색 품질에 한계                                   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  Advanced RAG (개선)                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  질문 → 쿼리 재작성 → 하이브리드 검색 → Re-ranking → LLM        │   │
│  │  • 쿼리 확장, HyDE, Multi-Query                                  │   │
│  │  • BM25 + 벡터 검색 조합                                         │   │
│  │  • Cross-encoder로 재순위화                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│  Modular RAG (최신)                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • 필요에 따라 모듈 조합                                         │   │
│  │  • Self-RAG: 검색 필요 여부 자체 판단                            │   │
│  │  • Corrective RAG: 검색 결과 검증 후 재검색                      │   │
│  │  • Adaptive RAG: 질문 유형별 다른 전략                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. 실무 적용 예시

### 1. 고객 지원 챗봇

```python
# Python으로 RAG 구현 (LangChain 사용)
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_pinecone import PineconeVectorStore
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate

# 상수 정의
DEFAULT_TOP_K = 3
DEFAULT_MODEL = "gpt-4o"
DEFAULT_TEMPERATURE = 0

class CustomerSupportRAG:
    # 시스템 프롬프트 템플릿
    PROMPT_TEMPLATE = """당신은 고객 지원 전문가입니다.
    아래 참고 문서를 바탕으로 질문에 답변해주세요.
    문서에 없는 내용은 "해당 정보를 찾을 수 없습니다"라고 답변하세요.

    참고 문서:
    {context}

    질문: {question}

    답변:"""

    def __init__(self, index_name: str = "support-docs"):
        self.embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
        self.vectorstore = PineconeVectorStore.from_existing_index(
            index_name=index_name,
            embedding=self.embeddings
        )
        self.llm = ChatOpenAI(model=DEFAULT_MODEL, temperature=DEFAULT_TEMPERATURE)

        self.prompt = PromptTemplate(
            template=self.PROMPT_TEMPLATE,
            input_variables=["context", "question"]
        )

        self.qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            chain_type="stuff",
            retriever=self.vectorstore.as_retriever(
                search_kwargs={"k": DEFAULT_TOP_K}
            ),
            return_source_documents=True,
            chain_type_kwargs={"prompt": self.prompt}
        )

    def answer(self, question: str) -> dict:
        result = self.qa_chain.invoke({"query": question})
        return {
            "answer": result["result"],
            "sources": [
                {
                    "content": doc.page_content[:200],
                    "source": doc.metadata.get("source", "unknown")
                }
                for doc in result["source_documents"]
            ]
        }

# 사용 예시
rag = CustomerSupportRAG()
response = rag.answer("환불 정책이 어떻게 되나요?")
# {
#   "answer": "구매 후 7일 이내 미개봉 상품은 전액 환불 가능합니다...",
#   "sources": [{"content": "...", "source": "환불정책.pdf"}]
# }
```

### 2. 모바일 앱 연동

```swift
// iOS - RAG API 호출
struct RAGRequest: Encodable {
    let question: String
    let conversationId: String?
    let filters: [String: String]?  // 메타데이터 필터링
}

struct RAGResponse: Decodable {
    let answer: String
    let sources: [Source]
    let confidence: Double

    struct Source: Decodable {
        let title: String
        let url: String
        let relevanceScore: Double
        let snippet: String
    }
}

class RAGService {
    private let baseURL: URL
    private let session: URLSession

    // 타임아웃 상수
    private static let requestTimeout: TimeInterval = 30

    init(baseURL: URL) {
        self.baseURL = baseURL
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = Self.requestTimeout
        self.session = URLSession(configuration: config)
    }

    func ask(_ question: String, filters: [String: String]? = nil) async throws -> RAGResponse {
        let request = RAGRequest(
            question: question,
            conversationId: currentConversationId,
            filters: filters
        )

        var urlRequest = URLRequest(url: baseURL.appendingPathComponent("/api/rag/query"))
        urlRequest.httpMethod = "POST"
        urlRequest.addValue("application/json", forHTTPHeaderField: "Content-Type")
        urlRequest.httpBody = try JSONEncoder().encode(request)

        let (data, _) = try await session.data(for: urlRequest)
        return try JSONDecoder().decode(RAGResponse.self, from: data)
    }

    // 스트리밍 응답 지원
    func askStreaming(_ question: String) -> AsyncThrowingStream<String, Error> {
        AsyncThrowingStream { continuation in
            Task {
                // SSE (Server-Sent Events) 처리
                // 실시간으로 응답을 받아 UI 업데이트
            }
        }
    }
}

// SwiftUI View
struct ChatView: View {
    @StateObject private var viewModel = ChatViewModel()
    @State private var inputText = ""

    var body: some View {
        VStack {
            ScrollView {
                LazyVStack(alignment: .leading) {
                    ForEach(viewModel.messages) { message in
                        MessageBubble(message: message)

                        // 출처 표시
                        if let sources = message.sources, !sources.isEmpty {
                            SourcesView(sources: sources)
                        }
                    }
                }
            }

            HStack {
                TextField("질문을 입력하세요", text: $inputText)
                Button("전송") {
                    viewModel.sendQuestion(inputText)
                    inputText = ""
                }
            }
            .padding()
        }
    }
}
```

### 3. 문서 인덱싱 파이프라인

```python
# Node.js/Python - 문서 인덱싱
from langchain_community.document_loaders import (
    PyPDFLoader, UnstructuredMarkdownLoader, WebBaseLoader
)
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore
import hashlib
from datetime import datetime

# 청킹 설정 상수
CHUNK_SIZE = 1000
CHUNK_OVERLAP = 200
SEPARATORS = ["\n\n", "\n", ".", " "]

class DocumentIndexer:
    def __init__(self, index_name: str):
        self.embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
        self.index_name = index_name
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=CHUNK_SIZE,
            chunk_overlap=CHUNK_OVERLAP,
            separators=SEPARATORS,
        )

    def load_document(self, file_path: str):
        """파일 유형에 따라 적절한 로더 선택"""
        if file_path.endswith('.pdf'):
            loader = PyPDFLoader(file_path)
        elif file_path.endswith('.md'):
            loader = UnstructuredMarkdownLoader(file_path)
        elif file_path.startswith('http'):
            loader = WebBaseLoader(file_path)
        else:
            raise ValueError(f"Unsupported file type: {file_path}")

        return loader.load()

    def index_documents(self, file_paths: list[str]):
        """문서들을 인덱싱"""
        all_chunks = []

        for file_path in file_paths:
            documents = self.load_document(file_path)

            # 청킹
            chunks = self.text_splitter.split_documents(documents)

            # 메타데이터 추가
            for chunk in chunks:
                chunk.metadata.update({
                    "source": file_path,
                    "indexed_at": datetime.now().isoformat(),
                    "chunk_hash": hashlib.md5(chunk.page_content.encode()).hexdigest()
                })

            all_chunks.extend(chunks)

        print(f"{len(file_paths)} 문서 → {len(all_chunks)} 청크 생성")

        # Vector DB에 저장
        PineconeVectorStore.from_documents(
            documents=all_chunks,
            embedding=self.embeddings,
            index_name=self.index_name
        )

        print("인덱싱 완료")
        return len(all_chunks)

    def update_document(self, file_path: str):
        """기존 문서 업데이트 (삭제 후 재인덱싱)"""
        # 기존 청크 삭제
        self._delete_by_source(file_path)
        # 새로 인덱싱
        self.index_documents([file_path])

    def _delete_by_source(self, source: str):
        """특정 소스의 문서 삭제"""
        # Pinecone 메타데이터 필터로 삭제
        pass

# 사용 예시
indexer = DocumentIndexer("support-docs")
indexer.index_documents([
    "docs/faq.pdf",
    "docs/policy.md",
    "https://company.com/help"
])
```

### 4. 고급 RAG 기법 구현

```python
# Multi-Query RAG: 여러 관점에서 검색
from langchain.retrievers.multi_query import MultiQueryRetriever

class AdvancedRAG:
    def __init__(self, vectorstore, llm):
        self.vectorstore = vectorstore
        self.llm = llm

        # Multi-Query Retriever: 질문을 여러 버전으로 재작성
        self.multi_query_retriever = MultiQueryRetriever.from_llm(
            retriever=vectorstore.as_retriever(search_kwargs={"k": 5}),
            llm=llm
        )

    def search_with_reranking(self, query: str, top_k: int = 3) -> list:
        """검색 후 Re-ranking"""
        # 1. 초기 검색 (더 많이 가져옴)
        initial_results = self.vectorstore.similarity_search(query, k=top_k * 3)

        # 2. Cross-encoder로 재순위화
        reranked = self._rerank(query, initial_results)

        return reranked[:top_k]

    def _rerank(self, query: str, documents: list) -> list:
        """Cross-encoder 기반 재순위화"""
        from sentence_transformers import CrossEncoder

        reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

        pairs = [(query, doc.page_content) for doc in documents]
        scores = reranker.predict(pairs)

        # 점수 기준 정렬
        scored_docs = list(zip(documents, scores))
        scored_docs.sort(key=lambda x: x[1], reverse=True)

        return [doc for doc, _ in scored_docs]

    def hybrid_search(self, query: str, alpha: float = 0.5) -> list:
        """키워드 + 시맨틱 하이브리드 검색"""
        # 벡터 검색 결과
        vector_results = self.vectorstore.similarity_search_with_score(query, k=10)

        # BM25 키워드 검색 (별도 인덱스 필요)
        keyword_results = self._bm25_search(query, k=10)

        # Reciprocal Rank Fusion으로 결합
        return self._rrf_fusion(vector_results, keyword_results, alpha)

    def _rrf_fusion(self, vec_results, kw_results, alpha):
        """RRF 알고리즘으로 결과 융합"""
        k = 60  # RRF 상수
        scores = {}

        for rank, (doc, _) in enumerate(vec_results):
            doc_id = doc.metadata.get("chunk_hash")
            scores[doc_id] = scores.get(doc_id, 0) + alpha / (k + rank)

        for rank, (doc, _) in enumerate(kw_results):
            doc_id = doc.metadata.get("chunk_hash")
            scores[doc_id] = scores.get(doc_id, 0) + (1 - alpha) / (k + rank)

        # 점수 기준 정렬 후 반환
        sorted_ids = sorted(scores.keys(), key=lambda x: scores[x], reverse=True)
        return sorted_ids[:10]
```

### RAG 아키텍처 예시

```
┌─────────────────────────────────────────────────────────────────┐
│                        모바일 앱                                │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway                                │
│                    (Rate Limiting, Auth)                        │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      RAG Service                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │Query Router │  │  Retriever  │  │  Generator  │             │
│  │(의도 분류)  │─▶│ (문서 검색) │─▶│ (답변 생성) │             │
│  └─────────────┘  └──────┬──────┘  └──────┬──────┘             │
│         │                │                │                     │
│         ▼                │                │                     │
│  ┌─────────────┐         │                │                     │
│  │ Query       │         │                │                     │
│  │ Rewriter    │         │                │                     │
│  │ (쿼리 확장) │         │                │                     │
│  └─────────────┘         │                │                     │
└──────────────────────────┼────────────────┼─────────────────────┘
                           │                │
           ┌───────────────┘                └───────────────┐
           ▼                                                ▼
┌─────────────────────┐                        ┌─────────────────────┐
│     Vector DB       │                        │    LLM API          │
│  ┌───────────────┐  │                        │  ┌───────────────┐  │
│  │   Pinecone    │  │                        │  │  GPT-4o       │  │
│  │   or Weaviate │  │                        │  │  / Claude     │  │
│  └───────────────┘  │                        │  └───────────────┘  │
│  ┌───────────────┐  │                        └─────────────────────┘
│  │   BM25 Index  │  │
│  │  (하이브리드) │  │
│  └───────────────┘  │
└─────────────────────┘
           ▲
           │ (오프라인 인덱싱)
┌──────────┴──────────┐
│  Document Pipeline  │
│  ┌────┐ ┌────┐     │
│  │PDF │ │DB  │ ... │
│  └────┘ └────┘     │
└─────────────────────┘
```

---

## 5. 장단점

### 장점

| 항목 | 설명 |
|------|------|
| **최신 정보** | 실시간으로 업데이트된 데이터 활용 가능 |
| **정확성** | 출처 기반 답변으로 환각 감소 |
| **도메인 특화** | 회사 내부 문서, 전문 지식 활용 |
| **비용 효율** | Fine-tuning 없이 지식 추가 가능 |
| **투명성** | 답변의 출처를 명시할 수 있음 |
| **데이터 보안** | 민감한 데이터를 LLM에 학습시키지 않음 |

### 단점

| 항목 | 설명 |
|------|------|
| **검색 품질 의존** | 잘못된 문서 검색 시 잘못된 답변 |
| **지연시간 증가** | 검색 + 생성으로 응답 시간 증가 |
| **인프라 비용** | Vector DB 운영 비용 발생 |
| **청킹 어려움** | 최적의 청크 크기 찾기가 어려움 |
| **복잡한 파이프라인** | 여러 컴포넌트 관리 필요 |

### RAG vs Fine-tuning 비교

```
                    RAG                     Fine-tuning
데이터 업데이트    ✅ 즉시 반영             ❌ 재학습 필요
비용              중간 (검색 인프라)        높음 (GPU 학습)
구현 복잡도       중간                      높음
응답 속도         느림 (검색+생성)          빠름 (생성만)
정확도           높음 (출처 기반)          도메인 특화
적합한 경우      - FAQ, 문서 검색          - 특정 스타일/톤
                 - 자주 바뀌는 정보         - 전문 용어 이해
                 - 출처가 중요한 경우       - 일관된 응답 필요

실무 권장:
┌─────────────────────────────────────────────────────────────────┐
│  1. 먼저 RAG로 시작 (빠른 구현, 데이터 업데이트 용이)           │
│  2. RAG로 부족하면 Fine-tuning 추가 검토                        │
│  3. 최적의 경우: RAG + Fine-tuned 모델 조합                     │
│     - Fine-tuned 모델로 도메인 이해도 향상                      │
│     - RAG로 최신/구체적 정보 제공                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 내 생각

```
이 섹션은 학습 후 직접 작성해보세요.

- RAG가 해결하는 핵심 문제는?
- 우리 서비스에 적용한다면 어떤 데이터를 인덱싱할까?
- 검색 품질을 높이려면 어떻게 해야 할까?
- RAG와 Fine-tuning 중 어떤 것을 선택할까?
```

---

## 7. 추가 질문

### 초급

1. RAG에서 "검색"과 "생성"은 각각 어떤 역할을 하나요?

> **답변**: RAG에서 "검색(Retrieval)"은 사용자 질문과 관련된 문서나 정보를 외부 데이터 소스(Vector DB, 검색 엔진 등)에서 찾아오는 역할을 합니다. 이 단계에서는 질문을 벡터로 변환하고, 유사한 벡터를 가진 문서 청크를 찾습니다. "생성(Generation)"은 검색된 문서를 컨텍스트로 LLM에 전달하고, 이를 바탕으로 자연스러운 답변을 만들어내는 역할입니다. 쉽게 비유하면, 검색은 "참고서에서 관련 페이지 찾기", 생성은 "찾은 내용을 바탕으로 답안 작성하기"입니다. 모바일 앱 개발 관점에서, 검색 결과의 품질이 최종 답변 품질을 결정하므로, 인덱싱과 검색 로직에 특히 신경 써야 합니다.

2. 청킹(Chunking)이 왜 필요한가요?

> **답변**: 청킹은 긴 문서를 작은 조각으로 나누는 과정으로, 세 가지 이유로 필수적입니다. 첫째, **LLM 컨텍스트 제한**: LLM은 한 번에 처리할 수 있는 토큰 수에 제한이 있어, 긴 문서 전체를 넣을 수 없습니다. 둘째, **검색 정확도**: 문서 전체보다 관련 있는 작은 부분만 검색하는 것이 더 정확합니다. "회사 연혁" 전체 문서보다 "2024년 신제품 출시" 부분만 찾는 것이 효율적입니다. 셋째, **비용 절감**: 컨텍스트가 짧을수록 LLM 호출 비용이 줄어듭니다. 실무에서는 보통 500-1000 토큰 크기로 청킹하고, 문맥 유지를 위해 50-200 토큰의 오버랩을 둡니다. 문서 유형에 따라 문단 단위, 섹션 단위 등 다양한 전략을 사용할 수 있습니다.

3. 임베딩(Embedding)이란 무엇인가요?

> **답변**: 임베딩은 텍스트를 숫자 벡터(배열)로 변환하는 과정입니다. 예를 들어 "맛있는 피자"라는 문장은 [0.23, -0.45, 0.12, ...] 같은 1536차원(OpenAI text-embedding-3-small 기준)의 숫자 배열이 됩니다. 중요한 점은, 의미가 비슷한 문장은 비슷한 벡터가 된다는 것입니다. "맛있는 피자"와 "pizza is delicious"는 언어가 달라도 벡터 공간에서 가까운 위치에 있습니다. 이 특성 덕분에 "비 오는 날 입을 옷"을 검색하면 "레인코트", "방수 자켓" 같은 의미적으로 관련된 결과를 찾을 수 있습니다. 모바일 앱에서는 OpenAI, Cohere, 또는 오픈소스(sentence-transformers) 임베딩 모델을 API로 호출하여 사용합니다.

### 중급

4. 청크 크기가 너무 크거나 작으면 어떤 문제가 생기나요?

> **답변**: 청크 크기 선택은 RAG 성능에 큰 영향을 미칩니다. **청크가 너무 크면**: (1) 불필요한 정보가 포함되어 LLM이 핵심을 파악하기 어렵습니다. (2) 검색 시 "큰 덩어리" 중 일부만 관련 있어도 전체가 반환되어 정확도가 떨어집니다. (3) 컨텍스트에 적은 수의 청크만 들어가 다양한 정보를 제공하지 못합니다. **청크가 너무 작으면**: (1) 문맥이 끊겨 의미가 불완전해집니다. (2) 중요한 정보가 여러 청크에 분산되어 전체 그림을 놓칠 수 있습니다. (3) 인덱싱해야 할 벡터 수가 많아져 비용과 검색 시간이 증가합니다. 실무 권장은 문서 유형에 따라 다르지만, 일반적으로 **500-1000 토큰** 크기에 **100-200 토큰 오버랩**이 좋은 시작점입니다. FAQ는 작게, 기술 문서는 크게 설정하는 것이 효과적입니다.

5. Hybrid Search(키워드 + 시맨틱)가 왜 효과적인가요?

> **답변**: Hybrid Search가 효과적인 이유는 키워드 검색과 시맨틱 검색이 서로의 약점을 보완하기 때문입니다. **키워드 검색(BM25)의 강점**: 정확한 용어, 고유명사, 코드, 제품명 검색에 뛰어납니다. "SKU-12345"나 "iPhone 15 Pro Max" 같은 정확한 매칭이 필요할 때 유리합니다. **시맨틱 검색의 강점**: 의미적 유사성을 파악합니다. "배가 아파요" 질문에 "복통 증상" 문서를 찾아줍니다. **한계 극복**: 시맨틱 검색만으로는 "에러 코드 E001"을 정확히 찾기 어렵고, 키워드 검색만으로는 "앱이 자꾸 꺼져요"의 의도를 파악하기 어렵습니다. Hybrid Search는 두 결과를 RRF(Reciprocal Rank Fusion) 같은 알고리즘으로 융합하여, 다양한 유형의 질문에 더 강건한 검색 결과를 제공합니다. 실무에서는 alpha=0.5로 시작하여 테스트 후 조정합니다.

6. Re-ranking이란 무엇이고 왜 필요한가요?

> **답변**: Re-ranking은 초기 검색 결과를 더 정교한 모델로 재평가하여 순서를 조정하는 기법입니다. 필요한 이유는 초기 검색(bi-encoder)과 재순위화(cross-encoder)의 trade-off 때문입니다. **Bi-encoder (초기 검색)**: 질문과 문서를 각각 독립적으로 벡터화합니다. 빠르지만, 질문-문서 간의 세밀한 상호작용을 놓칠 수 있습니다. **Cross-encoder (Re-ranking)**: 질문과 문서를 함께 입력받아 관련성을 직접 평가합니다. 정확하지만 느립니다. 실무 전략은 "빠른 검색으로 많이 가져온 후, 정교한 모델로 추리기"입니다. 예를 들어, 초기에 20개 문서를 빠르게 검색한 후, Cross-encoder로 상위 5개만 선별합니다. Cohere Rerank, BGE-reranker 같은 모델을 사용하면 검색 품질이 10-30% 향상됩니다. 다만, Re-ranking은 추가 지연시간과 비용이 발생하므로 필요성을 평가해야 합니다.

### 심화

7. Multi-Query RAG와 단순 RAG의 차이는?

> **답변**: Multi-Query RAG는 사용자의 원본 질문을 여러 버전으로 재작성하여 각각 검색한 후 결과를 통합하는 기법입니다. 단순 RAG는 원본 질문 하나로만 검색합니다. **작동 방식**: 사용자가 "AI 앱 만들기"를 질문하면, LLM이 이를 "1. 모바일 앱에서 AI 기능 구현 방법", "2. LLM API 연동하는 방법", "3. 온디바이스 ML 개발 방법"으로 재작성합니다. 각 쿼리로 검색하면 더 다양한 관련 문서를 찾을 수 있습니다. **장점**: (1) 모호한 질문의 여러 해석을 커버합니다. (2) 동의어나 관련 개념을 자동으로 확장합니다. (3) 검색 recall이 크게 향상됩니다. **단점**: LLM 호출이 추가되어 비용과 지연시간이 증가합니다. 실무에서는 복잡하거나 모호한 질문에 선택적으로 적용하고, 명확한 질문은 단순 RAG로 처리하는 것이 효율적입니다.

8. Graph RAG는 어떤 경우에 유용한가요?

> **답변**: Graph RAG는 문서 간의 관계를 그래프 구조로 표현하고, 이 관계 정보를 활용하여 검색하는 기법입니다. **유용한 경우**: (1) **복잡한 관계 추론**: "A 회사가 인수한 B 회사의 CEO는 누구인가요?" 같이 여러 엔티티 간 관계를 따라가야 하는 질문. (2) **지식 그래프가 있는 도메인**: 의학(질병-증상-치료), 법률(법조문-판례-해석), 제품 카탈로그(제품-부품-호환성) 등. (3) **전역적 요약**: "전체 문서에서 주요 테마는 무엇인가요?" 같은 질문에서, 개별 청크가 아닌 문서 전체 구조를 파악해야 할 때. **구현**: Microsoft의 GraphRAG 프레임워크가 대표적이며, 문서에서 엔티티와 관계를 추출하여 그래프를 구축합니다. 커뮤니티 탐지 알고리즘으로 관련 정보 클러스터를 찾습니다. 일반적인 FAQ나 단순 문서 검색에는 오버킬이지만, 복잡한 도메인 지식 베이스에는 매우 효과적입니다.

9. RAG 파이프라인의 평가 지표(Metrics)는 무엇이 있나요?

> **답변**: RAG 평가는 검색(Retrieval)과 생성(Generation) 두 단계로 나누어 측정합니다. **검색 평가 지표**: (1) **Recall@K**: 상위 K개 결과 중 정답 문서가 포함된 비율. 관련 문서를 놓치지 않는지 측정. (2) **Precision@K**: 상위 K개 중 실제로 관련 있는 문서의 비율. 불필요한 문서가 포함되지 않는지 측정. (3) **MRR (Mean Reciprocal Rank)**: 정답 문서가 몇 번째에 나타나는지의 평균 역수. 순위 품질 측정. (4) **NDCG**: 순위를 고려한 관련성 점수. **생성 평가 지표**: (1) **Faithfulness**: 답변이 검색된 문서 내용과 일치하는지 (환각 여부). (2) **Answer Relevancy**: 답변이 질문에 적절히 대응하는지. (3) **Context Relevancy**: 검색된 문서가 질문과 관련 있는지. RAGAS, TruLens 같은 프레임워크가 이 지표들을 자동 평가합니다. 실무에서는 이 지표들을 대시보드로 모니터링하고, 주기적으로 human evaluation을 병행합니다.

### 실습 과제

```
과제 1: 간단한 RAG 구현
- LangChain으로 PDF 문서 기반 Q&A 시스템 구축
- Streamlit으로 UI 만들기

과제 2: 검색 품질 개선
- 다양한 청킹 전략 비교 실험
- BM25 + 벡터 검색 하이브리드 구현

과제 3: 프로덕션 RAG
- 문서 업데이트 파이프라인 구축
- 검색 성능 모니터링 대시보드
```

### 실습 과제 예시 코드

```python
# 과제 1: Streamlit RAG 챗봇
import streamlit as st
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains import RetrievalQA

# 상수 정의
CHUNK_SIZE = 500
CHUNK_OVERLAP = 50
PERSIST_DIR = "./chroma_db"

@st.cache_resource
def load_rag_chain(pdf_path: str):
    """RAG 체인 초기화 (캐싱)"""
    # 문서 로드 및 청킹
    loader = PyPDFLoader(pdf_path)
    documents = loader.load()

    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=CHUNK_SIZE,
        chunk_overlap=CHUNK_OVERLAP
    )
    chunks = text_splitter.split_documents(documents)

    # Vector DB 생성
    embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory=PERSIST_DIR
    )

    # RAG 체인 구성
    llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=vectorstore.as_retriever(search_kwargs={"k": 3}),
        return_source_documents=True
    )

    return qa_chain

def main():
    st.title("PDF 문서 Q&A 챗봇")

    # PDF 업로드
    uploaded_file = st.file_uploader("PDF 파일 업로드", type="pdf")

    if uploaded_file:
        # 임시 파일 저장
        with open("temp.pdf", "wb") as f:
            f.write(uploaded_file.getvalue())

        qa_chain = load_rag_chain("temp.pdf")

        # 질문 입력
        question = st.text_input("질문을 입력하세요:")

        if question:
            with st.spinner("답변 생성 중..."):
                result = qa_chain.invoke({"query": question})

                st.markdown("### 답변")
                st.write(result["result"])

                st.markdown("### 참고 문서")
                for i, doc in enumerate(result["source_documents"]):
                    with st.expander(f"출처 {i+1}"):
                        st.write(doc.page_content[:500])

if __name__ == "__main__":
    main()
```

---

## 8. 실무 운영 시 주의사항

### RAG 시스템 운영 체크리스트

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      RAG 운영 체크리스트                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 문서 품질 관리                                                      │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 중복 문서 제거 (동일 내용 다른 버전)                       │   │
│     │  • 오래된 문서 아카이브 또는 삭제                              │   │
│     │  • 문서별 메타데이터 관리 (날짜, 카테고리, 버전)              │   │
│     │  • 주기적 문서 리뷰 및 업데이트                                │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  2. 검색 품질 모니터링                                                  │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • "관련 문서 없음" 응답 비율 추적                            │   │
│     │  • 사용자 피드백 수집 (유용함/유용하지 않음)                   │   │
│     │  • 검색 로그 분석으로 미흡한 쿼리 패턴 발견                    │   │
│     │  • A/B 테스트로 검색 전략 비교                                 │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  3. 비용 최적화                                                         │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 임베딩 모델: text-embedding-3-small vs large 비교          │   │
│     │  • 캐싱: 자주 묻는 질문 응답 캐시                              │   │
│     │  • 배치 임베딩: 실시간 대신 주기적 인덱싱                      │   │
│     │  • Vector DB: 관리형 vs 셀프호스팅 비용 비교                   │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  4. 보안 고려사항                                                       │
│     ┌─────────────────────────────────────────────────────────────┐   │
│     │  • 문서 접근 권한: 사용자별 검색 가능한 문서 제한             │   │
│     │  • PII(개인정보) 마스킹: 인덱싱 전 민감정보 제거              │   │
│     │  • 프롬프트 인젝션 방어: 검색 결과에 악의적 내용 포함 방지    │   │
│     │  • 로깅: 검색 쿼리와 결과를 안전하게 기록                      │   │
│     └─────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 성능 최적화 팁

```python
# 캐싱으로 비용 절감
import hashlib
from functools import lru_cache
import redis

class RAGCache:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.ttl = 3600  # 1시간 캐시

    def _cache_key(self, question: str) -> str:
        """질문의 해시값을 캐시 키로 사용"""
        return f"rag:{hashlib.md5(question.encode()).hexdigest()}"

    async def get_or_compute(self, question: str, rag_chain) -> dict:
        """캐시에서 찾거나 새로 계산"""
        cache_key = self._cache_key(question)

        # 캐시 확인
        cached = self.redis.get(cache_key)
        if cached:
            return json.loads(cached)

        # 새로 계산
        result = await rag_chain.ainvoke({"query": question})

        # 캐시 저장
        self.redis.setex(
            cache_key,
            self.ttl,
            json.dumps(result, ensure_ascii=False)
        )

        return result

# 시맨틱 캐싱 (유사한 질문도 캐시 히트)
class SemanticCache:
    def __init__(self, embeddings, threshold: float = 0.95):
        self.embeddings = embeddings
        self.threshold = threshold
        self.cache = {}  # {embedding: response}

    async def get_similar(self, question: str) -> dict | None:
        """유사한 질문의 캐시된 응답 반환"""
        q_embedding = self.embeddings.embed_query(question)

        for cached_embedding, response in self.cache.items():
            similarity = cosine_similarity(q_embedding, cached_embedding)
            if similarity > self.threshold:
                return response

        return None
```
