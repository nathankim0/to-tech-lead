# DNS (Domain Name System)

## 1. 한 줄 요약

**DNS는 사람이 읽을 수 있는 도메인 이름(api.example.com)을 컴퓨터가 이해하는 IP 주소(192.168.1.1)로 변환하는 인터넷의 전화번호부입니다.**

## 2. 쉽게 설명

### 모바일 개발자 관점에서

모바일 앱에서 `https://api.example.com/users`를 호출할 때, 실제로 연결되는 것은 IP 주소입니다. 이 변환 과정이 DNS 조회입니다.

**전화번호부에 비유하면:**
- 도메인 이름 = 친구 이름 ("김철수")
- IP 주소 = 전화번호 ("010-1234-5678")
- DNS 서버 = 전화번호부

우리는 친구 이름만 기억하면 되고, 전화번호부(DNS)가 실제 번호를 찾아줍니다.

**모바일 앱에서의 실제 경험:**
- 앱 실행 후 첫 API 호출이 느린 이유 → DNS 조회 시간 포함
- `UnknownHostException` 또는 `NSURLErrorDomain -1003` → DNS 조회 실패
- 비행기 모드 후 복구 시 연결 지연 → DNS 캐시 만료
- Charles/Proxyman에서 도메인으로 트래픽 필터링 → DNS가 해석한 후의 IP로 통신

### DNS 조회 과정

1. **브라우저/앱 캐시 확인**: 이미 조회한 적 있나?
2. **OS 캐시 확인**: 운영체제에 저장되어 있나?
3. **라우터 캐시 확인**: 공유기에 저장되어 있나?
4. **ISP DNS 서버 조회**: 통신사 DNS에 물어봄
5. **재귀적 조회**: Root → TLD → Authoritative DNS 순서로 조회

### DNS 레코드 타입

| 타입 | 용도 | 예시 | 모바일에서의 의미 |
|------|------|------|------------------|
| A | 도메인 → IPv4 주소 | example.com → 93.184.216.34 | 가장 기본, API 호출에 사용 |
| AAAA | 도메인 → IPv6 주소 | example.com → 2606:2800:220:1:... | IPv6 네트워크 지원 |
| CNAME | 도메인 → 다른 도메인 (별칭) | www.example.com → example.com | CDN 연결에 자주 사용 |
| MX | 메일 서버 지정 | example.com → mail.example.com | 이메일 앱 |
| TXT | 텍스트 정보 (인증 등) | 도메인 소유권 검증 | 딥링크 검증 |
| NS | 네임서버 지정 | example.com → ns1.example.com | - |
| SRV | 서비스 위치 지정 | _xmpp._tcp.example.com | 채팅 프로토콜 등 |

## 3. 구조 다이어그램

### DNS 조회 전체 흐름 (상세)

```
┌─────────────────┐
│   Mobile App    │  https://api.example.com/users 요청
│                 │
└────────┬────────┘
         │ 1. DNS 조회 시작
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      로컬 캐시 계층 (빠름)                            │
├─────────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐        │
│  │   앱 캐시     │ → │   OS 캐시     │ → │  라우터 캐시  │        │
│  │  (URLSession/ │   │  (iOS: mDNS   │   │  (공유기)     │        │
│  │   OkHttp)    │   │   Android:    │   │               │        │
│  │  TTL: 세션   │   │   NetD)       │   │  TTL: 설정값  │        │
│  └───────────────┘   └───────────────┘   └───────────────┘        │
└────────────────────────────┬────────────────────────────────────────┘
                             │ 캐시 미스 시
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ISP DNS 서버 (Recursive Resolver)                 │
│                    예: SKT DNS, KT DNS, Google 8.8.8.8             │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 캐시에 없으면 재귀적 조회 시작                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────────┘
                             │ 2. "api.example.com의 IP가 뭐야?"
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Root DNS Server (.)                             │
│                 전 세계 13개 루트 서버 클러스터                       │
│                 (실제로는 수백 개의 미러 서버)                        │
│                                                                     │
│  응답: ".com은 이 서버들이 관리해: a.gtld-servers.net..."           │
└────────────────────────────┬────────────────────────────────────────┘
                             │ 3. ".com TLD 서버로 질의"
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      TLD DNS Server (.com)                           │
│                 .com, .net, .org, .kr 등 최상위 도메인 관리          │
│                                                                     │
│  응답: "example.com은 이 네임서버가 관리해: ns1.example.com..."     │
└────────────────────────────┬────────────────────────────────────────┘
                             │ 4. "Authoritative 서버로 질의"
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│               Authoritative DNS Server (example.com)                 │
│                 실제 도메인 레코드를 가진 서버                        │
│                 (Route 53, Cloudflare DNS 등)                       │
│                                                                     │
│  응답: "api.example.com = 192.168.1.100, TTL=300"                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │ 5. 최종 IP 주소 반환 + 캐싱
                             ▼
                      192.168.1.100
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ISP 캐시 저장       라우터 캐시 저장      OS 캐시 저장
    (TTL 동안)          (TTL 동안)           (TTL 동안)
```

