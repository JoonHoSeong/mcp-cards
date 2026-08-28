# 🎴 Skill Card: tao-train-dino (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-train-dino`
- **도메인**: TAO / Model Training (Object Detection)
- **설명**: DINO (DETR with Improved DeNoising Anchor Boxes) 기반의 2D 객체 탐지 모델 학습 스킬입니다. 트랜스포머 구조에 디노이징(Denoising) 학습과 멀티스케일 피처를 결합하여 매우 높은 정밀도의 탐지 성능을 제공합니다.
- **핵심 목표**: 복잡한 배경이나 겹쳐진 객체가 많은 환경에서 고정밀 객체 탐지를 수행하는 모델을 학습시키고, 이를 TensorRT 엔진으로 최적화하여 배포하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **최고 수준의 탐지 정밀도 필요 시**: 실시간성보다는 정확도(mAP)가 최우선인 고정밀 객체 탐지 태스크를 수행할 때.
- **트랜스포머 기반 탐지 모델 학습**: NMS 과정 없이 End-to-End로 동작하는 최신 DETR 계열 모델을 학습시키고자 할 때.
- **COCO-format 데이터셋 활용**: 표준 COCO 또는 COCO-raw 포맷의 데이터셋을 통해 모델을 학습/검증할 때.
- **지식 증류 및 양자화**: 고성능 Teacher 모델의 지식을 전이하거나, FP16/INT8 양자화를 통해 추론 효율을 높여야 할 때.

---

## 🏗️ DINO 학습 아키텍처 및 요구사항

### 1. 필수 데이터셋 구성 (Mandatory Requirements)
DINO는 실행 시 **반드시 검증 데이터셋이 필요**합니다. 이를 누락하면 `FileNotFoundError`와 함께 즉시 종료됩니다.

| 필수 항목 | 요구사항 | 설명 | 주의 사항 |
| :--- | :--- | :--- | :--- |
| **Train Dataset** | 필수 | COCO 포맷의 학습 데이터 (S3/Local) | `images.tar.gz` + `annotations.json` 구조 |
| **Val Dataset** | **절대 필수** | COCO 포맷의 검증 데이터 | 별도 분할이 없다면 Train URI를 재사용할 것 |
| **`num_classes`** | 필수 | 객체 클래스 수 | 반드시 `max(category_id) + 1` 이상으로 설정 |

### 2. 핵심 액션(Action) 및 데이터 매핑
모든 액션 실행 시 `spec_overrides`를 통해 데이터 소스를 명시적으로 전달해야 합니다.

| 액션 (Action) | 필수 데이터 소스 (Spec Key) | 입력 파일 예시 | 목적 |
| :--- | :--- | :--- | :--- |
| **`train`** | `dataset.train_data_sources`, `dataset.val_data_sources` | `images.tar.gz`, `annotations.json` | 기본 모델 학습 및 검증 |
| **`distill`** | `dataset.train_data_sources`, `dataset.val_data_sources` | 위와 동일 | Teacher-Student 지식 전이 |
| **`quantize`** | `dataset.train_data_sources`, `dataset.val_data_sources` | 위와 동일 | INT8 양자화 캘리브레이션 |
| **`evaluate`** | `dataset.test_data_sources` | `images.tar.gz`, `annotations.json` | 최종 체크포인트 mAP 검증 |
| **`inference`** | `dataset.infer_data_sources` | `images.tar.gz`, `label_map.txt` | 실제 이미지 객체 탐지 및 박스 생성 |
| **`export`** | `export.checkpoint` | `model_epoch_*.pth` | ONNX 파일 생성 |

---

## 🛠️ 구현 가이드 및 핵심 워크플로우

### 1. AutoML 실행 정책 (AutoML Policy)
본 모델은 **AutoML이 활성화(`automl_enabled: true`)** 되어 있습니다.
- **기본 동작**: `automl_policy: on`으로 설정하여 `tao-run-automl`을 통해 `val_mAP` 또는 `mAP50`을 최대화하는 하이퍼파라미터 최적화(HPO)를 수행합니다.
- **수동 학습**: "Plain training", "No HPO" 등의 요청이 있을 때만 `automl_policy: off`로 설정하여 직접 학습을 수행합니다.

### 2. 데이터 소스 구성 (S3 Artifact Rule)
TAO DINO는 표준 아티팩트 구조를 기대합니다.
- **표준 구조**: `images.tar.gz` (이미지 압축 파일) + `annotations.json` (라벨 파일).
- **동작 방식**: SDK가 `images.tar.gz`를 다운로드하면 자동으로 압축을 해제하고, 파일 이름(stem)과 동일한 폴더(`images`)로 런타임 경로를 재작성합니다. 사용자는 원본 아카이브 파일명을 지정하십시오.

### 3. 체크포인트 및 하이퍼파라미터 설정
- **백본 설정**: 기본값은 `resnet_50`이며, `model.pretrained_backbone_path`를 통해 사전 학습된 가중치를 지정할 수 있습니다.
- **학습 주기**: 빠른 이터레이션은 10 epoch, 실제 프로덕션 데이터셋은 30-50 epoch 이상의 학습이 권장됩니다.
- **배치 사이즈**: `batch_size=4`가 기본이며, GPU VRAM 상황에 따라 조절하십시오.

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 치명적 함정 (Critical Pitfalls)
- **검증 데이터셋 누락**: `val_data_sources`를 생략하면 모델이 로드되기 전 `FileNotFoundError`가 발생합니다. 반드시 지정하십시오.
- **클래스 수 설정 오류**: `num_classes`를 실제 카테고리 ID보다 낮게 설정하면 `CUDA error: device-side assert triggered` 에러가 발생하며 프로세스가 강제 종료됩니다. 반드시 `max(category_id) + 1`로 설정하십시오.
- **멀티 GPU 일관성**: `train.num_gpus`를 늘릴 때 `train.gpu_ids`를 동일한 범위(예: `[0, 1, 2, 3]`)로 설정하지 않으면 분산 학습 시작 단계에서 타임아웃 또는 일관성 오류가 발생할 수 있습니다.

### 2. 주요 에러 대응 매트릭스
| 에러 현상 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **CUDA Out of Memory** | 배치 사이즈 과다 | `batch_size`를 4 이하로 축소하거나 더 가벼운 백본 선택 |
| **`FileNotFoundError` (Startup)** | `val_data_sources` 누락 | 학습 데이터 경로를 복제하여 `val_data_sources`에 할당 |
| **Device-side assert** | `num_classes` 부족 | `dataset.num_classes` 값을 `max(category_id) + 1`로 상향 조정 |
| **NCCL Timeout** | 멀티 GPU/노드 설정 불일치 | `train.gpu_ids` 설정 확인 및 `WORLD_SIZE`, `NODE_RANK` 환경 변수 검증 |

## 📚 관련 참조 스킬
- **배포 가이드**: [`tao-deploy-dino`](references/tao-deploy-dino.md) (TensorRT 엔진 생성 및 추론)
- **실행 플랫폼**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **데이터 검증**: [`tao-validate-dataset-format`](../tao-validate-dataset-format/SKILL.md)
