# HTTP/HTTPS

## 1. 한 줄 요약

**HTTP는 웹에서 데이터를 주고받는 약속(프로토콜)이고, HTTPS는 이 통신을 암호화하여 안전하게 만든 버전입니다.**

## 2. 쉽게 설명

### 모바일 개발자 관점에서

모바일 앱에서 `URLSession`(iOS)이나 `Retrofit`(Android)으로 API를 호출할 때, 여러분은 이미 HTTP를 사용하고 있습니다.

**HTTP를 편지에 비유하면:**
- HTTP는 엽서와 같습니다. 내용이 그대로 노출되어 배달 중 누구나 읽을 수 있습니다.
- HTTPS는 봉투에 넣어 봉인한 편지입니다. 받는 사람만 열어볼 수 있습니다.

**모바일 앱에서의 실제 경험:**
- iOS에서 Info.plist의 ATS(App Transport Security) 설정을 해본 적이 있다면, 그것이 바로 HTTPS 강제화입니다.
- Android에서 `android:usesCleartextTraffic="false"` 설정이 HTTP 차단입니다.
- Charles나 Proxyman 같은 프록시 도구로 API 디버깅할 때 보이는 것이 HTTP 요청/응답입니다.

### HTTP의 핵심 특징

1. **요청-응답 모델**: 클라이언트가 요청하면 서버가 응답합니다.
2. **무상태(Stateless)**: 각 요청은 독립적입니다. 서버는 이전 요청을 기억하지 않습니다.
3. **텍스트 기반**: 사람이 읽을 수 있는 형태로 통신합니다.

### HTTP 메서드

| 메서드 | 용도 | 예시 | 멱등성 | 안전성 |
|--------|------|------|--------|--------|
| GET | 데이터 조회 | 사용자 정보 가져오기 | O | O |
| POST | 데이터 생성 | 새 게시글 작성 | X | X |
| PUT | 데이터 전체 수정 | 프로필 전체 업데이트 | O | X |
| PATCH | 데이터 부분 수정 | 닉네임만 변경 | X | X |
| DELETE | 데이터 삭제 | 게시글 삭제 | O | X |
| HEAD | 헤더만 조회 | 파일 존재 여부 확인 | O | O |
| OPTIONS | 지원 메서드 확인 | CORS preflight | O | O |

> **멱등성(Idempotency)**: 같은 요청을 여러 번 보내도 결과가 같음
> **안전성(Safety)**: 서버의 상태를 변경하지 않음

### HTTP 상태 코드

| 범위 | 의미 | 자주 보는 코드 | 모바일 앱에서의 대응 |
|------|------|----------------|---------------------|
| 2xx | 성공 | 200 OK, 201 Created, 204 No Content | 정상 처리, UI 업데이트 |
| 3xx | 리다이렉션 | 301 Moved Permanently, 304 Not Modified | 캐시 활용, 새 URL로 재요청 |
| 4xx | 클라이언트 오류 | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 422 Unprocessable Entity, 429 Too Many Requests | 사용자에게 오류 안내, 재시도 로직 |
| 5xx | 서버 오류 | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable | 재시도 로직, 오프라인 모드 전환 |

## 3. 구조 다이어그램

### HTTP 요청/응답 흐름

```
┌─────────────────┐                              ┌─────────────────┐
│   Mobile App    │                              │     Server      │
│  (클라이언트)    │                              │                 │
└────────┬────────┘                              └────────┬────────┘
         │                                                │
         │  1. HTTP 요청 (Request)                        │
         │  ┌─────────────────────────────────┐          │
         │  │ GET /api/users/123 HTTP/1.1     │ ← 요청 라인 (메서드, 경로, 버전)
         │  │ Host: api.example.com           │ ← 필수 헤더
         │  │ Authorization: Bearer xxx       │ ← 인증 토큰
         │  │ Content-Type: application/json  │ ← 콘텐츠 타입
         │  │ Accept: application/json        │ ← 원하는 응답 형식
         │  │ Accept-Language: ko-KR          │ ← 언어 설정
         │  │ User-Agent: MyApp/1.0 iOS/17.0  │ ← 클라이언트 정보
         │  └─────────────────────────────────┘          │
         │ ─────────────────────────────────────────────>│
         │                                                │
         │  2. HTTP 응답 (Response)                       │
         │           ┌─────────────────────────────────┐ │
         │           │ HTTP/1.1 200 OK                 │ ← 상태 라인
         │           │ Content-Type: application/json  │ ← 응답 형식
         │           │ Content-Length: 89              │ ← 본문 크기
         │           │ Cache-Control: max-age=3600    │ ← 캐시 설정
         │           │ ETag: "abc123"                  │ ← 캐시 검증용
         │           │ X-Request-Id: uuid-here         │ ← 디버깅용 ID
         │           │                                 │
         │           │ {"id": 123, "name": "홍길동"}   │ ← 응답 본문
         │           └─────────────────────────────────┘ │
         │ <─────────────────────────────────────────────│
         │                                                │
```

