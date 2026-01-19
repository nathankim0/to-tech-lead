# Nginx

## 1. 한 줄 요약

**Nginx는 웹 서버이자 리버스 프록시로, 클라이언트 요청을 받아 정적 파일을 제공하거나 백엔드 서버로 트래픽을 전달하며, 로드 밸런싱, SSL 종료, 캐싱, 압축 등 다양한 기능을 제공하는 고성능 웹 인프라의 핵심 컴포넌트입니다.**

## 2. 쉽게 설명

### 모바일 개발자 관점에서

모바일 앱에서 `https://api.example.com/users`를 호출하면, 대부분 Nginx가 가장 먼저 그 요청을 받습니다. Nginx는 요청을 실제 애플리케이션 서버(Node.js, Spring 등)로 전달하고, 응답을 다시 앱으로 보내줍니다. 모바일 개발자가 API 호출 시 경험하는 응답 속도, 에러 처리, 타임아웃 등 많은 부분이 Nginx 설정에 영향을 받습니다.

**호텔 프론트 데스크에 비유하면:**
- **Nginx** = 호텔 프론트 데스크 (24시간 운영, 모든 요청의 첫 번째 접점)
- **백엔드 서버** = 호텔 직원들 (실제 서비스 담당)
- **클라이언트(앱)** = 호텔 손님
- **로드 밸런싱** = 여러 직원에게 업무 분배
- **캐싱** = 자주 묻는 질문 FAQ 게시판
- **SSL 종료** = 보안 검색대 (신원 확인 후 내부 출입)

손님(앱)이 "룸서비스 주세요"라고 프론트(Nginx)에 말하면, 프론트는 적절한 직원(백엔드)에게 요청을 전달하고, 결과를 손님에게 돌려줍니다. 만약 같은 요청이 자주 온다면 캐시된 응답을 바로 제공하기도 합니다.

### Nginx의 주요 역할

| 역할 | 설명 | 모바일 앱 관점 |
|------|------|----------------|
| **웹 서버** | 정적 파일(HTML, CSS, JS, 이미지) 직접 제공 | 웹뷰 콘텐츠 제공 |
| **리버스 프록시** | 요청을 백엔드 서버로 전달 | API 요청 중계 |
| **로드 밸런서** | 여러 서버에 트래픽 분산 | 안정적인 API 응답 |
| **SSL 종료** | HTTPS 암호화 처리 | 보안 통신 지원 |
| **캐싱** | 자주 요청되는 데이터 캐시 | 빠른 응답 |
| **압축** | 응답 데이터 gzip 압축 | 네트워크 사용량 절감 |

### 웹 서버 vs 리버스 프록시

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          동작 모드 비교                                       │
└─────────────────────────────────────────────────────────────────────────────┘

웹 서버로 동작할 때 (정적 파일 서빙):
  ┌────────────┐        ┌────────────────┐        ┌─────────────────┐
  │  Mobile    │ ──────▶│     Nginx      │──────▶ │  /var/www/html  │
  │    App     │◀────── │  (Web Server)  │◀────── │  정적 파일들      │
  └────────────┘        └────────────────┘        └─────────────────┘
       GET /image.png        파일 읽기              image.png 반환

리버스 프록시로 동작할 때 (API 요청 중계):
  ┌────────────┐        ┌────────────────┐        ┌─────────────────┐
  │  Mobile    │ ──────▶│     Nginx      │──────▶ │  Backend API    │
  │    App     │◀────── │(Reverse Proxy) │◀────── │  (Node/Spring)  │
  └────────────┘        └────────────────┘        └─────────────────┘
      POST /api/users        proxy_pass           JSON 응답 생성

