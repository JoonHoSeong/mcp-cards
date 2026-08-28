# 🎴 Skill Card: amc-run-sample-calibration (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `amc-run-sample-calibration`
- **도메인**: NVIDIA AI-IoT / AutoMagicCalib (AMC)
- **설명**: 번들로 제공되는 샘플 데이터셋(`sdg_08_2_sample_data_010926.zip`)을 사용하여 실행 중인 AMC 마이크로서비스의 엔드-투-엔드 기능을 검증하는 샌니티 체크(Sanity Check) 스킬입니다.
- **핵심 목표**: AMC 스택 설치 후 실제 데이터를 투입하기 전, 시스템이 정상적으로 동작하는지 확인하고 기준 성능 지표(Baseline Metrics)를 확보하는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 `amc-setup-calibration-stack`을 통해 서비스가 이미 런칭된 상태를 전제로 합니다.

### 1. [Step 1] 백엔드 탐색 및 연결 확인 (Backend Detection)
스크립트 실행 전, AMC 마이크로서비스가 가동 중인지 확인합니다.
- **동작**: 8000-8009 포트 범위를 스캔하여 `http://localhost:<port>/v1/ready` 엔드포인트가 `{"code": 0}`을 반환하는지 확인합니다.
- **실패 시**: 백엔드가 감지되지 않으면 사용자에게 `amc-setup-calibration-stack` 스킬을 먼저 실행하도록 안내하고 중단합니다.

### 2. [Step 2] 샘플 데이터 준비 (Data Extraction)
검증에 사용할 데이터를 준비합니다.
- **대상 파일**: `assets/sdg_08_2_sample_data_010926.zip`.
- **동작**: 해당 zip 파일을 `assets/.cache/sdg_08_2_sample_data_010926` 경로에 압축 해제합니다. (멱등성 유지: 이미 존재하면 스킵).
- **검증**: 압축 해제 후 `alignment_data/`, `GT.zip`, `videos/` 폴더가 정상적으로 존재하는지 확인합니다.

### 3. [Step 3] 캘리브레이션 스크립트 실행 (Execution)
번들된 Python 스크립트를 통해 API 시퀀스를 자동화하여 실행합니다.
- **실행 파일**: `scripts/run_sample_calibration.py`.
- **주요 파라미터**: `REPO_ROOT` (AMC 체크아웃 경로), `BASE_URL` (MS 주소), `SAMPLE_DIR` (압축 해제 경로).
- **내부 워크플로우**:
    1. `POST /v1/create_project` $\rightarrow$ 프로젝트 생성.
    2. `POST /v1/upload_video_files` $\rightarrow$ 4개의 샘플 비디오 업로드.
    3. `POST /v1/upload_alignment` 및 `upload_layout` $\rightarrow$ 정렬 및 레이아웃 데이터 업로드.
    4. `POST /v1/upload_gt_file` $\rightarrow$ Ground Truth 데이터 업로드.
    5. `POST /v1/verify_project` $\rightarrow$ `project_state: READY` 확인.
    6. `POST /v1/calibrate` $\rightarrow$ 캘리브레이션 시작 (기본 detector: `resnet`).
    7. **폴링**: `GET /v1/get_project_info`를 통해 `COMPLETED` 상태가 될 때까지 대기.

### 4. [Step 4] VGGT 정밀화 (Optional Refinement)
- **조건**: 프로젝트 상태가 `READY`이고 VGGT 모델이 설정되어 있는 경우.
- **동작**: `POST /v1/vggt/calibrate`를 호출하여 정밀도를 높이고, 최종 VGGT 평가 지표를 수집합니다.

### 5. [Step 5] 최종 결과 보고 (Reporting)
실행 완료 후 다음 지표를 사용자에게 보고합니다.
- **성능 지표**: Average L2 distance(m), Average reprojection error 0(px).
- **UI 링크**: 사용자가 웹 UI에서 결과를 직접 확인할 수 있도록 `http://<HOST_IP>:<UI_PORT>` 주소를 제공합니다.

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 진입 게이트
- [ ] `amc-setup-calibration-stack` 완료 및 서비스 가동 중.
- [ ] `assets/sdg_08_2_sample_data_010926.zip` 파일 존재 확인.
- [ ] Python 3 및 `requests` 라이브러리 설치 (없을 경우 스크립트가 임시 venv를 생성하여 자동 해결 시도).

### 2. 데이터셋 특이사항
- 본 스킬은 **합성 데이터(Synthetic Warehouse Cameras)** 4대를 사용하며, Ground Truth(GT)가 포함되어 있어 정량적 평가가 가능합니다.
- 자신의 비디오 파일을 사용하려는 경우 이 스킬이 아닌 `amc-run-video-calibration`을 사용해야 합니다.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **결과 조작 금지**: 캘리브레이션 출력값, 평가 지표, 궤적(Trajectory) 등을 임의로 생성하거나 조작하지 마십시오. 반드시 API 응답값을 그대로 보고하십시오.
2. **데이터 분리**: `sdg_08_2_sample_data_010926.zip` 이외의 다른 데이터셋을 이 스킬의 워크플로우로 처리하지 마십시오.

### 🚫 주요 제한 사항 (Limitations)
- **환경 종속성**: 백엔드 서비스가 실행 중이지 않으면 어떤 단계도 진행할 수 없습니다.
- **VGGT 의존성**: VGGT 모델이 없더라도 기본 AMC 캘리브레이션은 가능하지만, `vggt_state`가 `READY`가 아닐 경우 정밀화 단계는 자동으로 스킵됩니다.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| 백엔드 감지 실패 | 서비스 미가동 또는 포트 불일치 | `amc-setup-calibration-stack`을 재실행하여 서비스를 런칭하십시오. |
| `requests` 모듈 누락 | Python 환경 미설정 | `sudo apt install -y python3-venv python3-pip` 실행 후 다시 시도하십시오. |
| 비디오 파일 0개 발견 | 압축 해제 경로 오류 | `SAMPLE_DIR` 경로가 정확한지, `videos/` 폴더 내에 `cam_*.mp4`가 있는지 확인하십시오. |
| `verify_project` 실패 | 데이터 업로드 누락 | 비디오, 정렬, 레이아웃, GT 파일이 모두 정상적으로 업로드되었는지 확인하십시오. |
| 캘리브레이션 Timeout | 서버 리소스 부족 또는 정지 | `calibration.log`를 확인하여 "insufficient tracklets" 등의 오류가 있는지 분석하십시오. |

## 🚀 다음 단계
샘플 검증이 완료되었다면, 실제 보유하신 데이터셋으로 캘리브레이션을 진행하십시오:
- **커스텀 비디오 파일 사용**: `/amc-run-video-calibration`
- **실시간 RTSP 스트림 사용**: `/amc-run-rtsp-calibration`
