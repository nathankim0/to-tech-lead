# TCP/UDP

## 1. 한 줄 요약

**TCP는 신뢰성 있는 순서 보장 전송이고, UDP는 빠르지만 보장 없는 전송입니다.**

## 2. 쉽게 설명

### 모바일 개발자 관점에서

HTTP 통신을 할 때 내부적으로 TCP가 사용됩니다. 반면 실시간 게임이나 음성/영상 통화에서는 UDP가 주로 사용됩니다.

**택배와 편지에 비유하면:**

| TCP | UDP |
|-----|-----|
| 등기 우편 | 일반 우편 |
| 도착 확인 O | 도착 확인 X |
| 순서 보장 O | 순서 보장 X |
| 느리지만 확실함 | 빠르지만 유실 가능 |

**모바일 앱에서의 실제 경험:**
- Retrofit/URLSession으로 API 호출 → 내부적으로 TCP 사용
- WebSocket 채팅 → TCP 기반 (순서와 신뢰성 필요)
- 실시간 영상 통화 (WebRTC) → UDP 기반 (지연보다 실시간성 중요)
- DNS 조회 → UDP 사용 (빠른 단발성 요청)

### TCP의 핵심 특징

1. **연결 지향적 (Connection-oriented)**: 통신 전 3-way handshake로 연결을 수립합니다.
2. **신뢰성 보장**: 패킷 손실 시 재전송합니다.
3. **순서 보장**: 보낸 순서대로 도착합니다.
4. **흐름 제어 (Flow Control)**: 수신자의 처리 능력에 맞춰 전송 속도를 조절합니다. (슬라이딩 윈도우)
5. **혼잡 제어 (Congestion Control)**: 네트워크 상태에 따라 전송량을 조절합니다. (Slow Start, AIMD)

### UDP의 핵심 특징

1. **비연결 지향적 (Connectionless)**: 연결 수립 없이 바로 전송합니다.
2. **신뢰성 없음**: 패킷 손실을 책임지지 않습니다.
3. **순서 보장 없음**: 도착 순서가 바뀔 수 있습니다.
4. **오버헤드 최소화**: 헤더가 작고 빠릅니다. (8 bytes vs TCP 20+ bytes)
5. **브로드캐스트/멀티캐스트 지원**: 여러 대상에 동시 전송 가능합니다.

### 언제 무엇을 사용하나요?

| 사용 사례 | 프로토콜 | 이유 |
|-----------|----------|------|
| 웹 API 호출 | TCP (HTTP) | 데이터 정확성이 중요 |
| 파일 다운로드 | TCP | 누락 없이 완전한 파일 필요 |
| 실시간 게임 (위치 동기화) | UDP | 약간의 손실보다 속도가 중요 |
| 음성/영상 통화 | UDP (RTP) | 지연보다 실시간성이 중요 |
| DNS 조회 | UDP | 간단한 요청/응답, 빠른 처리 |
| 라이브 스트리밍 | UDP (RTP/RTSP) | 실시간 재생이 중요 |
| 이메일 전송 | TCP (SMTP) | 메시지 손실 불가 |
| 채팅 메시지 | TCP (WebSocket) | 메시지 순서와 도착 보장 필요 |

## 3. 구조 다이어그램

### TCP 3-Way Handshake (연결 수립) - 상세

```
┌─────────────────┐                              ┌─────────────────┐
│     Client      │                              │     Server      │
│   (Mobile App)  │                              │   (LISTEN 상태)  │
└────────┬────────┘                              └────────┬────────┘
         │                                                │
         │  1. SYN (seq=1000)                             │
         │  ┌──────────────────────────────────────────┐ │
         │  │ SYN=1, ACK=0                             │ │
         │  │ Sequence Number: 1000                    │ │
         │  │ "안녕하세요, 통신 시작해도 될까요?"       │ │
         │  └──────────────────────────────────────────┘ │
         │ ─────────────────────────────────────────────>│
         │                                    [SYN_RCVD] │
         │                                                │
         │  2. SYN + ACK (seq=5000, ack=1001)            │
         │  ┌──────────────────────────────────────────┐ │
         │  │ SYN=1, ACK=1                             │ │
         │  │ Sequence Number: 5000                    │ │
         │  │ Acknowledgment: 1001 (= 1000 + 1)        │ │
         │  │ "네, 좋습니다. 저도 준비됐어요"           │ │
         │  └──────────────────────────────────────────┘ │
         │ <─────────────────────────────────────────────│
 [SYN_SENT]                                               │
         │                                                │
         │  3. ACK (seq=1001, ack=5001)                   │
         │  ┌──────────────────────────────────────────┐ │
         │  │ SYN=0, ACK=1                             │ │
         │  │ Sequence Number: 1001                    │ │
         │  │ Acknowledgment: 5001 (= 5000 + 1)        │ │
         │  │ "확인했어요, 시작합시다!"                 │ │
         │  └──────────────────────────────────────────┘ │
         │ ─────────────────────────────────────────────>│
[ESTABLISHED]                                  [ESTABLISHED]
         │                                                │
         │  ═══════════ 양방향 데이터 전송 가능 ═══════════ │
         │                                                │
```

