# 🎴 Skill Card: tao-train-rtdetr (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-train-rtdetr`
- **도메인**: TAO / Model Training (Object Detection)
- **설명**: 2D 객체 탐지를 위한 RT-DETR (Real-Time DEtection TRansformer) 학습 스킬입니다. 높은 정확도를 유지하면서 실시간 추론이 가능하도록 설계되었으며, 배포 최적화를 위한 증류(Distillation) 및 양자화(Quantization)를 지원합니다.
- **핵심 목표**: 저지연(Low-latency)과 고정밀도를 동시에 갖춘 객체 탐지 모델을 학습시키고, 이를 TensorRT 엔진으로 최적화하여 실시간 서비스 환경에 배포하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **실시간 객체 탐지 모델 학습**: 추론 속도가 매우 중요한 실시간 애플리케이션을 위해 RT-DETR 모델을 학습시킬 때.
- **고성능 DETR 기반 학습**: 기존의 NMS(Non-Maximum Suppression) 과정 없이 End-to-End로 객체를 탐지하는 트랜스포머 기반 모델이 필요할 때.
- **모델 경량화 및 최적화**: Teacher 모델의 지식을 전이하는 증류(Distillation)나 INT8 양자화를 통해 엣지 디바이스 성능을 극대화해야 할 때.
- **COCO 포맷 데이터 학습**: COCO 또는 COCO-raw 포맷의 데이터셋을 사용하여 객체 탐지 학습을 수행할 때.

---

## 🏗️ RT-DETR 학습 아키텍처

### 1. 핵심 액션(Action) 및 데이터 요구사항
RT-DETR은 데이터 소스를 리스트 형태(`List[Dict]`)로 전달하는 특성이 있어, `spec_overrides` 작성 시 특히 주의해야 합니다.

| 액션 (Action) | 필수 데이터 소스 (Spec Key) | 입력 파일 예시 | 목적 |
| :--- | :--- | :--- | :--- |
| **`train`** | `dataset.train_data_sources`, `dataset.val_data_sources` | `images.tar.gz`, `annotations.json` | 기본 모델 학습 |
| **`distill`** | `dataset.train_data_sources`, `dataset.val_data_sources` | 위와 동일 | Teacher 모델 기반 지식 전이 |
| **`quantize`** | `dataset.train_data_sources`, `dataset.val_data_sources`, `dataset.quant_calibration_data_sources` | `images.tar.gz`, `annotations.json` | INT8 양자화 캘리브레이션 |
| **`evaluate`** | `dataset.test_data_sources` | `images.tar.gz`, `annotations.json` | mAP 성능 검증 |
| **`inference`** | `dataset.infer_data_sources` | `images.tar.gz`, `label_map.txt` | 실제 이미지 객체 탐지 및 박스 생성 |
| **`export`** | `export.checkpoint` | `model_epoch_*.pth` | ONNX 파일 생성 |

### 2. 런타임 실행 방식 (Launch Method)
RT-DETR은 Lightning-managed 방식이 아닌 **`torchrun`** 기반으로 실행됩니다.
- **실행 명령**: `torchrun --nnodes=N --nproc-per-node=M train.py`
- **멀티 GPU 설정**: `train.num_gpus`와 `train.gpu_ids`를 반드시 동일한 범위로 설정해야 합니다. (예: 8 GPU 사용 시 `num_gpus: 8`, `gpu_ids: [0,1,2,3,4,5,6,7]`)
- **분산 전략**: `ddp`(기본) 또는 `fsdp`(FP16 강제)를 지원합니다.

---

## 🛠️ 구현 가이드 및 핵심 워크플로우

### 1. AutoML 및 하이퍼파라미터 정책
본 모델은 **AutoML이 활성화(`automl_enabled: true`)** 되어 있습니다.
- **기본 라우팅**: `automl_policy: on` 설정 시 `tao-run-automl` 스킬을 통해 `val_mAP` 또는 `mAP50`을 최대화하는 방향으로 최적화합니다.
- **수동 제어**: `automl_policy: off` 설정 시 사용자가 정의한 하이퍼파라미터로 직접 학습합니다.

### 2. 데이터 소스 구성 (Mandatory Spec Overrides)
데이터 소스는 반드시 다음과 같은 딕셔너리 구조로 전달해야 합니다.

**Train 액션 예시:**
```python
{
    "dataset.num_classes": "<num_classes> + 1", # 배경 클래스 포함
    "dataset.train_data_sources": [
        {"image_dir": "s3://bucket/train/images.tar.gz", "json_file": "s3://bucket/train/annotations.json"}
    ],
    "dataset.val_data_sources": {
        "image_dir": "s3://bucket/val/images.tar.gz", "json_file": "s3://bucket/val/annotations.json"
    },
}
```

### 3. 증류(Distillation) 바인딩 설정
RT-DETR 증류 시에는 반드시 **IOU 특성 경로**를 바인딩해야 합니다. DINO 스타일의 `pred_logits`를 사용하면 실패합니다.
- **올바른 바인딩**: `student_module_name: srcs`, `teacher_module_name: srcs`, `criterion: IOU`

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 치명적 함정 (Critical Pitfalls)
- **카테고리 ID 불일치**: COCO ID가 0부터 시작하는 연속적인 값이 아닐 경우, `dataset.num_classes`를 `max(category_id) + 1`로 설정하고 `dataset.eval_class_ids`를 통해 실제 클래스 ID를 명시하십시오.
- **입력 크기 불일치**: Export 시 `640x640` 기본값을 유지하십시오. `960x544` 등 다른 크기 사용 시 `hybrid_encoder.py`에서 포지셔널 임베딩 크기 불일치 에러가 발생할 수 있습니다.
- **TensorRT 생성 경로**: PyT CLI 내부의 `gen_trt_engine`은 지원되지 않습니다. 반드시 별도의 **RT-DETR Deploy 워크플로우**(`tao-deploy-rtdetr`)를 사용하십시오.

### 2. 주요 에러 대응
| 에러 현상 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **CUDA Out of Memory** | 배치 사이즈 과다 | RT-DETR은 DINO보다 가볍지만, 16GB GPU에서는 `batch_size`를 8 이하로 조절하십시오. |
| **num_classes mismatch** | 기본값(80)과 데이터 불일치 | `dataset.num_classes`를 실제 데이터셋의 클래스 수에 맞춰 수정하십시오. |
| **Export shape mismatch** | 학습/배포 입력 크기 다름 | 학습 시 사용한 `dataset.augmentation.train_spatial_size`와 Export 시의 `input_height/width`를 일치시키십시오. |

## 📚 관련 참조 스킬
- **배포 가이드**: [`tao-deploy-rtdetr`](references/tao-deploy-rtdetr.md) (TensorRT 엔진 생성 및 추론)
- **실행 플랫폼**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **데이터 검증**: [`tao-validate-dataset-format`](../tao-validate-dataset-format/SKILL.md)
