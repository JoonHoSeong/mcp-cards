# 🎴 Skill Card: digital-health-clinical-asr-finetune (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `digital-health-clinical-asr-finetune`
- **도메인**: NVIDIA Healthcare / Clinical ASR Flywheel
- **설명**: Clinical ASR 플라이휠의 **Stage 4 (Fine-tune)** 단계입니다. Stage 3에서 측정된 KER(Keyword Error Rate)가 높은 경우, NeMo SFT(Supervised Fine-Tuning)를 통해 모델의 가중치를 업데이트하고 성능을 개선합니다.
- **핵심 목표**: stock NeMo SFT 레시피를 사용하여 특정 임상 용어에 대한 인식률을 높이고, **Cycle N $\rightarrow$ Cycle N+1**의 KER 델타를 통해 개선 효과를 정량적으로 검증하는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 단계는 GPU 자원을 사용하는 고비용/고부하 작업으로, 정교한 가이드라인 준수가 필수적입니다.

### 1. [Step 4a] GPU 호스트 프로비저닝 (Brev 활용)
SFT를 위해서는 최소 16GB(권장 24GB+) VRAM의 CUDA 호스트가 필요합니다.
- **권장 SKU**: Brev L40S (48GB VRAM).
- **비용 고지 (필수)**: 작업 시작 전, 시간당 비용(~$1.50/hr)과 인스턴스 미종료 시 발생하는 유휴 비용 위험을 사용자에게 명시하고 `YES` 확인을 받아야 합니다.
- **환경 구축**: NeMo 컨테이너(`nvcr.io/nvidia/nemo:25.11.01`)를 풀(pull)하고 데이터를 `rsync`로 전송합니다.

### 2. [Step 4b] 용어 인식 기반 Train/Val 분할 (Term-aware Split)
모델 성능을 정확히 측정하기 위해 전략적인 데이터 분할을 수행합니다.
- **방법**: `entity_category`별로 층화 추출(Stratified Split)하며, 기본 검증 비율은 0.2로 설정합니다.
- **특이사항**: 동일한 `term`이 학습셋과 검증셋 모두에 포함될 수 있으며, 이는 음향적/문맥적 강건성을 측정하기 위한 표준 ASR 적응 방식입니다.
- **중단 조건**: 우선 순위 카테고리의 행 수가 5개 미만인 경우, 신호 밀도가 너무 낮으므로 `/digital-health-clinical-asr-build`로 돌아가 데이터를 먼저 확장해야 합니다.

### 3. [Step 4c] 베이스 모델 선정
| 모델명 | SFT 적합성 | 비고 |
|---|---|---|
| **`nvidia/parakeet-tdt-0.6b-v2`** | ✅ **강력 추천** | 검증된 기본 모델. stock NeMo SFT 레시피가 완벽하게 작동함. |
| `nvidia/nemotron-speech-streaming-en-0.6b` | ❌ **사용 금지** | SFT 경로가 불안정하여 검증 단계에서 UNK 붕괴(Collapse) 발생 위험이 매우 큼. |

### 4. [Step 4d] Stock NeMo SFT 실행
커스텀 어댑터나 패치 없이, NeMo 컨테이너 내의 **표준 SFT 스크립트**(`/opt/NeMo/examples/asr/speech_to_text_finetune.py`)를 사용합니다.
- **핵심 하이퍼파라미터**:
    - `precision`: `bf16-mixed` (TDT 수치 안정성을 위해 필수).
    - `lr`: `3e-4` (CosineAnnealing 스케줄).
    - `epochs`: 3 (스모크 테스트) $\rightarrow$ 10~30 (프로덕션).
    - `batch_size`: 4 (16GB VRAM 기준) $\rightarrow$ 16 (L40S 48GB 기준).
- **주의**: TDT/RNNT 디코더에서 어댑터-믹스인(Adapter-mixin) 경로를 사용하면 NaN 텐서가 발생하므로 절대 사용하지 마십시오.

