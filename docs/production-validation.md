# Production Validation Checklist

## Interface

- Legacy Endpoint와 Proxy Routing 일치
- 업무/채널별 Routing 분리
- Request Method / Content-Type 유지
- Response Content-Type / 길이 / Encoding 검증

## Application

- Channel/Profile Configuration 누락 검증
- 정상 / 오류 / Timeout 처리
- Thread Pool 및 Connection Pool 영향
- Runtime별 선택적 재기동 가능 여부
- 다른 업무로 장애가 전파되는지 확인

## Protocol / File

- Fixed-length Header 길이
- Encoding
- File Size 범위
- Partial Read / EOF
- Archive Path Traversal
- XML XXE
- 파일 정리·보존 정책

## Database

- Sequence 및 Composite Key
- Master / Detail / Mapping Transaction
- Rollback 시 상태 원장 정합성
- DB Connection Pool
- 동일 건 Retry / Duplicate 처리

## Operations

- Channel / Request Correlation Log
- Error History
- 재처리 가능 여부
- 배포 / Rollback 절차
- CI Build/Test 성공 여부
- CI JAR SHA-256과 반입 JAR 비교

## Security

- 실제 Credential 저장소 제외
- 민감 전문/개인정보 로그 마스킹
- Application Port 접근통제
- TLS 적용 구간 확인
- 실제 운영 URI/IP/Schema가 공개 문서에 포함되지 않았는지 확인
