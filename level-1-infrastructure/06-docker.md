# Docker

## 1. 한 줄 요약

**Docker는 애플리케이션과 그 실행 환경(라이브러리, 설정, 의존성)을 컨테이너라는 표준화된 단위로 패키징하여, 개발/테스트/프로덕션 어느 환경에서든 동일하게 실행할 수 있게 해주는 컨테이너 플랫폼입니다.**

## 2. 쉽게 설명

### 모바일 개발자 관점에서

"제 컴퓨터에서는 되는데요?" - 이 문제를 해결하는 것이 Docker입니다. 모바일 개발자가 백엔드 API를 로컬에서 테스트하고 싶을 때, Docker를 사용하면 운영 서버와 동일한 환경을 단 한 줄의 명령어로 구축할 수 있습니다.

모바일 앱의 백엔드 서버를 로컬에서 테스트하고 싶을 때, Docker를 사용하면 서버 환경을 그대로 가져와서 실행할 수 있습니다. 더 이상 "Node.js 버전이 맞지 않아요", "MySQL 설치가 안 돼요", "환경변수 설정이 다릅니다" 같은 문제로 고민하지 않아도 됩니다.

**이사에 비유하면:**
- **기존 방식 (VM)**: 집 전체를 옮기기 (땅, 건물 기초, 모든 가구 포함)
- **Docker 방식**: 이삿짐 컨테이너에 필요한 짐만 담아 옮기기 (어디서든 언팩하면 동일한 환경)

**앱 번들에 비유하면:**
- iOS의 `.ipa` 파일처럼, Docker 이미지는 앱 실행에 필요한 모든 것을 담은 패키지입니다.
- 어느 기기(서버)에서든 동일하게 실행됩니다.
- 버전 관리도 가능: `myapp:1.0`, `myapp:2.0` 처럼 태그로 관리합니다.

**모바일 개발에서 Docker가 유용한 상황:**
- 백엔드 API를 로컬에서 실행하여 오프라인 테스트
- 새로 합류한 팀원에게 개발 환경 공유
- 다양한 백엔드 버전을 쉽게 전환하며 테스트
- CI/CD에서 일관된 테스트 환경 구축

### 핵심 개념

| 개념 | 설명 | 비유 |
|------|------|------|
| **Image (이미지)** | 컨테이너 실행에 필요한 패키지 | 앱 설치 파일 (.ipa, .apk) |
| **Container (컨테이너)** | 이미지를 실행한 인스턴스 | 실행 중인 앱 |
| **Dockerfile** | 이미지 빌드 설명서 | Xcode 빌드 설정 |
| **Registry** | 이미지 저장소 | App Store |
| **Volume** | 데이터 영구 저장소 | 앱 Documents 폴더 |
| **Network** | 컨테이너 간 통신 | 앱 간 URL Scheme |

### Docker vs 가상머신 (VM)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    가상 머신 (VM)                                   │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │    App A     │  │    App B     │  │    App C     │              │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤              │
│  │   Bins/Libs  │  │   Bins/Libs  │  │   Bins/Libs  │              │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤              │
│  │  Guest OS    │  │  Guest OS    │  │  Guest OS    │  ← 각각 OS   │
│  │  (Ubuntu)    │  │  (CentOS)    │  │  (Debian)    │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
├─────────────────────────────────────────────────────────────────────┤
│                     Hypervisor (VMware, VirtualBox)                 │
├─────────────────────────────────────────────────────────────────────┤
│                     Host OS (Windows, macOS)                        │
├─────────────────────────────────────────────────────────────────────┤
│                     Hardware                                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    Docker (Container)                               │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │    App A     │  │    App B     │  │    App C     │              │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤              │
│  │   Bins/Libs  │  │   Bins/Libs  │  │   Bins/Libs  │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
├─────────────────────────────────────────────────────────────────────┤
│                     Docker Engine                                   │
├─────────────────────────────────────────────────────────────────────┤
│                     Host OS (단일 OS 공유)                          │ ← OS 하나만!
├─────────────────────────────────────────────────────────────────────┤
│                     Hardware                                        │
└─────────────────────────────────────────────────────────────────────┘
```

## 3. 구조 다이어그램

### Docker 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Docker 아키텍처                            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────┐                    ┌─────────────────────────────┐
│   Docker CLI    │                    │      Docker Hub             │
│  (사용자 명령)   │                    │   (이미지 저장소)            │
│                 │                    │                             │
│ docker run      │                    │ ┌─────────────────────────┐ │
│ docker build    │                    │ │  nginx:latest           │ │
│ docker pull     │                    │ │  node:18                │ │
│ docker push     │                    │ │  mysql:8                │ │
└────────┬────────┘                    │ │  redis:alpine           │ │
         │                             │ └─────────────────────────┘ │
         │ REST API                    └──────────────┬──────────────┘
         ▼                                            │
┌─────────────────┐                                   │ push/pull
│  Docker Daemon  │<──────────────────────────────────┘
│   (dockerd)     │
│                 │
│ ┌─────────────┐ │
│ │   Images    │ │  이미지 관리
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │ Containers  │ │  컨테이너 실행/관리
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │  Networks   │ │  네트워크 관리
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │   Volumes   │ │  데이터 저장
│ └─────────────┘ │
└─────────────────┘
```