### 5. [Step 4e] Offline Cycle N+1 평가 (루프 폐쇄)
학습된 `.nemo` 모델을 사용하여 오프라인 전사를 수행하고 성능을 재측정합니다.
- **검증 방법**: Riva 배포 없이 NeMo의 `transcribe()` 함수를 사용하여 직접 측정.
- **의사결정 트리 (Cycle N vs N+1)**:
    - **KER 유의미하게 감소 ($\ge 20\%$ 상대 감소)**: 모델 유지 $\rightarrow$ 배포 단계로 진입.
    - **KER 소폭 감소/변화 없음**: `/digital-health-clinical-asr-build`로 돌아가 데이터셋 확장 (하이퍼파라미터 튜닝보다 데이터 밀도가 더 중요함).
    - **KER 악화**: 과적합(Overfitting) 발생. 데이터셋을 확장한 후 다시 학습.

### 6. [Step 4f] Riva NIM 배포 (선택 사항)
최종 검증된 `.nemo` 모델을 `/riva-asr-custom`으로 전달하여 프로덕션 서비스화합니다.
- **필수 정보**: 소스 아키텍처(TDT, CTC, RNNT 등)를 명시적으로 전달해야 합니다. (예: TDT $\rightarrow$ `decoder=nemo` flag $\rightarrow$ `parakeet-tdt-*` 컨테이너).

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 진입 게이트 (Stage 4 Gate)
다음 조건이 모두 충족되어야 SFT를 시작할 수 있습니다.
- **KER 지표**: 우선 순위 카테고리의 KER가 **0.3 초과**.
- **데이터 규모**: 전체 매니페스트 **100행 이상** 및 우선 순위 카테고리당 **5행 이상**.
- **선행 작업**: `/digital-health-clinical-asr-eval`을 통한 베이스라인 측정 완료.

### 2. 컴퓨팅 요구사항
- **VRAM**: 최소 16GB $\rightarrow$ 권장 24GB+ (L40S 48GB 추천).
- **컨테이너**: `nvcr.io/nvidia/nemo:25.11.01`.
- **툴킷**: NVIDIA Container Toolkit + Docker.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **Stock SFT 전용**: 커스텀 어댑터나 외부 패치를 적용하지 마십시오. 검증된 표준 `speech_to_text_finetune.py` 스크립트만 사용하십시오.
2. **SFT 금지 모델**: `nemotron-speech-streaming-en-0.6b` 모델에 대해 SFT를 시도하지 마십시오. 스트리밍 모델이 필요하더라도 Parakeet TDT v2를 학습시킨 후 Riva에서 청크(chunk) 처리하는 것이 정석입니다.
3. **비용 관리**: GPU 인스턴스 생성 후 반드시 `brev stop` 또는 `brev delete`를 통해 과금을 중단하십시오.

### 🚫 주요 제한 사항 (Limitations)
- **TDT/RNNT 어댑터 결함**: TDT 및 RNNT 디코더 기반 모델에서 LinearAdapter-mixin과 같은 어댑터 방식의 SFT를 시도하면 72개 이상의 NaN 텐서가 발생하며 학습이 붕괴됩니다. 반드시 Full-model SFT를 사용하십시오.
- **데이터 밀도 의존성**: 100행 미만의 매우 작은 매니페스트에서는 하이퍼파라미터 튜닝보다 데이터 추가 확보가 성능 개선에 훨씬 효과적입니다.
- **영어 전용**: 본 가이드는 en-US 모델 및 정규화 기준을 따릅니다. 타 언어의 경우 베이스 모델 및 SFT 레시피의 재검증이 필요합니다.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **학습 첫 단계 후 UNK 붕괴** | `nemotron-speech-streaming` 베이스 사용 | `nvidia/parakeet-tdt-0.6b-v2`로 베이스 모델을 교체하십시오. |
| **매니페스트 경로 인식 불가** | 파일 경로 오타 또는 `rsync` 누락 | `ls -R`로 경로를 확인하고 다시 전송하십시오. |
| **VRAM Out of Memory** | `batch_size` 과다 | `batch_size`를 1~4 사이로 낮추고 `accumulation_steps`를 높이십시오. |
| **NaN Loss 발생** | `precision` 설정 오류 | 반드시 `bf16-mixed`를 사용하고, LR을 $10^{-4}$ 수준으로 낮추십시오. |

## 🚀 다음 단계
SFT가 완료되고 KER 감소가 확인되면, Riva NIM 배포 스킬인 `/riva-asr-custom`으로 이동하여 모델을 프로덕션 환경에 적용하십시오.
