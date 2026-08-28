# 🎴 Skill Card: vss-summarize-video (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `vss-summarize-video`
- **도메인**: VSS (Video Search and Summarization)
- **설명**: LVS 요약 마이크로서비스(HITL-gated) 또는 VLM 폴백을 사용하여 녹화된 비디오의 서술적 요약 및 타임스탬프 이벤트 리스트를 생성합니다.
- **핵심 목표**: 비디오 클립에 대해 단일한 정제된 내러티브 요약 및 이벤트 목록 제공.

## ⚠️ 제약 사항 및 주의사항
- **사용 금지 대상**:
    - 실시간 RTSP 캡셔닝 $\rightarrow$ `vss-deploy-dense-captioning` 사용.
    - 리포트 생성 (사고/경고 윈도우 리포트 등) $\rightarrow$ `vss-generate-video-report` (Mode B) 사용.
    - 아카이브 전체 시맨틱 검색 $\rightarrow$ `vss-search-archive` 사용.
- **출력 원칙**: 백엔드 출력을 **최소한의 변형**으로 제공하십시오. 패러프레이징, 이모지 추가, 재포맷팅을 금지하며 백엔드의 텍스트를 그대로 렌더링합니다. (One backend call $\rightarrow$ one rendering).
- **라우팅**: 비디오 길이에 상관없이 `/v1/ready` 상태 코드(HTTP 200)에 따라 LVS 서비스 또는 VLM 폴백 경로를 결정합니다.

## 🛠️ 실행 워크플로우

### Step 0: 환경 설정 및 가용성 확인 (Setup)

#### 1. 엔드포인트 및 모델 설정
- **VLM / RT-VLM**: `${VLM_BASE_URL}` (기본값: `http://${HOST_IP:-localhost}:8018`)
- **LVS 서비스**: `${LVS_BACKEND_URL}` (기본값: `http://${HOST_IP:-localhost}:38111`)
- **모델 이름**: `${VLM_NAME}` (기본값: `nim_nvidia_cosmos3-nano-reasoner_bf16-final`)

#### 2. 가용성 프로브 (Availability Checks)
반드시 아래 스크립트를 실행하여 라우팅 경로를 결정하십시오. (LVS의 경우 503 Warmup 리트라이 포함)

```bash
VLM="${VLM_BASE_URL:-${RTVI_VLM_BASE_URL:-http://${HOST_IP:-localhost}:8018}}"
VLM="${VLM%/v1}"

# VLM / RT-VLM 확인: /v1/models 가 200이어야 함
vlm_code=$(curl -s -o /dev/null -w '%{http_code}' --connect-timeout 3 --max-time 10 "$VLM/v1/models")
[ "$vlm_code" = "200" ] && echo "VLM OK" || echo "VLM not reachable (HTTP $vlm_code)"

# LVS 서비스 확인: /v1/ready 가 200이어야 함 (503 시 최대 30초 리트라이)
VIDEO_SUMMARIZATION_URL=${LVS_BACKEND_URL:-http://${HOST_IP:-localhost}:38111}
video_sum_code=000
for i in $(seq 1 10); do
  video_sum_code=$(curl -s -o /dev/null -w '%{http_code}' --connect-timeout 3 --max-time 10 "$VIDEO_SUMMARIZATION_URL/v1/ready")
  case "$video_sum_code" in
    200) echo "video summarization OK"; break ;;
    503) sleep 3 ;; # warming up
    *) break ;;
  esac
done
[ "$video_sum_code" = "200" ] || echo "video summarization service not reachable (HTTP $video_sum_code)"
```

#### 3. 라우팅 결정
| `/v1/ready` 결과 | 선택할 백엔드 | 호출 엔드포인트 |
|---|---|---|
| **HTTP 200** | LVS microservice (HITL) | `POST ${LVS_BACKEND_URL}/v1/summarize` |
| **그 외 모든 결과** | VLM / RT-VLM (Fallback) | `POST ${VLM}/v1/chat/completions` |

---

### Step 1: 비디오 클립 URL 확보 (Sub-task)
**반드시 `vss-manage-video-io-storage` 스킬을 호출하여 수행하십시오.** 직접 curl 호출을 금지합니다.
- **수집 정보**: `streamId`, 타임라인(`startTime`, `endTime`), 및 `container=mp4` 형식의 임시 MP4 클립 URL(`.videoUrl`).
- **주의**: 클립 URL 확보는 중간 단계입니다. 이를 최종 답변으로 제공하지 말고 Step 2로 즉시 진행하십시오.

---

### Step 2: 요약 생성 (Main Process)

#### 경로 A: LVS 마이크로서비스 (Primary - HITL 필수)
1. **HITL 수행**: 사용자에게 `scenario`와 `events`를 요청하십시오.
   - **자율 모드(Autonomous) 기본값**: 사용자가 요청한 경우에만 `scenario="activity monitoring"`, `events=["notable activity"]`를 사용합니다.
