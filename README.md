# Golf-swing-form

좋아하는 프로 골퍼를 고르고 내 스윙 영상을 올리면, 프로의 기준 스윙과 비교해 관절 각도·궤적 차이를 보여주는 개인용 웹 플랫폼입니다. AWS 프리티어로 실제 배포해 운영합니다.

## 아키텍처

```
React ──▶ Spring Boot (EC2) ──▶ RDS (PostgreSQL/MySQL, Free Tier)
  │              │
  │ presigned    │ 분석 요청
  ▼ URL PUT      ▼
  S3  ◀──────── SQS ──▶ Python Lambda (컨테이너 이미지)
  (영상)                 MediaPipe Pose + DTW 비교
```

1. 프론트가 백엔드에서 presigned URL을 받아 S3에 영상을 직접 업로드
2. 업로드 완료를 알리면 백엔드가 SQS에 분석 요청 발행
3. Lambda가 영상에서 포즈 시퀀스를 추출하고, 프로 기준 시퀀스와 DTW로 정렬·비교
4. 결과가 백엔드에 반영되고, 프론트는 상태 조회 API를 폴링해 결과를 표시

## 레포 구조

| 경로 | 내용 | 담당 에이전트 |
|---|---|---|
| `frontend/` | React 웹앱 | 프론트 |
| `backend/` | Spring Boot API 서버 | 백엔드 |
| `ai/` | 포즈 추출·DTW 분석, Lambda 핸들러, 프로 기준 시퀀스 배치 | AI |
| `infra/` | IaC, 배포 스크립트, CI/CD | 클라우드/배포 |
| `contracts/` | 영역 간 계약 (API 명세, 메시지·결과 스키마, S3 키 규칙) | 공동 (PM 승인) |
| `docs/agents/` | 에이전트별 역할 프롬프트와 협업 규칙 | PM |

## 개발 방식

영역별 AI 에이전트 4개(프론트·백엔드·AI·클라우드)가 나눠 개발하고, 영역 간 약속은 `contracts/`에 문서로 고정합니다. 모든 변경은 브랜치 → PR → 검토 후 머지합니다. 자세한 규칙은 [docs/agents/README.md](docs/agents/README.md)를 보세요.
