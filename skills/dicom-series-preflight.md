# 🎴 Skill Card: dicom-series-preflight (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `dicom-series-preflight`
- **도메인**: NVIDIA MedTech / DICOM Preflight Analysis
- **설명**: 변환 또는 추론 단계 이전에 단일 DICOM 시리즈 폴더의 헤더만을 검사하는 프리플라이트(Preflight) 도구입니다.
- **핵심 목표**: DICOM 시리즈의 인벤토리, 방향성(Orientation) Axcodes, PHI 존재 여부 및 기타 소견을 신속하게 스캔하여 `pass`, `warn`, `fail` 형태의 판정(`preflight.verdict`)을 내리는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 픽셀 데이터를 디코딩하지 않고 헤더만 검사하므로 매우 빠르게 작동하며, 후속 작업의 적합성을 판단하는 게이트(Gate) 역할을 합니다.

### 1. 기본 실행 방법 (CLI)
DICOM 시리즈가 저장된 디렉토리를 대상으로 실행합니다.
\`\`\`bash
# DICOM 시리즈 디렉토리 스캔
python scripts/preflight_series.py PATH_TO_DICOM_DIR
\`\`\`

### 2. 에이전트 환경에서의 실행 (run_script)
호스트 에이전트의 `run_script` 기능을 사용하여 정형화된 실행을 수행합니다.
\`\`\`python
run_script("scripts/preflight_series.py", args=["PATH_TO_DICOM_DIR"])
\`\`\`

### 3. 신뢰 실행 및 워크플로우 검증
신뢰할 수 있는 실행 팩을 생성하거나 정의된 워크플로우 게이트를 통해 검증합니다.
\`\`\`bash
# 신뢰 실행 (Trusted Run)
make run-trusted SKILL=dicom_series_preflight \\
  FIXTURE=skills/dicom-series-preflight/fixtures/clean_no_phi \\
  OUT=runs/dicom_preflight_demo

# 워크플로우 게이트 실행
make run-workflow \\
  WORKFLOW=examples/workflows/dicom_preflight_gate.yaml \\
  WORKFLOW_INPUT=skills/dicom-series-preflight/fixtures/clean_no_phi \\
  WORKFLOW_OUT=runs/dicom_preflight_gate
\`\`\`

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 런타임 요구사항
- **필수 라이브러리**: `skill_manifest.yaml`의 `runtime.side_effects.pip_packages`에 정의된 패키지들.
- **입력 데이터**: 단일 DICOM 시리즈 폴더 (`dicom_dir`).
- **출력 형식**: JSON (`preflight_json`).

### 2. 검증 프로세스 (Quality Gate)
출력된 JSON 결과는 다음 검증기를 통해 신뢰성을 확보해야 합니다.
- **검증기**: `verifiers/dicom_preflight_quality_v1`

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **헤더 전용 검사**: 본 스킬은 헤더만 검사합니다. 픽셀 데이터를 디코딩하거나 이미지 내에 내장된(Burnt-in) PHI를 감지하려 하지 마십시오.
2. **래퍼 유지**: 제공된 래퍼 스크립트를 그대로 사용하십시오. 임의로 구현한 엔트리포인트로 교체하는 것을 금지합니다.
3. **단일 디렉토리 원칙**: 한 폴더 내에 하나의 시리즈만 있다고 가정합니다. 트리 구조 내의 여러 스터디(Study)를 자동으로 조정(Reconcile)하지 않습니다.

### 🚫 주요 제한 사항 (Limitations)
- **방향성 가정**: 정준 방향성 게이트(Canonical orientation gate)는 LPS 기반 CT axcodes (L, P, S)를 가정합니다.
- **미지원 포맷**: 압축된 전송 구문(Compressed transfer syntax) 및 멀티프레임 인스턴스는 경고를 발생시키며, 디코딩되지 않습니다.
- **사용 금지 영역**: 임상 배치, 규제 준수용 비식별화, 자율 진단, 검증된 컨버터 없는 프로덕션 인제스천(Ingestion)에 사용해서는 안 됩니다.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **Dependency/Import Error** | `skill_manifest.yaml`과 현재 환경의 런타임 패키지 불일치 | 매니페스트에 선언된 패키지를 재설치하거나 문서화된 셋업 명령어를 실행하십시오. |
| **Empty/Invalid JSON Output** | 잘못된 입력 경로, 지원되지 않는 모달리티 또는 상위 프로세스 실패 | 알려진 픽스처(Fixture) 파일로 재실행하여 래퍼 JSON 및 stderr를 확인하십시오. |
| **Validation Gate Failure** | 출력값이 정의된 엔지니어링 불변성(Invariant)을 위반함 | 실패한 Evidence Pack을 보관하고, 게이트 메시지를 참조하여 입력값 또는 래퍼 코드를 수정하십시오. |
