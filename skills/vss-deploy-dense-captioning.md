# 🎴 Skill Card: vss-deploy-dense-captioning (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `vss-deploy-dense-captioning`
- **도메인**: VSS (Video Search and Summarization)
- **설명**: RT-VLM(Real-Time Vision-Language Model) Dense Captioning 마이크로서비스를 단독(Standalone) 배포하고, 관련 REST API를 통해 비디오 캡셔닝, 파일 업로드, 라이브 스트림 관리 및 채팅 완성을 수행합니다.
- **핵심 목표**: 전체 VSS 프로필 배포 없이 RT-VLM 서비스만을 독립적으로 구성하여 비디오의 시각적 내용을 텍스트로 상세히 기술(Dense Captioning)하는 기능을 확보하는 것.

## ⚠️ 제약 사항 및 주의사항
- **배포 구분**: 
    - 전체 VSS 프로필(Warehouse/Alerts 등) 배포가 필요한 경우 $\rightarrow$ `vss-deploy-profile` 스킬을 사용하십시오. 본 스킬은 **RT-VLM 단독 서비스** 배포 전용입니다.
- **리소스 요구사항**: NVIDIA GPU 및 NVIDIA Container Toolkit이 설치되어 있어야 하며, GPU 메모리가 선택한 모델(Cosmos Reason 시리즈 등)의 요구량을 충족해야 합니다.
- **보안**: `NGC_CLI_API_KEY` 및 `RTVI_VLM_API_KEY`는 절대 로그에 노출하거나 평문으로 공유하지 마십시오. `.env` 파일 또는 환경 변수로 관리하십시오.
- **권한**: Docker 및 sudo 권한이 필요합니다. 상호작용이 불가능한 환경에서는 `sudo -n` 가드를 사용하여 권한 부족 시 즉시 중단하고 사용자에게 요청하십시오.

## 🛠️ 실행 워크플로우