### HTTPS TLS 핸드셰이크 (상세)

```
┌─────────────────┐                              ┌─────────────────┐
│     Client      │                              │     Server      │
│   (Mobile App)  │                              │   (API Server)  │
└────────┬────────┘                              └────────┬────────┘
         │                                                │
         │  1. Client Hello                               │
         │  ┌─────────────────────────────────────────┐  │
         │  │ • 지원하는 TLS 버전 (1.2, 1.3)          │  │
         │  │ • 지원하는 암호화 알고리즘 목록         │  │
         │  │ • 클라이언트 랜덤 값 (32 bytes)         │  │
         │  │ • 세션 ID (재연결 시 사용)              │  │
         │  └─────────────────────────────────────────┘  │
         │ ─────────────────────────────────────────────>│
         │                                                │
         │  2. Server Hello + Certificate                 │
         │  ┌─────────────────────────────────────────┐  │
         │  │ • 선택한 TLS 버전                       │  │
         │  │ • 선택한 암호화 알고리즘                │  │
         │  │ • 서버 랜덤 값 (32 bytes)               │  │
         │  │ • 서버 인증서 (공개키 포함)             │  │
         │  └─────────────────────────────────────────┘  │
         │ <─────────────────────────────────────────────│
         │                                                │
         │  3. 클라이언트 측 처리                         │
         │  ┌─────────────────────────────────────────┐  │
         │  │ • 인증서 유효성 검증 (CA 체인 확인)     │  │
         │  │ • 도메인 일치 확인                      │  │
         │  │ • 만료 여부 확인                        │  │
         │  │ • Pre-Master Secret 생성               │  │
         │  │ • 서버 공개키로 암호화                  │  │
         │  └─────────────────────────────────────────┘  │
         │                                                │
         │  4. Client Key Exchange + Change Cipher Spec   │
         │ ─────────────────────────────────────────────>│
         │                                                │
         │  5. 서버 측 처리                               │
         │           ┌─────────────────────────────────┐ │
         │           │ • 개인키로 Pre-Master Secret 복호화│ │
         │           │ • Master Secret 생성             │ │
         │           │ • 세션 키 도출                   │ │
         │           └─────────────────────────────────┘ │
         │                                                │
         │  6. Server Finished + Change Cipher Spec       │
         │ <─────────────────────────────────────────────│
         │                                                │
         │  ═══════════════════════════════════════════  │
         │       이후 모든 통신은 대칭키로 암호화         │
         │  ═══════════════════════════════════════════  │
         │                                                │
```

### HTTP/1.1 vs HTTP/2 vs HTTP/3 비교

```
HTTP/1.1                    HTTP/2                      HTTP/3
┌──────────────┐           ┌──────────────┐           ┌──────────────┐
│  Request 1   │           │   Stream 1   │           │   Stream 1   │
│  ──────────> │           │   ──────────>│           │   ──────────>│
│  Response 1  │           │   Stream 2   │           │   Stream 2   │
│  <────────── │           │   ──────────>│           │   ──────────>│
│  Request 2   │           │   Stream 3   │           │   Stream 3   │
│  ──────────> │           │   ──────────>│           │   ──────────>│
│  Response 2  │           │              │           │              │
│  <────────── │           │  (다중화)     │           │  (다중화)     │
└──────────────┘           └──────────────┘           └──────────────┘
       │                          │                          │
   순차 처리               TCP 기반 다중화            UDP(QUIC) 기반
 (Head-of-Line           (바이너리 프레임)           (스트림 독립성)
   Blocking)                    │                          │
       │                   HPACK 헤더 압축             0-RTT 연결 재개
       │                        │                          │
  6개 연결 제한            하나의 연결로 처리         패킷 손실 시
       │                        │                   다른 스트림 영향 X
```