### DNS 캐싱 계층과 TTL

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DNS 캐시 계층                                │
└─────────────────────────────────────────────────────────────────────┘

Level 1: 앱 내부 캐시
┌────────────────────────────────────────────────────────────────────┐
│  URLSession / OkHttp 내부 DNS 캐시                                  │
│  • 앱 실행 중에만 유효                                              │
│  • 재시작하면 초기화                                                 │
│  • 가장 빠른 조회                                                    │
└────────────────────────────────────────────────────────────────────┘
                              ↓ 미스
Level 2: OS 레벨 캐시
┌────────────────────────────────────────────────────────────────────┐
│  iOS: mDNSResponder 데몬                                            │
│  Android: netd DNS 캐시                                             │
│  • DNS 레코드의 TTL 값 존중                                         │
│  • 설정 > Wi-Fi > DNS에서 사용자 지정 DNS 가능                      │
│  예시: TTL=300이면 5분간 캐시                                       │
└────────────────────────────────────────────────────────────────────┘
                              ↓ 미스
Level 3: 네트워크 레벨 캐시
┌────────────────────────────────────────────────────────────────────┐
│  공유기(Router) DNS 캐시                                            │
│  • 같은 네트워크의 모든 기기가 공유                                  │
│  • 공유기 재시작 시 초기화                                          │
└────────────────────────────────────────────────────────────────────┘
                              ↓ 미스
Level 4: ISP DNS 서버 캐시
┌────────────────────────────────────────────────────────────────────┐
│  통신사 DNS 서버 (KT, SKT, LG U+)                                   │
│  또는 Public DNS (8.8.8.8, 1.1.1.1)                                │
│  • 수백만 사용자가 공유하므로 히트율 높음                           │
│  • 인기 있는 도메인은 거의 항상 캐시됨                              │
└────────────────────────────────────────────────────────────────────┘

💡 TTL (Time To Live) 예시:
   api.example.com    TTL=60     → 1분 캐시 (빠른 장애 대응)
   www.example.com    TTL=3600   → 1시간 캐시 (안정적)
   static.example.com TTL=86400  → 24시간 캐시 (거의 안 바뀜)
```

### DNS 레코드 예시 (실제 Zone 파일)

```
; example.com의 DNS Zone 파일 예시
; AWS Route 53 또는 Cloudflare에서 관리

$TTL 3600       ; 기본 TTL 1시간

; =========== 네임서버 설정 ===========
example.com.        IN  NS      ns1.example.com.
example.com.        IN  NS      ns2.example.com.
ns1.example.com.    IN  A       10.0.0.1
ns2.example.com.    IN  A       10.0.0.2

; =========== 웹 서버 ===========
example.com.        IN  A       93.184.216.34
www.example.com.    IN  CNAME   example.com.    ; www는 별칭

; =========== API 서버 (로드밸런싱) ===========
; 여러 A 레코드 → 라운드로빈 DNS
api.example.com.    IN  A       10.0.1.1
api.example.com.    IN  A       10.0.1.2
api.example.com.    IN  A       10.0.1.3

; =========== CDN 연결 ===========
; CNAME으로 CDN 도메인 연결
cdn.example.com.    IN  CNAME   d123456.cloudfront.net.
images.example.com. IN  CNAME   example.b-cdn.net.

; =========== 환경별 분리 ===========
api-prod.example.com.   IN  A       10.0.1.1
api-staging.example.com.IN  A       10.0.2.1
api-dev.example.com.    IN  A       10.0.3.1

; =========== 모바일 딥링크 검증 ===========
; Apple App Site Association 용
example.com.        IN  TXT     "apple-app-site-association"
; Android App Links 용
example.com.        IN  TXT     "android-app-link=com.example.app"