### Step 0: 사전 요구사항 및 환경 검증 (Pre-flight)
배포 전 하드웨어 및 런타임 상태를 확인합니다.
```bash
# GPU 상태 확인
nvidia-smi --query-gpu=index,name --format=csv,noheader

# NVIDIA Container Toolkit 확인
nvidia-container-cli info

# Docker 및 Compose 버전 확인
docker compose version

# GPU 가속 컨테이너 실행 테스트
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

### Step 1: 단독 배포 실행 (Standalone Deployment Flow)
전체 스택의 `depends_on` 의존성을 제거한 독립형 배포 프로세스를 따릅니다.

1. **설정 파일 준비**:
   ```bash
   # 1. compose 파일 복사 (쓰기 가능한 디렉토리로)
   cp $REPO/deploy/docker/services/rtvi/rtvi-vlm/rtvi-vlm-docker-compose.yml ./standalone-rtvlm-compose.yml
   
   # 2. 단독 배포를 위해 불필요한 depends_on 블록 제거 (필수)
   # (참조: references/deploy-rt-vlm-service.md 의 가이드에 따라 편집)
   ```

2. **환경 변수 설정 (`rtvi-vlm.env` 생성)**:
   다음 핵심 변수를 포함하는 `.env` 파일을 생성합니다.
   | 변수명 | 목적 | 권장값/예시 |
   |---|---|---|
   | `NGC_CLI_API_KEY` | 이미지 풀 및 모델 다운로드 | `<Your-NGC-API-Key>` |
   | `RTVI_VLM_PORT` | 호스트 API 포트 | `8018` |
   | `HOST_IP` | Kafka 브로커 주소 설정용 | `${HOST_IP}` |
   | `VSS_DATA_DIR` | 클립 저장소 바인드 마운트 경로 | `/tmp/vss-data` |
   | `RTVI_VLM_MODEL_TO_USE` | 사용할 백엔드 모델 선택 | `cosmos-reason3` 또는 `openai-compat` |
   | `RTVI_VLM_MODEL_PATH` | 로컬 모델 경로 (Local 사용 시) | `ngc:nim/nvidia/cosmos3-nano-reasoner:bf16-final` |

3. **배포 및 헬스 체크**:
   ```bash
   # Dry-run 검증
   docker compose --env-file rtvi-vlm.env -f standalone-rtvlm-compose.yml config --quiet

   # 이미지 풀 및 실행
   docker compose --env-file rtvi-vlm.env -f standalone-rtvlm-compose.yml up -d rtvi-vlm

   # 서비스 준비 상태 확인
   curl -fsS "http://localhost:${RTVI_VLM_PORT:-8018}/v1/health/ready"
   ```

### Step 2: API 활용 (Quick Start - Local Video)
배포된 서비스에 비디오를 업로드하고 캡션을 생성하는 기본 흐름입니다.

1. **비디오 파일 업로드**:
   ```bash
   export BASE_URL="http://localhost:${RTVI_VLM_PORT:-8018}"
   export API_KEY="${NGC_CLI_API_KEY}"

   FILE_ID=$(curl -fsS -X POST "$BASE_URL/v1/files" \
     -H "Authorization: Bearer $API_KEY" \
     -F "file=@/path/to/video.mp4" \
     -F "purpose=vision" \
     -F "media_type=video" | jq -r '.id')
   ```

2. **Dense Caption 생성 (SSE 스트림)**:
   ```bash
   # 모델 ID 확인
   MODEL_ID=$(curl -fsS "$BASE_URL/v1/models" -H "Authorization: Bearer $API_KEY" | jq -r '.data[0].id // .id')

   # 캡션 생성 요청
   curl -N -X POST "$BASE_URL/v1/generate_captions" \
     -H "Authorization: Bearer $API_KEY" \
     -H "Content-Type: application/json" \
     -d "{
       \"id\": \"$FILE_ID\",
       \"prompt\": \"Write a concise dense caption for each 10-second segment of this video.\",
       \"model\": \"$MODEL_ID\",
       \"chunk_duration\": 10,
       \"stream\": true
     }"
   ```

---

## 📤 API Reference (Core Endpoints)

- **`POST /v1/files`**: 멀티파트 미디어 업로드 $\rightarrow$ `id` 반환.
- **`POST /v1/generate_captions`**: 파일 또는 스트림 기반 캡션 생성.
- **`POST /v1/streams/add`**: RTSP 라이브 스트림 등록.
- **`DELETE /v1/streams/delete/{stream_id}`**: 스트림 해제.
- **`POST /v1/chat/completions`**: OpenAI 호환 멀티모달 채팅 API.
- **`GET /v1/health/ready`**: 서비스 준비 상태 확인.

---

## 🔍 트러블슈팅 (Troubleshooting)

| 증상 (Symptom) | 원인 (Cause) | 해결책 (Fix) |
|---|---|---|
| Connection Refused | 마이크로서비스 미실행 | `/v1/health/ready` 확인 및 `docker compose up` 재시도 |
| HTTP 401/403 (NGC) | API Key 누락/만료 | `docker login nvcr.io` 수행 및 `NGC_CLI_API_KEY` 업데이트 |
| Container OOM / Model Load Fail | GPU 메모리 부족 | 더 작은 모델 변체(Variant)로 변경하거나 다른 GPU 프로세스 종료 |
| RTSP 등록 실패 | 스트림 URL 유효하지 않음 | `ffprobe` 또는 `gst-discoverer-1.0`으로 RTSP URL 유효성 검증 |
| 400 Bad Request (Chat) | 텍스트 전용 `/v1/completions` 호출 | 최신 빌드에서는 멀티모달 `/v1/chat/completions`를 사용하십시오. |

## 🔗 참조 및 연결
- **vss-deploy-profile**: 전체 VSS 스택 배포 (본 스킬의 상위 집합).
- **vss-manage-video-io-storage**: 비디오 파일 업로드 및 스토리지 관리 상세.
- **Detailed Docs**: 
    - API 상세 스펙: `references/api-surface-26.05.md`
    - 단독 배포 상세 가이드: `references/deploy-rt-vlm-service.md`
    - Kafka 워크플로우: `references/kafka-workflows.md`
