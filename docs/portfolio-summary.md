# Application Architecture Modernization · Portfolio Summary

## Overview

금융 IT 운영환경에서 업무·고객사별로 분산되어 있던 Application을 분석하고, **외부 Interface를 유지하면서 내부 구조를 공통화·표준화**하는 Architecture 개선을 수행했습니다.

핵심은 프로젝트 수나 WAR/JAR 개수를 줄이는 것이 아니라 다음 문제를 동시에 해결하는 것이었습니다.

- 동일 기능의 반복 구현과 중복 운영 포인트
- 업무별 설정이 소스 코드에 침투하는 구조
- 배포·재기동 단위와 장애 영향 범위의 불명확성
- Legacy Interface 변경에 따른 전환 리스크
- Binary/File/DB 처리 과정의 장애 추적 및 재처리 어려움
- 폐쇄망 배포 환경에서 Source와 실제 배포 Binary의 정합성 확인 어려움

이를 위해 두 종류의 통합 사례에서 동일한 Architecture 원칙을 적용했습니다.

---

## Case A. 52개 진입점 → 4개 업무 영역

### 기존 문제

고객사·서비스별 진입점이 증가하면서 동일한 인증, 파일 등록, 조회, 통계, 사용자·권한 관리 등의 기능이 여러 위치에서 반복 관리되는 구조였습니다.

업무별 WAR 분리도 검토했지만, WAR 수가 증가할수록 배포 스크립트, Alias, DB 코드, 환경 설정과 운영 교육 포인트가 함께 증가한다는 문제가 있었습니다.

### Architecture 판단

기능 책임을 기준으로 52개 진입점을 4개 업무 영역으로 재구성하고, **단일 소스/공통 배포 구조 + Profile 기반 설정 분리 + Runtime 격리** 방식을 선택했습니다.

```text
AS-IS
52 Entry Points
    │
    ├─ duplicated configuration
    ├─ duplicated deployment logic
    └─ repeated maintenance points

TO-BE
Web Routing
    │
    ▼
Common Application
    │
    ├─ Business Area A
    ├─ Business Area B
    ├─ Business Area C
    └─ Business Area D
    │
Profile / Environment Configuration
    │
Multiple Runtime Units
```

### 핵심 설계

- 공통 기능과 업무별 설정 분리
- Web 계층에서 업무별 Routing 유지
- 동일 소스 구조를 사용하면서 Runtime 단위를 분리해 선택적 재기동 가능
- 신규 업무 추가 시 신규 프로젝트 복제보다 Configuration 확장을 우선
- 배포 단순화와 장애 격리 사이의 균형 유지

### 결과

52개 진입점을 4개 영역으로 정리해 소스·설정·배포 관리 포인트를 축소하고, 공통 기능 변경 시 반복 개발 범위를 줄였습니다.

---

## Case B. 분산 중계 수신 → 공통 Integration Application

### 기존 문제

업무별 수신 Application은 고정길이 Binary 전문, 파일 수신, 압축 해제, XML Metadata 분석, DB 원장 등록이라는 유사한 처리 흐름을 가지고 있었지만 업무별로 분리되어 있었습니다.

이 구조에서는 Protocol 변경, 오류 처리 개선, 보안 보완 또는 운영 로그 개선 시 동일한 변경을 여러 Application에 반복 적용해야 했습니다.

### Architecture 판단

외부 호출 주소를 일괄 변경하지 않고 Reverse Proxy를 이용해 Legacy Endpoint를 유지하면서 내부적으로는 하나의 공통 수신 API와 Channel Resolver를 사용하도록 설계했습니다.

```text
External Partner
      │ Legacy Interface
      ▼
Web / Reverse Proxy
      │ Internal Routing
      ▼
Common Receive API
      │
      ├─ Channel Resolver
      ├─ Fixed-length Protocol Reader
      ├─ Binary Streaming
      ├─ Archive Validation / Extraction
      ├─ XML Metadata Parser
      ├─ Document Mapping
      ├─ Transaction / Ledger Registration
      └─ Legacy-compatible Response
```

### 핵심 설계

- Legacy Interface Compatibility 유지
- 업무 식별을 URI 문자열 분석이 아닌 명시적 Channel Code로 처리
- 전체 Request를 메모리에 올리지 않고 Stream 기반으로 파일 저장
- Archive Path Traversal 방어 및 XML XXE 방어
- Legacy DAO를 JPA Composite Key / Sequence 기반으로 재설계
- Master / Detail / Mapping 데이터의 Transaction Boundary 명확화
- 오류 발생 시 수신 이력을 별도 상태로 남길 수 있는 구조
- Legacy 응답 전문의 고정길이/Offset 호환 검증
- GitHub Actions에서 Build/Test/JAR 생성 및 SHA-256 Artifact 검증

### Troubleshooting 사례

Web Server를 통한 수신 테스트 중 HTTP 오류를 단순 Application 문제로 판단하지 않고 계층별로 분리했습니다.

```text
VHOST Mapping
→ Reverse Proxy
→ Content-Type
→ Spring Security
→ Controller
→ Binary Reader
→ File / XML
→ JPA / Database
```

VHOST 불일치, Content-Type 오류, Security의 2차 Error 처리 등을 순차적으로 구분하여 실제 최초 실패 지점을 확인했습니다.

---

## Common Architecture Principles

두 프로젝트에서 공통으로 적용한 원칙은 다음과 같습니다.

**1. Interface를 먼저 보호합니다.**  
외부 시스템과 동시에 변경하지 않고 Web/Proxy 계층에서 Compatibility를 유지해 내부 전환 리스크를 분리합니다.

**2. 업무 차이는 Configuration으로 밀어냅니다.**  
업무 코드마다 같은 Service를 복제하지 않고 공통 흐름을 유지한 상태에서 Profile/Channel 설정으로 차이를 표현합니다.

**3. 통합과 장애 격리를 동시에 봅니다.**  
통합 Application 하나가 전체 장애점이 되지 않도록 Runtime, Thread Pool, Connection Pool, 재기동 및 Scale-out 단위를 같이 검토합니다.

**4. 성공 기준을 End-to-End로 정의합니다.**  
HTTP 200이나 파일 저장만으로 성공 처리하지 않고 DB 원장, 상태, 응답 전문까지 완료되어야 하나의 정상 Transaction으로 판단합니다.

**5. 배포 Binary까지 추적합니다.**  
CI에서 검증한 JAR과 폐쇄망 서버에 반입된 JAR의 SHA-256을 비교해 Source Commit과 실제 배포 Artifact의 연결성을 확보합니다.

---

## Architecture Value

이 경험은 특정 Framework 사용 경험보다 다음 역량을 보여줍니다.

- Legacy 구조를 기능·운영·장애 관점에서 분해하는 능력
- 공통화 대상과 격리 대상의 경계를 결정하는 능력
- 기존 Interface를 유지한 단계적 전환 설계
- Web / WAS / Application / DB / File / External Interface를 하나의 처리 흐름으로 보는 관점
- 운영 중 장애와 재처리를 고려한 Transaction 및 상태 설계
- 개발 완료 이후 Build, 배포, 검증까지 연결하는 Architecture 관점

정량 성과는 실제로 확인 가능한 `52개 진입점 → 4개 업무 영역`과 같은 수치만 사용하며, 측정하지 않은 성능 개선율이나 장애 감소율은 별도 KPI로 주장하지 않습니다.