### Docker 이미지 레이어

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Docker Image Layers                            │
└─────────────────────────────────────────────────────────────────────┘

            Dockerfile                          Image Layers
┌───────────────────────────────┐      ┌───────────────────────────┐
│ FROM node:18-alpine           │ ───> │ Layer 1: node:18-alpine   │ (공유됨)
├───────────────────────────────┤      ├───────────────────────────┤
│ WORKDIR /app                  │ ───> │ Layer 2: /app 디렉토리    │
├───────────────────────────────┤      ├───────────────────────────┤
│ COPY package*.json ./         │ ───> │ Layer 3: package.json     │
├───────────────────────────────┤      ├───────────────────────────┤
│ RUN npm install               │ ───> │ Layer 4: node_modules     │ (캐시됨)
├───────────────────────────────┤      ├───────────────────────────┤
│ COPY . .                      │ ───> │ Layer 5: 소스 코드        │
├───────────────────────────────┤      ├───────────────────────────┤
│ CMD ["node", "server.js"]     │ ───> │ Layer 6: 실행 명령        │
└───────────────────────────────┘      └───────────────────────────┘

※ 이전 레이어가 변경되지 않으면 캐시 사용 → 빠른 빌드
```

### 컨테이너 네트워크

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Docker Network Types                             │
└─────────────────────────────────────────────────────────────────────┘

1. Bridge Network (기본값)
   ┌─────────────────────────────────────────────────────────────────┐
   │                     Docker Host                                 │
   │  ┌─────────────────────────────────────────────────────────────┐│
   │  │               docker0 (bridge)                              ││
   │  │                 172.17.0.1                                  ││
   │  └─────────────────┬───────────────────┬───────────────────────┘│
   │                    │                   │                        │
   │  ┌─────────────────┴────┐  ┌──────────┴─────────────┐          │
   │  │   Container A        │  │   Container B          │          │
   │  │   172.17.0.2         │  │   172.17.0.3           │          │
   │  │   (api-server)       │  │   (database)           │          │
   │  └──────────────────────┘  └────────────────────────┘          │
   └─────────────────────────────────────────────────────────────────┘

2. Host Network
   컨테이너가 Host의 네트워크를 직접 사용 (격리 없음)
   - 장점: 최고의 네트워크 성능
   - 단점: 포트 충돌 가능, 보안 격리 없음

3. Custom Bridge Network (권장)
   ┌─────────────────────────────────────────────────────────────────┐
   │  my-network                                                     │
   │  ┌─────────────────────┐   ┌─────────────────────┐             │
   │  │   api               │   │   db                │             │
   │  │   (컨테이너 이름으로 │   │   (DNS 자동 등록)    │             │
   │  │    통신 가능)        │──>│                     │             │
   │  │   curl http://db:5432│   │                     │             │
   │  └─────────────────────┘   └─────────────────────┘             │
   └─────────────────────────────────────────────────────────────────┘
   - 장점: 컨테이너 이름으로 DNS 조회, 격리된 네트워크
   - Docker Compose 사용 시 자동 생성
```

### 컨테이너 라이프사이클

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       Container Lifecycle                                   │
└─────────────────────────────────────────────────────────────────────────────┘

                    docker create
    ┌─────────┐ ─────────────────▶ ┌─────────┐
    │  Image  │                    │ Created │
    └─────────┘                    └────┬────┘
                                        │ docker start
                                        ▼
                                   ┌─────────┐
                           ┌───▶  │ Running │ ◀───┐
                           │      └────┬────┘     │
                           │           │          │
              docker restart           │          │ docker start
                           │           │          │
                           │           ▼          │
                           │      ┌─────────┐     │
                           └───── │ Stopped │ ────┘
                                  └────┬────┘
                                       │ docker rm
                                       ▼
                                  ┌─────────┐
                                  │ Removed │
                                  └─────────┘

docker run = docker create + docker start (한 번에 실행)

상태별 명령어:
- Running → Stopped: docker stop <container>
- Running → Paused:  docker pause <container>
- Stopped → Removed: docker rm <container>
- Running → Removed: docker rm -f <container> (강제)
```

## 4. 실무 적용 예시

### 예시 1: 기본 Docker 명령어

```bash
# ==================== 이미지 관리 ====================

# 이미지 다운로드
docker pull nginx:latest
docker pull node:18-alpine

# 이미지 목록 확인
docker images
# REPOSITORY   TAG         IMAGE ID       SIZE
# nginx        latest      605c77e624dd   142MB
# node         18-alpine   a0b787b0d53e   175MB

# 이미지 삭제
docker rmi nginx:latest
docker image prune          # 사용하지 않는 이미지 정리


# ==================== 컨테이너 실행 ====================

# 기본 실행
docker run nginx

# 백그라운드 실행 (-d: detached)
docker run -d nginx

# 이름 지정하여 실행
docker run -d --name my-nginx nginx

# 포트 매핑 (-p host:container)
docker run -d -p 8080:80 --name my-nginx nginx
# 이제 http://localhost:8080 으로 접속 가능