2. **API 호출**:
```bash
VIDEO_SUMMARIZATION_URL=${LVS_BACKEND_URL:-http://${HOST_IP:-localhost}:38111}
SCENARIO='warehouse monitoring' # HITL 결과 반영
EVENTS_JSON='["notable activity"]' # HITL 결과 반영
OBJECTS_JSON='' # 선택 사항

curl -s --max-time 300 -X POST "$VIDEO_SUMMARIZATION_URL/v1/summarize" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg url "<clip_url_from_vss_manage_video_io_storage>" \
        --arg model "${VLM_NAME:-nim_nvidia_cosmos3-nano-reasoner_bf16-final}" \
        --arg scenario "$SCENARIO" \
        --argjson events "$EVENTS_JSON" \
        --argjson objects "${OBJECTS_JSON:-null}" '{
    url: $url,
    model: $model,
    scenario: $scenario,
    events: $events,
    chunk_duration: 10,
    num_frames_per_second_or_fixed_frames_chunk: 20,
    use_fps_for_chunking: false,
    seed: 1
  } + (if $objects == null then {} else {objects_of_interest: $objects} end)')" \
  | jq -r '.choices[0].message.content' \
  | jq '{video_summary, events}'
```

#### 경로 B: VLM 직접 호출 (Fallback)
LVS 서비스가 불가능할 때만 사용합니다. **HITL을 수행하지 않습니다.**
1. **폴백 경고 문구 삽입**: 답변 최상단에 다음을 **그대로** 복사하여 추가하십시오.
   > ⚠ **Note:** Input video `<name>` is `<N>`s long.
   > The video summarization service is not deployed, so this summary was
   > produced by the VLM alone with a generic default prompt. Deploy the
   > `lvs` profile for higher-quality summaries with scenario/events
   > targeting.
2. **API 호출**:
```bash
VLM="${VLM_BASE_URL:-${RTVI_VLM_BASE_URL:-http://${HOST_IP:-localhost}:8018}}"
VLM="${VLM%/v1}"
PROMPT='Describe in detail what is happening in this video,
including all visible people, vehicles, equipments, objects,
actions, and environmental conditions.
OUTPUT REQUIREMENTS:
[timestamp-timestamp] Description of what is happening.
EXAMPLE:
[0.0s-4.0s] <description of the first event>
[4.0s-12.0s] <description of the second event>'

curl -s --max-time 300 -X POST "$VLM/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d "$(jq -n \
        --arg model "${VLM_NAME:-nim_nvidia_cosmos3-nano-reasoner_bf16-final}" \
        --arg text "$PROMPT" \
        --arg url "<clip_url_from_vss_manage_video_io_storage>" \
        '{
          model: $model,
          temperature: 0.0,
          max_tokens: 1024,
          messages: [{
            role: "user",
            content: [
              {type: "text", text: $text},
              {type: "video_url", video_url: {url: $url}}
            ]
          }]
        }')" | jq -r '.choices[0].message.content'
```

---

## 📤 최종 출력 및 렌더링 규칙

### 1. 헤더 작성
`Summary of <video_name> (<duration>)` (예: `Summary of warehouse_clip_01 (3m 30s)`)

### 2. 본문 렌더링
- **LVS 결과**: `video_summary` 내용을 **그대로** 렌더링하십시오. `events` 리스트의 각 항목(시작/종료 시간, 타입, 설명)을 표 또는 리스트 형태로 제공합니다.
- **VLM 결과**: `choices[0].message.content` 내용을 그대로 렌더링하십시오. Cosmos 모델의 `<think>` 블록은 제거하고 `<answer>` 부분만 보여줍니다.
- **금지 사항**: 패러프레이징, 이모지 추가, 임의의 재포맷팅을 엄격히 금지합니다.

---

## 🔍 트러블슈팅 (Troubleshooting)

| 증상 (Symptom) | 원인 (Cause) | 해결책 (Fix) |
|---|---|---|
| `/v1/ready` 가 503을 반복 반환 | LVS 서비스 웜업(Warmup) 중 | 위 Setup의 리트라이 루프대로 최대 30초간 대기 |
| `video_summary` 및 `events` 가 비어있음 | 클립에 요청한 이벤트가 없음 | `scenario` 또는 `events`를 더 넓게 설정하여 재시도 |
| VLM 응답에 `<think>` 블록 포함 | Cosmos 추론 모드 작동 중 | `</think>` 이후의 내용만 렌더링 |
| `curl /v1/ready` 의 표준 출력(stdout)이 비어있음 | 서비스가 바디 없이 200 OK 반환 | `-o /dev/null -w '%{http_code}'` 로 상태 코드만 확인 |

## 🔗 참조 및 연결
- **Upstream**: `vss-deploy-profile` (VSS lvs 프로필 배포)
- **Dependent**: `vss-manage-video-io-storage` (VIOS API 관리)
- **Related**: `vss-search-archive` (아카이브 시맨틱 검색)
- **Detailed Docs**:
    - API 상세: `references/video-summarization-api.md`
    - 배포 및 운영: `references/video-summarization-deployment.md`
    - 환경 변수: `references/video-summarization-environment-variables.md`
    - 디버깅 가이드: `references/video-summarization-debugging.md`