### TCP 4-Way Handshake (연결 종료)

```
┌─────────────────┐                              ┌─────────────────┐
│     Client      │                              │     Server      │
│ [ESTABLISHED]   │                              │ [ESTABLISHED]   │
└────────┬────────┘                              └────────┬────────┘
         │                                                │
         │  1. FIN                                        │
         │  "저는 보낼 데이터가 없어요"                    │
         │ ─────────────────────────────────────────────>│
  [FIN_WAIT_1]                                            │
         │                                                │
         │  2. ACK                                        │
         │  "네, 알겠어요 (하지만 저는 아직 보낼 게 있어요)"│
         │ <─────────────────────────────────────────────│
  [FIN_WAIT_2]                                  [CLOSE_WAIT]
         │                                                │
         │         (서버가 남은 데이터 전송...)            │
         │                                                │
         │  3. FIN                                        │
         │  "이제 저도 보낼 데이터가 없어요"              │
         │ <─────────────────────────────────────────────│
                                                 [LAST_ACK]
         │                                                │
         │  4. ACK                                        │
         │  "확인했어요, 안녕히!"                         │
         │ ─────────────────────────────────────────────>│
  [TIME_WAIT]                                    [CLOSED]
         │                                                │
         │    (2MSL 대기 후 완전 종료)                    │
         │                                                │
  [CLOSED]                                                │
```

### TCP vs UDP 패킷 구조

```
TCP 헤더 (20~60 bytes) - 복잡하지만 기능이 풍부
┌────────────────────────────────────────────────────────────┐
│  Source Port (16)       │  Destination Port (16)           │
├────────────────────────────────────────────────────────────┤
│              Sequence Number (32)                          │  ← 순서 보장용
├────────────────────────────────────────────────────────────┤
│           Acknowledgment Number (32)                       │  ← 수신 확인용
├────────────────────────────────────────────────────────────┤
│ Offset│Reserved│Flags      │     Window Size (16)          │
│  (4)  │  (3)   │URG|ACK|PSH│                                │
│       │        │RST|SYN|FIN│                                │  ← 연결 제어
├────────────────────────────────────────────────────────────┤
│    Checksum (16)          │     Urgent Pointer (16)        │  ← 무결성 검증
├────────────────────────────────────────────────────────────┤
│                    Options (variable, 0~40 bytes)          │
└────────────────────────────────────────────────────────────┘

UDP 헤더 (8 bytes) - 단순하고 빠름
┌────────────────────────────────────────────────────────────┐
│  Source Port (16)         │  Destination Port (16)         │
├────────────────────────────────────────────────────────────┤
│    Length (16)            │       Checksum (16)            │
└────────────────────────────────────────────────────────────┘
                         그게 전부입니다!
```

### OSI 7계층과 TCP/IP 4계층에서의 위치

```
        OSI 7계층                    TCP/IP 4계층              프로토콜 예시
┌─────────────────────┐      ┌─────────────────────┐
│  Application (7)    │      │                     │      HTTP, HTTPS, FTP
├─────────────────────┤      │    Application      │      SMTP, WebSocket
│  Presentation (6)   │      │                     │      DNS, DHCP
├─────────────────────┤      │                     │
│  Session (5)        │      └─────────────────────┘
├─────────────────────┤      ┌─────────────────────┐
│  Transport (4)      │  ←←  │    Transport        │  ←←  ★ TCP, UDP ★
├─────────────────────┤      └─────────────────────┘
│  Network (3)        │      ┌─────────────────────┐
│                     │  ←←  │    Internet         │  ←←  IP, ICMP, ARP
├─────────────────────┤      └─────────────────────┘
│  Data Link (2)      │      ┌─────────────────────┐
├─────────────────────┤  ←←  │  Network Access     │  ←←  Ethernet, Wi-Fi
│  Physical (1)       │      │                     │      Bluetooth
└─────────────────────┘      └─────────────────────┘

📱 모바일 앱 관점:
   앱에서 URLSession.data(for:) 호출
        ↓
   HTTP 요청 생성 (Application)
        ↓
   TCP 세그먼트로 분할 (Transport)  ← 여기서 TCP/UDP 선택
        ↓
   IP 패킷으로 캡슐화 (Internet)
        ↓
   Wi-Fi/LTE 프레임으로 전송 (Network Access)
```