# 환경변수 설정
docker run -d \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  --name my-mysql \
  mysql:8

# 볼륨 마운트 (-v host:container)
docker run -d \
  -p 3306:3306 \
  -v /my/local/data:/var/lib/mysql \
  --name my-mysql \
  mysql:8


# ==================== 컨테이너 관리 ====================

# 실행 중인 컨테이너 확인
docker ps
# CONTAINER ID   IMAGE   STATUS         PORTS                  NAMES
# abc123         nginx   Up 2 minutes   0.0.0.0:8080->80/tcp   my-nginx

# 모든 컨테이너 확인 (중지된 것 포함)
docker ps -a

# 컨테이너 중지/시작/재시작
docker stop my-nginx
docker start my-nginx
docker restart my-nginx

# 컨테이너 삭제
docker rm my-nginx          # 중지된 컨테이너만 삭제 가능
docker rm -f my-nginx       # 실행 중이어도 강제 삭제

# 모든 중지된 컨테이너 삭제
docker container prune


# ==================== 컨테이너 디버깅 ====================

# 로그 확인
docker logs my-nginx
docker logs -f my-nginx     # 실시간 로그 (tail -f처럼)
docker logs --tail 100 my-nginx  # 최근 100줄

# 컨테이너 내부 접속
docker exec -it my-nginx /bin/bash
docker exec -it my-nginx sh  # bash 없는 이미지용

# 컨테이너 상태 확인
docker inspect my-nginx
docker stats my-nginx       # CPU/메모리 사용량
```

### 예시 2: Dockerfile 작성

```dockerfile
# Dockerfile - Node.js API 서버

# 베이스 이미지 (경량 Alpine Linux 사용)
FROM node:18-alpine

# 작업 디렉토리 설정
WORKDIR /app

# package.json 먼저 복사 (캐싱 최적화)
COPY package*.json ./

# 의존성 설치 (프로덕션만)
RUN npm ci --only=production

# 소스 코드 복사
COPY . .

# 앱이 사용하는 포트
EXPOSE 3000

# 비-root 사용자로 실행 (보안)
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# 환경변수 기본값
ENV NODE_ENV=production

# 헬스체크
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# 실행 명령
CMD ["node", "server.js"]
```

```bash
# 이미지 빌드
docker build -t my-api:1.0.0 .

# 빌드 + 태깅
docker build -t myregistry/my-api:latest -t myregistry/my-api:1.0.0 .

# 빌드 캐시 없이 (문제 해결 시)
docker build --no-cache -t my-api:1.0.0 .
```

### 예시 3: Docker Compose (다중 컨테이너)

```yaml
# docker-compose.yml
# 로컬 개발 환경: API 서버 + DB + Redis

version: '3.8'

services:
  # API 서버
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./api:/app              # 소스 코드 마운트 (개발 시 핫리로드)
      - /app/node_modules       # node_modules는 제외
    depends_on:
      - db
      - redis
    networks:
      - app-network

  # PostgreSQL 데이터베이스
  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  # Redis 캐시
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  # Nginx 리버스 프록시
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - app-network

# 볼륨 정의 (데이터 영구 저장)
volumes:
  postgres_data:
  redis_data:

# 네트워크 정의
networks:
  app-network:
    driver: bridge
```

```bash
# Docker Compose 명령어

# 모든 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f api

# 특정 서비스만 재시작
docker-compose restart api

# 서비스 스케일 아웃
docker-compose up -d --scale api=3

# 모든 서비스 중지 및 삭제
docker-compose down

# 볼륨까지 삭제 (데이터 초기화)
docker-compose down -v
```

### 예시 4: 모바일 개발자를 위한 로컬 백엔드 환경

```yaml
# docker-compose.local.yml
# 모바일 앱 테스트를 위한 로컬 백엔드

version: '3.8'

services:
  # 백엔드 API
  api:
    image: company/backend-api:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=local
      - DATABASE_URL=jdbc:postgresql://db:5432/myapp
    depends_on:
      - db

  # 데이터베이스
  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=app
      - POSTGRES_PASSWORD=localpass
      - POSTGRES_DB=myapp
    volumes:
      - ./seed-data.sql:/docker-entrypoint-initdb.d/seed.sql  # 초기 데이터

  # 모의 결제 서버
  mock-payment:
    image: mockserver/mockserver:latest
    ports:
      - "1080:1080"
    environment:
      - MOCKSERVER_INITIALIZATION_JSON_PATH=/config/expectations.json
    volumes:
      - ./mocks/payment.json:/config/expectations.json

  # 로컬 S3 (이미지 업로드 테스트)
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3
      - DEFAULT_REGION=ap-northeast-2
```

```bash
# 모바일 개발자 사용법

# 1. 백엔드 환경 시작
docker-compose -f docker-compose.local.yml up -d

# 2. 상태 확인
docker-compose -f docker-compose.local.yml ps

# 3. 앱에서 http://localhost:8080/api 로 요청

# 4. 로그 확인 (API 디버깅)
docker-compose -f docker-compose.local.yml logs -f api

