# 🎴 Skill Card: dicom-metadata-extract (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `dicom-metadata-extract`
- **도메인**: NVIDIA MedTech / DICOM Metadata Analysis
- **설명**: 단일 DICOM 파일에서 지정된 메타데이터를 추출하고, 표준 태그 내에 개인 건강 정보(PHI, Protected Health Information)의 존재 여부를 플래그하는 도구입니다.
- **핵심 목표**: DICOM 헤더의 리터럴 값을 정확하게 추출하여 JSON 형태로 제공하고, PHI 포함 여부를 신속하게 식별하는 것입니다. 본 스킬은 익명화(Anonymization)나 임상적 진단 목적이 아닌, 메타데이터 분석 및 검증용으로 설계되었습니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 래퍼(Wrapper) 스크립트를 통해 실행되며, 사용자는 문서화된 명령어를 정확히 사용하여 결과물을 생성해야 합니다.

### 1. 기본 실행 방법 (CLI)
가장 일반적인 실행 방법으로, `scripts/extract_metadata.py`를 직접 호출합니다.
\`\`\`bash
# 기본 실행 (결과는 stdout으로 출력)
python scripts/extract_metadata.py PATH_TO_DICOM

# 결과를 파일로 저장하여 실행
python scripts/extract_metadata.py PATH_TO_DICOM --output result.json
\`\`\`

### 2. 에이전트 환경에서의 실행 (run_script)
호스트 에이전트가 `run_script` 기능을 제공하는 경우, 다음과 같이 호출하여 정형화된 실행을 수행합니다.
\`\`\`python
run_script("scripts/extract_metadata.py", args=["PATH_TO_DICOM", "--output", "result.json"])
\`\`\`

### 3. 신뢰할 수 있는 실행(Trusted Run) 및 검증
증거 기반 리뷰를 위해 `eval_engine`을 통한 신뢰 실행을 권장합니다.
\`\`\`bash
python -m eval_engine.run_trusted skills/dicom-metadata-extract \\
  --fixture skills/dicom-metadata-extract/fixtures/sample_ct.dcm \\
  --out runs/dicom_metadata_trusted
\`\`\`

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 런타임 요구사항
- **필수 라이브러리**: `pydicom` 및 `skill_manifest.yaml`에 정의된 Python 패키지들.
- **입력 데이터**: 유효한 단일 DICOM 파일 (`dicom_path`).
- **출력 형식**: JSON (`metadata_json`).

### 2. 검증 프로세스 (Quality Gate)
추출된 JSON 결과물이 정확한지 확인하기 위해 다음 검증기를 실행해야 합니다.
- **검증기**: `medagent.verifiers.dicom_metadata_quality_v1`
- **절차**: 생성된 Evidence Pack을 검증기에 통과시켜 데이터 무결성을 확인한 후 최종 증거로 채택하십시오.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **리터럴 추출**: DICOM 헤더에 명시된 리터럴 값만 추출하십시오. 모달리티, 신체 부위, 진단명 등에 대해 추론(Inference)하거나 임상적 의미를 부여해서는 안 됩니다.
2. **래퍼 유지**: 제공된 래퍼 스크립트를 그대로 사용하십시오. 임의로 작성한 구현체로 엔트리포인트를 교체하지 마십시오.
3. **출력 정렬**: 모든 출력 필드는 `validators/output_schema.json`에 정의된 스키마와 정확히 일치해야 합니다.

### 🚫 주요 제한 사항 (Limitations)
- **태그 범위**: PS3.15 표준 태그의 일부 서브셋만 지원합니다. 완전한 '기본 애플리케이션 기밀성 프로필(Basic Application Confidentiality Profile)' 구현체가 아닙니다.
- **미지원 항목**:
    - 프라이빗 태그(Private Tags)는 체크하지 않습니다.
    - 이미지 픽셀 내에 내장된(Burnt-in) PHI는 감지할 수 없습니다.
    - 멀티프레임(Multi-frame) 처리 기능이 매우 제한적입니다.
- **사용 금지 영역**: 임상 배치, 규제 준수용 비식별화, 자율 진단, 환자 직접 대면 서비스에 사용해서는 안 됩니다.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **Dependency/Import Error** | `skill_manifest.yaml`에 정의된 런타임 패키지와 현재 환경의 불일치 | 매니페스트에 선언된 패키지를 다시 설치하거나 문서화된 셋업 명령어를 실행하십시오. |
| **Empty/Invalid JSON Output** | 잘못된 입력 경로, 지원되지 않는 모달리티 또는 상위 프로세스 실패 | 알려진 픽스처(Fixture) 파일로 재실행하여 래퍼 JSON 및 stderr를 확인하십시오. |
| **Validation Gate Failure** | 출력값이 정의된 엔지니어링 불변성(Invariant)을 위반함 | 실패한 Evidence Pack을 보관하고, 게이트 메시지를 참조하여 입력값 또는 래퍼 코드를 수정하십시오. |