### TCP 흐름 제어 (슬라이딩 윈도우)

```
수신자가 처리할 수 있는 만큼만 전송

┌─────────────────────────────────────────────────────────────────────────┐
│                          TCP 슬라이딩 윈도우                              │
└─────────────────────────────────────────────────────────────────────────┘

Sender                                                          Receiver
┌──────────────────────────────────────┐                       Window=3000
│ 1000  1001  1002  1003  1004  1005   │
│  ✓     ✓    [====WINDOW====]         │ ← 보낼 수 있는 범위
└──────────────────────────────────────┘

1. Sender: 1002, 1003, 1004 전송 (윈도우 크기만큼)
            ─────────────────────────────────────────>

2. Receiver: 처리 완료, ACK=1005, Window=2000 (처리 느려짐)
            <─────────────────────────────────────────

3. Sender: 윈도우 크기가 줄었으므로 전송량 감소
┌──────────────────────────────────────┐
│ 1000  1001  1002  1003  1004  1005   │
│  ✓     ✓     ✓     ✓     ✓   [==]    │ ← 윈도우 축소
└──────────────────────────────────────┘

💡 모바일에서의 의미:
   서버 과부하 시 → 윈도우 크기 감소 → 자동으로 전송량 조절
   앱 개발자가 별도 처리 불필요 (TCP가 자동으로 처리)
```

## 4. 실무 적용 예시

### 예시 1: WebSocket (TCP 기반 실시간 통신)

```swift
// iOS에서 WebSocket 사용 (TCP 기반)
import Foundation

class ChatService: NSObject {
    private var webSocket: URLSessionWebSocketTask?
    private var session: URLSession!
    private var isConnected = false

    // 재연결 설정
    private let maxReconnectAttempts = 5
    private var reconnectAttempts = 0
    private var reconnectDelay: TimeInterval = 1.0

    override init() {
        super.init()
        session = URLSession(
            configuration: .default,
            delegate: self,
            delegateQueue: OperationQueue()
        )
    }

    func connect() {
        guard !isConnected else { return }

        let url = URL(string: "wss://chat.example.com/ws")!
        webSocket = session.webSocketTask(with: url)
        webSocket?.resume()

        receiveMessage()
    }

    func sendMessage(_ text: String) {
        guard isConnected else {
            print("WebSocket not connected")
            return
        }

        let message = URLSessionWebSocketTask.Message.string(text)
        webSocket?.send(message) { [weak self] error in
            if let error = error {
                print("Send error: \(error)")
                self?.handleDisconnection()
            }
        }
    }

    private func receiveMessage() {
        webSocket?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    DispatchQueue.main.async {
                        NotificationCenter.default.post(
                            name: .chatMessageReceived,
                            object: text
                        )
                    }
                case .data(let data):
                    print("Received binary: \(data.count) bytes")
                @unknown default:
                    break
                }
                self?.receiveMessage() // 계속 수신 대기
            case .failure(let error):
                print("Receive error: \(error)")
                self?.handleDisconnection()
            }
        }
    }

    // Ping/Pong으로 연결 상태 확인 (TCP Keep-Alive와 별개)
    func startHeartbeat() {
        Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { [weak self] _ in
            self?.webSocket?.sendPing { error in
                if let error = error {
                    print("Ping failed: \(error)")
                    self?.handleDisconnection()
                }
            }
        }
    }

    private func handleDisconnection() {
        isConnected = false
        attemptReconnection()
    }

    private func attemptReconnection() {
        guard reconnectAttempts < maxReconnectAttempts else {
            print("Max reconnection attempts reached")
            return
        }

        reconnectAttempts += 1
        let delay = reconnectDelay * pow(2, Double(reconnectAttempts - 1)) // 지수 백오프

        DispatchQueue.main.asyncAfter(deadline: .now() + delay) { [weak self] in
            self?.connect()
        }
    }

    func disconnect() {
        webSocket?.cancel(with: .normalClosure, reason: nil)
        isConnected = false
    }
}

extension ChatService: URLSessionWebSocketDelegate {
    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didOpenWithProtocol protocol: String?) {
        isConnected = true
        reconnectAttempts = 0
        print("WebSocket connected")
    }

    func urlSession(_ session: URLSession,
                    webSocketTask: URLSessionWebSocketTask,
                    didCloseWith closeCode: URLSessionWebSocketTask.CloseCode,
                    reason: Data?) {
        isConnected = false
        print("WebSocket closed: \(closeCode)")
    }
}

extension Notification.Name {
    static let chatMessageReceived = Notification.Name("chatMessageReceived")
}
```