# 5. 환경 종료
docker-compose -f docker-compose.local.yml down
```

### 예시 5: 유용한 Docker 팁

```bash
# ==================== 디스크 정리 ====================

# 사용하지 않는 모든 리소스 정리
docker system prune -a

# 볼륨까지 포함하여 정리
docker system prune -a --volumes

# 현재 Docker 디스크 사용량 확인
docker system df


# ==================== 이미지 최적화 ====================

# 멀티스테이지 빌드 (작은 이미지 생성)
# Dockerfile.multistage

# Stage 1: 빌드
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: 실행 (최종 이미지)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]


# ==================== 보안 ====================

# 컨테이너 취약점 스캔
docker scan my-image:latest

# 비-root 사용자 확인
docker run --rm my-image:latest whoami
```

### 예시 6: 모바일 앱에서 로컬 Docker 환경 연동

#### iOS (Swift) - 로컬 Docker 백엔드 연결

```swift
import Foundation

// MARK: - 환경별 API 설정
enum APIEnvironment {
    case local          // Docker 로컬 환경
    case development    // 개발 서버
    case staging        // 스테이징 서버
    case production     // 프로덕션 서버

    var baseURL: String {
        switch self {
        case .local:
            // Docker로 실행 중인 로컬 백엔드
            // 시뮬레이터: localhost 사용 가능
            // 실제 기기: Mac의 IP 주소 사용 (예: 192.168.1.100)
            #if targetEnvironment(simulator)
            return "http://localhost:8080"
            #else
            return "http://192.168.1.100:8080"  // Mac IP
            #endif
        case .development:
            return "https://dev-api.example.com"
        case .staging:
            return "https://staging-api.example.com"
        case .production:
            return "https://api.example.com"
        }
    }

    var isSecure: Bool {
        self != .local
    }
}

// MARK: - 로컬 환경용 네트워크 설정
class LocalDockerAPIClient {
    static let shared = LocalDockerAPIClient()

    private var currentEnvironment: APIEnvironment = .local

    private lazy var session: URLSession = {
        let config = URLSessionConfiguration.default

        // 로컬 Docker 환경은 타임아웃을 길게 (컨테이너 시작 시간 고려)
        if currentEnvironment == .local {
            config.timeoutIntervalForRequest = 30
        }

        return URLSession(configuration: config)
    }()

    func switchEnvironment(to env: APIEnvironment) {
        self.currentEnvironment = env
        print("API Environment switched to: \(env)")
    }

    func checkDockerBackendHealth() async throws -> Bool {
        guard currentEnvironment == .local else { return true }

        let healthURL = URL(string: "\(currentEnvironment.baseURL)/health")!

        do {
            let (_, response) = try await session.data(from: healthURL)
            return (response as? HTTPURLResponse)?.statusCode == 200
        } catch {
            print("Docker backend not ready: \(error.localizedDescription)")
            print("Did you run: docker-compose up -d ?")
            throw DockerError.backendNotRunning
        }
    }
}

enum DockerError: Error, LocalizedError {
    case backendNotRunning
    case containerNotFound

    var errorDescription: String? {
        switch self {
        case .backendNotRunning:
            return "로컬 Docker 백엔드가 실행 중이 아닙니다. 'docker-compose up -d'를 실행하세요."
        case .containerNotFound:
            return "필요한 Docker 컨테이너를 찾을 수 없습니다."
        }
    }
}

// MARK: - Info.plist 설정 (로컬 HTTP 허용)
/*
 로컬 Docker 환경 테스트를 위해 Info.plist에 추가:

 <key>NSAppTransportSecurity</key>
 <dict>
     <key>NSAllowsLocalNetworking</key>
     <true/>
 </dict>

 또는 특정 IP만 허용:
 <key>NSExceptionDomains</key>
 <dict>
     <key>localhost</key>
     <dict>
         <key>NSExceptionAllowsInsecureHTTPLoads</key>
         <true/>
     </dict>
 </dict>
*/
```

#### Android (Kotlin) - 로컬 Docker 백엔드 연결

```kotlin
import android.os.Build
import okhttp3.OkHttpClient
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

// 환경별 설정
enum class ApiEnvironment(val baseUrl: String, val isSecure: Boolean) {
    LOCAL(
        // Android 에뮬레이터에서 host machine 접근
        // 10.0.2.2는 에뮬레이터에서 호스트 머신을 가리킴
        baseUrl = if (isEmulator()) "http://10.0.2.2:8080" else "http://192.168.1.100:8080",
        isSecure = false
    ),
    DEVELOPMENT("https://dev-api.example.com", true),
    STAGING("https://staging-api.example.com", true),
    PRODUCTION("https://api.example.com", true);

    companion object {
        fun isEmulator(): Boolean {
            return (Build.FINGERPRINT.startsWith("generic")
                    || Build.FINGERPRINT.startsWith("unknown")
                    || Build.MODEL.contains("google_sdk")
                    || Build.MODEL.contains("Emulator")
                    || Build.MODEL.contains("Android SDK"))
        }
    }
}

// 로컬 Docker 환경용 API 클라이언트
object LocalDockerApiClient {

