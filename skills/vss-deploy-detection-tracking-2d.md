# 🎴 Skill Card: vss-deploy-detection-tracking-2d (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `vss-deploy-detection-tracking-2d`
- **도메인**: VSS (Video Search and Summarization)
- **설명**: RTVI-CV(Real-Time Video Intelligence CV) 2D 탐지 및 추적 마이크로서비스를 배포, 운영, 디버깅하고 관련 REST API를 호출합니다.
- **핵심 목표**: `rtvi-cv` (metropolis_perception_app) 컨테이너를 구축하고, 스트림 관리, 헬스 체크, 메트릭 수집 및 텍스트 임베딩 생성을 수행하는 것.

## ⚠️ 제약 사항 및 주의사항
- **대상 구분**: 본 스킬은 2D 탐지/추적 전용입니다. VLM, 임베딩 전용 서비스 또는 분석 API가 필요한 경우 각각 `vss-deploy-dense-captioning` 또는 `vss-setup-video-analytics-api` 스킬을 사용하십시오.
- **리소스 요구사항**: NVIDIA dGPU (T4, A100, L40, H100, B200, RTX) 또는 Jetson/SBSA 플랫폼이 필요하며, 선택한 모델(Sparse4D, GDINO, RT-DETR 등)에 맞는 GPU 메모리가 확보되어야 합니다.
- **보안**: `NGC_CLI_API_KEY` 및 `NVIDIA_API_KEY`는 민감한 정보입니다. `.env` 파일 등을 통해 안전하게 관리하십시오.
- **권한**: Docker 및 sudo 권한이 필요합니다. 비대화형 환경에서는 `sudo -n` 가드를 사용하여 권한 부족 시 즉시 중단하고 사용자에게 요청하십시오.

## 🛠️ 실행 워크플로우 (Action Routing)

사용자의 의도에 따라 다음 두 가지 주요 흐름 중 하나를 선택하여 실행합니다.

### Flow A: 배포, 중단 및 디버깅 (DEPLOY / TEARDOWN / DEBUG)
**의도**: "rtvi-cv 배포해줘", "perception 컨테이너 중지해", "rtvi-cv 로그 확인해줘" 등.
**참조 문서**: `references/deploy-vss-detection-tracking-2d.md`

1. **배포 준비**: `assets/deploy-defaults.yml`을 기반으로 플랫폼(x86, SBSA, Jetson)과 유스케이스(Warehouse, SmartCity 등)를 결정합니다.
2. **단계별 실행 (Strict Sequence)**:
    - **Step 1 (Deploy targets)**: 타겟 하드웨어, 모델, 이미지 태그 확정.
    - **Step 2 (Pipeline configuration)**: 스트림 개수, Sink 타입(EGL, File 등), 스트림 모드(Static/Dynamic) 설정.
    - **Step 3 (Container)**: `docker run` 명령어를 합성하여 컨테이너 실행.
    - **Step 4 (Apply configuration)**: 컨테이너 내부의 `apply_config.sh`를 호출하여 배치 사이즈, 싱크, 소스 리스트 등을 상세 설정.
    - **Step 5 (Plan & Results)**: 앱 실행 계획을 출력하고, 최종 FPS 및 준비 상태 결과를 확인.
3. **결과 출력 (Box Format)**: 각 단계의 결과는 반드시 **128자 너비의 고정폭 박스(`┌ ─ ┐`)** 형태로 출력해야 합니다.
4. **후속 조치 (Step 6)**: 배포 완료 후 사용자에게 메트릭 확인, 스트림 관리, 로그 확인, 또는 중단 여부를 묻는 메뉴를 제공합니다.

### Flow B: API 사용 (API USAGE)
**의도**: "스트림 추가해줘", "rtvi-cv 헬스체크 해줘", "FPS 알려줘", "임베딩 생성해줘" 등.
**참조 문서**: `references/usage-vss-detection-tracking-2d.md`
**기본 포트**: `9000` (`http://<host>:9000/api/v1`)

1. **헬스 체크**: `/live`, `/ready`, `/startup` 엔드포인트를 통해 서비스 상태를 확인합니다.
2. **스트림 관리**:
    - **추가**: `POST /stream/add` (RTSP/File URI 전달)
    - **제거**: `DELETE /stream/remove/{stream_id}`
    - **조회**: `/stream/get-stream-info`
3. **메트릭 수집**: `/api/v1/metrics`를 통해 GPU 사용량 및 처리 FPS를 확인합니다.
4. **임베딩 생성**: 텍스트 쿼리를 전달하여 비디오 검색을 위한 텍스트 임베딩을 생성합니다.

---

## ⌨️ 주요 헬퍼 스크립트 (Scripts)
모든 스크립트는 `$SKILL_DIR/scripts/` 경로에서 호출하며, `run_script` 도구를 사용합니다.

| 스크립트 | 목적 | 주요 인자 |
|---|---|---|
| `load_defaults.sh` | 플랫폼 감지 및 YAML 기본값 로드 | `--usecase <name>` |
| `fetch_resources.sh` | NGC 리소스 다운로드 및 추출 | `--ngc-ref <ref>` |
| `apply_in_container.sh` | 실행 중인 컨테이너에 설정 적용 (Wrapper) | `<container_name>` |
| `start_app_in_container.sh` | 컨테이너 내부 앱 실행 및 대기 (Wrapper) | `<container_name>` |
| `add_streams.sh` | REST API를 통한 스트림 추가 | `<uri>...` |
| `collect_metrics.sh` | `/api/v1/metrics` 스냅샷 수집 | - |
| `synthesize_docker_run.sh` | 플랫폼별 최적화된 `docker run` 라인 생성 | - |

---

## 🔍 트러블슈팅 (Troubleshooting)

| 증상 (Symptom) | 원인 (Cause) | 해결책 (Fix) |
|---|---|---|
| Connection Refused | `rtvi-cv` 서비스 미실행 | `/docs` 또는 `/health` 확인 후 `vss-deploy-detection-tracking-2d` 재수행 |
| HTTP 401/403 (NGC) | API Key 누락 또는 만료 | `docker login nvcr.io` 수행 및 `NGC_CLI_API_KEY` 재설정 |
| Container OOM | GPU 메모리 부족 | 모델 변체를 더 작은 것으로 변경하거나 다른 GPU 프로세스 종료 |
| 스트림 추가 실패 | RTSP URL 유효하지 않음 | `ffprobe` 등으로 스트림 접근 가능 여부 선검증 |
| `sudo` 프롬프트 발생 | 비대화형 환경에서 sudo 권한 부족 | `sudo -n` 사용. 실패 시 사용자에게 수동 실행 요청 |

## 🔗 참조 및 연결
- **vss-deploy-profile**: 전체 VSS 스택 배포 가이드.
- **vss-deploy-dense-captioning**: RT-VLM 기반 캡셔닝 서비스 배포.
- **Detailed Docs**:
    - 배포 상세 워크플로우: `references/deploy-vss-detection-tracking-2d.md`
    - API 사용 상세 가이드: `references/usage-vss-detection-tracking-2d.md`
    - 설정 적용 상세 리스트: `references/apply-config.md`