; =========== 메일 서버 ===========
example.com.        IN  MX  10  mail1.example.com.
example.com.        IN  MX  20  mail2.example.com.  ; 백업

; =========== SPF/DKIM (이메일 인증) ===========
example.com.        IN  TXT     "v=spf1 include:_spf.google.com ~all"
```

### GeoDNS / Latency-based DNS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GeoDNS 동작 원리                                  │
│              (같은 도메인이지만 위치에 따라 다른 IP 반환)             │
└─────────────────────────────────────────────────────────────────────┘

                        api.example.com 조회

한국 사용자                                        미국 사용자
┌─────────────────┐                              ┌─────────────────┐
│   Seoul, Korea  │                              │  California, US │
└────────┬────────┘                              └────────┬────────┘
         │                                                │
         ▼                                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    GeoDNS / Route 53 / Cloudflare                   │
│                                                                     │
│  if (user.location == "Korea")                                     │
│      return "43.200.xxx.xxx"  // Seoul Region                      │
│  else if (user.location == "USA")                                  │
│      return "52.94.xxx.xxx"   // US-West Region                    │
│  else                                                               │
│      return "nearest_server_ip"                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
         │                                                │
         ▼                                                ▼
┌─────────────────┐                              ┌─────────────────┐
│  Seoul Server   │                              │  US-West Server │
│  43.200.xxx.xxx │                              │  52.94.xxx.xxx  │
│  Latency: 20ms  │                              │  Latency: 15ms  │
└─────────────────┘                              └─────────────────┘

💡 모바일 앱에서의 의미:
   - 글로벌 서비스에서 자동으로 가까운 서버 연결
   - DNS 응답에 따라 레이턴시 최적화
   - 장애 시 다른 리전으로 자동 전환 가능
```

## 4. 실무 적용 예시

### 예시 1: DNS 조회 시간 측정 및 최적화 (iOS)

```swift
import Foundation
import Network

class DNSProfiler {
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "dns.profiler")

    // DNS 조회 시간 측정
    func measureDNSResolution(for hostname: String) async -> DNSResult {
        let startTime = CFAbsoluteTimeGetCurrent()

        // getaddrinfo를 사용한 DNS 조회
        var hints = addrinfo(
            ai_flags: 0,
            ai_family: AF_UNSPEC,  // IPv4와 IPv6 모두
            ai_socktype: SOCK_STREAM,
            ai_protocol: 0,
            ai_addrlen: 0,
            ai_canonname: nil,
            ai_addr: nil,
            ai_next: nil
        )

        var result: UnsafeMutablePointer<addrinfo>?
        let status = getaddrinfo(hostname, nil, &hints, &result)

        let endTime = CFAbsoluteTimeGetCurrent()
        let duration = (endTime - startTime) * 1000 // ms

        defer { freeaddrinfo(result) }

        if status != 0 {
            return DNSResult(
                hostname: hostname,
                success: false,
                durationMs: duration,
                addresses: [],
                error: String(cString: gai_strerror(status))
            )
        }

        // 모든 주소 수집
        var addresses: [String] = []
        var ptr = result
        while ptr != nil {
            if let addr = ptr?.pointee.ai_addr {
                var hostBuffer = [CChar](repeating: 0, count: Int(NI_MAXHOST))
                if getnameinfo(addr, socklen_t(ptr!.pointee.ai_addrlen),
                               &hostBuffer, socklen_t(hostBuffer.count),
                               nil, 0, NI_NUMERICHOST) == 0 {
                    addresses.append(String(cString: hostBuffer))
                }
            }
            ptr = ptr?.pointee.ai_next
        }

        return DNSResult(
            hostname: hostname,
            success: true,
            durationMs: duration,
            addresses: addresses,
            error: nil
        )
    }

    // 앱 시작 시 주요 도메인 프리페칭
    func prefetchDomains(_ domains: [String]) async {
        await withTaskGroup(of: Void.self) { group in
            for domain in domains {
                group.addTask {
                    let result = await self.measureDNSResolution(for: domain)
                    print("Prefetched \(domain): \(result.durationMs)ms, IPs: \(result.addresses)")
                }
            }
        }
    }

    // 자주 사용하는 도메인 목록
    static let commonDomains = [
        "api.example.com",
        "cdn.example.com",
        "images.example.com",
        "analytics.example.com"
    ]
}

struct DNSResult {
    let hostname: String
    let success: Bool
    let durationMs: Double
    let addresses: [String]
    let error: String?
}

// 앱 시작 시 사용
class AppDelegate: UIResponder, UIApplicationDelegate {
    let dnsProfiler = DNSProfiler()

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        // 백그라운드에서 DNS 프리페칭
        Task.detached(priority: .utility) {
            await self.dnsProfiler.prefetchDomains(DNSProfiler.commonDomains)
        }

        return true
    }
}
```

