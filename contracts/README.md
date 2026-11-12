# 계약 (Contracts)

영역 간 약속을 모아 두는 곳입니다. 구현은 항상 이 문서를 따르며, 바꿀 때는 계약 변경 PR을 먼저 올립니다.

| 파일 | 내용 | 소유 | 사용하는 영역 | 상태 |
|---|---|---|---|---|
| `api/openapi.yaml` | REST API 명세 | 백엔드 | 프론트 | 미작성 |
| `db/erd.md` | DB 스키마 | 백엔드 | (참고: AI·클라우드) | 미작성 |
| `messages/analysis-request.schema.json` | SQS 분석 요청 메시지 | 백엔드 | AI | 미작성 |
| `messages/analysis-result.schema.json` | 분석 결과 JSON | AI | 백엔드, 프론트 | 미작성 |
| `pose/pose-sequence.md` | 포즈 시퀀스·기준 시퀀스 포맷 | AI | (참고: 백엔드) | 미작성 |
| `storage/s3-keys.md` | S3 버킷·키 규칙 | 클라우드 | 백엔드, AI | 미작성 |
| `infra/architecture.md` | 인프라 구성도, 환경변수, 권한 | 클라우드 | 전체 | 미작성 |

## 상태 표기

각 계약 파일 맨 위에 다음을 적습니다.

```
status: draft | approved
owner: backend | ai | cloud
version: 0.1.0
```

- `draft`: 제안 단계. 다른 영역이 의견을 남길 수 있음
- `approved`: PM 승인 완료. 이후 변경은 버전을 올리고 영향 영역을 명시