혼합 모드 (실제 운영 환경):
  ┌────────────┐        ┌────────────────┐
  │  Mobile    │        │     Nginx      │
  │    App     │───────▶│                │─┬─▶ /static/*  → 파일 직접 서빙
  │            │◀───────│  라우팅 판단    │ │
  └────────────┘        └────────────────┘ ├─▶ /api/*     → Backend 프록시
                                           │
                                           └─▶ /ws/*      → WebSocket 프록시
```

### 포워드 프록시 vs 리버스 프록시

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    프록시 유형 비교                                           │
└─────────────────────────────────────────────────────────────────────────────┘

포워드 프록시 (Forward Proxy):
- 클라이언트를 대신하여 외부 서버에 요청
- 클라이언트의 IP를 숨김
- 예: 회사 프록시, VPN

  ┌────────────┐     ┌──────────────┐     ┌────────────────┐
  │   Client   │────▶│Forward Proxy │────▶│ External Server│
  │  (사내 PC)  │     │  (회사 프록시) │     │   (인터넷)      │
  └────────────┘     └──────────────┘     └────────────────┘
       내부 IP           프록시 IP로 접근

리버스 프록시 (Reverse Proxy):
- 서버를 대신하여 클라이언트 요청 수신
- 백엔드 서버의 IP를 숨김
- 예: Nginx, HAProxy

  ┌────────────┐     ┌──────────────┐     ┌────────────────┐
  │   Client   │────▶│Reverse Proxy │────▶│ Backend Server │
  │ (Mobile App)│     │   (Nginx)    │     │  (내부 서버)    │
  └────────────┘     └──────────────┘     └────────────────┘
      외부에서 접근        공개 IP            내부 IP (보호됨)
```

## 3. 구조 다이어그램

### Nginx 전체 아키텍처

```
                                    ┌─────────────────────┐
                                    │   Mobile App        │
                                    │   (iOS/Android)     │
                                    └──────────┬──────────┘
                                               │
                                    HTTPS 요청 │ (443 포트)
                                               │
                              ┌────────────────▼────────────────┐
                              │           Nginx                  │
                              │  ┌────────────────────────────┐ │
                              │  │  1. SSL 종료 (HTTPS→HTTP)   │ │
                              │  │  2. 요청 라우팅 판단         │ │
                              │  │  3. 로드 밸런싱             │ │
                              │  │  4. 캐싱 확인               │ │
                              │  │  5. 압축 적용               │ │
                              │  └────────────────────────────┘ │
                              └──────────────┬──────────────────┘
                                             │
           ┌─────────────────────────────────┼─────────────────────────────────┐
           │                                 │                                 │
           ▼                                 ▼                                 ▼
┌─────────────────────┐        ┌─────────────────────┐        ┌─────────────────────┐
│  정적 파일 서버      │        │   API Server #1     │        │   API Server #2     │
│  (/static/*)        │        │   (Node.js)         │        │   (Node.js)         │
│                     │        │   localhost:3001    │        │   localhost:3002    │
│  - images/          │        │                     │        │                     │
│  - css/             │        │   /api/*            │        │   /api/*            │
│  - js/              │        │                     │        │                     │
└─────────────────────┘        └─────────────────────┘        └─────────────────────┘
```

### 요청 처리 흐름

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Nginx 요청 처리 흐름                               │
└─────────────────────────────────────────────────────────────────────────────┘

  [1] 요청 수신                [2] 처리 결정              [3] 응답 반환
      │                            │                          │
      ▼                            ▼                          ▼

GET /api/users HTTP/1.1    ┌─ /api/* → upstream    ┌─ 캐시 있음 → 캐시 반환
Host: api.example.com      │  /static/* → 정적파일  │  캐시 없음 → 백엔드 호출
                           │  / → 기본 index.html   │
                           └─────────────────────── └─────────────────────────

  요청 URL 분석                location 블록 매칭        proxy_pass / try_files
```

### 로드 밸런싱 방식

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          로드 밸런싱 알고리즘                                 │
└─────────────────────────────────────────────────────────────────────────────┘

1. Round Robin (기본값) - 순차적 분배
   ┌────────────┐
   │   Client   │
   └─────┬──────┘
         │  1번째 → Server A
         │  2번째 → Server B
         │  3번째 → Server C
         │  4번째 → Server A (반복)
         ▼
   ┌─────────────────────────────┐
   │ Server A │ Server B │ Server C │
   └─────────────────────────────┘

   설정: upstream backend { server a; server b; server c; }  # 기본값

2. Least Connections - 연결 적은 서버로
   ┌────────────┐
   │   Client   │
   └─────┬──────┘
         │  현재 연결 수 확인
         │  가장 적은 서버로 전달
         ▼
   ┌─────────────────────────────────────┐
   │ Server A (5) │ Server B (2) │ Server C (8) │
   │              │      ▲       │              │
   └──────────────┴──────┴───────┴──────────────┘
                    선택!

   설정: upstream backend { least_conn; server a; server b; }

3. IP Hash - 같은 클라이언트는 같은 서버로
   ┌────────────┐
   │ Client IP  │ → hash(192.168.1.100) % 3 = 1 → Server B
   │192.168.1.100│    항상 같은 서버로 연결 (세션 유지)
   └────────────┘

   설정: upstream backend { ip_hash; server a; server b; }

4. Weighted Round Robin - 서버 성능에 따른 가중치 분배
   ┌────────────┐
   │   Client   │
   └─────┬──────┘
         │  Server A (weight=5): 50% 트래픽
         │  Server B (weight=3): 30% 트래픽
         │  Server C (weight=2): 20% 트래픽
         ▼
   ┌─────────────────────────────────────────────┐
   │ Server A     │ Server B   │ Server C        │
   │ (고성능 8코어) │ (중간 4코어) │ (저성능 2코어)  │
   └─────────────────────────────────────────────┘

   설정: upstream backend { server a weight=5; server b weight=3; server c weight=2; }
```

### Nginx 이벤트 기반 아키텍처

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Nginx vs Apache 아키텍처 비교                             │
└─────────────────────────────────────────────────────────────────────────────┘

Apache (프로세스/스레드 기반):
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Apache HTTP Server                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │ Process #1  │ │ Process #2  │ │ Process #3  │ │ Process #N  │   ...     │
│  │ Client A    │ │ Client B    │ │ Client C    │ │ Client N    │           │
│  │ (점유 중)    │ │ (점유 중)    │ │ (대기 중)    │ │ (점유 중)    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘           │
│                                                                             │
│  문제: 10,000 동시 연결 = 10,000 프로세스 필요 → 메모리 폭발 (C10K 문제)       │
└─────────────────────────────────────────────────────────────────────────────┘

Nginx (이벤트 기반):
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Nginx                                          │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                     Master Process (관리자)                             │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                   │                                         │
│           ┌───────────────────────┼───────────────────────┐                 │
│           ▼                       ▼                       ▼                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐           │
│  │   Worker #1     │   │   Worker #2     │   │   Worker #N     │           │
│  │ ┌─────────────┐ │   │ ┌─────────────┐ │   │ ┌─────────────┐ │           │
│  │ │Event Queue  │ │   │ │Event Queue  │ │   │ │Event Queue  │ │           │
│  │ │ - Client A  │ │   │ │ - Client D  │ │   │ │ - Client G  │ │           │
│  │ │ - Client B  │ │   │ │ - Client E  │ │   │ │ - Client H  │ │           │
│  │ │ - Client C  │ │   │ │ - Client F  │ │   │ │ - Client I  │ │           │
│  │ │ - ...       │ │   │ │ - ...       │ │   │ │ - ...       │ │           │
│  │ └─────────────┘ │   │ └─────────────┘ │   │ └─────────────┘ │           │
│  │  수천 연결 처리   │   │  수천 연결 처리   │   │  수천 연결 처리   │           │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘           │
│                                                                             │
│  장점: 4 Worker로 40,000+ 동시 연결 처리 가능 → 메모리 효율적                   │
└─────────────────────────────────────────────────────────────────────────────┘

epoll (Linux) / kqueue (macOS) 이벤트 루프:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  while (true) {                                                            │
│      events = epoll_wait(fd_list)  // I/O 이벤트 대기 (블로킹 없음)          │
│                                                                            │
│      for (event in events) {                                               │
│          if (event.type == READ_READY) {                                   │
│              read_request(event.fd)     // 요청 읽기                        │
│          }                                                                 │
│          if (event.type == WRITE_READY) {                                  │
│              send_response(event.fd)    // 응답 보내기                      │
│          }                                                                 │
│      }                                                                     │
│  }                                                                         │
│                                                                            │
│  → 단일 스레드가 수천 연결을 논블로킹으로 처리                                  │
└────────────────────────────────────────────────────────────────────────────┘
```

## 4. 실무 적용 예시

### 예시 1: 기본 Nginx 설정

```nginx
# /etc/nginx/nginx.conf

# 워커 프로세스 수 (CPU 코어 수에 맞춤)
worker_processes auto;

# 이벤트 설정
events {
    worker_connections 1024;  # 워커당 최대 연결 수
    use epoll;               # Linux에서 효율적인 이벤트 처리
}

http {
    # MIME 타입 설정
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 로그 포맷
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    '$request_time';  # 응답 시간 추가

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    # 성능 최적화
    sendfile        on;      # 파일 전송 최적화
    tcp_nopush      on;      # 헤더와 파일 함께 전송
    tcp_nodelay     on;      # 작은 패킷도 즉시 전송
    keepalive_timeout 65;    # Keep-Alive 연결 유지 시간

    # Gzip 압축
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;    # 1KB 이상만 압축

    # 서버 설정 include
    include /etc/nginx/conf.d/*.conf;
}
```

### 예시 2: API 리버스 프록시 설정

```nginx
# /etc/nginx/conf.d/api.conf

# 업스트림 서버 정의 (로드 밸런싱)
upstream api_servers {
    # 로드 밸런싱 방식 (기본: round-robin)
    # least_conn;  # 연결 수가 적은 서버로
    # ip_hash;     # 같은 IP는 같은 서버로

    server 127.0.0.1:3001 weight=3;  # 가중치 높음
    server 127.0.0.1:3002 weight=2;
    server 127.0.0.1:3003 backup;    # 백업 서버

    # 헬스체크 (실패 시 제외)
    # max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name api.example.com;

    # HTTP → HTTPS 리다이렉트
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL 인증서
    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    # SSL 설정
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers on;

    # 보안 헤더
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;

    # API 요청 → 백엔드 서버로 프록시
    location /api/ {
        proxy_pass http://api_servers;

        # 프록시 헤더 설정
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 타임아웃 설정
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # 버퍼 설정
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # 헬스체크 엔드포인트
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }

    # 정적 파일 (이미지, JS, CSS)
    location /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 에러 페이지
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

### 예시 3: 모바일 앱용 API 캐싱 설정

```nginx
# /etc/nginx/conf.d/api-cache.conf

# 캐시 저장소 정의
proxy_cache_path /var/cache/nginx/api
                 levels=1:2
                 keys_zone=api_cache:10m      # 메모리 10MB
                 max_size=100m                # 디스크 100MB
                 inactive=60m                 # 60분 미사용 시 삭제
                 use_temp_path=off;

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # ... SSL 설정 생략 ...

    # 캐싱 가능한 읽기 전용 API
    location /api/v1/products {
        proxy_pass http://api_servers;

        # 캐시 설정
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;       # 200 응답은 10분 캐시
        proxy_cache_valid 404 1m;        # 404는 1분 캐시
        proxy_cache_use_stale error timeout updating;  # 오류 시 오래된 캐시 사용

        # 캐시 키 정의
        proxy_cache_key "$scheme$request_method$host$request_uri";

        # 캐시 상태 헤더 추가 (디버깅용)
        add_header X-Cache-Status $upstream_cache_status;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 캐싱하면 안 되는 API (사용자별 데이터)
    location /api/v1/users {
        proxy_pass http://api_servers;

        # 캐시 비활성화
        proxy_cache off;
        add_header Cache-Control "no-store, no-cache, must-revalidate";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 예시 4: Rate Limiting (요청 제한)

```nginx
# /etc/nginx/conf.d/rate-limit.conf

# Rate Limit 정의
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
# $binary_remote_addr: IP 주소 기준
# zone=api_limit:10m: 10MB 메모리 사용
# rate=10r/s: 초당 10개 요청

limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
# 동시 연결 수 제한용

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # 로그인 API - 더 엄격한 제한
    location /api/auth/login {
        limit_req zone=api_limit burst=5 nodelay;
        # burst=5: 순간적으로 5개까지 허용
        # nodelay: 지연 없이 즉시 처리

        # 제한 초과 시 응답
        limit_req_status 429;  # Too Many Requests

        proxy_pass http://api_servers;
    }

    # 일반 API
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        limit_conn conn_limit 10;  # IP당 동시 연결 10개

        proxy_pass http://api_servers;
    }
}
```

### 예시 5: 자주 사용하는 Nginx 명령어

```bash
# ==================== 서비스 관리 ====================

# Nginx 시작/중지/재시작
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx    # 설정 변경 시 무중단 반영 ★

# 상태 확인
sudo systemctl status nginx


# ==================== 설정 확인 ====================

# 설정 문법 검사 (반드시 reload 전에!)
sudo nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# 현재 적용된 설정 확인
sudo nginx -T


# ==================== 로그 확인 ====================

# 액세스 로그 실시간 확인
sudo tail -f /var/log/nginx/access.log

# 에러 로그 확인
sudo tail -f /var/log/nginx/error.log

# 특정 상태 코드 필터링
sudo grep " 500 " /var/log/nginx/access.log
sudo grep " 502 " /var/log/nginx/access.log


# ==================== 캐시 관리 ====================

# 캐시 디렉토리 비우기
sudo rm -rf /var/cache/nginx/*
sudo systemctl reload nginx
```

### 예시 6: 모바일 앱에서 Nginx 연동 상황

#### iOS (Swift) - Nginx 프록시 뒤의 API 호출

```swift
import Foundation

// MARK: - Nginx 특성을 고려한 네트워크 클라이언트
class APIClient {
    static let shared = APIClient()

    private let session: URLSession

    private init() {
        let config = URLSessionConfiguration.default

        // Nginx proxy_read_timeout과 일치시키는 것이 중요
        config.timeoutIntervalForRequest = 60  // Nginx 기본값 60초
        config.timeoutIntervalForResource = 300

        // Keep-Alive 활용 (Nginx keepalive_timeout 설정과 연관)
        config.httpShouldUsePipelining = true

        // 캐시 정책 (Nginx Cache-Control 헤더 존중)
        config.requestCachePolicy = .useProtocolCachePolicy

        self.session = URLSession(configuration: config)
    }

    func fetchData<T: Decodable>(from endpoint: String) async throws -> T {
        guard let url = URL(string: "https://api.example.com\(endpoint)") else {
            throw APIError.invalidURL
        }

        var request = URLRequest(url: url)
        request.setValue("application/json", forHTTPHeaderField: "Accept")

        // Nginx 로그 분석에 도움이 되는 커스텀 헤더
        request.setValue("iOS-App/1.0", forHTTPHeaderField: "X-Client-App")
        request.setValue(UIDevice.current.identifierForVendor?.uuidString ?? "unknown",
                         forHTTPHeaderField: "X-Device-ID")

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw APIError.invalidResponse
        }

        // Nginx 캐시 상태 확인 (디버깅용)
        if let cacheStatus = httpResponse.value(forHTTPHeaderField: "X-Cache-Status") {
            print("Nginx Cache: \(cacheStatus)")  // HIT, MISS, EXPIRED 등
        }

        // Nginx Rate Limit 처리
        if httpResponse.statusCode == 429 {
            let retryAfter = httpResponse.value(forHTTPHeaderField: "Retry-After")
            throw APIError.rateLimited(retryAfter: Int(retryAfter ?? "60") ?? 60)
        }

        // Nginx 502/503/504 에러 처리
        switch httpResponse.statusCode {
        case 502:
            throw APIError.badGateway  // 백엔드 서버 문제
        case 503:
            throw APIError.serviceUnavailable  // 백엔드 과부하
        case 504:
            throw APIError.gatewayTimeout  // 백엔드 응답 지연
        case 200..<300:
            return try JSONDecoder().decode(T.self, from: data)
        default:
            throw APIError.httpError(code: httpResponse.statusCode)
        }
    }
}

enum APIError: Error {
    case invalidURL
    case invalidResponse
    case rateLimited(retryAfter: Int)
    case badGateway
    case serviceUnavailable
    case gatewayTimeout
    case httpError(code: Int)
}
```

#### Android (Kotlin) - Nginx 프록시와 OkHttp 최적화

```kotlin
import okhttp3.*
import okhttp3.logging.HttpLoggingInterceptor
import java.util.concurrent.TimeUnit

// Nginx 특성을 고려한 OkHttp 클라이언트 구성
object ApiClient {

    private val client: OkHttpClient by lazy {
        OkHttpClient.Builder()
            // Nginx proxy_read_timeout과 일치
            .connectTimeout(60, TimeUnit.SECONDS)
            .readTimeout(60, TimeUnit.SECONDS)
            .writeTimeout(60, TimeUnit.SECONDS)

            // Keep-Alive 연결 풀 설정 (Nginx keepalive_timeout 고려)
            .connectionPool(ConnectionPool(5, 5, TimeUnit.MINUTES))

            // gzip 자동 처리 (Nginx gzip on과 연동)
            // OkHttp는 기본적으로 Accept-Encoding: gzip을 보내고
            // 응답을 자동 압축 해제함

            // Nginx 커스텀 헤더 추가
            .addInterceptor(NginxHeaderInterceptor())

            // Nginx 에러 처리
            .addInterceptor(NginxErrorInterceptor())

            // 로깅
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.HEADERS
            })
            .build()
    }

    fun getClient(): OkHttpClient = client
}

// Nginx 로그 분석에 유용한 헤더 추가
class NginxHeaderInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()

        val newRequest = originalRequest.newBuilder()
            .header("X-Client-App", "Android-App/1.0")
            .header("X-Device-ID", Settings.Secure.ANDROID_ID)
            .header("X-Request-ID", UUID.randomUUID().toString())  // 추적용
            .build()

        val response = chain.proceed(newRequest)

        // Nginx 캐시 상태 로깅
        response.header("X-Cache-Status")?.let { cacheStatus ->
            Log.d("NginxCache", "Status: $cacheStatus")
        }

        return response
    }
}

// Nginx 관련 에러 처리
class NginxErrorInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val response = chain.proceed(chain.request())

        when (response.code) {
            429 -> {
                // Rate Limit - Nginx limit_req_zone 초과
                val retryAfter = response.header("Retry-After")?.toIntOrNull() ?: 60
                throw RateLimitException(retryAfter)
            }
            502 -> throw BadGatewayException("백엔드 서버 연결 실패")
            503 -> throw ServiceUnavailableException("서비스 일시 중단")
            504 -> throw GatewayTimeoutException("백엔드 응답 시간 초과")
        }

        return response
    }
}

// Retrofit 설정 예시
interface ApiService {
    @GET("/api/v1/products")
    suspend fun getProducts(): Response<List<Product>>
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com")
    .client(ApiClient.getClient())
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

### 예시 7: WebSocket 프록시 설정

```nginx
# /etc/nginx/conf.d/websocket.conf

# 채팅/실시간 알림용 WebSocket 서버
upstream websocket_servers {
    # IP Hash로 세션 유지 (같은 사용자는 같은 서버로)
    ip_hash;

    server 127.0.0.1:8080;
    server 127.0.0.1:8081;

    # 연결 유지 (WebSocket은 장시간 연결)
    keepalive 100;
}

server {
    listen 443 ssl http2;
    server_name ws.example.com;

    # SSL 설정 생략...

    location /ws/ {
        proxy_pass http://websocket_servers;

        # WebSocket 업그레이드 핵심 설정
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 클라이언트 실제 IP 전달
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # WebSocket은 오래 연결되므로 타임아웃 늘림
        proxy_read_timeout 3600s;   # 1시간
        proxy_send_timeout 3600s;

        # 버퍼링 비활성화 (실시간 메시지)
        proxy_buffering off;
    }

    # 일반 HTTP API는 별도 처리
    location /api/ {
        proxy_pass http://api_servers;
        # ... 일반 프록시 설정
    }
}
```

## 4.5 실무에서 자주 겪는 문제와 해결책

### 문제 1: 502 Bad Gateway 에러

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 모바일 앱에서 간헐적으로 502 에러 발생                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인 분석:                                                                   │
│ 1. 백엔드 서버 다운                                                          │
│ 2. upstream 서버 연결 실패                                                   │
│ 3. 백엔드 응답 전에 연결 끊김                                                 │
└─────────────────────────────────────────────────────────────────────────────┘

# 에러 로그 확인
sudo tail -f /var/log/nginx/error.log | grep "502"

# 해결 방법 1: 백엔드 헬스체크 추가
upstream api_servers {
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3003 backup;  # 백업 서버
}

# 해결 방법 2: 연결 타임아웃 조정
location /api/ {
    proxy_connect_timeout 5s;      # 연결 타임아웃 (짧게)
    proxy_read_timeout 60s;        # 읽기 타임아웃
    proxy_next_upstream error timeout http_502 http_503;  # 실패 시 다음 서버로
    proxy_next_upstream_tries 2;   # 최대 2번 재시도
}
```

### 문제 2: 413 Request Entity Too Large

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 이미지/파일 업로드 시 413 에러                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: Nginx 기본 업로드 제한 1MB                                              │
└─────────────────────────────────────────────────────────────────────────────┘

# 해결: client_max_body_size 조정
http {
    client_max_body_size 50M;  # 전역 설정
}

# 또는 특정 location에만 적용
location /api/upload {
    client_max_body_size 100M;     # 업로드 API만 100MB
    proxy_pass http://upload_servers;

    # 대용량 파일을 위한 버퍼 설정
    proxy_request_buffering off;   # 버퍼링 없이 스트리밍
    proxy_read_timeout 300s;       # 업로드 시간 확보
}
```

### 문제 3: 504 Gateway Timeout

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 오래 걸리는 API 호출 시 504 에러 (예: 리포트 생성, 대량 데이터 조회)        │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: proxy_read_timeout 초과 (기본 60초)                                    │
└─────────────────────────────────────────────────────────────────────────────┘

# 해결 방법 1: 타임아웃 늘리기 (비추천 - 임시 해결)
location /api/reports {
    proxy_pass http://api_servers;
    proxy_read_timeout 300s;  # 5분
}

# 해결 방법 2: 비동기 처리 패턴 (권장)
# 요청 → 202 Accepted 즉시 반환 → 폴링으로 결과 확인
location /api/reports {
    proxy_pass http://api_servers;
    proxy_read_timeout 10s;  # 빠른 응답만 기대
}

location /api/reports/status {
    proxy_pass http://api_servers;
    # 폴링용 - 빠른 응답
}
```

### 문제 4: 모바일 앱 CORS 문제

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: WebView나 하이브리드 앱에서 CORS 에러                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: Nginx CORS 헤더 미설정                                                 │
└─────────────────────────────────────────────────────────────────────────────┘

# 해결: CORS 헤더 설정
location /api/ {
    # Preflight 요청 처리
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, X-Requested-With';
        add_header 'Access-Control-Max-Age' 86400;  # 24시간 캐시
        add_header 'Content-Length' 0;
        return 204;
    }

    # 실제 요청에도 CORS 헤더 추가
    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type' always;

    proxy_pass http://api_servers;
}
```

### 문제 5: SSL 인증서 만료

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 갑자기 모든 HTTPS 요청 실패, iOS에서 ATS 에러                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: Let's Encrypt 인증서 갱신 실패 (90일 만료)                               │
└─────────────────────────────────────────────────────────────────────────────┘

# 인증서 상태 확인
sudo certbot certificates

# 수동 갱신 테스트
sudo certbot renew --dry-run

# 자동 갱신 cron 설정 확인
sudo crontab -l

# 권장: systemd timer로 자동 갱신
# /etc/systemd/system/certbot.timer
[Unit]
Description=Run certbot twice daily

[Timer]
OnCalendar=*-*-* 00,12:00:00
RandomizedDelaySec=43200
Persistent=true

[Install]
WantedBy=timers.target
```

### 디버깅 체크리스트

```bash
# 1. Nginx 상태 확인
sudo systemctl status nginx

# 2. 설정 문법 검사
sudo nginx -t

# 3. 에러 로그 실시간 확인
sudo tail -f /var/log/nginx/error.log

# 4. 특정 도메인/경로 요청 필터링
sudo tail -f /var/log/nginx/access.log | grep "api.example.com"

# 5. 백엔드 서버 연결 테스트
curl -v http://127.0.0.1:3001/health

# 6. 포트 사용 확인
sudo netstat -tlnp | grep nginx
sudo lsof -i :80
sudo lsof -i :443

# 7. 프로세스 상태
ps aux | grep nginx

# 8. 연결 수 확인
sudo netstat -an | grep :443 | wc -l

# 9. upstream 연결 상태 (stub_status 모듈 필요)
curl http://localhost/nginx_status
```

## 5. 장단점

### Nginx의 장점

| 장점 | 설명 |
|------|------|
| 높은 성능 | 비동기 이벤트 기반으로 적은 메모리로 많은 연결 처리 |
| 안정성 | 검증된 소프트웨어, 대규모 서비스에서 사용 |
| 유연한 설정 | 다양한 라우팅, 리라이트, 캐싱 규칙 |
| 로드 밸런싱 | 내장 로드 밸런서로 트래픽 분산 |
| 무중단 설정 반영 | reload로 서비스 중단 없이 설정 변경 |
| 풍부한 모듈 | SSL, gzip, rate limit 등 다양한 기능 |

### Nginx의 단점 및 고려사항

| 단점 | 설명 |
|------|------|
| 설정 복잡도 | 처음 배우기 어려울 수 있음 |
| 동적 콘텐츠 | PHP, Python 등은 별도 처리 필요 (FastCGI) |
| 실시간 설정 변경 | 일부 설정은 reload 필요 |
| 디버깅 | 문제 발생 시 원인 파악이 어려울 수 있음 |

### Nginx vs Apache

| 비교 항목 | Nginx | Apache |
|-----------|-------|--------|
| 아키텍처 | 비동기, 이벤트 기반 | 프로세스/스레드 기반 |
| 정적 파일 | 매우 빠름 | 빠름 |
| 동적 콘텐츠 | 외부 처리 필요 | mod_php 등 내장 |
| 동시 연결 | 매우 많은 연결 처리 | 연결당 프로세스 필요 |
| 설정 | 중앙 집중식 | .htaccess 지원 |
| 메모리 사용 | 적음 | 상대적으로 많음 |

## 6. 내 생각

```
(이 공간은 학습 후 자신의 생각을 정리하는 곳입니다)

- Nginx의 역할을 이해한 후, 서버 아키텍처가 어떻게 보이는지:


- 모바일 앱의 API 응답 속도와 Nginx의 관계:


- 앞으로 배포 구성할 때 Nginx를 어떻게 활용할 수 있을지:


```

## 7. 추가 질문

1. **Nginx와 AWS ALB(Application Load Balancer)의 차이점은 무엇인가요?** 언제 각각을 사용하면 좋을까요?

> **답변**: Nginx와 AWS ALB는 모두 로드 밸런싱 기능을 제공하지만 운영 방식과 특성이 크게 다릅니다. Nginx는 직접 서버에 설치하여 관리하는 소프트웨어 로드 밸런서로, 세밀한 설정 제어가 가능하고 온프레미스 환경에서도 사용할 수 있으며, 리버스 프록시, 캐싱, 정적 파일 서빙 등 다양한 기능을 한 곳에서 처리할 수 있습니다. 반면 AWS ALB는 완전 관리형 서비스로, 서버 관리 없이 AWS 콘솔에서 설정만 하면 자동으로 스케일링되고, AWS 서비스(ECS, EKS, Lambda)와 네이티브 통합되며, AWS WAF와 연동한 보안 기능도 쉽게 적용할 수 있습니다. 실무에서는 AWS 환경에서 오토스케일링 그룹과 함께 사용할 때는 ALB가 적합하고, 세밀한 캐싱 전략이나 복잡한 라우팅 규칙이 필요하거나 멀티 클라우드/온프레미스 환경에서는 Nginx가 더 유연합니다. 많은 대규모 서비스에서는 ALB 뒤에 Nginx를 두어 ALB는 트래픽 분산과 SSL 종료를, Nginx는 세부 라우팅과 캐싱을 담당하는 하이브리드 구성을 사용합니다.

2. **WebSocket 연결을 Nginx에서 어떻게 프록시하나요?** 채팅 앱 백엔드를 위한 설정은?

> **답변**: WebSocket을 Nginx에서 프록시하려면 HTTP 1.1의 Connection Upgrade 메커니즘을 명시적으로 설정해야 합니다. 핵심 설정은 `proxy_http_version 1.1;`, `proxy_set_header Upgrade $http_upgrade;`, `proxy_set_header Connection "upgrade";` 세 줄로, 이를 통해 HTTP 연결을 WebSocket으로 업그레이드할 수 있습니다. 채팅 앱의 경우 연결이 오래 유지되므로 `proxy_read_timeout`을 3600초(1시간) 이상으로 설정하고, 실시간 메시지 전달을 위해 `proxy_buffering off;`로 버퍼링을 비활성화해야 합니다. 또한 `ip_hash`를 사용하면 같은 사용자가 항상 같은 백엔드 서버에 연결되어 채팅방 상태를 유지할 수 있습니다. 모바일 앱 관점에서는 네트워크 전환(Wi-Fi ↔ LTE) 시 WebSocket 재연결 로직이 중요한데, 이때 Nginx가 빠르게 연결을 수락하고 백엔드로 전달할 수 있도록 `keepalive` 연결 풀을 설정하는 것이 좋습니다.

3. **Nginx에서 API 버전 관리(v1, v2)를 어떻게 할 수 있나요?** URL 기반 vs 헤더 기반 라우팅

> **답변**: Nginx에서 API 버전 관리는 크게 URL 기반과 헤더 기반 두 가지 방식으로 구현할 수 있습니다. URL 기반(`/api/v1/users`, `/api/v2/users`)은 location 블록으로 쉽게 라우팅할 수 있어 직관적이고, 브라우저에서 직접 테스트하기 쉬우며, CDN 캐싱도 용이합니다. 설정 예시로 `location /api/v1/ { proxy_pass http://api_v1_servers; }`와 `location /api/v2/ { proxy_pass http://api_v2_servers; }`처럼 각 버전을 다른 upstream으로 보낼 수 있습니다. 헤더 기반은 `Accept: application/vnd.api.v2+json`이나 커스텀 헤더 `X-API-Version: 2`를 사용하며, Nginx에서는 `map` 지시어와 조건문으로 구현합니다. 모바일 앱에서는 URL 기반이 더 선호되는데, 앱 버전별로 다른 API 버전을 호출하기 쉽고 디버깅이 간편하기 때문입니다. 실무 팁으로, 새 버전 출시 시 점진적 마이그레이션을 위해 가중치 기반 라우팅(`weight`)으로 트래픽을 천천히 이동시키는 카나리 배포도 Nginx로 쉽게 구현할 수 있습니다.

4. **Nginx 로그를 분석하여 API 성능을 모니터링하는 방법은?** Prometheus, Grafana 연동은 어떻게?

> **답변**: Nginx 로그 기반 성능 모니터링은 log_format에 `$request_time`(전체 처리 시간), `$upstream_response_time`(백엔드 응답 시간), `$status`(HTTP 상태 코드)를 포함시키는 것부터 시작합니다. Prometheus 연동은 nginx-prometheus-exporter를 사용하거나, 더 상세한 메트릭이 필요하면 nginx-vts-module을 설치하여 가상 호스트별, location별 통계를 수집할 수 있습니다. Grafana 대시보드에서는 p99 응답 시간, 초당 요청 수(RPS), 에러율(4xx/5xx 비율), upstream 서버별 상태를 시각화하여 한눈에 서비스 상태를 파악할 수 있습니다. 모바일 앱 팀에게 유용한 메트릭으로는 API 엔드포인트별 평균 응답 시간, 지역별 응답 시간 차이(CDN 효과 측정), 피크 시간대 에러율 등이 있습니다. 실시간 알림을 위해 Grafana Alert나 Prometheus Alertmanager를 설정하면 에러율 급증이나 응답 시간 저하 시 즉시 알림을 받을 수 있어, 모바일 사용자에게 영향이 가기 전에 대응할 수 있습니다.

5. **Nginx에서 CORS(Cross-Origin Resource Sharing) 헤더를 어떻게 설정하나요?** 모바일 웹뷰에서 필요한 설정은?

> **답변**: CORS 설정은 Preflight 요청(OPTIONS)과 실제 요청 두 가지를 모두 처리해야 합니다. OPTIONS 요청에는 `add_header 'Access-Control-Allow-Origin' '*'`, `add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS'`, `add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type'`를 설정하고 `return 204;`로 빈 응답을 반환합니다. 실제 요청에도 같은 헤더를 `always` 플래그와 함께 추가해야 에러 응답에서도 CORS 헤더가 포함됩니다. 모바일 웹뷰에서는 특히 주의할 점이 있는데, iOS WKWebView와 Android WebView는 각각 약간 다른 CORS 동작을 보이므로 `Access-Control-Allow-Credentials: true`가 필요할 때는 `Allow-Origin`에 와일드카드(`*`) 대신 구체적인 도메인을 명시해야 합니다. 또한 모바일 앱의 로컬 파일(`file://`)에서 요청하는 경우 특별한 처리가 필요할 수 있어, 하이브리드 앱에서는 네이티브 브릿지를 통한 API 호출이 더 안정적인 경우가 많습니다.

6. **Let's Encrypt를 사용한 무료 SSL 인증서 자동 갱신은 어떻게 설정하나요?** certbot과 Nginx 연동 방법은?

> **답변**: Let's Encrypt 인증서는 certbot 도구로 쉽게 발급받을 수 있으며, `sudo certbot --nginx -d example.com -d www.example.com` 명령으로 Nginx 설정까지 자동으로 수정됩니다. certbot은 `.well-known/acme-challenge/` 경로를 통한 도메인 소유권 검증을 수행하므로, Nginx에서 해당 경로가 접근 가능해야 합니다. 인증서는 90일마다 만료되므로 자동 갱신이 필수인데, `sudo certbot renew --dry-run`으로 갱신 테스트를 하고, systemd timer나 cron으로 `certbot renew && systemctl reload nginx`를 하루 2회 실행하도록 설정합니다. 갱신 후 Nginx reload가 자동으로 되도록 `--deploy-hook "systemctl reload nginx"` 옵션을 사용하면 좋습니다. 모바일 앱 관점에서 인증서 갱신이 중요한 이유는, 인증서가 만료되면 iOS ATS와 Android Network Security Config에 의해 모든 HTTPS 요청이 차단되어 앱이 완전히 동작하지 않게 되기 때문입니다. 따라서 인증서 만료 14일 전 알림을 설정하고, 갱신 실패 시 즉시 알림을 받을 수 있는 모니터링을 구축하는 것이 중요합니다.
