# 🎴 Skill Card: amc-run-video-calibration (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `amc-run-video-calibration`
- **도메인**: NVIDIA AI-IoT / AutoMagicCalib (AMC)
- **설명**: 사전 녹화된 MP4 비디오 파일들을 사용하여 AutoMagicCalib REST API를 통해 새로운 데이터셋의 캘리브레이션을 수행하는 스킬입니다.
- **핵심 목표**: 사용자가 제공한 커스텀 비디오 데이터를 AMC 백엔드에 업로드하고, 정렬(Alignment) 및 레이아웃 설정을 거쳐 최종 캘리브레이션 결과와 평가 지표를 도출하는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 `amc-setup-calibration-stack`을 통해 서비스가 이미 런칭된 상태를 전제로 합니다.

### 1. [Step 1] 프로젝트 생성 및 데이터 업로드
- **프로젝트 생성**: `POST /v1/create_project`를 호출하여 고유한 `project_id`를 확보합니다.
- **비디오 업로드**: `POST /v1/upload_video_files/{project_id}`를 통해 `cam_*.mp4` 파일들을 **알파벳 순으로 정렬하여** 업로드합니다. (업로드 순서가 카메라 인덱스가 됩니다).

### 2. [Step 2] 설정 파일 및 정렬 데이터 해결 (Resolution)
자동 스캔과 사용자 입력을 병행하여 필요한 설정 파일을 확보합니다.
- **대상 파일**: 
    - **Calibration Settings**: `settings.json`, `config.json` 등.
    - **Alignment JSON**: `alignment_data.json`.
    - **Layout PNG**: `layout.png`.
- **해결 프로세스**:
    1. **Auto-scan**: 비디오 디렉토리 및 하위 1단계 폴더를 검색하여 일치하는 파일이 하나만 있을 때 자동 채택합니다.
    2. **사용자 확인**: 검색 결과가 없거나 여러 개인 경우, 사용자에게 명시적 경로를 요청합니다.
    3. **UI Fallback**: 로컬 파일이 없는 경우, 사용자에게 웹 UI의 **Step 3(Parameters)** 및 **Step 4(Alignment)** 단계에서 수동으로 설정을 완료하도록 안내하고 확인을 기다립니다.

### 3. [Step 3] 데이터 업로드 및 검증 (Upload & Verify)
확보된 파일을 API를 통해 업로드하고 프로젝트 상태를 확인합니다.
- **업로드 시퀀스**:
    - `POST /v1/config/{project_id}` $\rightarrow$ 캘리브레이션 설정 적용.
    - `POST /v1/upload_alignment/{project_id}` $\rightarrow$ 정렬 데이터 업로드.
    - `POST /v1/upload_layout/{project_id}` $\rightarrow$ 레이아웃 이미지 업로드.
    - `POST /v1/upload_gt_file/{project_id}` (옵션) $\rightarrow$ Ground Truth zip 업로드 (평가 지표 산출용).
    - `POST /v1/upload_focal_length/{project_id}` (옵션) $\rightarrow$ 초점 거리 오버라이드 적용.
- **검증**: `POST /v1/verify_project/{project_id}`를 호출하여 `project_state: READY` 상태가 되는지 확인합니다.

### 4. [Step 4] 캘리브레이션 실행 및 모니터링 (Calibrate & Poll)
- **계획 확인**: 실행 전, 사용된 **Detector 타입**(`resnet` 또는 `transformer`)과 설정 파일 경로를 요약하여 사용자에게 최종 확인을 받습니다.
- **실행**: `POST /v1/calibrate/{project_id}`를 호출하여 프로세스를 시작합니다.
- **폴링**: `GET /v1/get_project_info/{project_id}`를 10초 간격으로 호출하여 `project_state`가 `COMPLETED`가 될 때까지 대기합니다. (보통 10~60분 소요).

