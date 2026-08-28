# 🎴 Skill Card: vss-setup-video-analytics-api (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `vss-setup-video-analytics-api`
- **도메인**: VSS (Video Search and Summarization)
- **설명**: `vss-video-analytics-api` REST 서비스를 단독(Standalone)으로 배포하고 설정합니다.
- **핵심 목표**: 전체 warehouse 스택 없이, 분석 API 서버만을 독립적으로 구성하여 REST 인터페이스를 제공하는 것.

## ⚠️ 제약 사항 및 주의사항
- **사용 금지 대상**: 전체 warehouse 블루프린트 스택 배포 $\rightarrow$ `vss-deploy-profile` (profile: `warehouse` 또는 `alerts`) 사용.
- **인프라 의존성**: Elasticsearch가 반드시 실행 중이고 연결 가능해야 합니다. 서버는 시작 시 ES를 핑(Ping)하며, 연결 실패 시 종료됩니다.
- **보안 주의**: `NGC_CLI_API_KEY`는 민감한 자격 증명입니다. 절대 채팅창에 붙여넣거나 `/tmp`에 저장하지 마십시오. `read -rs` 또는 시크릿 매니저를 통해 로드하십시오.

## 🛠️ 실행 워크플로우

### Step 0: 사전 요구사항 확인 (Prerequisites)
1. **레포지토리 및 경로**: `$VSS_APPS_DIR`가 `<repo>/deploy/docker/`를 가리키고 있는지 확인합니다.
2. **인증**: `$NGC_CLI_API_KEY`가 설정되어 `nvcr.io`에서 이미지를 풀(pull)할 수 있는지 확인합니다.
3. **런타임**: Docker Engine **28.3.3** 및 Docker Compose plugin **v2.39.1+** 버전인지 확인합니다.
4. **인프라**:
    - **Elasticsearch (필수)**: 설정된 URL에서 접근 가능해야 합니다.
      - 필요 시 실행: `docker compose -f services/infra/compose.yml up -d elasticsearch`
    - **Kafka (선택)**: 브로커 없이도 실행 가능하지만, 동적 설정/보정/RTLS/AMR 기능을 사용하려면 연결이 필요합니다.
5. **저장소**: `$VSS_DATA_DIR/data_log/vss_video_analytics_api` 경로가 쓰기 가능하도록 설정되어 있는지 확인합니다.

### Step 1: 배포 전략 결정 (User Input)
사용자로부터 다음 사항을 결정합니다:
1. **설정 소스**: 이미지 내장 기본값, 서비스 제공 기본값, 또는 사용자 정의(Custom) 설정 중 선택.
2. **데이터-로그 볼륨**: 파일 업로드 처리를 위해 호스트 볼륨 바인딩이 필요한지 결정.
3. **인프라 연결**: Elasticsearch 주소 확인 및 Kafka 브로커 연결 여부 결정.

### Step 2: 배포 및 검증 (Deploy & Verify)
1. **배포 실행**:
```bash
cd $REPO/deploy/docker
export VSS_APPS_DIR=$(pwd)
export VSS_DATA_DIR=${VSS_DATA_DIR:-/tmp/vss-data}
mkdir -p "$VSS_DATA_DIR/data_log/vss_video_analytics_api"

# Standalone 배포 실행
docker compose -f services/analytics/compose.yml up -d vss-video-analytics-api
```

2. **헬스 체크 (Health Check)**:
```bash
# 서비스가 완전히 떴는지 확인 (HTTP 200 OK)
curl -sf http://localhost:8081/livez
```
- **성공**: `200 OK` (또는 응답 성공).
- **실패**: `docker compose logs vss-video-analytics-api`를 확인하여 Elasticsearch 연결 오류인지 확인하십시오.

---

## 📤 API 사용 가이드 (Quick Start)

### 1. API 문서 확인
서비스가 실행되면 브라우저 또는 curl을 통해 Swagger UI를 확인할 수 있습니다:
`http://localhost:8081/swagger-ui/index.html`

### 2. 주요 엔드포인트 호출 예시
- **상태 확인**: `GET /livez` 또는 `GET /health`
- **설정 조회**: `GET /config`
- **분석 요청**: `POST /analyze` (페이로드에 비디오 URL 및 분석 파라미터 포함)

---

## 🔍 트러블슈팅 (Troubleshooting)

| 증상 (Symptom) | 원인 (Cause) | 해결책 (Fix) |
|---|---|---|
| 컨테이너가 즉시 종료됨 (Exited) | Elasticsearch 연결 불가 | `docker compose logs` 확인 후 ES 엔드포인트 및 네트워크 설정 재검토 |
| `403 Forbidden` 발생 | NGC 이미지 풀 권한 없음 | `docker login nvcr.io` 수행 및 `$NGC_CLI_API_KEY` 확인 |
| API 응답 속도가 매우 느림 | Kafka 브로커 타임아웃 | Kafka를 사용하지 않는 경우 설정에서 `kafka.brokers: []` 확인 |
| 볼륨 마운트 오류 (Permission Denied) | 호스트 경로 권한 부족 | `chmod -R 777 $VSS_DATA_DIR/data_log/vss_video_analytics_api` 수행 |

## 🔗 참조 및 연결
- **Upstream**: `vss-deploy-profile` (전체 스택 배포)
- **Dependent**: `vss-summarize-video` (분석 API 결과를 기반으로 요약 수행)
- **Related**: `vss-setup-behavior-analytics` (분석 API의 상위 설정/보정 도구)
- **Detailed Docs**:
    - API 스펙: `src/app/specification/openapi.json`
    - Docker 설정 가이드: `references/deploy-video-analytics-api-service.md`
