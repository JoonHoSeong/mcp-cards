# 🎴 Skill Card: vss-ask-video (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `vss-ask-video`
- **도메인**: VSS (Video Search and Summarization)
- **설명**: VSS Agent의 `video_understanding` 도구를 사용하여 비디오 클립에 대해 시각적인 질문(Visual Q&A)을 수행합니다.
- **핵심 목표**: 기존 요약이나 메타데이터만으로는 알 수 없는, **실제 픽셀 데이터 기반의 구체적인 시각적 사실**을 VLM(Vision-Language Model)을 통해 확인하는 것.

## ⚠️ 제약 사항 및 주의사항
- **사용 시점**: 
    - 사용자가 "비디오에서 무슨 일이 일어나는가?", "특정 객체의 색상은 무엇인가?", "안전 수칙을 지켰는가?" 등 **시각적 확인**이 필요한 질문을 했을 때 사용합니다.
    - 이미 데이터베이스, MCP 결과, 또는 이전 대화 요약에 답이 있는 경우에는 이 스킬을 사용하지 마십시오 (단, 사용자가 비디오를 통한 **재검증**을 요청한 경우는 제외).
- **프로필 요구사항**: `video_understanding` 도구가 포함된 VSS 프로필(보통 `base` 또는 `lvs`)이 배포되어 있어야 합니다.

## 🛠️ 실행 워크플로우

### Step 0: 배포 상태 확인 (Deployment Probe)
VSS Agent가 실행 중인지 확인합니다.
```bash
# HOST_IP는 VSS Agent가 리스닝 중인 호스트 주소
curl -sf --max-time 5 "http://${HOST_IP}:8000/docs" >/dev/null
```
- **성공**: 다음 단계로 진행합니다.
- **실패**: 사용자에게 알리고 `vss-deploy-profile -p base` (또는 `lvs`)를 통해 배포를 먼저 수행하십시오.

### Step 1: 센서 존재 여부 확인 (Sensor Prerequisite)
`/generate` 호출 전, 대상 비디오(센서)가 VST storage에 실제로 존재하는지 반드시 확인해야 합니다. (사용자가 이미 업로드했다고 주장하더라도 생략하지 마십시오.)

1. **센서 리스트 조회**:
   ```bash
   curl -sf --max-time 5 "http://${HOST_IP}:30888/vst/api/v1/sensor/list" | jq '.[].name'
   ```
2. **매칭 확인**: 반환된 이름 목록과 사용자가 제공한 `<sensor-id>`(또는 파일명 stem)를 비교합니다.
3. **업로드 처리 (필요 시)**: 매칭되는 센서가 없다면 비디오를 먼저 업로드하십시오.
   ```bash
   # <filename>: 공백 없는 파일명, <timestamp>: ISO 8601 UTC (기본값: 2025-01-01T00:00:00.000Z)
   curl -s -X PUT "http://${HOST_IP}:30888/vst/api/v1/storage/file/<filename>?timestamp=<timestamp>" \
     -H "Content-Type: application/octet-stream" \
     -H "Content-Length: <file_size_in_bytes>" \
     --upload-file /path/to/<filename> | jq .
   ```
   *(상세 업로드 시맨틱은 `vss-manage-video-io-storage` 스킬 참조)*

### Step 2: VSS Agent 쿼리 및 응답 추출 (Agent Workflow)
센서 존재가 확인되면 VSS Agent에게 `video_understanding` 도구 사용을 요청합니다.

1. **쿼리 실행**:
   ```bash
   export VSS_AGENT_BASE_URL="http://localhost:8000"

   curl -s -X POST "${VSS_AGENT_BASE_URL}/generate" \
     -H "Content-Type: application/json" \
     -d '{"input_message": "Call video_understanding tool to answer the following question about <sensor-id>: <user query>"}' | jq .
   ```

2. **응답 정제 (Response Cleaning)**:
   `/generate` 응답의 `.value` 필드에는 `<agent-think>...</agent-think>` 블록(추론 과정)이 포함되어 있습니다. 사용자에게는 **최종 답변만** 제공해야 합니다.
   
   **정제 명령어 (Python/Bash 조합)**:
   ```bash
   curl -s -X POST "${VSS_AGENT_BASE_URL}/generate" \
     -H "Content-Type: application/json" \
     -d '{"input_message": "Call video_understanding tool to answer the following question about <sensor-id>: <user query>"}' \
     | jq -r '.value' \
     | python3 -c 'import re,sys; t=sys.stdin.read(); t=re.sub(r"<agent-think>.*?</agent-think>\s*", "", t, flags=re.S); print(t.strip())'
   ```

---

## 🔍 트러블슈팅 (Troubleshooting)

| 증상 (Symptom) | 원인 (Cause) | 해결책 (Fix) |
|---|---|---|
| Agent 응답이 없음 / Connection Refused | VSS Agent 미배포 또는 포트 불일치 | `http://${HOST_IP}:8000/docs` 접속 확인 후 `vss-deploy-profile` 수행 |
| 센서 목록에 파일이 없음 | 업로드 실패 또는 파일명 불일치 | VST 업로드 API 호출 후 `sensor/list` 재조회 |
| 응답에 `<agent-think>` 태그가 남아있음 | 정제 스크립트 미적용 | 위에서 제공한 `python3` 정규식 필터를 사용하여 응답을 정제하십시오. |
| 답변이 부정확함 (VLM Hallucination) | 질문이 너무 모호하거나 클립이 너무 김 | 질문을 더 구체적으로 수정하거나, 특정 타임스탬프 구간을 지정하여 다시 질문하십시오. |

## 🔗 참조 및 연결
- **vss-manage-video-io-storage**: 유효한 `VIDEO_URL` 생성을 위한 VST 스토리지 관리.
- **vss-generate-video-report**: 정기 리포트 생성 기능 (본 스킬은 ad-hoc Q&A 전용).
- **vss-deploy-profile**: `base` 또는 `lvs` 프로필 배포 가이드.
