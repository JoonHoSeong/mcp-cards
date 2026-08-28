# 🎴 Skill Card: tao-convert-dataset-format (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-convert-dataset-format`
- **도메인**: TAO / Dataset Management (DAFT)
- **설명**: NVIDIA TAO DAFT(Dataset and Format Toolkit) 데이터셋을 지원되는 다양한 포맷 간에 변환하는 도구입니다. VLM(Vision-Language Model) 학습, Cosmos-Reason 등의 최신 AI 모델 학습을 위해 데이터셋의 구조를 변경하거나 특정 모델이 요구하는 포맷으로 패키징할 때 사용합니다.
- **핵심 목표**: `tao-daft convert` CLI를 통해 데이터셋의 논리적 구조(메타데이터, 레이블, 미디어 경로 등)를 유지하면서, 타겟 모델이 요구하는 엄격한 포맷 규격으로 변환하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **VLM 학습 데이터 준비**: QA, 요약, 시계열 분석 등의 태스크를 VLM 학습용 포맷으로 패키징해야 할 때.
- **포맷 마이그레이션**: 구버전 DAFT 데이터셋을 최신 버전(`metropolis-v3.0`, `cosmos-reason-v1.0` 등)으로 업데이트해야 할 때.
- **학습 전처리**: `meta.json` 스타일의 학습 세트를 생성하여 TAO 학습 파이프라인에 입력하기 전.
- **데이터셋 구조 변경**: 데이터셋의 소스 포맷을 유지하면서 타겟 포맷의 제약 조건(미디어 처리 방식 등)에 맞춰 변환이 필요할 때.

---

## 🏗️ DAFT 변환 아키텍처 (Conversion Architecture)

### 1. CLI 구조 및 명령 체계
`tao-daft`는 중첩된 argparse 서브커맨드 구조를 가집니다. 명령어의 순서와 플래그 사용법이 매우 엄격합니다.

**표준 명령어 형태:**
```bash
tao-daft convert <source-format> <target-format> --path <input_path> --output <output_path>
```

| 구성 요소 | 유형 | 설명 | 주의 사항 |
| :--- | :--- | :--- | :--- |
| `<source-format>` | positional | 소스 데이터셋 포맷 슬러그 (예: `metropolis-v2.0`) | `--from` 플래그를 사용하지 말 것 |
| `<target-format>` | positional | 타겟 데이터셋 포맷 슬러그 (예: `cosmos-reason-v1.0`) | `--to` 플래그를 사용하지 말 것 |
| `--path` | flag | 변환할 소스 데이터셋 또는 상위 디렉토리 경로 | 반드시 플래그와 함께 전달 |
| `--output` | flag | 변환된 데이터셋이 저장될 경로 | 반드시 플래그와 함께 전달 |

### 2. 포맷 슬러그 (Format Slugs)
포맷 이름은 버전이 포함된 소문자, 점-구분 형식(예: `metropolis-v3.0`)을 사용합니다. 지원되는 포맷 리스트는 런타임에 `--help`를 통해 동적으로 확인해야 합니다.

---

## 🛠️ 구현 가이드 및 워크플로우 (Implementation Workflow)

### 1. 정밀 탐색 시퀀스 (Discovery Sequence)
설치된 버전마다 지원 포맷이 다르므로, 추측하지 말고 다음 순서로 탐색하십시오.

1. **버전 확인**: `tao-daft --version` $ightarrow$ 설치 상태 및 버전 핀 고정.
2. **소스 포맷 확인**: `tao-daft convert --help` $ightarrow$ 현재 지원하는 모든 소스 포맷 리스트 확인.
3. **타겟 포맷 확인**: `tao-daft convert <source> --help` $ightarrow$ 해당 소스에서 변환 가능한 모든 타겟 포맷 확인.
4. **최종 플래그 확인**: `tao-daft convert <source> <target> --help` $ightarrow$ 해당 변환 쌍에 특화된 추가 플래그(미디어 복사 방식, 태스크 서브셋 등) 확인.

### 2. 변환 실행 및 결과 처리
- **계층적 변환**: `--path`에 단일 데이터셋뿐만 아니라 **상위 디렉토리**를 지정하면, 컨버터가 하위 트리를 모두 탐색하며 일괄 변환을 수행합니다.
- **사후 검증**: 변환 완료 후 반드시 `tao-validate-dataset-format` 스킬을 사용하여 구조적/의미적 무결성을 검증하십시오.

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 치명적 함정 (Critical Pitfalls)
- **위치 인자 오류**: `<source>`와 `<target>`을 플래그(`--from`, `--to`)로 전달하면 명령어가 실패합니다. 반드시 **포지셔널 인자**로 전달하십시오.
- **경로 플래그 누락**: `--path`와 `--output`을 포지셔널 인자로 전달하면 실패합니다. 반드시 **플래그**로 전달하십시오.
- **비-DAFT 데이터**: COCO, YOLO, JSONL 등 일반 포맷 $ightarrow$ DAFT 변환은 본 스킬의 범위가 아닙니다. 해당 작업은 `nvidia-tao-daft` 상위 레포지토리의 전용 컨버터를 사용하십시오.

### 2. 에러 코드별 대응
| 에러 메시지 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **`tao-daft: command not found`** | 패키지 미설치 | `pip install nvidia-tao-daft` 설치 후 확인 |
| **`invalid choice: '<format>'`** | 지원되지 않는 포맷 | `tao-daft convert --help`로 현재 버전의 정확한 슬러그 확인 |
| **`argument --path/--output is required`** | 플래그 누락/위치 오류 | `--path` 및 `--output` 플래그가 명시적으로 포함되었는지 확인 |
| **Validation 실패** | 플래그 설정 오류 | `tao-daft convert <source> <target> --help`에서 미디어 핸들링 또는 태스크 서브셋 플래그를 다시 확인 |

## 📚 관련 참조 스킬
- **데이터 검증**: [`tao-validate-dataset-format`](../tao-validate-dataset-format/SKILL.md) (변환 후 필수 단계)
- **런타임 플랫폼**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **환경 구축**: [`tao-setup-nvidia-gpu-host`](../tao-setup-nvidia-gpu-host/SKILL.md)