    private var currentEnvironment = ApiEnvironment.LOCAL

    private val client: OkHttpClient by lazy {
        OkHttpClient.Builder()
            // 로컬 Docker 환경은 시작 시간 고려하여 타임아웃 길게
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .addInterceptor { chain ->
                val request = chain.request()
                try {
                    chain.proceed(request)
                } catch (e: Exception) {
                    if (currentEnvironment == ApiEnvironment.LOCAL) {
                        throw DockerConnectionException(
                            "로컬 Docker 백엔드에 연결할 수 없습니다.\n" +
                            "확인사항:\n" +
                            "1. docker-compose up -d 실행 여부\n" +
                            "2. docker-compose ps로 컨테이너 상태 확인\n" +
                            "3. 에뮬레이터 사용 시 10.0.2.2 주소 사용"
                        )
                    }
                    throw e
                }
            }
            .build()
    }

    val retrofit: Retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(currentEnvironment.baseUrl)
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    suspend fun checkDockerHealth(): Boolean {
        return try {
            val response = client.newCall(
                okhttp3.Request.Builder()
                    .url("${currentEnvironment.baseUrl}/health")
                    .build()
            ).execute()
            response.isSuccessful
        } catch (e: Exception) {
            false
        }
    }

    fun switchEnvironment(env: ApiEnvironment) {
        currentEnvironment = env
    }
}

class DockerConnectionException(message: String) : Exception(message)

// res/xml/network_security_config.xml (로컬 HTTP 허용)
/*
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- 프로덕션: HTTPS만 허용 -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>

    <!-- 로컬 개발: HTTP 허용 -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
        <domain includeSubdomains="true">localhost</domain>
        <domain includeSubdomains="true">192.168.1.100</domain>
    </domain-config>
</network-security-config>

AndroidManifest.xml에 추가:
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
*/
```

### 예시 7: CI/CD용 Docker 빌드 스크립트

```bash
#!/bin/bash
# build-and-push.sh - CI/CD 파이프라인용

set -e  # 에러 발생 시 즉시 종료

# 변수 설정
REGISTRY="ghcr.io"
IMAGE_NAME="mycompany/backend-api"
VERSION=${GITHUB_SHA:-$(git rev-parse --short HEAD)}
BRANCH=${GITHUB_REF_NAME:-$(git branch --show-current)}

echo "Building Docker image..."
echo "Registry: $REGISTRY"
echo "Image: $IMAGE_NAME"
echo "Version: $VERSION"
echo "Branch: $BRANCH"

# 태그 설정
if [ "$BRANCH" = "main" ]; then
    TAGS="-t $REGISTRY/$IMAGE_NAME:latest -t $REGISTRY/$IMAGE_NAME:$VERSION"
elif [ "$BRANCH" = "develop" ]; then
    TAGS="-t $REGISTRY/$IMAGE_NAME:develop -t $REGISTRY/$IMAGE_NAME:dev-$VERSION"
else
    TAGS="-t $REGISTRY/$IMAGE_NAME:$BRANCH-$VERSION"
fi

# 멀티 플랫폼 빌드 (ARM + AMD64)
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --push \
    $TAGS \
    --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") \
    --build-arg VERSION=$VERSION \
    --build-arg GIT_COMMIT=$VERSION \
    .

echo "Successfully built and pushed: $REGISTRY/$IMAGE_NAME:$VERSION"
```

## 4.5 실무에서 자주 겪는 문제와 해결책

### 문제 1: 포트 충돌 (Port Already in Use)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: docker: Error response from daemon: Ports are not available:          │
│       listen tcp 0.0.0.0:8080: bind: address already in use                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: 호스트의 8080 포트가 이미 다른 프로세스에서 사용 중                        │
└─────────────────────────────────────────────────────────────────────────────┘

# 해결 방법 1: 사용 중인 포트 확인
lsof -i :8080
# COMMAND   PID USER   FD   TYPE    SIZE/OFF NODE NAME
# node    12345 user   23u  IPv4    0x...    TCP *:8080 (LISTEN)

# 해결 방법 2: 다른 포트 사용
docker run -d -p 8081:8080 my-api  # 호스트 8081 → 컨테이너 8080

# 해결 방법 3: 충돌하는 프로세스 종료
kill 12345

# 해결 방법 4: 중지된 Docker 컨테이너가 포트 점유 중인지 확인
docker ps -a | grep 8080
docker rm $(docker ps -a -q --filter "status=exited")
```

### 문제 2: 컨테이너가 즉시 종료됨

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: docker run 실행 후 컨테이너가 바로 Exited 상태                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: 포그라운드 프로세스가 없거나, 앱이 시작 직후 크래시                         │
└─────────────────────────────────────────────────────────────────────────────┘

# 디버깅 방법 1: 로그 확인
docker logs <container_id>

# 디버깅 방법 2: 컨테이너 내부 쉘 접속
docker run -it --entrypoint /bin/sh my-image

# 디버깅 방법 3: 종료된 컨테이너 상태 확인
docker inspect <container_id> --format='{{.State.ExitCode}}'
docker inspect <container_id> --format='{{.State.Error}}'