### 예시 2: DNS-over-HTTPS 사용 (Android)

```kotlin
// Android에서 보안 DNS 사용 (DNS over HTTPS)
import okhttp3.OkHttpClient
import okhttp3.dnsoverhttps.DnsOverHttps
import okhttp3.HttpUrl.Companion.toHttpUrl
import java.net.InetAddress
import java.net.UnknownHostException

class SecureDnsManager(private val context: Context) {

    companion object {
        // 공용 DoH 서버 목록
        const val CLOUDFLARE_DOH = "https://1.1.1.1/dns-query"
        const val GOOGLE_DOH = "https://dns.google/dns-query"
        const val QUAD9_DOH = "https://dns.quad9.net/dns-query"
    }

    // 기본 OkHttpClient (DoH 부트스트랩용)
    private val bootstrapClient = OkHttpClient.Builder()
        .connectTimeout(5, TimeUnit.SECONDS)
        .readTimeout(5, TimeUnit.SECONDS)
        .build()

    // Cloudflare DoH 설정
    private val dns = DnsOverHttps.Builder()
        .client(bootstrapClient)
        .url(CLOUDFLARE_DOH.toHttpUrl())
        .bootstrapDnsHosts(
            // DoH 서버 자체의 IP (부트스트랩)
            InetAddress.getByName("1.1.1.1"),
            InetAddress.getByName("1.0.0.1"),
            InetAddress.getByName("2606:4700:4700::1111"),
            InetAddress.getByName("2606:4700:4700::1001")
        )
        .includeIPv6(true)  // IPv6 지원
        .build()

    // DoH를 사용하는 OkHttpClient 생성
    fun createSecureClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .dns(dns)
            .connectTimeout(10, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    // 수동 DNS 조회
    suspend fun resolveDomain(hostname: String): DnsLookupResult {
        return withContext(Dispatchers.IO) {
            val startTime = System.currentTimeMillis()

            try {
                val addresses = dns.lookup(hostname)
                val duration = System.currentTimeMillis() - startTime

                DnsLookupResult(
                    hostname = hostname,
                    addresses = addresses.map { it.hostAddress ?: "" },
                    durationMs = duration,
                    success = true,
                    usedDoH = true
                )
            } catch (e: UnknownHostException) {
                DnsLookupResult(
                    hostname = hostname,
                    addresses = emptyList(),
                    durationMs = System.currentTimeMillis() - startTime,
                    success = false,
                    error = e.message
                )
            }
        }
    }

    // DoH vs 일반 DNS 성능 비교
    suspend fun compareDnsPerformance(hostname: String): DnsComparison {
        val regularResult = measureRegularDns(hostname)
        val dohResult = resolveDomain(hostname)

        return DnsComparison(
            hostname = hostname,
            regularDnsMs = regularResult.durationMs,
            dohDnsMs = dohResult.durationMs,
            dohOverhead = dohResult.durationMs - regularResult.durationMs
        )
    }

    private suspend fun measureRegularDns(hostname: String): DnsLookupResult {
        return withContext(Dispatchers.IO) {
            val startTime = System.currentTimeMillis()
            try {
                val addresses = InetAddress.getAllByName(hostname)
                DnsLookupResult(
                    hostname = hostname,
                    addresses = addresses.map { it.hostAddress ?: "" },
                    durationMs = System.currentTimeMillis() - startTime,
                    success = true,
                    usedDoH = false
                )
            } catch (e: UnknownHostException) {
                DnsLookupResult(
                    hostname = hostname,
                    addresses = emptyList(),
                    durationMs = System.currentTimeMillis() - startTime,
                    success = false,
                    error = e.message
                )
            }
        }
    }
}

data class DnsLookupResult(
    val hostname: String,
    val addresses: List<String>,
    val durationMs: Long,
    val success: Boolean,
    val usedDoH: Boolean = false,
    val error: String? = null
)

data class DnsComparison(
    val hostname: String,
    val regularDnsMs: Long,
    val dohDnsMs: Long,
    val dohOverhead: Long
)

// 사용 예시
val secureClient = SecureDnsManager(context).createSecureClient()
val request = Request.Builder()
    .url("https://api.example.com/data")
    .build()

// DNS 조회가 HTTPS로 암호화됨 (ISP가 볼 수 없음)
secureClient.newCall(request).enqueue(object : Callback {
    override fun onResponse(call: Call, response: Response) {
        // 응답 처리
    }
    override fun onFailure(call: Call, e: IOException) {
        // 오류 처리
    }
})
```

