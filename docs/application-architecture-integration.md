# Application Architecture Integration Experience

## 1. 왜 Application 통합이 필요했는가

운영 중인 금융 시스템에서 Application이 고객사·업무별로 증가하면 개별 시스템은 단순해 보이지만 전체 운영 관점에서는 동일한 인증, 파일 처리, 로그, 예외 처리, 배포 스크립트와 설정이 반복됩니다.

통합의 목적을 “프로젝트 수 감소”로 두면 공통 Application의 장애 영향 범위가 커지거나 Thread/Connection Resource가 과도하게 공유되는 문제가 발생할 수 있습니다. 따라서 다음 질문을 기준으로 통합 범위를 결정했습니다.

- 실제로 중복된 기능인가?
- 업무별 차이를 Configuration으로 표현할 수 있는가?
- 장애 발생 시 영향 범위를 어디까지 허용할 것인가?
- 어떤 단위로 배포·재기동·Scale-out 해야 하는가?
- 기존 Interface를 변경하지 않고 내부를 전환할 수 있는가?

## 2. Case A · 사용자/채널 Application 통합

### AS-IS

52개 진입점이 고객사·서비스 기준으로 분산되어 있었고, 공통 기능 변경 시 여러 운영 포인트를 함께 수정해야 했습니다.

### 검토한 대안

**업무별 WAR 분리**는 격리 측면에서는 단순하지만 업무 증가에 따라 WAR, 배포 스크립트, Alias, 환경 설정과 교육 포인트가 계속 증가합니다.

### 선택한 구조

단일 공통 Application을 유지하되 Profile/설정으로 업무별 차이를 분리하고, Runtime/JVM 단위를 나누어 선택적 배포·재기동이 가능하도록 구성했습니다.

```text
Web Routing
    ↓
Common Application Artifact
    ↓
Profile / Business Configuration
    ↓
Runtime A / Runtime B / Runtime C / Runtime D
```

### Architecture 의미

소스와 공통 로직은 하나로 관리하면서 운영 장애 단위는 Runtime으로 분리했습니다. 즉 “소스 통합”과 “운영 격리”를 같은 단위로 강제하지 않았습니다.

## 3. Case B · 중계 수신 Application 통합

### AS-IS

업무별 수신 Application이 비슷한 Binary 전문 수신 → 파일 저장 → 압축 해제 → Metadata 분석 → DB 등록 흐름을 반복하고 있었습니다.

### 선택한 구조

Reverse Proxy에서 기존 외부 Interface를 유지한 채 내부 API를 Channel별로 Routing하고, Spring Boot Application 내부에서는 공통 Receive Service를 사용하도록 설계했습니다.

```text
Legacy Endpoint
    ↓
Reverse Proxy
    ↓
/channel/{channelCode}/receive
    ↓
Channel Resolver
    ↓
Common Receive Pipeline
```

### 처리 Pipeline

```text
Fixed-length Header
→ File Metadata
→ Stream Copy
→ Archive Validation
→ XML Parsing
→ Domain Mapping
→ Transactional Persistence
→ Legacy-compatible Response
```

### Architecture 의미

업무별 차이는 Channel Configuration으로 분리하고, Protocol/File/DB 처리 로직을 하나의 공통 흐름으로 유지했습니다. 신규 업무가 추가될 때 기존 Application을 복제하는 대신 Channel과 Configuration을 확장할 수 있는 구조를 목표로 했습니다.

## 4. 공통 설계 원칙

### 외부 Compatibility와 내부 구조를 분리

외부 시스템에 변경을 요구하지 않고 Reverse Proxy/Web Routing을 이용해 Legacy Endpoint를 유지했습니다. 이를 통해 외부 Interface Cut-over와 내부 Application 전환을 분리했습니다.

### Configuration over branching

업무별 ID, 경로, 처리 옵션 같은 차이를 코드의 `if/else`로 누적하지 않고 Configuration으로 이동시켰습니다.

### Resource Isolation

통합 Application에서는 특정 업무의 트래픽이나 장애가 다른 업무로 전파될 수 있으므로 Thread, Connection Pool, Runtime/JVM, 파일 경로, 재기동 단위를 함께 검토했습니다.

### Transaction과 상태 추적

파일 수신만 성공하고 DB 등록이 실패하는 부분 성공 상태를 방치하지 않도록 처리 단계와 상태 변경을 분리하고, 실패한 수신 건을 추적할 수 있도록 설계했습니다.

### 운영 검증 가능성

로그에 Channel/수신 식별자를 남기고, CI에서 Build/Test 후 JAR과 SHA-256을 생성해 실제 배포 Binary까지 추적할 수 있도록 했습니다.

## 5. Troubleshooting 접근

문제가 발생한 Layer에서 바로 Root Cause를 확정하지 않고 다음 순서로 확인했습니다.

```text
Client
→ Web Server / Reverse Proxy
→ Security
→ Application Controller
→ Protocol Parser
→ File System
→ Transaction
→ Database
→ Response
```

예를 들어 HTTP 403이 보이더라도 실제 최초 오류가 Content-Type 불일치이고 `/error` 처리 과정에서 Security가 403으로 덮어쓰는 경우처럼, 최종 Status와 최초 실패 지점을 구분해 분석했습니다.

## 6. 공통 성과

- 52개 진입점 → 4개 업무 영역으로 구조 재편
- 분산된 중계 수신 흐름을 공통 Receive Pipeline으로 표준화
- 외부 Interface 변경 없이 내부 Architecture 전환
- 공통 소스와 업무별 Configuration 분리
- 통합에 따른 장애 범위 확대를 Runtime/Resource 단위로 제어
- Build/Test → Artifact → SHA-256까지 연결한 배포 검증 체계 구성

## 7. 이 경험이 보여주는 역량

이 두 프로젝트는 특정 Framework를 교체한 경험보다 **분산된 운영 시스템을 어디까지 통합하고 어디를 격리할지 판단한 경험**에 가깝습니다.

Application Architect 관점에서는 공통화율을 최대화하는 것보다 운영 안정성, 데이터 정합성, 장애 영향 범위와 신규 업무 확장성을 동시에 만족시키는 경계를 정하는 것이 핵심이었습니다.