### 예시 2: Android에서 UDP 사용 (실시간 게임)

```kotlin
// Android에서 UDP 통신 (게임 위치 동기화)
import java.net.DatagramPacket
import java.net.DatagramSocket
import java.net.InetAddress
import kotlinx.coroutines.*
import java.nio.ByteBuffer

class GameNetworkManager(
    private val serverHost: String = "game.example.com",
    private val serverPort: Int = 9999
) {
    private var socket: DatagramSocket? = null
    private val serverAddress: InetAddress by lazy {
        InetAddress.getByName(serverHost)
    }
    private var receiveJob: Job? = null
    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())

    // 패킷 시퀀스 번호 (순서 확인용 - UDP는 순서 보장 X)
    private var sequenceNumber = 0

    fun start() {
        socket = DatagramSocket().apply {
            soTimeout = 5000 // 5초 타임아웃
        }
    }

    // 플레이어 위치 전송 (UDP - 빠른 전송이 중요)
    fun sendPlayerPosition(playerId: String, x: Float, y: Float, z: Float) {
        scope.launch {
            try {
                // 패킷 구조: [seq(4)][playerId(16)][x(4)][y(4)][z(4)] = 32 bytes
                val buffer = ByteBuffer.allocate(32)
                buffer.putInt(sequenceNumber++)
                buffer.put(playerId.take(16).padEnd(16).toByteArray())
                buffer.putFloat(x)
                buffer.putFloat(y)
                buffer.putFloat(z)

                val data = buffer.array()
                val packet = DatagramPacket(data, data.size, serverAddress, serverPort)
                socket?.send(packet)

                // UDP는 전송 확인을 하지 않음
                // 손실되어도 다음 위치 업데이트가 곧 전송됨 (초당 30-60회)
            } catch (e: Exception) {
                // 오류 발생해도 게임은 계속 진행
                // 로그만 남기고 다음 패킷 전송 시도
                e.printStackTrace()
            }
        }
    }

    // 다른 플레이어 위치 수신
    fun startReceiving(onPositionReceived: (playerId: String, x: Float, y: Float, z: Float) -> Unit) {
        receiveJob = scope.launch {
            val buffer = ByteArray(1024)
            var lastSequence = mutableMapOf<String, Int>()

            while (isActive) {
                try {
                    val packet = DatagramPacket(buffer, buffer.size)
                    socket?.receive(packet)

                    val data = ByteBuffer.wrap(packet.data, 0, packet.length)
                    val seq = data.getInt()
                    val playerIdBytes = ByteArray(16)
                    data.get(playerIdBytes)
                    val playerId = String(playerIdBytes).trim()
                    val x = data.getFloat()
                    val y = data.getFloat()
                    val z = data.getFloat()

                    // 오래된 패킷 무시 (네트워크 지연으로 순서가 뒤바뀔 수 있음)
                    val lastSeq = lastSequence[playerId] ?: -1
                    if (seq > lastSeq) {
                        lastSequence[playerId] = seq
                        withContext(Dispatchers.Main) {
                            onPositionReceived(playerId, x, y, z)
                        }
                    }
                } catch (e: java.net.SocketTimeoutException) {
                    // 타임아웃은 정상 상황 (수신할 패킷이 없음)
                    continue
                } catch (e: Exception) {
                    e.printStackTrace()
                }
            }
        }
    }

    fun stop() {
        receiveJob?.cancel()
        socket?.close()
        scope.cancel()
    }
}

// 사용 예시
class GameActivity : AppCompatActivity() {
    private val networkManager = GameNetworkManager()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        networkManager.start()

        // 다른 플레이어 위치 수신
        networkManager.startReceiving { playerId, x, y, z ->
            updateOtherPlayerPosition(playerId, x, y, z)
        }

        // 내 위치를 초당 30회 전송
        lifecycleScope.launch {
            while (isActive) {
                val myPosition = getMyPosition()
                networkManager.sendPlayerPosition(
                    "player123",
                    myPosition.x,
                    myPosition.y,
                    myPosition.z
                )
                delay(33) // ~30 FPS
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        networkManager.stop()
    }
}
```