### 예시 3: DNS 오류 처리 및 사용자 안내

```swift
// iOS에서 DNS 오류 처리
class NetworkErrorHandler {

    enum DnsError: Error, LocalizedError {
        case noInternetConnection
        case dnsResolutionFailed(hostname: String)
        case dnsTimeout
        case serverNotFound(hostname: String)

        var errorDescription: String? {
            switch self {
            case .noInternetConnection:
                return "인터넷에 연결되어 있지 않습니다"
            case .dnsResolutionFailed(let hostname):
                return "서버 주소를 찾을 수 없습니다: \(hostname)"
            case .dnsTimeout:
                return "서버 연결에 시간이 너무 오래 걸립니다"
            case .serverNotFound(let hostname):
                return "서버를 찾을 수 없습니다: \(hostname)"
            }
        }

        var recoverySuggestion: String? {
            switch self {
            case .noInternetConnection:
                return "Wi-Fi 또는 셀룰러 데이터 연결을 확인해주세요."
            case .dnsResolutionFailed:
                return "네트워크 설정을 확인하거나 잠시 후 다시 시도해주세요."
            case .dnsTimeout:
                return "네트워크 연결이 느립니다. 다른 네트워크에서 시도해주세요."
            case .serverNotFound:
                return "서버 점검 중일 수 있습니다. 잠시 후 다시 시도해주세요."
            }
        }
    }

    func handleURLError(_ error: URLError, for url: URL) -> DnsError {
        let hostname = url.host ?? "unknown"

        switch error.code {
        case .notConnectedToInternet:
            return .noInternetConnection

        case .cannotFindHost:
            // DNS 조회 실패
            return .dnsResolutionFailed(hostname: hostname)

        case .timedOut:
            // 타임아웃 (DNS 또는 연결)
            return .dnsTimeout

        case .cannotConnectToHost:
            // 호스트에 연결 불가 (DNS는 성공, 서버가 응답 안 함)
            return .serverNotFound(hostname: hostname)

        case .dnsLookupFailed:
            return .dnsResolutionFailed(hostname: hostname)

        default:
            return .serverNotFound(hostname: hostname)
        }
    }

    // 재시도 가능한 에러인지 판단
    func isRetryable(_ error: DnsError) -> Bool {
        switch error {
        case .noInternetConnection:
            return false  // 연결 없으면 재시도 무의미
        case .dnsResolutionFailed, .dnsTimeout, .serverNotFound:
            return true   // 일시적 문제일 수 있음
        }
    }

    // 네트워크 상태 확인
    func checkNetworkAndRetry<T>(
        maxRetries: Int = 3,
        operation: () async throws -> T
    ) async throws -> T {
        var lastError: Error?

        for attempt in 0..<maxRetries {
            do {
                return try await operation()
            } catch let urlError as URLError {
                let dnsError = handleURLError(urlError, for: urlError.failureURLString.flatMap { URL(string: $0) } ?? URL(string: "https://unknown")!)
                lastError = dnsError

                if !isRetryable(dnsError) {
                    throw dnsError
                }

                // 지수 백오프
                let delay = Double(1 << attempt) // 1, 2, 4초
                try await Task.sleep(nanoseconds: UInt64(delay * 1_000_000_000))
            }
        }

        throw lastError ?? DnsError.serverNotFound(hostname: "unknown")
    }
}

// 사용 예시
class ApiClient {
    private let errorHandler = NetworkErrorHandler()

    func fetchData() async throws -> Data {
        try await errorHandler.checkNetworkAndRetry {
            let url = URL(string: "https://api.example.com/data")!
            let (data, _) = try await URLSession.shared.data(from: url)
            return data
        }
    }
}
```

### 예시 4: 커스텀 DNS 서버 설정 (Android 9+)