### 5. [Step 5] 결과 분석 및 정밀화 (Analysis & Refinement)
- **결과 수집**: `GET /v1/result/{project_id}/evaluation_statistics`를 통해 L2 distance 및 Reprojection error를 확인합니다.
- **VGGT 정밀화 (Optional)**:
    - **조건**: `vggt_state: READY` 상태인 경우.
    - **동작**: 사용자 확인 후 `POST /v1/vggt/calibrate/{project_id}`를 실행하고, 완료 후 정밀화된 지표를 보고합니다.
- **결과 파일**: 서버의 `projects/project_<id>/output/` 경로에 최종 `camInfo_XX.yaml` 파일들이 생성됩니다.

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 진입 게이트
- [ ] `amc-setup-calibration-stack` 완료 및 서비스 가동 중.
- [ ] `cam_00.mp4`, `cam_01.mp4` 등 시간 동기화된 비디오 파일 확보.
- [ ] (옵션) `GT.zip` 파일 (정량적 평가가 필요한 경우).

### 2. 데이터 가이드라인
- **비디오 규격**: 약 1920x1080 해상도 권장.
- **파일명**: 반드시 `cam_XX.mp4` 형식을 유지하여 업로드 순서와 카메라 인덱스를 일치시켜야 합니다.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **알파벳 순 업로드**: 비디오 파일을 업로드할 때 반드시 알파벳 순으로 정렬하여 전송하십시오. 그렇지 않으면 카메라 인덱스가 뒤섞여 캘리브레이션이 실패합니다.
2. **UI Fallback 확인**: 정렬/레이아웃 데이터가 없는 경우 UI에서 수동 설정을 완료했는지 반드시 확인(`ls` 또는 API 호출)한 후 다음 단계로 진행하십시오.
3. **계획 최종 확인**: `/calibrate` 호출 전, Detector 타입(`resnet` vs `transformer`)을 사용자에게 명시적으로 확인받으십시오.

### 🚫 주요 제한 사항 (Limitations)
- **데이터 프라이버시**: 업로드된 비디오는 REST API를 통해 백엔드로 전송됩니다. 신뢰할 수 있는 네트워크 환경에서만 실행하십시오.
- **처리 시간**: 비디오 길이와 Detector 타입에 따라 완료까지 상당한 시간이 소요되므로, 폴링 루프의 타임아웃을 넉넉하게 설정하십시오.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| `verify_project` $\neq$ `READY` | 필수 데이터 누락 | 비디오, 정렬, 레이아웃 파일이 모두 정상적으로 업로드되었는지 확인하십시오. |
| 수동 정렬 파일 미발견 | UI에서 Save 미클릭 | 사용자가 UI에서 정렬을 마친 후 반드시 'Save'를 클릭했는지 확인하고 `manual_adjustment/` 폴더를 체크하십시오. |
| 캘리브레이션 정체 (`RUNNING` 지속) | 트랙렛 부족 (Static Scene) | `calibration.log`를 확인하여 유효한 특징점 궤적이 충분히 추출되었는지 분석하십시오. |
| 즉각적인 `ERROR` 상태 | 파일명 규칙 위반 | 파일명이 `cam_00.mp4` 형태로 연속적으로 구성되었는지 확인하십시오. |
| L2 거리 낮으나 Reprojection 높음 | 초점 거리 추정 오류 | `upload_focal_length` API를 통해 정확한 Focal Length 값을 오버라이드하십시오. |
| VGGT 상태 `INIT` 유지 | 캘리브레이션 미완료 또는 설정 누락 | 기본 AMC 캘리브레이션이 `COMPLETED`가 된 후 `READY`로 전환됩니다. 설정 확인은 `amc-setup-calibration-stack`을 참조하십시오. |

## 🚀 다음 단계
캘리브레이션이 완료된 프로젝트 ID는 다음 단계의 3D 트래킹 및 분석을 위한 입력값으로 사용됩니다:
- **MV3DT 결과 내보내기**: `GET /v1/result/{project_id}/mv3dt_result?result_type=amc` 호출을 통해 `transforms.yml` 확보.
- **VGGT 정밀 결과 내보내기**: `?result_type=vggt` 옵션으로 정밀화된 결과 확보.