### 예시 3: TCP Keep-Alive 및 타임아웃 설정 (iOS)

```swift
// URLSession 설정에서 TCP 옵션 조정
class NetworkConfiguration {

    static func createOptimizedSession() -> URLSession {
        let configuration = URLSessionConfiguration.default

        // 연결 타임아웃: TCP 3-way handshake 완료 시간
        configuration.timeoutIntervalForRequest = 30

        // 리소스 타임아웃: 전체 요청 완료 시간
        configuration.timeoutIntervalForResource = 300 // 5분 (대용량 다운로드용)

        // HTTP 파이프라이닝 (HTTP/1.1에서 성능 향상)
        configuration.httpShouldUsePipelining = true

        // 연결당 최대 동시 요청 수
        configuration.httpMaximumConnectionsPerHost = 6

        // 셀룰러 네트워크 사용 허용
        configuration.allowsCellularAccess = true

        // 백그라운드 세션 (앱이 백그라운드일 때도 전송)
        // let background = URLSessionConfiguration.background(withIdentifier: "com.app.background")

        return URLSession(configuration: configuration)
    }

    // 네트워크 품질에 따른 동적 타임아웃
    static func createAdaptiveSession(for networkType: NetworkType) -> URLSession {
        let configuration = URLSessionConfiguration.default

        switch networkType {
        case .wifi:
            configuration.timeoutIntervalForRequest = 10
            configuration.httpMaximumConnectionsPerHost = 6
        case .cellular4G:
            configuration.timeoutIntervalForRequest = 20
            configuration.httpMaximumConnectionsPerHost = 4
        case .cellular3G:
            configuration.timeoutIntervalForRequest = 40
            configuration.httpMaximumConnectionsPerHost = 2
        case .unknown:
            configuration.timeoutIntervalForRequest = 30
            configuration.httpMaximumConnectionsPerHost = 4
        }

        return URLSession(configuration: configuration)
    }
}

enum NetworkType {
    case wifi
    case cellular4G
    case cellular3G
    case unknown
}
```

### 예시 4: Socket 연결 상태 모니터링 및 재시도

```kotlin
// Android에서 네트워크 상태에 따른 처리
import android.content.Context
import android.net.ConnectivityManager
import android.net.Network
import android.net.NetworkCapabilities
import android.net.NetworkRequest
import kotlinx.coroutines.*

class NetworkStateManager(private val context: Context) {
    private val connectivityManager =
        context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager

    // 현재 네트워크 상태
    var currentNetworkType: NetworkType = NetworkType.NONE
        private set

    // 네트워크 변경 콜백
    private val networkCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            updateNetworkType()
            onNetworkAvailable?.invoke()
        }

        override fun onLost(network: Network) {
            currentNetworkType = NetworkType.NONE
            onNetworkLost?.invoke()
        }

        override fun onCapabilitiesChanged(
            network: Network,
            networkCapabilities: NetworkCapabilities
        ) {
            updateNetworkType()
        }
    }

    var onNetworkAvailable: (() -> Unit)? = null
    var onNetworkLost: (() -> Unit)? = null

    fun startMonitoring() {
        val request = NetworkRequest.Builder()
            .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
            .build()
        connectivityManager.registerNetworkCallback(request, networkCallback)
        updateNetworkType()
    }

    fun stopMonitoring() {
        connectivityManager.unregisterNetworkCallback(networkCallback)
    }

    private fun updateNetworkType() {
        val network = connectivityManager.activeNetwork
        val capabilities = connectivityManager.getNetworkCapabilities(network)

        currentNetworkType = when {
            capabilities == null -> NetworkType.NONE
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> NetworkType.WIFI
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> {
                if (capabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_NOT_METERED)) {
                    NetworkType.CELLULAR_FAST
                } else {
                    NetworkType.CELLULAR_SLOW
                }
            }
            else -> NetworkType.OTHER
        }
    }

    fun isConnected(): Boolean {
        return currentNetworkType != NetworkType.NONE
    }

    // TCP 연결 실패 시 재시도 로직 (지수 백오프)
    suspend fun <T> retryWithExponentialBackoff(
        maxRetries: Int = 5,
        initialDelayMs: Long = 1000,
        maxDelayMs: Long = 32000,
        factor: Double = 2.0,
        block: suspend () -> T
    ): T {
        var currentDelay = initialDelayMs
        var lastException: Exception? = null

        repeat(maxRetries) { attempt ->
            try {
                return block()
            } catch (e: java.net.ConnectException) {
                // TCP 연결 실패 (서버 도달 불가)
                lastException = e
            } catch (e: java.net.SocketTimeoutException) {
                // TCP 타임아웃 (서버 응답 없음)
                lastException = e
            } catch (e: java.net.UnknownHostException) {
                // DNS 조회 실패
                lastException = e
            } catch (e: javax.net.ssl.SSLException) {
                // TLS 핸드셰이크 실패
                lastException = e
            }

            if (attempt < maxRetries - 1) {
                // 네트워크 없으면 재시도 무의미
                if (!isConnected()) {
                    throw NetworkUnavailableException("No network connection")
                }

                delay(currentDelay)
                currentDelay = minOf((currentDelay * factor).toLong(), maxDelayMs)
            }
        }

        throw lastException ?: Exception("Unknown error after $maxRetries retries")
    }
}

enum class NetworkType {
    WIFI,
    CELLULAR_FAST,  // 4G/5G
    CELLULAR_SLOW,  // 3G 이하
    OTHER,
    NONE
}

class NetworkUnavailableException(message: String) : Exception(message)
```