```kotlin
// Android에서 Private DNS 설정 확인 및 안내
class DnsConfigurationHelper(private val context: Context) {

    // 현재 DNS 설정 확인
    fun getCurrentDnsServers(): List<String> {
        val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE)
            as ConnectivityManager

        val network = connectivityManager.activeNetwork ?: return emptyList()
        val linkProperties = connectivityManager.getLinkProperties(network) ?: return emptyList()

        return linkProperties.dnsServers.map { it.hostAddress ?: "" }
    }

    // Private DNS (DoT) 설정 상태 확인 (Android 9+)
    @RequiresApi(Build.VERSION_CODES.P)
    fun getPrivateDnsStatus(): PrivateDnsStatus {
        val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE)
            as ConnectivityManager

        val network = connectivityManager.activeNetwork ?: return PrivateDnsStatus.OFF
        val linkProperties = connectivityManager.getLinkProperties(network) ?: return PrivateDnsStatus.OFF

        return when {
            linkProperties.isPrivateDnsActive -> PrivateDnsStatus.ACTIVE
            linkProperties.privateDnsServerName != null -> PrivateDnsStatus.CONFIGURED
            else -> PrivateDnsStatus.OFF
        }
    }

    // Private DNS 설정 화면으로 이동
    fun openPrivateDnsSettings() {
        val intent = Intent(Settings.ACTION_WIRELESS_SETTINGS)
        context.startActivity(intent)
    }

    // DNS 관련 디버깅 정보 수집
    fun getDnsDebugInfo(): DnsDebugInfo {
        return DnsDebugInfo(
            dnsServers = getCurrentDnsServers(),
            privateDnsStatus = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
                getPrivateDnsStatus().name
            } else {
                "NOT_SUPPORTED"
            },
            isUsingMobileData = isUsingMobileData(),
            networkType = getNetworkType()
        )
    }

    private fun isUsingMobileData(): Boolean {
        val cm = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        val network = cm.activeNetwork ?: return false
        val capabilities = cm.getNetworkCapabilities(network) ?: return false
        return capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR)
    }

    private fun getNetworkType(): String {
        val cm = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        val network = cm.activeNetwork ?: return "NONE"
        val capabilities = cm.getNetworkCapabilities(network) ?: return "UNKNOWN"

        return when {
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> "WIFI"
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> "CELLULAR"
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_ETHERNET) -> "ETHERNET"
            else -> "OTHER"
        }
    }
}

enum class PrivateDnsStatus {
    OFF,
    CONFIGURED,
    ACTIVE
}

data class DnsDebugInfo(
    val dnsServers: List<String>,
    val privateDnsStatus: String,
    val isUsingMobileData: Boolean,
    val networkType: String
)
```

## 5. 장단점

### DNS의 장점

| 장점 | 설명 |
|------|------|
| 사람 친화적 | 숫자 IP 대신 기억하기 쉬운 도메인 사용 |
| 유연한 서버 관리 | IP 변경 시 DNS만 수정하면 됨 |
| 로드 밸런싱 | 여러 IP를 반환하여 트래픽 분산 |
| 지역 기반 라우팅 | GeoDNS로 가까운 서버 연결 |
| 장애 대응 | 빠른 DNS 전환으로 서버 장애 대응 |
| 캐싱 | 반복 조회 시 빠른 응답 |

### DNS의 단점 및 주의사항

| 단점 | 설명 |
|------|------|
| 추가 지연 시간 | DNS 조회에 수십~수백 ms 소요 가능 |
| 캐시 문제 | TTL 동안 오래된 IP로 연결될 수 있음 |
| 보안 취약점 | DNS 스푸핑, 캐시 포이즈닝 위험 |
| 단일 장애점 | DNS 서버 장애 시 서비스 접속 불가 |
| 전파 지연 | DNS 변경 후 전파까지 시간 필요 |

### 모바일 앱에서의 DNS 고려사항

| 상황 | 대응 방안 |
|------|----------|
| 느린 DNS 응답 | DNS 프리페칭, 로컬 캐싱 |
| 불안정한 네트워크 | 타임아웃 설정, 재시도 로직 |
| 보안 | DoH/DoT 사용 고려 |
| 네트워크 전환 | DNS 캐시 무효화 고려 |
| 중국/특수 지역 | 로컬 DNS 서버, 대체 IP 하드코딩 |

## 6. 실무에서 자주 겪는 문제와 해결책

### 문제 1: 앱 첫 실행 시 느린 로딩

