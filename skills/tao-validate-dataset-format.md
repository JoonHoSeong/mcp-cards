# 🎴 Skill Card: tao-validate-dataset-format (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-validate-dataset-format`
- **도메인**: TAO / Dataset Management (DAFT)
- **설명**: NVIDIA TAO DAFT(Dataset and Format Toolkit) 데이터셋의 구조, 스키마 및 상호 참조 무결성을 검증하는 도구입니다. 학습 시작 전, 데이터셋이 TAO 모델이 요구하는 엄격한 포맷 규격을 준수하는지 확인하여 런타임 에러를 사전에 방지합니다.
- **핵심 목표**: `tao-daft validate` CLI를 통해 데이터셋 내의 메타데이터-미디어 간 연결성, 필수 파일 존재 여부, 스키마 정합성을 검증하여 **"학습 가능한 상태(Train-Ready)"**임을 보장하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **학습 직전 최종 검증**: `tao-convert-dataset-format`으로 변환을 마친 후, 실제 학습 세션에 투입하기 전.
- **데이터셋 무결성 진단**: 데이터셋 업로드/다운로드 과정에서 파일 누락이나 경로 손상이 의심될 때.
- **스키마 오류 디버깅**: TAO 학습 중 데이터 로딩 관련 에러가 발생하여, 원천 데이터셋의 구조적 결함을 찾아야 할 때.
- **CI/CD 파이프라인 통합**: 데이터셋 업데이트 시 자동으로 포맷 유효성을 검사하여 파이프라인 진입 여부를 결정할 때.

---

## 🏗️ DAFT 검증 아키텍처 (Validation Architecture)

### 1. CLI 구조 및 명령 체계
`tao-daft validate`는 특정 포맷을 서브커맨드로 받는 구조이며, 대상 경로는 반드시 플래그로 전달해야 합니다.

**표준 명령어 형태:**
```bash
tao-daft validate <format> --path <dataset-or-parent-dir>
```

| 구성 요소 | 유형 | 설명 | 주의 사항 |
| :--- | :--- | :--- | :--- |
| `<format>` | positional | 검증할 데이터셋 포맷 슬러그 (예: `metropolis-v3.0`) | `--format` 플래그를 사용하지 말 것 |
| `--path` | flag | 검증 대상 데이터셋 경로 또는 상위 디렉토리 경로 | 반드시 플래그와 함께 전달 |

### 2. 검증 범위 (Validation Scope)
단순한 파일 존재 확인을 넘어 다음의 3단계 검증을 수행합니다:
1. **구조 검증 (Structural)**: `meta.json`, `media/`, `text/` 등 포맷별 필수 디렉토리 및 파일 구조 확인.
2. **스키마 검증 (Schema)**: JSON/YAML 메타데이터 내의 필드 타입, 필수 키 존재 여부 및 값의 범위 확인.
3. **상호 참조 검증 (Cross-Reference)**: 메타데이터에 기재된 미디어 파일 경로가 실제 디스크 상에 존재하는지, 텍스트 레이블과 미디어가 1:1로 매칭되는지 확인.

---

## 🛠️ 구현 가이드 및 워크플로우 (Implementation Workflow)

### 1. 포맷 추론 전략 (Format Inference)
사용자가 포맷을 명시하지 않은 경우, 다음의 **디렉토리 마커(Directory Markers)**를 통해 포맷을 추론합니다. (파일명이 아닌 디렉토리 구조 기준)

| 디렉토리 마커 (Marker) | 추론 포맷 (Inferred Format) | 비고 |
| :--- | :--- | :--- |
| `meta.json` + `media/` + `text/` | `cosmos-reason-v1.0` | VLM/Reasoning 특화 구조 |
| `contextual/` (alongside `raw/`, `task/`) | `metropolis-v3.0` | Metropolis 스마트 시티 특화 구조 |
| 위 마커 모두 없음 | **추론 불가** | 사용자에게 직접 확인 요청 |

### 2. 검증 실행 시퀀스
1. **환경 확인**: `tao-daft --version`으로 SDK 설치 및 버전 확인.
2. **포맷 결정**: 위 추론 전략을 사용하거나 `tao-daft validate --help`에서 지원 포맷 확인 후 결정.
3. **상세 플래그 확인**: `tao-daft validate <format> --help`를 실행하여 해당 포맷 전용 검증 플래그(예: 범위 제한, 엄격 모드 등) 확인.
4. **검증 수행**: `tao-daft validate <format> --path <path>` 실행.
5. **결과 해석**: 하단의 `VALIDATION RESULTS` 블록과 최종 상태(`✅ PASSED` / `❌ FAILED`) 확인.

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 치명적 함정 (Critical Pitfalls)
- **포지셔널 경로 오류**: `--path`를 생략하고 경로를 포지셔널 인자로 전달하면 `argument --path is required` 에러가 발생합니다.
- **포맷 슬러그 불일치**: 설치된 `tao-daft` 버전에서 지원하지 않는 구버전 슬러그를 사용하면 `invalid choice` 에러가 발생합니다. 반드시 최신 `--help`를 확인하십시오.
- **대규모 데이터셋 로그 폭주**: 데이터셋 규모가 클 경우 stdout에 수만 줄의 에러가 출력될 수 있습니다. 이 경우 출력을 파일로 리다이렉션하고 슬라이스 단위로 읽으십시오.

### 2. 에러 대응 매트릭스
| 결과 상태 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **`❌ VALIDATION FAILED`** | 구조/스키마/참조 오류 | `VALIDATION RESULTS` 섹션의 구체적인 에러 메시지(예: `File not found`, `Missing key`) 확인 |
| **`invalid choice`** | 잘못된 포맷 슬러그 | `tao-daft validate --help`에서 정확한 이름 확인 |
| **`command not found`** | SDK 미설치 | `pip install nvidia-tao-daft` 수행 |
| **경고 발생 (Warning)** | 비치명적 불일치 | 학습에 지장이 없는 수준인지 확인하거나, `--strict` 플래그를 통해 경고를 에러로 승격시켜 정밀 수정 |

## 📚 관련 참조 스킬
- **데이터 변환**: [`tao-convert-dataset-format`](../tao-convert-dataset-format/SKILL.md) (검증 실패 시 수정 후 재변환)
- **런타임 플랫폼**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **환경 구축**: [`tao-setup-nvidia-gpu-host`](../tao-setup-nvidia-gpu-host/SKILL.md)