# 흔한 원인과 해결책:
# 1. CMD가 백그라운드 실행으로 되어 있음
#    잘못: CMD ["npm", "start", "&"]
#    올바름: CMD ["npm", "start"]

# 2. 환경변수 누락으로 앱 시작 실패
docker run -e DATABASE_URL=... -e API_KEY=... my-image

# 3. 헬스체크 실패로 컨테이너 재시작 반복
docker inspect <container_id> --format='{{.State.Health}}'
```

### 문제 3: 볼륨 권한 문제 (Permission Denied)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 볼륨 마운트된 디렉토리에서 파일 읽기/쓰기 실패                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: 컨테이너 내부 사용자 UID와 호스트 파일 소유자 UID 불일치                    │
└─────────────────────────────────────────────────────────────────────────────┘

# 문제 상황
docker run -v /my/data:/app/data my-image
# Error: EACCES: permission denied, open '/app/data/file.txt'

# 해결 방법 1: 호스트 디렉토리 권한 변경
chmod -R 777 /my/data  # (주의: 보안 위험)

# 해결 방법 2: 컨테이너를 특정 사용자로 실행
docker run -u $(id -u):$(id -g) -v /my/data:/app/data my-image

# 해결 방법 3: Dockerfile에서 동일한 UID 사용
# Dockerfile
ARG UID=1000
RUN useradd -u $UID -m appuser
USER appuser

# 빌드 시 UID 전달
docker build --build-arg UID=$(id -u) -t my-image .

# 해결 방법 4: Docker Compose에서 user 지정
services:
  app:
    image: my-image
    user: "1000:1000"
    volumes:
      - ./data:/app/data
```

### 문제 4: 이미지 빌드 캐시 무효화

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: 작은 코드 변경에도 npm install이 매번 실행되어 빌드가 오래 걸림             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: Dockerfile 레이어 순서 최적화 미흡                                       │
└─────────────────────────────────────────────────────────────────────────────┘

# 잘못된 Dockerfile (매번 npm install 실행)
FROM node:18
WORKDIR /app
COPY . .                    # 코드 변경 시 여기서 캐시 무효화
RUN npm install             # 매번 재실행됨
CMD ["npm", "start"]

# 올바른 Dockerfile (의존성 캐싱)
FROM node:18
WORKDIR /app
COPY package*.json ./       # 의존성 파일만 먼저 복사
RUN npm ci                  # 의존성 설치 (package.json 변경 시만 재실행)
COPY . .                    # 소스 코드 복사
CMD ["npm", "start"]

# 결과:
# - package.json 변경 없으면 npm ci 스킵 → 빌드 시간 대폭 단축
# - 레이어 캐싱 원리: 변경된 레이어 이후 모든 레이어 재빌드
```

### 문제 5: macOS에서 Docker 성능 저하

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 증상: macOS에서 Docker 볼륨 마운트 시 파일 I/O가 매우 느림                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ 원인: macOS의 파일시스템과 Linux 컨테이너 간 동기화 오버헤드                     │
└─────────────────────────────────────────────────────────────────────────────┘

# 해결 방법 1: 볼륨 마운트 옵션 사용
docker run -v /my/code:/app:delegated my-image  # 쓰기 지연 허용
docker run -v /my/code:/app:cached my-image     # 읽기 캐시 사용

# 해결 방법 2: node_modules 제외 (가장 효과적)
# docker-compose.yml
volumes:
  - ./:/app
  - /app/node_modules       # node_modules는 컨테이너 내부에만 존재

# 해결 방법 3: 소스 코드만 마운트
volumes:
  - ./src:/app/src          # 소스 코드 디렉토리만 마운트
  - ./package.json:/app/package.json:ro

# 해결 방법 4: Docker Desktop 설정 (VirtioFS 사용)
# Docker Desktop → Settings → General → VirtioFS 활성화

# 해결 방법 5: Mutagen 사용 (고급)
# 파일 동기화 도구로 성능 개선
```

### 디버깅 체크리스트

```bash
# ==================== 컨테이너 상태 확인 ====================

# 1. 실행 중인 컨테이너 확인
docker ps

# 2. 모든 컨테이너 확인 (중지된 것 포함)
docker ps -a

# 3. 컨테이너 로그 확인
docker logs <container_id>
docker logs -f --tail 100 <container_id>  # 실시간, 최근 100줄

# 4. 컨테이너 상세 정보
docker inspect <container_id>

# 5. 컨테이너 리소스 사용량
docker stats <container_id>


# ==================== 네트워크 디버깅 ====================

# 6. 컨테이너 내부에서 네트워크 테스트
docker exec -it <container_id> sh
# 내부에서:
curl http://other-container:port/health
ping other-container
nslookup other-container

# 7. 네트워크 목록 및 상세 정보
docker network ls
docker network inspect <network_name>

# 8. 포트 매핑 확인
docker port <container_id>


# ==================== 이미지 디버깅 ====================

# 9. 이미지 레이어 확인
docker history <image_name>

# 10. 이미지 상세 정보
docker inspect <image_name>

# 11. 이미지 크기 분석
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}"


# ==================== 시스템 정리 ====================

# 12. 디스크 사용량 확인
docker system df

# 13. 미사용 리소스 정리
docker system prune         # 컨테이너, 네트워크, 이미지
docker system prune -a      # 모든 미사용 이미지 포함
docker volume prune         # 미사용 볼륨
```