```swift
// 원인: 콜드 스타트 시 DNS 캐시가 비어있음
// 해결책: 스플래시 화면에서 DNS 프리페칭

class SplashViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        // 백그라운드에서 DNS 미리 조회
        Task {
            await DNSProfiler().prefetchDomains([
                "api.example.com",
                "cdn.example.com"
            ])

            // DNS 완료 후 메인 화면으로 이동
            await MainActor.run {
                navigateToMain()
            }
        }
    }
}
```

### 문제 2: 특정 네트워크에서만 연결 실패

```kotlin
// 원인: 회사/학교 네트워크의 DNS 필터링
// 해결책: 대체 DNS 사용 또는 에러 메시지 안내

class NetworkDiagnostics(context: Context) {
    suspend fun diagnoseConnection(hostname: String): DiagnosisResult {
        // 1. 시스템 DNS로 시도
        val systemResult = tryResolve(hostname, useDoH = false)

        if (systemResult.success) {
            return DiagnosisResult.SUCCESS
        }

        // 2. DoH로 시도
        val dohResult = tryResolve(hostname, useDoH = true)

        if (dohResult.success) {
            return DiagnosisResult.DNS_BLOCKED_USE_DOH
        }

        // 3. 둘 다 실패
        return DiagnosisResult.SERVER_UNREACHABLE
    }
}

enum class DiagnosisResult {
    SUCCESS,
    DNS_BLOCKED_USE_DOH,    // 네트워크 DNS가 차단, DoH 사용 권장
    SERVER_UNREACHABLE      // 서버 자체가 접속 불가
}
```

## 7. 내 생각

```
(이 공간은 학습 후 자신의 생각을 정리하는 곳입니다)

- DNS 조회 과정을 이해한 후 네트워크 지연에 대해 새롭게 알게 된 점:


- 내 앱에서 DNS 관련 최적화가 필요한 부분:


- DNS 장애 상황을 경험했거나, 이에 대비하는 방법:


```

## 8. 추가 질문

1. **DNS TTL(Time To Live)이란 무엇이고, 서비스 운영에서 어떻게 설정해야 하나요?** 너무 짧거나 긴 TTL의 문제점은?

> **답변**: TTL은 DNS 레코드가 캐시에 저장되는 시간(초)입니다. 클라이언트와 중간 DNS 서버들은 TTL 동안 캐시된 결과를 재사용하고, 만료 후에는 다시 조회합니다.
>
> TTL 설정 가이드: (1) 짧은 TTL (60-300초): 장애 시 빠른 전환이 필요한 경우, 블루-그린 배포 시, 마이그레이션 준비 단계. 단점은 DNS 서버 부하 증가와 조회 지연 빈발. (2) 긴 TTL (3600-86400초): 거의 변경되지 않는 서버, CDN의 정적 콘텐츠 도메인. 단점은 IP 변경 시 전파가 느림, 장애 대응이 어려움.
>
> 실무 권장: API 서버는 300초(5분), CDN은 3600초(1시간), 마이그레이션 24시간 전에 TTL을 60초로 낮추고, 완료 후 다시 올림. 모바일 앱에서는 TTL을 직접 제어할 수 없으므로, 서버 측 설정이 중요합니다.

2. **DNS-over-HTTPS(DoH)와 DNS-over-TLS(DoT)의 차이점은 무엇인가요?** 모바일 앱에서 보안 DNS를 사용하는 방법은?

> **답변**: DoH와 DoT 모두 DNS 쿼리를 암호화하여 ISP나 중간자가 어떤 도메인에 접속하는지 볼 수 없게 합니다. DoH는 HTTPS(포트 443)를 사용하고, DoT는 TLS(포트 853)를 사용합니다.
>
> 차이점: DoH는 일반 HTTPS 트래픽과 구분이 안 되어 차단이 어렵고, 웹 브라우저에서 주로 사용합니다. DoT는 전용 포트를 사용하여 방화벽에서 쉽게 식별/차단 가능하지만, 시스템 레벨 설정이 가능합니다.
>
> 모바일에서: Android 9+는 설정에서 Private DNS(DoT)를 지원합니다. iOS는 시스템 레벨 DoH/DoT가 없지만, 앱에서 OkHttp의 DnsOverHttps나 직접 구현할 수 있습니다. 프라이버시가 중요한 앱(VPN, 보안 앱)에서는 DoH를 직접 구현하는 것이 좋습니다.

3. **CDN(Content Delivery Network)에서 DNS는 어떤 역할을 하나요?** GeoDNS와 Anycast DNS의 차이점은?

