# Yohanesty · Financial IT Backend / Application Architecture

금융 IT 환경에서 Java/Spring 기반 Backend, 대내외 Integration, 운영 안정화와 Application Architecture 개선을 수행해 왔습니다.

관심 영역은 **Legacy Modernization, Application Consolidation, Integration Architecture, Reliability/Operations**입니다.

## Architecture

### [Financial Application Architecture Portfolio](https://github.com/Yohanesty/financial-application-architecture-portfolio)
분산 Application을 기능 책임 중심으로 통합하고, Legacy Interface를 유지하면서 Configuration과 Runtime Isolation으로 업무 차이와 장애 범위를 분리한 Architecture Case Study입니다.

- 52개 진입점 → 4개 업무 영역 재구성
- 분산 중계 수신 → 공통 Receive Pipeline 표준화
- Web/Reverse Proxy 기반 Legacy Interface Compatibility
- Transaction / Resource Isolation / CI Artifact 검증

### [Financial E-Document Gateway Portfolio](https://github.com/Yohanesty/financial-edoc-gateway-portfolio)
Legacy native/socket 기반 전자문서 수신 구조를 Java/Netty 기반으로 재구성한 Integration Modernization 사례입니다.

- Netty 기반 메시지 수신 Pipeline
- Transport / Validation / Business Logic 분리
- 오류 이력 및 재처리
- 비동기 Downstream 처리

## Backend & Security

### [Spring Security MFA Portfolio](https://github.com/Yohanesty/spring-security-mfa-portfolio)
Legacy Spring Web 인증 흐름에 TOTP 기반 MFA를 확장한 인증 고도화 사례입니다.

- 기존 ID/PW 인증 흐름 유지
- TOTP 2차 인증
- 폐쇄망 배포와 운영 절차 고려
- 인증 실패/예외 처리 분리

### [JWT / Redis Image Viewer Portfolio](https://github.com/Yohanesty/jwt-redis-image-viewer-portfolio-)
JWT와 Redis를 활용해 외부 이미지 조회 세션과 접근 범위를 제어한 Backend Integration 사례입니다.

## Operations & Observability

### [Zabbix Monitoring & Operations Portfolio](https://github.com/Yohanesty/zabbix-monitoring-portfolio)
서버·Application 통합 모니터링과 안전한 조건부 자동조치 구조를 정리한 Operations Engineering 사례입니다.

- Linux / AIX / Windows Monitoring
- Process / Log / JVM 관측
- Template / Macro / LLD 표준화
- 조건부 Auto-Recovery
- 운영 검증 Checklist

## Earlier Development

### [Android Development Portfolio](https://github.com/Yohanesty/android-development-portfolio)
Android 기반 문서관리 Prototype과 서버 API 연계 경험을 일반화해 정리한 초기 개발 Portfolio입니다.

## Engineering Principles

- 안정성 > 데이터 정합성 > 운영성 > 성능 > 개발 편의성
- Interface Compatibility와 내부 구조 변경을 분리
- 공통화와 장애 격리를 동시에 검토
- HTTP Status가 아니라 최초 실패 Layer를 추적
- 측정하지 않은 성능/장애 개선율은 성과로 주장하지 않음
- 공개 Portfolio에는 실제 운영 비밀정보와 고객 데이터를 포함하지 않음
