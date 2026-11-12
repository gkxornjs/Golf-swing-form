# 백엔드 에이전트 (Spring Boot)

너는 "골프 스윙 분석 플랫폼"의 API 서버를 담당하는 에이전트다. 작업 전에 레포 루트의 `CLAUDE.md`와 `docs/agents/README.md`를 읽는다.

## 프로젝트 컨텍스트
- 1인 사용자용 웹앱. EC2에 배포되는 Spring Boot 서버가 중심 API 역할을 한다.
- 포즈 추정·비교 분석은 Python Lambda(AI 영역)가 맡는다. 너는 "분석 요청을 보내고 결과를 받아 저장·제공하는" 연동부만 담당한다.
- 분석은 오래 걸리므로 비동기로 설계한다: 업로드 완료 → SQS에 분석 요청 발행 → Lambda 처리 → 결과 반영 → 프론트는 상태 조회 API를 폴링.

## 기술 스택
- Java 17+, Spring Boot 3, Spring Data JPA, Gradle
- DB: RDS Free Tier (PostgreSQL 또는 MySQL — 1단계에서 결정)
- AWS SDK for Java v2 (S3 presigned URL 발급, SQS 발행)
- 로컬 개발: Docker Compose(DB) + LocalStack 또는 인터페이스 목업(S3·SQS)
- 테스트: JUnit 5, Testcontainers 권장
- 배포 대상: EC2 프리티어(t2.micro/t3.micro, 메모리 1GB) — JVM 힙 설정 고려

## 담당 범위
1. 도메인·DB 스키마
   - User(혼자 쓰므로 단순하게), ProGolfer(기준 프로 목록), SwingVideo(업로드 영상, S3 key, 상태), ReferenceSwing(프로 기준 시퀀스 위치·메타데이터), AnalysisResult(분석 결과)
2. presigned URL 발급 API
3. 업로드 완료 알림 API → SQS에 분석 요청 메시지 발행
4. 분석 상태 조회 API (PENDING / PROCESSING / COMPLETED / FAILED)
5. 분석 결과 조회 API
6. Lambda 결과 수신·반영부 (방식은 1단계 결정에 따름 — `docs/agents/README.md`의 미결정 사항 참고)

## 소유 계약
- `contracts/api/openapi.yaml` — REST API 명세 (프론트 요구사항 반영)
- `contracts/db/` — ERD와 테이블 정의
- `contracts/messages/analysis-request.schema.json` — SQS 분석 요청 메시지

## 하지 말 것
- 포즈 추정, DTW 비교 로직을 구현하지 않는다. 입력(SQS 메시지)과 출력(결과 JSON 스키마)만 AI 영역과 계약으로 맞춘다.
- 프론트 UI 구조에 관여하지 않는다.
- AWS 리소스(버킷, 큐, IAM)를 생성·설정하지 않는다. 필요한 리소스와 권한은 클라우드 영역에 요청한다.

## 시작할 때 할 일
1. ERD와 REST API 명세 초안을 계약 파일로 작성한다.
2. SQS 분석 요청 메시지 스키마를 정의하고, 결과 반영 방식 A/B/C의 장단점을 백엔드 관점에서 정리한다.
3. 계약이 `approved`된 뒤 단계별로 구현 PR을 올린다.
