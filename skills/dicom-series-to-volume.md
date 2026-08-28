# 🎴 Skill Card: dicom-series-to-volume (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `dicom-series-to-volume`
- **도메인**: NVIDIA MedTech / DICOM to NIfTI Conversion
- **설명**: 하나의 CT DICOM 시리즈 폴더를 HU(Hounsfield Unit) NIfTI 볼륨과 아핀(Affine) 증거 데이터로 변환하는 도구입니다.
- **핵심 목표**: DICOM 슬라이스를 `ImagePositionPatient` 기준으로 정렬하고, `RescaleSlope` 및 `RescaleIntercept`를 적용하여 정확한 HU 값을 가진 NIfTI 파일(`.nii.gz`)과 상세 요약 JSON을 생성하는 것입니다.

## ⚙️ 구현 및 실행 가이드

### 1. 기본 실행 방법 (CLI)
DICOM 시리즈 폴더를 입력받아 NIfTI 볼륨을 생성합니다.
\`\`\`bash
python scripts/series_to_volume.py PATH_TO_DICOM_DIR --output PATH_TO_OUT.nii.gz
\`\`\`

### 2. 에이전트 환경에서의 실행 (run_script)
호스트 에이전트의 `run_script` 기능을 사용하는 경우:
\`\`\`python
run_script("scripts/series_to_volume.py", args=["PATH_TO_DICOM_DIR", "--output", "PATH_TO_OUT.nii.gz"])
\`\`\`

### 3. 신뢰 실행 및 검증
변환된 볼륨의 품질을 보장하기 위해 `dicom_volume_quality_v1` 검증기와 함께 실행합니다.
\`\`\`bash
python -m eval_engine.run_trusted skills/dicom-series-to-volume \\
  --fixture PATH_TO_DICOM_DIR \\
  --out runs/dicom_series_to_volume_trusted
\`\`\`

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 런타임 요구사항
- **필수 라이브러리**: `skill_manifest.yaml`에 정의된 Python 패키지들.
- **입력 데이터**: 단일 CT DICOM 시리즈 폴더 (`dicom_dir`).
- **출력 데이터**: NIfTI 볼륨 (`nifti_volume`) 및 결과 JSON (`result_json`).

### 2. 주요 출력 필드 (Key Output Fields)
결과 JSON에서 다음 필드를 확인하여 변환 무결성을 검증하십시오.
- `n_slices`: 처리된 슬라이스 수
- `series_instance_uid`: DICOM 시리즈 고유 ID
- `output.shape` / `output.spacing`: 생성된 볼륨의 크기와 간격
- `output.axcodes` / `output.affine`: 방향성 코드 및 아핀 행렬
- `hu_range`: HU 값의 범위
- `runtime.conversion_seconds`: 변환 소요 시간

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **단일 시리즈 원칙**: 하나의 폴더에는 하나의 시리즈만 포함되어야 합니다. 멀티 시리즈 입력은 프리플라이트 단계에서 거부됩니다.
2. **래퍼 유지**: 제공된 래퍼 스크립트를 그대로 사용하십시오. 임의로 구현한 엔트리포인트로 교체하는 것을 금지합니다.
3. **방향성 확인**: 본 스킬은 픽셀 재방향성(Reorientation)을 수행하지 않습니다. NIfTI/RAS 좌표계로 변환된 아핀 행렬을 기반으로, 후속 단계(예: `expected_axcodes` 게이트)에서 반드시 방향성을 검증한 후 세그멘테이션 모델에 입력하십시오.

### 🚫 주요 제한 사항 (Limitations)
- **멀티프레임 미지원**: 파일당 프레임 수가 1개 초과인 멀티프레임 DICOM은 지원하지 않습니다.
- **압축 전송 구문 미지원**: JPEG, JPEG2000, RLE 등 압축된 전송 구문은 지원하지 않습니다.
- **사용 금지 영역**: 임상 배치, 자율 진단, 규제 제출용으로 사용해서는 안 됩니다. 프로덕션 수준의 변환이 필요한 경우 `dcm2niix`와 같이 검증된 컨버터를 사용하십시오.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **Dependency/Import Error** | `skill_manifest.yaml`과 현재 환경의 런타임 패키지 불일치 | 매니페스트에 선언된 패키지를 재설치하거나 문서화된 셋업 명령어를 실행하십시오. |
| **Empty/Invalid JSON Output** | 잘못된 입력 경로, 지원되지 않는 모달리티 또는 상위 프로세스 실패 | 알려진 픽스처(Fixture) 파일로 재실행하여 래퍼 JSON 및 stderr를 확인하십시오. |
| **Validation Gate Failure** | 출력값이 정의된 엔지니어링 불변성(Invariant)을 위반함 | 실패한 Evidence Pack을 보관하고, 게이트 메시지를 참조하여 입력값 또는 래퍼 코드를 수정하십시오. |
