# StockFlow Server

주식 커뮤니티 + KIS 모의투자 자동매매 플랫폼 백엔드

---

## 프로젝트 개요

StockFlow는 한국투자증권(KIS) 모의투자 API를 연동한 자동매매 기능과 종목 분석·전략 공유 커뮤니티를 결합한 플랫폼입니다.

---

## 기술 스택

| 분류 | 기술 |
|------|------|
| Language | Java 17 |
| Framework | Spring Boot 3.5 |
| ORM | Spring Data JPA |
| SQL Mapper | MyBatis |
| Database | PostgreSQL |
| Cache | Redis |
| Auth | Spring Security + JWT |
| API Docs | SpringDoc OpenAPI (Swagger UI) |
| External API | KIS 한국투자증권 모의투자 REST API |

---

## 도메인 구조

### 핵심 도메인
- **Member** — 회원 가입, 로그인, JWT 인증
- **Account** — KIS 모의투자 계좌 연동
- **Order** — 매수/매도 자동매매 주문
- **Trade** — 체결 내역 관리
- **Strategy** — 자동매매 전략 등록/실행

### 커뮤니티 도메인
- **Post** — 종목 분석, 전략 공유 게시글
- **Comment** — 댓글
- **Like** — 좋아요

### 동시성 처리 전략
| 기능 | 전략 |
|------|------|
| 주문(매수/매도) | 비관적 락 or Redis 분산락 |
| 좋아요 | 낙관적 락 or Redis |
| 조회수 | Redis incr |

---

## 로컬 실행

### 사전 조건
- Java 17
- Docker (PostgreSQL, Redis 실행용)
- KIS 모의투자 API 키

### 1. 환경 변수 설정

`src/main/resources/application.yaml`에 아래 항목을 채워넣으세요:

```yaml
spring:
  application:
    name: stockflowserver
  datasource:
    url: jdbc:postgresql://localhost:5432/stockflow
    username: your_db_user
    password: your_db_password
  jpa:
    hibernate:
      ddl-auto: update

jwt:
  secret: your_jwt_secret
  expiration: 86400000  # 24h (ms)

kis:
  app-key: your_kis_app_key
  app-secret: your_kis_app_secret
  base-url: https://openapivts.koreainvestment.com:29443  # 모의투자 URL
```

> `application.yaml`은 `.gitignore`에 포함되어 있습니다. 직접 생성하거나 `application.yaml.example`을 복사해 사용하세요.

### 2. 인프라 실행

```bash
docker-compose up -d
```

### 3. 서버 실행

```bash
./gradlew bootRun
```

### 4. API 문서 확인

```
http://localhost:8080/swagger-ui/index.html
```

---

## 패키지 구조 (예정)

```
src/main/java/stockflow/stockflowserver/
├── member/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   └── domain/
├── account/
├── order/
├── trade/
├── strategy/
├── post/
├── comment/
├── like/
└── global/
    ├── config/       # Security, JPA, Redis 설정
    ├── exception/
    └── jwt/
```

---

## 주요 API (예정)

| Method | URL | 설명 |
|--------|-----|------|
| POST | /api/auth/signup | 회원가입 |
| POST | /api/auth/login | 로그인 |
| POST | /api/accounts/connect | KIS 계좌 연동 |
| POST | /api/orders | 주문 생성 |
| GET | /api/orders | 주문 내역 조회 |
| GET | /api/trades | 체결 내역 조회 |
| POST | /api/strategies | 전략 등록 |
| GET | /api/posts | 게시글 목록 |
| POST | /api/posts | 게시글 작성 |
| POST | /api/posts/{id}/like | 좋아요 |

---

## KIS 모의투자 API 참고

- [한국투자증권 Open API 포털](https://apiportal.koreainvestment.com)
- 모의투자 Base URL: `https://openapivts.koreainvestment.com:29443`
- 실전투자 Base URL: `https://openapi.koreainvestment.com:9443`