## 5. 장단점

### Docker의 장점

| 장점 | 설명 |
|------|------|
| 환경 일관성 | 개발, 테스트, 프로덕션 환경 동일 |
| 빠른 배포 | 이미지 pull 후 바로 실행 |
| 격리성 | 앱 간 독립적인 실행 환경 |
| 버전 관리 | 이미지 태그로 버전 관리 |
| 자원 효율성 | VM보다 가볍고 빠름 |
| 생태계 | Docker Hub에 다양한 이미지 |

### Docker의 단점 및 고려사항

| 단점 | 설명 |
|------|------|
| 학습 비용 | Dockerfile, Compose 등 새로운 개념 |
| 네트워크 복잡성 | 컨테이너 간 통신 설정 |
| 데이터 관리 | 볼륨 관리에 주의 필요 |
| 디버깅 어려움 | 컨테이너 내부 문제 파악 |
| macOS 성능 | Linux보다 느림 (가상화 오버헤드) |

### 언제 Docker를 사용하면 좋은가?

| 상황 | Docker 사용 추천 |
|------|------------------|
| 로컬에서 백엔드 테스트 | 매우 추천 |
| 팀원과 환경 공유 | 매우 추천 |
| CI/CD 파이프라인 | 매우 추천 |
| 마이크로서비스 운영 | 추천 |
| 단순 스크립트 실행 | 비추천 (오버엔지니어링) |

## 6. 내 생각

```
(이 공간은 학습 후 자신의 생각을 정리하는 곳입니다)

- Docker를 사용하면서 개발 워크플로우가 어떻게 바뀔 수 있을지:


- 로컬에서 백엔드 환경을 구축할 때 Docker가 어떻게 도움이 될지:


- 앞으로 Docker를 사용해보고 싶은 상황:


```

## 7. 추가 질문

1. **Kubernetes는 Docker와 어떻게 다른가요?** 언제 Kubernetes가 필요한가요?

> **답변**: Docker는 개별 컨테이너를 생성하고 실행하는 "컨테이너 런타임"이고, Kubernetes(K8s)는 수십~수천 개의 컨테이너를 여러 서버에 배포하고 관리하는 "컨테이너 오케스트레이션" 플랫폼입니다. Docker가 "컨테이너를 만드는 도구"라면, Kubernetes는 "컨테이너들을 지휘하는 지휘자"에 비유할 수 있습니다. Kubernetes가 필요한 시점은 서비스가 성장하여 단일 서버로 감당하기 어려워지거나, 자동 스케일링/롤링 업데이트/자가 치유(self-healing) 같은 기능이 필요할 때입니다. 예를 들어 모바일 앱 백엔드가 트래픽 급증 시 자동으로 Pod를 늘리고, 배포 실패 시 자동 롤백하는 기능이 필요하다면 Kubernetes를 도입할 시점입니다. 반면 팀이 작고 서버가 몇 대 이하라면 Docker Compose나 Docker Swarm으로 충분하며, Kubernetes의 학습 비용과 운영 복잡도가 오히려 부담이 될 수 있습니다.

2. **Docker 이미지를 최적화하여 크기를 줄이는 방법은?** Alpine Linux, 멀티스테이지 빌드 외에 다른 방법은?

> **답변**: 이미지 크기 최적화는 빌드 시간 단축, 배포 속도 향상, 보안 표면 축소에 중요합니다. Alpine Linux와 멀티스테이지 빌드 외에도 여러 기법이 있습니다. 첫째, `.dockerignore` 파일로 불필요한 파일(node_modules, .git, 테스트 파일, 문서)을 빌드 컨텍스트에서 제외합니다. 둘째, RUN 명령을 체이닝하여 레이어 수를 줄이고, 같은 RUN에서 캐시/임시 파일을 삭제합니다(`RUN apt-get update && apt-get install -y pkg && rm -rf /var/lib/apt/lists/*`). 셋째, 프로덕션 의존성만 설치합니다(`npm ci --only=production`, `pip install --no-cache-dir`). 넷째, distroless 이미지(gcr.io/distroless)를 사용하면 쉘도 없는 최소 이미지로 보안과 크기 모두 개선됩니다. 다섯째, `docker-slim`이나 `dive` 같은 도구로 이미지 레이어를 분석하여 불필요한 파일을 찾아 제거할 수 있습니다. 예를 들어 Node.js 앱의 경우 일반 node 이미지 900MB → alpine 175MB → 멀티스테이지 + distroless 50MB까지 줄일 수 있습니다.

3. **Docker 컨테이너에서 발생하는 로그는 어떻게 관리하나요?** 로그 드라이버와 중앙 집중식 로깅