## 5. 장단점

### TCP

| 장점 | 단점 |
|------|------|
| 데이터 전송의 신뢰성 보장 | 연결 수립/해제 오버헤드 (3-way, 4-way handshake) |
| 패킷 순서 보장 | 패킷 손실 시 재전송으로 인한 지연 |
| 흐름 제어로 수신자 보호 | 헤더 크기가 큼 (20~60 bytes) |
| 혼잡 제어로 네트워크 보호 | Head-of-Line Blocking 문제 |
| 오류 검출 및 복구 | 실시간 애플리케이션에 부적합할 수 있음 |

### UDP

| 장점 | 단점 |
|------|------|
| 낮은 지연 시간 (연결 수립 불필요) | 신뢰성 보장 없음 (패킷 손실 가능) |
| 작은 헤더 크기 (8 bytes) | 순서 보장 없음 |
| 브로드캐스트/멀티캐스트 지원 | 흐름/혼잡 제어 없음 |
| 단순한 구조로 빠른 처리 | 애플리케이션에서 직접 신뢰성 구현 필요 |
| 실시간 애플리케이션에 적합 | 방화벽에 의해 차단될 수 있음 |

## 6. 실무에서 자주 겪는 문제와 해결책

### 문제 1: TCP 연결 타임아웃

```swift
// 모바일 네트워크에서 자주 발생
// 원인: 3G/4G 전환, 터널 진입, 약한 신호

// 해결책: 적절한 타임아웃 + 재시도 + 사용자 피드백
class ResilientNetworkManager {
    func request(_ url: URL) async throws -> Data {
        let startTime = Date()

        do {
            return try await performRequest(url)
        } catch let error as URLError {
            switch error.code {
            case .timedOut:
                // 타임아웃: 네트워크 상태 확인 후 재시도
                if isNetworkAvailable() {
                    return try await performRequest(url, timeout: 60) // 더 긴 타임아웃
                } else {
                    throw NetworkError.noConnection
                }
            case .networkConnectionLost:
                // 연결 끊김: Wi-Fi ↔ 셀룰러 전환 시 발생
                // 잠시 대기 후 재시도
                try await Task.sleep(nanoseconds: 1_000_000_000) // 1초
                return try await performRequest(url)
            default:
                throw error
            }
        }
    }
}
```

### 문제 2: WebSocket 연결 유지

