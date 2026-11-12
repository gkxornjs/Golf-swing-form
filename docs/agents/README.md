# 에이전트 협업 가이드

## 구성

| 에이전트 | 프롬프트 | 담당 폴더 | 소유 계약 |
|---|---|---|---|
| 프론트 | [frontend.md](frontend.md) | `frontend/` | — (API 요구사항만 제안) |
| 백엔드 | [backend.md](backend.md) | `backend/` | `contracts/api/`, `contracts/db/`, `contracts/messages/analysis-request.*` |
| AI | [ai.md](ai.md) | `ai/` | `contracts/messages/analysis-result.*`, `contracts/pose/` |
| 클라우드/배포 | [cloud.md](cloud.md) | `infra/` | `contracts/storage/`, `contracts/infra/` |

PM(레포 주인)은 모든 계약 PR을 승인하고, 영역 간 충돌을 최종 결정한다. 오케스트레이터(Claude)는 에이전트를 띄우고, 결과를 모아 PR을 만들고, 어긋난 부분을 PM에게 보고한다.

## 소통 방식

에이전트끼리 직접 대화하지 않는다. 대신:
1. **계약 파일**(`contracts/`)이 공식 소통 채널이다. 모든 에이전트는 작업 전에 관련 계약을 읽는다.
2. **PR 본문의 "다른 영역에 요청"** 섹션으로 요청을 남기면, 오케스트레이터가 해당 에이전트에게 전달한다.
3. 결정이 필요하면 `docs/decisions/`에 선택지를 정리하고 PM에게 넘긴다.

## 진행 단계

| 단계 | 참여 에이전트 | 산출물 | 완료 조건 |
|---|---|---|---|
| 1. 계약 확정 | 백엔드, AI, 클라우드 | API 명세, DB 스키마, SQS 메시지·결과 JSON 스키마, S3 키 규칙, 인프라 구성도 | 계약 파일 전부 `approved` |
| 2. AI 파이프라인 로컬 검증 | AI | 프로 영상 1개 + 내 영상 1개로 DTW 비교 결과 | 결과가 계약 스키마를 만족하고 의미 있는 차이를 보여줌 |
| 3. 백엔드·프론트 로컬 연동 | 백엔드, 프론트 | API 서버, 화면, 로컬 목업(SQS·S3 대체) | 업로드 → 분석중 → 결과 화면까지 로컬에서 동작 |
| 4. AWS 배포 | 클라우드 (+ 각 영역 지원) | IaC, CI/CD, 배포 가이드 | 실제 AWS에서 전체 흐름 동작 |

## 미결정 사항 (1단계에서 결정)

- **Lambda 결과 반영 방식**: Lambda가 RDS에 직접 쓰려면 VPC 안에 들어가야 하고, 그러면 S3/SQS 접근에 NAT Gateway(유료) 또는 VPC 엔드포인트가 필요하다. 후보:
  - (A) Lambda → 결과 JSON을 S3에 저장 → S3 이벤트 또는 백엔드 폴링으로 RDS 반영
  - (B) Lambda → 백엔드 내부 콜백 API 호출 (인증 토큰 필요)
  - (C) Lambda → RDS 직접 쓰기 (VPC + 엔드포인트 구성)
- **DB 엔진**: PostgreSQL vs MySQL
- **IaC 도구**: Terraform vs AWS CDK
- **프론트 호스팅**: S3 정적 호스팅(+CloudFront) vs EC2에서 함께 서빙

## 조율 체크리스트 (PR 머지 전 점검)

- [ ] REST API 명세가 백엔드 구현·프론트 호출과 일치하는가
- [ ] SQS 메시지 포맷이 백엔드 발행부와 Lambda 파싱부에서 같은가
- [ ] 분석 결과 JSON 스키마가 Lambda 출력과 백엔드 저장부에서 같은가
- [ ] S3 키 규칙을 백엔드(presigned 발급), AI(읽기·쓰기), 클라우드(버킷 정책·이벤트)가 동일하게 가정하는가
- [ ] DB 스키마 변경이 관련 계약과 다른 영역에 반영되었는가