> **답변**: Docker는 다양한 로그 드라이버를 지원하여 컨테이너 로그를 유연하게 관리할 수 있습니다. 기본 `json-file` 드라이버는 로그를 호스트의 `/var/lib/docker/containers/`에 JSON 형식으로 저장하며, `--log-opt max-size=10m --log-opt max-file=3` 옵션으로 로테이션을 설정할 수 있습니다. 프로덕션 환경에서는 중앙 집중식 로깅이 권장되는데, `fluentd`나 `syslog` 드라이버로 로그를 수집 서버로 전송하거나, ELK(Elasticsearch + Logstash + Kibana) 스택, Loki + Grafana, 또는 클라우드 서비스(AWS CloudWatch, Datadog)로 전송합니다. 모바일 백엔드 운영 시 유용한 패턴은 구조화된 JSON 로깅(winston, pino 등)과 함께 요청 ID를 포함시켜, 모바일 앱에서 발생한 에러를 API 로그에서 추적할 수 있게 하는 것입니다. `docker logs` 명령은 개발 환경에서는 유용하지만, 컨테이너가 삭제되면 로그도 사라지므로 프로덕션에서는 반드시 외부 저장소로 로그를 전송해야 합니다.

4. **컨테이너 보안을 위해 신경 써야 할 것들은 무엇인가요?** 이미지 취약점 스캔, 비-root 실행 등

> **답변**: 컨테이너 보안은 이미지 빌드 단계부터 런타임까지 전 과정에서 고려해야 합니다. 첫째, 이미지 취약점 스캔으로 `trivy`, `snyk`, `docker scout` 같은 도구를 CI/CD에 통합하여 알려진 CVE를 검출합니다. 둘째, 반드시 비-root 사용자로 실행하여 컨테이너 탈출 시 피해를 최소화합니다(Dockerfile에 `USER nonroot` 추가). 셋째, 읽기 전용 파일시스템(`--read-only`)과 필요한 capabilities만 허용(`--cap-drop ALL --cap-add NET_BIND_SERVICE`)하여 공격 표면을 줄입니다. 넷째, 비밀 정보(API 키, DB 비밀번호)는 환경변수나 이미지에 넣지 말고 Docker Secrets나 외부 비밀 관리 서비스(AWS Secrets Manager, HashiCorp Vault)를 사용합니다. 다섯째, 신뢰할 수 있는 베이스 이미지만 사용하고 공식 이미지나 검증된 퍼블리셔의 이미지를 선택합니다. 모바일 앱 백엔드는 사용자 데이터를 다루므로, 특히 네트워크 격리(커스텀 브릿지 네트워크)와 불필요한 포트 노출 방지에 신경 써야 합니다.

5. **Docker Desktop 대신 사용할 수 있는 대안은 무엇인가요?** Colima, Podman, Rancher Desktop 비교

> **답변**: Docker Desktop의 유료화 정책(대기업 유료) 이후 여러 대안이 등장했습니다. **Colima**는 macOS/Linux에서 가장 인기 있는 대안으로, Lima VM 기반으로 Docker CLI와 완벽 호환되며 설치가 간단합니다(`brew install colima && colima start`). Docker Compose도 그대로 사용 가능하여 기존 워크플로우를 유지할 수 있습니다. **Podman**은 Red Hat이 개발한 데몬리스 컨테이너 엔진으로, rootless 실행이 기본이라 보안이 강화되었고, `alias docker=podman`으로 대부분의 명령어가 호환됩니다. 다만 Docker Compose 대신 `podman-compose`를 사용해야 하며 일부 호환성 이슈가 있을 수 있습니다. **Rancher Desktop**은 GUI를 제공하며 containerd/dockerd를 선택할 수 있고, Kubernetes까지 내장하여 K8s 학습에 유리합니다. 모바일 개발자에게는 Colima가 가장 권장되는데, Docker Desktop과 동일한 경험을 무료로 제공하면서 메모리 사용량도 적기 때문입니다.

6. **GitHub Actions에서 Docker 이미지를 빌드하고 배포하는 방법은?** CI/CD 파이프라인 구성

> **답변**: GitHub Actions에서 Docker 이미지를 빌드하고 레지스트리에 푸시하는 것은 현대적인 CI/CD의 표준 패턴입니다. 기본 워크플로우는 checkout → Docker Buildx 설정 → 레지스트리 로그인 → 빌드 및 푸시 순서로 진행됩니다. `docker/build-push-action`을 사용하면 멀티 플랫폼 빌드(linux/amd64, linux/arm64), 레이어 캐싱(GitHub Actions Cache 또는 Registry Cache), 자동 태깅(git 태그, 커밋 SHA 기반)을 쉽게 구성할 수 있습니다. GitHub Container Registry(ghcr.io)는 GitHub과 자연스럽게 통합되어 별도 설정 없이 `GITHUB_TOKEN`으로 인증할 수 있어 편리합니다. 모바일 앱 백엔드의 경우, main 브랜치 푸시 시 자동으로 이미지를 빌드하고, 태그 생성 시 프로덕션 배포를 트리거하는 워크플로우가 일반적입니다. 빌드 캐싱을 적극 활용하면 빌드 시간을 10분에서 2분 이하로 단축할 수 있어, PR마다 테스트용 이미지를 빌드하는 것도 현실적으로 가능해집니다.