```kotlin
// 모바일에서 WebSocket이 자주 끊어지는 문제
// 원인: NAT 타임아웃, 네트워크 전환, Doze 모드

class ReliableWebSocket(private val url: String) {
    private var webSocket: WebSocket? = null
    private val client = OkHttpClient.Builder()
        .pingInterval(15, TimeUnit.SECONDS) // Ping으로 연결 유지
        .build()

    // Heartbeat으로 연결 상태 확인
    private val heartbeatJob = CoroutineScope(Dispatchers.IO).launch {
        while (isActive) {
            delay(30_000) // 30초마다
            if (!checkConnection()) {
                reconnect()
            }
        }
    }

    // Android Doze 모드 대응
    fun handleDozeMode(isDozeMode: Boolean) {
        if (isDozeMode) {
            // Doze 모드에서는 WebSocket 사용 불가
            // FCM으로 대체
            disconnect()
        } else {
            connect()
        }
    }
}
```

## 7. 내 생각

```
(이 공간은 학습 후 자신의 생각을 정리하는 곳입니다)

- TCP와 UDP의 차이를 이해하고 나서 새롭게 보이는 것들:


- 내가 개발한 앱에서 사용하고 있는 프로토콜은 무엇인지:


- 실시간 기능을 구현한다면 어떤 프로토콜을 선택할지와 그 이유:


```

## 8. 추가 질문

1. **QUIC 프로토콜이란 무엇인가요?** HTTP/3에서 사용되는 이 프로토콜은 TCP와 UDP 중 어느 것 위에서 동작하나요?

> **답변**: QUIC(Quick UDP Internet Connections)는 Google이 개발하고 IETF가 표준화한 전송 계층 프로토콜로, UDP 위에서 동작합니다. 하지만 UDP 위에 TCP의 장점(신뢰성, 순서 보장, 혼잡 제어)을 애플리케이션 레벨에서 구현했습니다.
>
> QUIC의 주요 특징: (1) 0-RTT 연결 재개 - 이전에 연결했던 서버에는 핸드셰이크 없이 즉시 데이터 전송 가능. (2) 스트림 독립성 - TCP는 하나의 패킷 손실이 모든 데이터를 막지만(HOL Blocking), QUIC는 손실된 스트림만 영향을 받음. (3) Connection Migration - IP 주소가 바뀌어도(Wi-Fi→LTE) 연결 유지. (4) 내장 TLS 1.3 - 암호화가 프로토콜에 통합되어 있음.
>
> 모바일 환경에서 QUIC는 특히 유리합니다. 네트워크 전환이 빈번하고, 패킷 손실이 잦은 환경에서 더 나은 성능을 보입니다. iOS 15+와 Android 10+에서 HTTP/3(QUIC)를 지원하며, URLSession과 OkHttp가 서버 지원 시 자동으로 활용합니다.

2. **TCP의 Head-of-Line Blocking 문제란 무엇이고, HTTP/2와 HTTP/3에서는 이를 어떻게 해결하나요?**

> **답변**: Head-of-Line(HOL) Blocking은 앞선 패킷의 문제가 뒤따르는 패킷의 처리를 막는 현상입니다. TCP는 순서 보장을 위해 패킷 1이 손실되면 패킷 2, 3, 4가 도착해도 애플리케이션에 전달하지 않고 기다립니다. HTTP/1.1에서는 한 연결에서 요청이 순차 처리되어 앞선 요청이 느리면 뒤 요청도 지연됩니다.
>
> HTTP/2는 하나의 TCP 연결에서 여러 스트림을 멀티플렉싱하지만, TCP 레벨의 HOL Blocking은 여전히 존재합니다. 하나의 패킷 손실이 모든 스트림을 막습니다.
>
> HTTP/3(QUIC)는 UDP 위에서 자체 스트림 관리를 하여 스트림 간 독립성을 보장합니다. 스트림 A의 패킷 손실은 스트림 B, C에 영향을 주지 않습니다. 이는 모바일 환경처럼 패킷 손실이 빈번한 상황에서 큰 성능 향상을 가져옵니다.

3. **모바일 환경에서 TCP 연결이 자주 끊어지는 이유는 무엇인가요?** (Wi-Fi에서 셀룰러로 전환 시 등)