## 4. 실무 적용 예시

### 예시 1: 로그인 API 호출 (iOS)

```swift
// iOS에서 로그인 API 호출
func login(email: String, password: String) async throws -> User {
    let url = URL(string: "https://api.example.com/auth/login")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("application/json", forHTTPHeaderField: "Accept")
    request.setValue("ko-KR", forHTTPHeaderField: "Accept-Language")

    // 요청 ID 추가 (디버깅용)
    let requestId = UUID().uuidString
    request.setValue(requestId, forHTTPHeaderField: "X-Request-Id")

    let body = ["email": email, "password": password]
    request.httpBody = try JSONEncoder().encode(body)

    let (data, response) = try await URLSession.shared.data(for: request)

    guard let httpResponse = response as? HTTPURLResponse else {
        throw NetworkError.invalidResponse
    }

    // 응답 헤더에서 유용한 정보 추출
    let serverRequestId = httpResponse.value(forHTTPHeaderField: "X-Request-Id")
    let rateLimitRemaining = httpResponse.value(forHTTPHeaderField: "X-RateLimit-Remaining")

    print("Request ID: \(requestId), Server Request ID: \(serverRequestId ?? "N/A")")
    print("Rate Limit Remaining: \(rateLimitRemaining ?? "N/A")")

    switch httpResponse.statusCode {
    case 200:
        return try JSONDecoder().decode(User.self, from: data)
    case 401:
        throw NetworkError.unauthorized
    case 429:
        // Rate limit 초과 시 Retry-After 헤더 확인
        let retryAfter = httpResponse.value(forHTTPHeaderField: "Retry-After")
        throw NetworkError.rateLimited(retryAfter: Int(retryAfter ?? "60") ?? 60)
    case 400...499:
        // 에러 응답 본문 파싱
        if let errorResponse = try? JSONDecoder().decode(ErrorResponse.self, from: data) {
            throw NetworkError.clientError(httpResponse.statusCode, errorResponse.message)
        }
        throw NetworkError.clientError(httpResponse.statusCode, nil)
    case 500...599:
        throw NetworkError.serverError(httpResponse.statusCode)
    default:
        throw NetworkError.unknown
    }
}

// 에러 응답 구조체
struct ErrorResponse: Decodable {
    let code: String
    let message: String
}

// 네트워크 에러 정의
enum NetworkError: Error {
    case invalidResponse
    case unauthorized
    case rateLimited(retryAfter: Int)
    case clientError(Int, String?)
    case serverError(Int)
    case unknown
}
```

### 예시 2: Retrofit을 이용한 API 호출 (Android)

```kotlin
// Android Retrofit 인터페이스
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Long): Response<User>

    @POST("auth/login")
    suspend fun login(@Body request: LoginRequest): Response<LoginResponse>

    @Headers(
        "Accept: application/json",
        "Accept-Language: ko-KR"
    )
    @GET("products")
    suspend fun getProducts(
        @Query("page") page: Int,
        @Query("limit") limit: Int
    ): Response<ProductListResponse>
}

// OkHttpClient 인터셉터로 공통 헤더 추가
val client = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val original = chain.request()
        val requestId = UUID.randomUUID().toString()

        val request = original.newBuilder()
            .header("X-Request-Id", requestId)
            .header("X-App-Version", BuildConfig.VERSION_NAME)
            .header("X-Platform", "Android")
            .build()

        val response = chain.proceed(request)

        // 응답 헤더 로깅
        val serverRequestId = response.header("X-Request-Id")
        Log.d("Network", "Request: $requestId, Server: $serverRequestId")

        response
    }
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = if (BuildConfig.DEBUG) {
            HttpLoggingInterceptor.Level.BODY
        } else {
            HttpLoggingInterceptor.Level.NONE
        }
    })
    .build()

// 사용 예시
class UserRepository(private val api: ApiService) {
    suspend fun getUser(userId: Long): Result<User> {
        return try {
            val response = api.getUser(userId)
            when {
                response.isSuccessful -> Result.success(response.body()!!)
                response.code() == 404 -> Result.failure(UserNotFoundException())
                response.code() == 429 -> {
                    val retryAfter = response.headers()["Retry-After"]?.toIntOrNull() ?: 60
                    Result.failure(RateLimitException(retryAfter))
                }
                else -> Result.failure(ApiException(response.code()))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 예시 3: iOS App Transport Security (ATS) 설정

```xml
<!-- Info.plist -->
<!-- 프로덕션 권장 설정: 기본값 사용 (HTTPS 강제) -->

