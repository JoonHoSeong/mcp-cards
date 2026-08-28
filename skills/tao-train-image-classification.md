# 🎴 Skill Card: tao-train-image-classification (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-train-image-classification`
- **도메인**: TAO / Model Training (Image Classification)
- **설명**: PyTorch 기반의 TAO 이미지 분류 모델 학습 스킬입니다. FAN, EfficientNet, ResNet 등 다양한 백본(Backbone)을 지원하며, 배포 최적화를 위한 증류(Distillation) 및 양자화(Quantization) 기능을 포함합니다.
- **핵심 목표**: 고성능 이미지 분류기를 학습시키고, 이를 TensorRT 엔진 등으로 변환하여 실제 추론 환경에 최적화된 형태로 배포하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **이미지 분류 모델 학습**: 특정 도메인의 이미지 데이터를 기반으로 분류 모델을 처음부터 학습시키거나 사전 학습 모델을 파인튜닝할 때.
- **모델 최적화 (증류/양자화)**: 거대 모델(Teacher)의 지식을 작은 모델(Student)로 전이하거나, FP16/INT8 양자화를 통해 추론 속도를 높여야 할 때.
- **모델 평가 및 추론**: 학습된 체크포인트의 정확도(`val_acc_1`)를 검증하거나, 실제 테스트 이미지에 대해 추론을 수행할 때.
- **배포 파일 생성**: 학습된 PyT 모델을 ONNX 또는 TensorRT(`.engine`) 파일로 변환하여 엣지 디바이스에 배포하고자 할 때.

---

## 🏗️ 모델 아키텍처 및 학습 파이프라인

### 1. 지원 백본 (Supported Backbones)
다양한 입력 해상도와 성능 요구사항에 맞춰 백본을 선택할 수 있습니다. (일부 백본은 384, 512, 768 등 비표준 해상도 필요)

| 패밀리 | 지원 타입 (예시) | 특이 사항 |
| :--- | :--- | :--- |
| **FAN** | `fan_tiny`, `fan_small_12_p4_hybrid`, `fan_base_16_p4_hybrid` | 효율적인 엣지 추론 최적화 |
| **GCViT** | `gcvit_tiny` $ightarrow$ `gcvit_large` | 글로벌 컨텍스트 캡처 강화 |
| **FasterViT** | `fastervit_0` $ightarrow$ `fastervit_6` | 고속 비전 트랜스포머 |
| **ViT/EVA/DINO** | `vit_large_patch14_dinov2`, `eva02_large_patch14` | 최신 파운데이션 모델 기반 |
| **SigLIP** | `ViT-H-14-SigLIP-CLIPA-224` | Contrastive Language-Image Pretraining |

### 2. 핵심 액션(Action) 및 데이터 요구사항
모든 액션 실행 시 `spec_overrides`를 통해 데이터 경로를 반드시 명시해야 합니다.

| 액션 (Action) | 필수 데이터 소스 (Spec Key) | 입력 파일 예시 | 목적 |
| :--- | :--- | :--- | :--- |
| **`train`** | `dataset.train_dataset.images_dir`, `dataset.classes_file`, `dataset.val_dataset.images_dir` | `images_train`, `classes.txt`, `images_val` | 기본 모델 학습 |
| **`distill`** | `dataset.train_dataset.images_dir`, `dataset.classes_file`, `dataset.val_dataset.images_dir` | 위와 동일 | 교사-학생 모델 지식 전이 |
| **`quantize`** | `dataset.train_dataset.images_dir`, `dataset.quant_calibration_dataset.images_dir` | `images_train` (Calibration용) | INT8 양자화 최적화 |
| **`evaluate`** | `dataset.val_dataset.images_dir`, `dataset.test_dataset.images_dir` | `images_val`, `images_test` | 체크포인트 성능 검증 |
| **`inference`** | `dataset.test_dataset.images_dir`, `dataset.classes_file` | `images_test`, `classes.txt` | 실제 이미지 클래스 예측 |
| **`export`** | `dataset.root_dir` | 추출된 데이터셋 루트 | ONNX/TensorRT 변환 준비 |

---

## 🛠️ 구현 가이드 및 핵심 워크플로우

### 1. AutoML 실행 정책 (AutoML Policy)
본 모델은 모델 레이어에서 **AutoML이 활성화(`automl_enabled: true`)** 되어 있습니다.
- **기본 동작**: 특별한 요청이 없는 한 `automl_policy: on`으로 설정하여 `tao-run-automl` 스킬을 통해 하이퍼파라미터 최적화(HPO)를 수행합니다.
- **수동 학습**: "AutoML 끄기", "Plain training", "No HPO" 등의 요청이 있을 때만 `automl_policy: off`로 설정하여 직접 학습을 수행합니다.

### 2. 데이터 경로 설정 (Mandatory Overrides)
로컬 Docker 환경에서는 S3 아카이브를 그대로 전달하지 말고, **반드시 압축을 해제한 후 폴더 경로를 지정**하십시오.
- **잘못된 예**: `"dataset.train_dataset.images_dir": "images_train.tar.gz"`
- **올바른 예**: `"dataset.train_dataset.images_dir": "/workspace/data/extracted/train/images_train"`

### 3. 체크포인트 핸드오프 (Checkpoint Handoff)
학습 결과로 생성된 `model_epoch_*.pth` 중 특정 에포크를 선택하여 후속 작업(`evaluate`, `export` 등)에 연결해야 합니다.
- SDK의 리졸버를 사용하여 정확한 에포크 파일을 선택하십시오.
- 명시적 요청이 없는 한 `classifier_model_latest.pth` 심볼릭 링크보다 **특정 에포크 파일**을 사용하는 것이 재현성 측면에서 권장됩니다.

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 하드웨어 요구사항
- **최소**: GPU 1장 (16GB+ VRAM, V100/A100 권장).
- **최적**: GPU 2장 이상. 224x224 해상도, `batch_size=8` 기준 16GB VRAM에서 안정적으로 동작합니다.

### 2. 주요 에러 패턴 및 해결책
| 에러 현상 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **CUDA Out of Memory** | 배치 사이즈 또는 백본 크기 과다 | `dataset.batch_size` 축소 또는 더 작은 백본(예: `fan_tiny`) 선택 |
| **num_classes mismatch** | 클래스 수 설정 불일치 | `dataset.num_classes` 값이 `classes.txt` 및 실제 폴더 수와 일치하는지 확인 |
| **Empty class directory** | 특정 클래스에 이미지 없음 | 모든 클래스 폴더에 최소 1장 이상의 이미지가 포함되어 있는지 확인 |
| **Distill scheduler error** | 잘못된 스케줄러 정책 사용 | `train.optim.policy: step` 설정을 유지하십시오. (`linear`는 현재 버전에서 미지원 가능성) |

## 📚 관련 참조 스킬
- **배포 가이드**: [`tao-deploy-image-classification`](references/tao-deploy-image-classification.md)
- **실행 플랫폼**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **데이터 검증**: [`tao-validate-dataset-format`](../tao-validate-dataset-format/SKILL.md)