> **답변**: TCP 연결은 4-tuple(소스IP, 소스포트, 목적지IP, 목적지포트)로 식별됩니다. Wi-Fi→셀룰러 전환 시 IP 주소가 바뀌므로 서버 입장에서는 완전히 다른 연결로 인식하여 기존 연결이 끊어집니다.
>
> 추가 원인들: (1) NAT 타임아웃 - 이동통신사 NAT 장비가 일정 시간(보통 2-5분) 비활동 연결을 끊음. (2) 안드로이드 Doze 모드 - 배터리 절약을 위해 백그라운드 네트워크 차단. (3) iOS 백그라운드 제한 - 앱이 백그라운드로 가면 네트워크 연결 제한.
>
> 해결책: (1) Keep-Alive 패킷 전송 (30초~1분 간격). (2) QUIC/HTTP/3 사용 시 Connection Migration 활용. (3) 앱이 포그라운드로 돌아올 때 연결 상태 확인 및 재연결. (4) 중요한 실시간 기능은 FCM/APNs 푸시로 대체.

4. **WebSocket과 Server-Sent Events(SSE)의 차이점은 무엇인가요?** 각각 어떤 상황에서 사용하면 좋을까요?

> **답변**: WebSocket은 양방향(Full-duplex) 통신으로 클라이언트와 서버 모두 언제든 메시지를 보낼 수 있습니다. SSE는 단방향(Server→Client)으로 서버만 클라이언트에게 데이터를 푸시합니다.
>
> WebSocket의 특징: 별도 프로토콜(ws://, wss://), 바이너리와 텍스트 모두 지원, 연결 수립 후 오버헤드 최소화. SSE의 특징: 일반 HTTP 사용, 텍스트만 지원, 자동 재연결 내장, 기존 HTTP 인프라와 호환.
>
> 사용 사례 구분: WebSocket - 채팅, 실시간 게임, 양방향 실시간 협업(Figma 같은). SSE - 주식 시세, 뉴스 피드, 알림, AI 스트리밍 응답(ChatGPT 같은).
>
> 모바일에서는 배터리와 연결 유지 관점에서 SSE가 더 유리할 수 있습니다(자동 재연결, 더 적은 오버헤드). 하지만 양방향 통신이 필요하면 WebSocket이 필수입니다.

5. **TCP Nagle 알고리즘이란 무엇이고, 실시간 애플리케이션에서 왜 비활성화하나요?**

> **답변**: Nagle 알고리즘은 네트워크 효율성을 위해 작은 패킷들을 모아서 한 번에 전송하는 TCP의 기능입니다. 예를 들어 1바이트씩 10번 보내는 대신, 10바이트를 모아서 1번 전송합니다. 이렇게 하면 헤더 오버헤드(40바이트)가 줄어들어 네트워크 효율이 좋아집니다.
>
> 하지만 실시간 애플리케이션(게임, 채팅, 트레이딩)에서는 작은 지연도 치명적입니다. Nagle이 패킷을 모으느라 수십 ms의 지연이 발생할 수 있습니다. 특히 Delayed ACK와 결합되면 200ms 이상의 지연이 발생할 수도 있습니다.
>
> 해결책: `TCP_NODELAY` 소켓 옵션을 설정하여 Nagle을 비활성화합니다. iOS의 CFSocketStream이나 Android의 Socket.setTcpNoDelay(true)로 설정합니다. HTTP 클라이언트(URLSession, OkHttp)는 기본적으로 이를 적절히 처리합니다.

6. **모바일 앱에서 네트워크 전환(Wi-Fi ↔ 셀룰러) 시 TCP 연결을 유지하는 방법은 무엇인가요?** MPTCP(Multipath TCP)란?

> **답변**: MPTCP(Multipath TCP)는 하나의 TCP 연결을 여러 네트워크 경로(Wi-Fi + 셀룰러)로 동시에 사용할 수 있게 해주는 TCP 확장입니다. 한 경로가 끊어져도 다른 경로로 seamless하게 전환되어 연결이 유지됩니다.
>
> iOS는 MPTCP를 네이티브로 지원합니다. Siri, Apple Music 등에서 사용 중이며, URLSessionConfiguration.multipathServiceType으로 설정할 수 있습니다. Android는 일부 제조사와 커널에서 지원하지만 표준 API가 없습니다.
>
> MPTCP 외의 대안: (1) 애플리케이션 레벨에서 연결 상태 모니터링 및 재연결 구현. (2) QUIC/HTTP/3 사용 - Connection ID 기반으로 IP가 바뀌어도 연결 유지. (3) VPN 사용 - VPN 터널 내에서는 실제 IP 변경이 보이지 않음. (4) 중요 작업은 네트워크 전환 시 일시 정지 후 재개.
>
> 실무에서는 네트워크 전환을 감지(ConnectivityManager/NWPathMonitor)하고, 전환 후 연결을 재수립하는 것이 가장 현실적입니다.