<!-- 개발 환경에서 특정 도메인 HTTP 허용 -->
<key>NSAppTransportSecurity</key>
<dict>
    <!-- 전역 설정 -->
    <key>NSAllowsArbitraryLoads</key>
    <false/>  <!-- 기본값: HTTPS만 허용 -->

    <!-- 도메인별 예외 설정 -->
    <key>NSExceptionDomains</key>
    <dict>
        <!-- 로컬 개발 서버 -->
        <key>localhost</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSIncludesSubdomains</key>
            <true/>
        </dict>

        <!-- 레거시 API (마이그레이션 필요) -->
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.0</string>  <!-- 오래된 TLS 허용 -->
        </dict>

        <!-- 메인 API (TLS 1.2 이상 강제) -->
        <key>api.example.com</key>
        <dict>
            <key>NSExceptionRequiresForwardSecrecy</key>
            <true/>  <!-- Forward Secrecy 필수 -->
            <key>NSExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
        </dict>
    </dict>
</dict>
```

### 예시 4: Android Network Security Config

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- 기본 설정: HTTPS만 허용 -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>

    <!-- 프로덕션 API: HTTPS 강제 + Certificate Pinning -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2025-12-31">
            <!-- 주 인증서 핀 -->
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- 백업 인증서 핀 -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>

    <!-- 개발용: 로컬 HTTP 허용 -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="false">10.0.2.2</domain>  <!-- 에뮬레이터 localhost -->
        <domain includeSubdomains="false">localhost</domain>
    </domain-config>

    <!-- 디버그 빌드: 사용자 인증서 신뢰 (Charles, Proxyman용) -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="user"/>
            <certificates src="system"/>
        </trust-anchors>
    </debug-overrides>
</network-security-config>

<!-- AndroidManifest.xml에 적용 -->
<!-- <application android:networkSecurityConfig="@xml/network_security_config" ... > -->
```

### 예시 5: HTTP 캐싱 구현

```swift
// iOS에서 HTTP 캐싱 활용
class CachingNetworkManager {
    private lazy var session: URLSession = {
        let config = URLSessionConfiguration.default

        // 캐시 설정
        let cache = URLCache(
            memoryCapacity: 50 * 1024 * 1024,  // 50MB 메모리
            diskCapacity: 100 * 1024 * 1024,   // 100MB 디스크
            diskPath: "http_cache"
        )
        config.urlCache = cache
        config.requestCachePolicy = .useProtocolCachePolicy

        return URLSession(configuration: config)
    }()

    // ETag를 활용한 조건부 요청
    func fetchWithCache(url: URL, etag: String?) async throws -> (Data, String?) {
        var request = URLRequest(url: url)

        // 이전에 받은 ETag가 있으면 조건부 요청
        if let etag = etag {
            request.setValue(etag, forHTTPHeaderField: "If-None-Match")
        }

        let (data, response) = try await session.data(for: request)
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        switch httpResponse.statusCode {
        case 200:
            // 새 데이터
            let newEtag = httpResponse.value(forHTTPHeaderField: "ETag")
            return (data, newEtag)
        case 304:
            // 변경 없음 - 캐시 사용
            throw CacheError.notModified
        default:
            throw NetworkError.serverError(httpResponse.statusCode)
        }
    }
}

enum CacheError: Error {
    case notModified
}
```