> **답변**: CDN에서 DNS는 사용자를 가장 가까운 엣지 서버로 연결하는 핵심 역할을 합니다. cdn.example.com을 조회하면 사용자 위치에 따라 다른 IP가 반환됩니다.
>
> GeoDNS: DNS 서버가 사용자의 IP 주소(지역)를 보고 가까운 서버의 IP를 반환합니다. 예: 한국 사용자 → 서울 서버 IP, 미국 사용자 → LA 서버 IP. 장점은 정확한 지역 타겟팅, 단점은 DNS 서버 위치와 사용자 위치가 다를 수 있음(VPN 사용 시 등).
>
> Anycast DNS: 여러 서버가 동일한 IP 주소를 가지고, BGP 라우팅이 가장 가까운 서버로 연결합니다. 예: 1.1.1.1(Cloudflare)은 전 세계에서 같은 IP지만 가까운 서버로 연결됨. 장점은 자동 페일오버, 단점은 구성이 복잡함.
>
> 실제 CDN은 두 가지를 조합하여 사용합니다. Cloudflare, AWS CloudFront 등이 이 방식을 사용합니다.

4. **모바일 앱에서 DNS 캐싱을 직접 구현해야 하는 상황은 언제인가요?** URLSession/OkHttp의 기본 DNS 캐싱과 차이점은?

> **답변**: 대부분의 경우 시스템 DNS 캐시로 충분하지만, 직접 구현이 필요한 경우가 있습니다: (1) 오프라인 지원 - 마지막으로 알려진 IP로 연결 시도. (2) 중국 등 DNS 검열 지역 - 하드코딩된 IP 또는 대체 DNS. (3) 빠른 재연결 - 앱 재시작 시 즉시 연결. (4) DNS 장애 대비 - 백업 IP 목록 유지.
>
> 시스템 캐시와 차이점: 시스템 캐시는 TTL 만료 시 삭제되고, 앱 재시작이나 네트워크 전환 시 초기화될 수 있습니다. 직접 구현하면 영구 저장, 커스텀 TTL, 다중 IP 관리가 가능합니다. 하지만 IP가 변경되었을 때 오래된 IP로 연결하는 문제가 있으므로, 주기적 갱신과 폴백 로직이 필요합니다.

5. **DNS Rebinding 공격이란 무엇이고, 모바일 앱에서 어떻게 방어하나요?**

> **답변**: DNS Rebinding 공격은 공격자가 자신의 도메인의 DNS 레코드를 빠르게 변경하여, 피해자의 브라우저/앱이 내부 네트워크 리소스에 접근하게 만드는 공격입니다. 예: 처음에 evil.com → 공격자 서버 IP, 잠시 후 evil.com → 192.168.1.1(피해자 공유기)로 변경.
>
> 모바일 앱에서의 위험: WebView를 사용하는 앱에서 주로 발생. 악성 웹페이지가 앱의 WebView에서 로드되고, DNS rebinding으로 내부 API에 접근할 수 있음.
>
> 방어 방법: (1) 서버 측에서 Host 헤더 검증 - 허용된 도메인만 처리. (2) WebView에서 private IP 대역 차단. (3) 내부 API는 인증 필수. (4) WebView의 네트워크 접근을 제한하거나, 신뢰할 수 있는 도메인만 허용.

6. **서버 마이그레이션 시 DNS 변경은 어떻게 진행해야 하나요?** Blue-Green 배포와 DNS의 관계는?

> **답변**: 서버 마이그레이션 시 DNS 변경은 신중해야 합니다. TTL 때문에 모든 사용자에게 즉시 반영되지 않기 때문입니다.
>
> 권장 절차: (1) 마이그레이션 24-48시간 전: TTL을 60-300초로 낮춤. (2) 새 서버 준비: 새 IP에서 서비스 가동 및 테스트. (3) DNS 변경: A 레코드를 새 IP로 변경. (4) 모니터링: 구/신 서버 모두 모니터링 (구 TTL 시간 동안). (5) 완료 후: TTL을 원래 값으로 복원, 구 서버 종료.
>
> Blue-Green 배포: 두 개의 동일한 환경(Blue=현재, Green=새 버전)을 유지하고, DNS 또는 로드밸런서로 트래픽을 전환합니다. DNS 방식은 전환이 느리므로(TTL), 로드밸런서 방식이 더 빠릅니다. AWS에서는 Route 53 가중치 기반 라우팅으로 점진적 전환이 가능합니다. 문제 발생 시 DNS를 다시 Blue로 변경하여 롤백합니다.
