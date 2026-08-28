---
name: nemo-automodel-model-onboarding
description: Guide for onboarding new model architectures into NeMo AutoModel, including architecture discovery, implementation patterns, registration, and validation.
version: 0.1.0
license: Apache-2.0
metadata:
  author: NVIDIA
  tags:
    - nemo-automodel
    - model-onboarding
  domain: foundation
---

# 🎴 Skill Card: nemo-automodel-model-onboarding (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `nemo-automodel-model-onboarding`
- **도메인**: NeMo AutoModel Model Foundation
- **설명**: 새로운 모델 아키텍처(LLM, VLM, MoE 등)를 NeMo AutoModel 프레임워크에 온보딩하기 위한 엔드-투-엔드 가이드입니다. 아키텍처 분석부터 구현, 레지스트리 등록, 수치적 동등성 검증(Parity Testing)까지의 전 과정을 다룹니다.
- **핵심 목표**: Hugging Face 등 외부 프레임워크의 모델을 NeMo AutoModel의 표준 레이아웃으로 정확하게 이식하여, 분산 학습 및 추론 최적화 기능을 즉시 사용할 수 있도록 하는 것입니다.

## 🚀 온보딩 파이프라인: Discovery $\rightarrow$ Implement $\rightarrow$ Register $\rightarrow$ Verify $\rightarrow$ Parity

### 1. Discovery (아키텍처 분석)
코드 작성 전, 대상 모델의 `config.json`을 분석하여 정체성을 분류합니다.

| 모델 타입 | 핵심 식별자 (Indicators) | 패턴 참조 파일 |
|---|---|---|
| **Dense LLM** | `ForCausalLM` 포함, 전문가(Expert) 관련 필드 없음 | `llm-patterns.md` |
| **MoE LLM** | `n_routed_experts`, `num_local_experts` 등 존재 | `moe-patterns.md` |
| **VLM** | `ForConditionalGeneration` 포함, `vision_config` + `text_config` 존재 | `vlm-patterns.md` |

- **필수 추출 정보**: `hidden_size`, `intermediate_size`, `num_hidden_layers`, `vocab_size`, `tie_word_embeddings` (매우 중요), `hidden_act`.

### 2. Implementation (구현 전략)
다음의 디렉토리 구조와 순서로 구현을 진행합니다: `components/models/<name>/`
1. **`config.py`**: HF `AutoConfig`로 파싱 불가능한 경우 커스텀 `PretrainedConfig` 구현.
2. **`rope_utils.py`**: YaRN, NTK-aware 등 커스텀 RoPE 구현.
3. **`layers.py`**: MLA, MoE 라우터 등 비표준 레이어 구현.
4. **`model.py`**: 메인 모델 클래스 구현 및 `HFCheckpointingMixin` 상속.
5. **`state_dict_adapter.py`**: HF 가중치 $\rightarrow$ NeMo 레이아웃 변환 로직 구현.
6. **`__init__.py`**: 메인 모델 클래스 export.

### 3. Registration (레지스트리 등록)
프레임워크가 모델을 인식할 수 있도록 `_transformers/registry.py`에 등록합니다.
- **`MODEL_ARCH_MAPPING`**: `(HF_Class_Name, (Module_Path, Class_Name))` 형태로 등록.
- **`_CUSTOM_CONFIG_REGISTRATIONS`**: 커스텀 설정 클래스가 필요한 경우 등록.
- **Capabilities 선언**: `ModelCapabilities` 데이터클래스 또는 `get_capabilities()` 메서드를 통해 지원 가능한 병렬 전략(TP, PP, FSDP 등)을 정의하십시오.

### 4. Verification (검증)
전체 체크포인트를 로드하기 전, **Tiny Config(약 1M 파라미터 이하)**를 사용하여 기본 동작을 검증합니다.
- **Forward Shape Test**: 입력 텐서에 대해 기대하는 출력 shape가 나오는지 확인.
- **Adapter Round-trip**: `from_hf` $\rightarrow$ `to_hf` 수행 시 가중치 값과 이름이 보존되는지 확인.
- **Layer Equivalence**: 재작성한 레이어(Attention, MLP 등)가 HF 원본과 수치적으로 동일한지 확인.

### 5. Parity Testing (최종 정밀 검증)
가장 중요한 단계로, 다음 세 수준의 비교를 수행하여 수치적 일치성을 보장합니다.
1. **State-dict Parity**: 변환된 가중치 $\rightarrow$ 다시 HF 포맷으로 내보내기 $\rightarrow$ 원본과 비교.
2. **Component Parity**: 고정된 시드와 동일한 dtype으로 개별 컴포넌트 출력 비교.
3. **End-to-End Parity**: 동일한 토큰 입력에 대해 **Logits, Hidden States, Loss**가 일치하는지 확인.

---

## ⚠️ 기술적 제약 및 주의 사항 (Critical Gotchas)

### 1. Word Embedding Tying (가중치 공유)
Causal LM 헤드를 가진 모든 모델은 `TieSupport` 정책을 반드시 선언해야 합니다.
- `BOTH`: Tied/Untied 모두 지원.
- `TIED_ONLY`: Tied 설정만 지원.
- `UNTIED_ONLY`: Untied 설정만 지원.
- **주의**: `from_pretrained` 시 체크포인트에 저장된 `tie_word_embeddings` 값이 최우선이며, 이를 임의로 뒤집는 설정은 거부되어야 합니다.

### 2. MoE State-dict 매핑
MoE 모델은 단순 로딩이 아니라 다음 사항을 명시적으로 매핑해야 합니다.
- 라우터 가중치 및 바이어스 $\rightarrow$ 전문가 가중치 (인덱스 순서 보존) $\rightarrow$ 공유 전문가(Shared Experts) 구분.
- 단순 `from_pretrained()` 호출만으로 검증하지 말고, 반드시 키 매핑 테스트를 수행하십시오.

### 3. 정밀도 민감 파라미터 (Precision-Sensitive)
Mamba의 `A_log`/`dt_bias`, MoE 시그모이드 게이트 바이어스, attention-sink 바이어스 등은 샤딩 시에도 **반드시 fp32로 유지**되어야 합니다. 이를 위해 `_keep_in_fp32_modules_strict` 리스트에 등록하십시오.

## 🔍 진단 래더 (Diagnostic Ladder)

| 증상 | 체크포인트 | 해결책 |
|---|---|---|
| **Load Error (Key Mismatch)** | `state_dict_adapter.py` 확인 | HF 가중치 키 이름과 NeMo 레이어 이름 간의 매핑 테이블을 다시 검증하십시오. |
| **Numerical Divergence** | RoPE/Normalization 구현 확인 | RoPE 스케일링 방식이나 Norm의 epsilon 값이 HF와 일치하는지 확인하십시오. |
| **OOM during Load** | `ModelCapabilities` 확인 | 모델 크기에 비해 너무 큰 TP/PP 설정을 사용 중인지 확인하십시오. |
| **Weight Tying Failure** | `TieSupport` 정책 확인 | 모델의 `tie_word_embeddings` 설정과 실제 가중치 텐서의 공유 여부를 확인하십시오. |

## 🔗 코드 앵커 (Code Anchors)
- `_transformers/registry.py`: 모델 및 설정 레지스트리.
- `components/models/common/combined_projection/`: Combined QKV/MLP 구현체.
- `components/models/common/hf_checkpointing_mixin.py`: HF 체크포인트 로드/저장 믹스인.
- `components/moe/layers.py`: MoE 레이어 및 전문가 그룹 구현.