## 5. 장단점

### HTTP

| 장점 | 단점 |
|------|------|
| 단순하고 이해하기 쉬움 | 데이터가 암호화되지 않아 보안에 취약 |
| 디버깅이 용이함 (텍스트 기반) | 중간자 공격(MITM)에 노출 |
| 오버헤드가 적음 | 데이터 변조 가능 |
| 프록시 도구로 쉽게 분석 | 앱스토어 제출 시 거부될 수 있음 |

### HTTPS

| 장점 | 단점 |
|------|------|
| 데이터 암호화로 도청 방지 | 초기 연결 시 TLS 핸드셰이크 오버헤드 |
| 데이터 무결성 보장 | 인증서 관리 필요 |
| 신원 확인 (인증서를 통해) | 약간의 성능 저하 (현대에는 미미함) |
| SEO에 유리, 브라우저 신뢰 표시 | 인증서 비용 (Let's Encrypt로 무료화) |
| iOS/Android에서 기본 요구사항 | 디버깅 시 추가 설정 필요 (프록시 인증서) |
| HTTP/2, HTTP/3 사용 가능 | |

## 6. 실무에서 자주 겪는 문제와 해결책

### 문제 1: "SSL 인증서 오류" (iOS/Android)

```swift
// 원인: 자체 서명 인증서, 만료된 인증서, 도메인 불일치
// 해결책 1: (개발용) ATS 예외 설정 또는 Network Security Config

// 해결책 2: (테스트용) URLSession delegate로 인증서 검증 우회
class InsecureSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // ⚠️ 프로덕션에서 절대 사용 금지!
        if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
           let serverTrust = challenge.protectionSpace.serverTrust {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.performDefaultHandling, nil)
        }
    }
}
```

### 문제 2: "타임아웃 발생"

```kotlin
// 원인: 네트워크 불안정, 서버 응답 지연
// 해결책: 적절한 타임아웃 설정 + 재시도 로직

val client = OkHttpClient.Builder()
    .connectTimeout(10, TimeUnit.SECONDS)  // 연결 타임아웃
    .readTimeout(30, TimeUnit.SECONDS)     // 읽기 타임아웃
    .writeTimeout(30, TimeUnit.SECONDS)    // 쓰기 타임아웃
    .retryOnConnectionFailure(true)        // 연결 실패 시 재시도
    .addInterceptor { chain ->
        var request = chain.request()
        var response: Response? = null
        var tryCount = 0
        val maxRetries = 3

        while (tryCount < maxRetries) {
            try {
                response = chain.proceed(request)
                if (response.isSuccessful) break
            } catch (e: IOException) {
                tryCount++
                if (tryCount >= maxRetries) throw e
                Thread.sleep(1000L * tryCount)  // 지수 백오프
            }
        }
        response ?: throw IOException("Max retries exceeded")
    }
    .build()
```

### 문제 3: "401 Unauthorized 응답 처리"

```swift
// 토큰 갱신 후 자동 재시도
class AuthenticatedNetworkManager {
    private let tokenManager: TokenManager

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        do {
            return try await performRequest(endpoint)
        } catch NetworkError.unauthorized {
            // 토큰 갱신 시도
            try await tokenManager.refreshToken()
            // 갱신된 토큰으로 재시도
            return try await performRequest(endpoint)
        }
    }
}
```

## 7. 내 생각

```
(이 공간은 학습 후 자신의 생각을 정리하는 곳입니다)

- HTTP/HTTPS를 학습하면서 새롭게 알게 된 점:


- 모바일 개발에서 이 지식이 어떻게 활용될 수 있을지:


- 더 궁금한 점이나 추가로 학습하고 싶은 내용:


```

## 8. 추가 질문

1. **HTTP/1.1, HTTP/2, HTTP/3의 차이점은 무엇인가요?** 각 버전별 개선사항과 모바일 앱에서의 영향은?

> **답변**: HTTP/1.1은 텍스트 기반 프로토콜로 하나의 TCP 연결에서 요청/응답이 순차적으로 처리되어 Head-of-Line Blocking 문제가 있습니다. 이를 우회하기 위해 브라우저는 도메인당 6개의 동시 연결을 사용했습니다. HTTP/2는 바이너리 프레이밍과 멀티플렉싱을 도입하여 하나의 TCP 연결에서 여러 스트림을 동시에 처리할 수 있고, 헤더 압축(HPACK)과 서버 푸시 기능이 추가되었습니다. HTTP/3는 UDP 기반의 QUIC 프로토콜 위에서 동작하여 TCP의 Head-of-Line Blocking 문제를 완전히 해결하고, 0-RTT 연결 재개로 레이턴시를 크게 줄였습니다.
>
> 모바일 앱에서는 iOS의 URLSession과 Android의 OkHttp가 자동으로 HTTP/2를 지원하며, 서버가 지원하면 자동으로 업그레이드됩니다. 특히 모바일 환경에서는 네트워크 전환(Wi-Fi → LTE)이 빈번한데, HTTP/3의 Connection Migration 기능은 IP 변경 시에도 연결을 유지할 수 있어 매우 유용합니다. 배터리와 데이터 사용량 측면에서도 연결 수가 줄고 헤더가 압축되어 효율적입니다.

2. **Certificate Pinning(인증서 고정)이란 무엇이고, 왜 모바일 앱에서 중요한가요?** 구현 방법과 주의사항은?

> **답변**: Certificate Pinning은 앱이 특정 인증서나 공개키만 신뢰하도록 하드코딩하는 보안 기법입니다. 일반적으로 앱은 OS의 신뢰 저장소에 있는 CA(인증 기관)가 서명한 모든 인증서를 신뢰하지만, 이는 악성 CA 인증서가 설치된 경우 중간자 공격(MITM)에 취약합니다. 금융 앱이나 민감한 데이터를 다루는 앱에서 특히 중요합니다.
>
> iOS에서는 Network Security Config의 `pin-set`이나 URLSession의 `URLAuthenticationChallenge`에서 인증서를 검증하고, Android에서는 `network_security_config.xml`의 `<pin-set>` 또는 OkHttp의 `CertificatePinner`를 사용합니다. 주의사항으로는: (1) 백업 핀을 반드시 포함해야 합니다 - 인증서 갱신 시 앱이 동작 불능이 될 수 있습니다. (2) 핀 만료일을 설정하고 모니터링해야 합니다. (3) 인증서가 아닌 공개키(SPKI)를 피닝하면 인증서 갱신 시에도 앱 업데이트 없이 동작합니다. (4) 개발/테스트 환경에서는 debug-override로 우회할 수 있게 설정해야 합니다.

3. **REST API와 GraphQL의 차이점은 무엇인가요?** 각각 어떤 상황에서 유리한가요?

> **답변**: REST API는 리소스 중심으로 설계되어 각 엔드포인트가 특정 리소스를 나타내고 HTTP 메서드(GET, POST, PUT, DELETE)로 작업을 구분합니다. 반면 GraphQL은 쿼리 언어로 클라이언트가 필요한 데이터의 형태를 직접 정의합니다.
>
> REST의 장점: 캐싱이 쉽고(HTTP 캐시 활용), 간단하고 예측 가능하며, 파일 업로드가 쉽고, 대부분의 도구와 호환됩니다. 단점: Over-fetching(필요 이상의 데이터 수신)과 Under-fetching(여러 번 요청 필요) 문제가 있습니다.
>
> GraphQL의 장점: 정확히 필요한 데이터만 요청 가능(모바일에서 데이터 절약), 한 번의 요청으로 연관 데이터 조회, 강력한 타입 시스템과 자동 문서화입니다. 단점: 캐싱이 복잡하고, 파일 업로드가 불편하며, 쿼리 복잡도 제한이 필요하고, 학습 곡선이 있습니다.
>
> 실무에서는 단순한 CRUD 앱은 REST, 복잡한 데이터 관계가 있거나 다양한 클라이언트(웹, iOS, Android)가 서로 다른 데이터를 필요로 할 때는 GraphQL이 유리합니다.

4. **Keep-Alive 연결이란 무엇이고, 모바일 앱의 네트워크 효율성에 어떤 영향을 주나요?**

> **답변**: HTTP/1.0에서는 매 요청마다 TCP 연결을 새로 맺었지만, HTTP/1.1부터 Keep-Alive가 기본값이 되어 하나의 TCP 연결을 여러 요청에 재사용합니다. 이는 TCP 3-way handshake(~50-100ms)와 TLS handshake(~100-200ms)의 오버헤드를 줄여줍니다.
>
> 모바일 환경에서 Keep-Alive는 특히 중요합니다: (1) 배터리 절약 - 연결 수립은 라디오 칩을 활성화시키므로 배터리 소모가 큽니다. (2) 레이턴시 감소 - 이미 연결이 맺어져 있으면 즉시 요청이 가능합니다. (3) 데이터 절약 - TLS 핸드셰이크 패킷이 줄어듭니다.
>
> iOS의 URLSession과 Android의 OkHttp는 기본적으로 연결 풀을 관리하여 Keep-Alive를 자동으로 활용합니다. OkHttp의 경우 기본적으로 5분간 idle 연결을 유지하고 최대 5개의 연결을 풀에 보관합니다. 서버 측에서도 `Keep-Alive: timeout=5, max=100` 같은 헤더로 연결 정책을 알려줍니다.

5. **모바일 앱에서 네트워크 요청을 캐싱하는 전략은 무엇이 있나요?** ETag, Last-Modified 헤더는 어떻게 활용하나요?

> **답변**: HTTP 캐싱 전략은 크게 두 가지입니다: (1) 강제 캐싱(Freshness) - `Cache-Control: max-age=3600` 헤더로 지정된 시간 동안 서버에 요청 없이 캐시를 사용합니다. (2) 조건부 요청(Validation) - 캐시가 만료되었을 때 `If-None-Match`(ETag)나 `If-Modified-Since`(Last-Modified) 헤더로 변경 여부를 확인하고, 변경이 없으면 304 Not Modified 응답으로 본문 전송을 생략합니다.
>
> 실무 활용 예시: 상품 목록 API는 `Cache-Control: max-age=300`으로 5분간 캐시하고, 상품 상세 정보는 ETag를 활용해 변경 시에만 새 데이터를 받습니다. 사용자 프로필처럼 자주 변경되는 데이터는 `Cache-Control: no-cache`로 항상 서버에 확인하되 ETag로 변경 없으면 304를 받습니다.
>
> iOS URLSession은 기본적으로 HTTP 캐시 헤더를 존중하며, Android OkHttp도 마찬가지입니다. 추가로 앱 레벨에서 Room(Android)이나 Core Data(iOS)로 오프라인 캐시를 구현하면 네트워크 없이도 앱 사용이 가능합니다.

6. **API Rate Limiting은 무엇이고, 클라이언트에서 어떻게 대응해야 하나요?** 429 Too Many Requests를 받으면 어떻게 처리해야 할까요?

> **답변**: Rate Limiting은 서버가 클라이언트의 요청 빈도를 제한하는 메커니즘으로, DDoS 방어, 공정한 리소스 분배, 비용 관리를 위해 사용됩니다. 보통 `X-RateLimit-Limit`(최대 요청 수), `X-RateLimit-Remaining`(남은 요청 수), `X-RateLimit-Reset`(리셋 시간) 헤더로 현재 상태를 알려줍니다.
>
> 429 응답을 받으면: (1) `Retry-After` 헤더를 확인하여 지정된 시간 후 재시도합니다. (2) 헤더가 없으면 지수 백오프(exponential backoff)로 1초, 2초, 4초... 간격으로 재시도합니다. (3) 사용자에게 "잠시 후 다시 시도해주세요" 메시지를 표시합니다.
>
> 예방 전략: (1) 요청을 배치로 묶어 횟수를 줄입니다. (2) 캐싱을 적극 활용합니다. (3) 디바운싱/스로틀링으로 중복 요청을 방지합니다. (4) `X-RateLimit-Remaining` 값을 모니터링하여 한계에 가까워지면 요청 빈도를 줄입니다. (5) 중요 요청과 덜 중요한 요청을 분류하여 우선순위를 관리합니다.
