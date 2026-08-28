---
name: nemo-automodel-recipe-development
description: Create and modify NeMo AutoModel training and evaluation recipes, including YAML structure, builders, and execution flow.
version: 0.1.0
license: Apache-2.0
metadata:
  author: NVIDIA
  tags:
    - nemo-automodel
    - recipe-development
  domain: feature
---

# 🎴 Skill Card: nemo-automodel-recipe-development (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `nemo-automodel-recipe-development`
- **도메인**: NeMo AutoModel Recipe Feature
- **설명**: NeMo AutoModel의 학습 및 평가 레시피(Recipe)를 생성하고 수정하기 위한 종합 가이드입니다. YAML 설정 구조, 빌더 함수 패턴, 실행 흐름 및 CLI 오버라이드 방법을 다룹니다.
- **핵심 목표**: 모델 아키텍처, 데이터셋, 최적화 알고리즘을 결합한 최적의 학습 파이프라인을 YAML 기반으로 정의하고, 이를 안정적으로 실행 및 검증하는 것입니다.

## 🚀 레시피 아키텍처 및 실행 흐름

### 1. 실행 파이프라인 (Execution Flow)
`CLI (automodel finetune llm -c config.yaml)` $\rightarrow$ `app.py (파싱)` $\rightarrow$ `Recipe Script (e.g., train_ft.py)` $\rightarrow$ `Recipe Class (.setup())` $\rightarrow$ `Component Builders` $\rightarrow$ `.run_train_validation_loop()`

### 2. 빌더 패턴 (Builder Pattern)
모든 구성 요소는 전용 빌더 함수를 통해 생성되어 유연성과 재사용성을 확보합니다.
- `build_model()`: 설정 기반 모델 인스턴스화.
- `build_optimizer()`: AdamW 등 최적화 도구 생성.
- `build_dataloader()`: 학습/검증 데이터 로더 설정.
- `build_loss_module()`: 손실 함수 정의.
- `build_lr_scheduler()`: 학습률 스케줄러 설정.
- `build_step_scheduler()`: 학습 진행(에폭, 스텝, 인터벌) 제어.

### 3. 인프라 적용 순서 (Infrastructure Application Order)
빌드 후 구성 요소에 적용되는 엄격한 순서입니다. 순서가 바뀌면 모델 상태가 오염되거나 런타임 에러가 발생할 수 있습니다.
`PEFT (LoRA)` $\rightarrow$ `FP8 Quantization` $\rightarrow$ `QAT` $\rightarrow$ `Checkpoint Load` $\rightarrow$ `Parameter Freezing` $\rightarrow$ `Sharding (FSDP2/Megatron-FSDP/DDP)` $\rightarrow$ `Device Placement` $\rightarrow$ `torch.compile` $\rightarrow$ `Context Parallelism Hooks`

---

## 🛠️ YAML 설정 상세 분석 (Config Anatomy)

### 1. 핵심 섹션 구성
- **`step_scheduler`**: `max_steps`, `val_check_interval`, `checkpoint_interval` 등 학습 주기 제어.
- **`distributed`**: `strategy` (fsdp2, megatron_fsdp, ddp) 및 `tp_size`, `cp_size`, `dp_size` 정의.
- **`model`**: `_target_`를 통한 모델 클래스 지정 및 하이퍼파라미터 전달.
- **`optimizer` / `lr_scheduler`**: 최적화 알고리즘 및 학습률 전략.
- **`dataset` / `validation_dataset`**: 데이터 소스 및 토크나이저 설정.

### 2. `_target_` 패턴 (Dynamic Instantiation)
`_target_` 키는 호출할 Python 가산함수(Callable)의 완전한 경로를 지정하며, 동일 레벨의 나머지 키들은 해당 함수의 키워드 인자(kwargs)로 전달됩니다.
*예시*:
```yaml
optimizer:
  _target_: torch.optim.AdamW
  lr: 2.0e-5
  weight_decay: 0.01
```
$\Rightarrow$ `torch.optim.AdamW(lr=2.0e-5, weight_decay=0.01)`

### 3. CLI 오버라이드 (Dynamic Override)
설정 파일 수정 없이 명령줄에서 특정 값만 변경하여 빠르게 실험할 수 있습니다.
`--optimizer.lr 1e-4` $\rightarrow$ YAML의 `optimizer` 섹션 내 `lr` 값을 1e-4로 덮어씁니다.

---

## ⚠️ 제약 사항 및 주의 사항 (Gotchas)

- **배치 사이즈 정렬 (Batch Size Alignment)**: `global_batch_size`는 반드시 `local_batch_size * dp_size * grad_accumulation_steps`로 나누어 떨어져야 합니다. 그렇지 않으면 첫 스텝에서 런타임 에러가 발생합니다.
- **사일런트 설정 에러 (Silent Config Errors)**: `_target_` 경로에 오타가 있을 경우, 임포트 에러가 발생하며 실행이 중단됩니다. 모듈 경로와 클래스 이름을 다시 확인하십시오.
- **검증 시 OOM (Validation OOM)**: 검증 단계에서 메모리 부족이 발생할 경우, `torch.no_grad()` 적용 여부를 확인하고 `validation_dataset`의 배치 사이즈를 줄이십시오.
- **체크포인트 복구 실패**: 복구하려는 체크포인트의 모델 아키텍처(레이어 수, hidden dim 등)가 현재 YAML 설정과 정확히 일치해야 합니다.

## 🔍 진단 래더 (Diagnostic Ladder)

| 증상 | 체크포인트 | 해결책 |
|---|---|---|
| **Training Crash at Step 0** | 배치 사이즈 계산식 확인 | `global_batch_size` 정렬 상태를 검증하십시오. |
| **Recipe Not Found in CLI** | CLI route 등록 확인 | 신규 레시피 추가 시 `app.py` 또는 관련 라우터에 alias가 등록되었는지 확인하십시오. |
| **Forward Pass Shape Mismatch** | Collate 함수 출력 확인 | 데이터셋의 collate 함수가 반환하는 텐서 키/형태가 모델 입력 시그니처와 일치하는지 확인하십시오. |
| **Checkpoint Restore Error** | 모델 설정 vs 체크포인트 비교 | 모델 아키텍처 설정이 저장된 체크포인트와 동일한지 확인하십시오. |

## 🔗 코드 앵커 (Code Anchors)
- `nemo_automodel/recipes/llm/train_ft.py`: LLM 파인튜닝/사전학습 메인 스크립트.
- `nemo_automodel/recipes/vlm/finetune.py`: VLM 파인튜닝 레시피.
- `nemo_automodel/recipes/diffusion/train.py`: 디퓨전 모델 학습 레시피.
- `nemo_automodel/recipes/retrieval/`: Bi-encoder 및 Cross-encoder 학습 레시피.
