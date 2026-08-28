# 🎴 Skill Card: digital-health-clinical-asr-eval (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `digital-health-clinical-asr-eval`
- **도메인**: NVIDIA Healthcare / Clinical ASR Flywheel
- **설명**: Clinical ASR 플라이휠의 **Stage 3 (Eval)** 단계입니다. NeMo 포맷의 매니페스트를 전사하고, 4가지 지표(WER, CER, KER, SER)를 측정하여 5개 섹션으로 구성된 KER 리더보드를 생성합니다.
- **핵심 목표**: 단순한 WER을 넘어, 임상적으로 치명적인 오류를 잡아내는 **KER (Keyword Error Rate)**를 측정하고, 결과에 따라 모델 미세조정(Fine-tune) 여부나 발음 교정(Build) 단계로의 회귀를 결정하는 의사결정 게이트 역할을 합니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 오디오 생성 없이, 이미 존재하는 매니페스트를 기반으로 스코어링을 수행하는 'Score-and-Route' 단계입니다.

### 1. [Step 3a] ASR NIM 선정 및 확인
사용할 ASR 모델을 결정하고 설정값을 확인합니다.
- **기본 모델**: `nvidia/parakeet-tdt-0.6b-v2` (NVCF function-id: `d3fe9151-442b-4204-a70d-5fcc597fd610`).
- **설정 옵션**: `ASR_MODEL_NAME`(표시 이름), `ASR_NVCF_FUNCTION_ID`(모델 변경), `ASR_ENDPOINT`(자체 호스팅 gRPC 주소).
- **주의**: API 크레딧 소모 전, 선택된 NIM과 Resolved Function-ID를 사용자에게 명시적으로 확인시켜야 합니다.

### 2. [Step 3b] 전사 (Transcription) 수행
매니페스트의 각 행을 ASR NIM을 통해 전사합니다.
- **방법**: `riva.client.ASRService.offline_recognize`를 이용한 gRPC 호출.
- **결과물**: `per_sample.json` (ref, hyp, term, entity_category, ipa_source 등 모든 메타데이터 포함).
- **복구 전략**: `RESOURCE_EXHAUSTED` 에러 발생 시 해당 행부터 재시도합니다.

### 3. [Step 3c] 4가지 핵심 지표 측정 (Scoring)
모든 전사 결과에 대해 다음 정규화 과정을 거친 후 지표를 계산합니다.
- **정규화 (Normalization)**: ① 소문자화 $\rightarrow$ ② NFKD 정규화 $\rightarrow$ ③ 하이픈(-) 제외 모든 구두점 제거 $\rightarrow$ ④ 연속 공백을 단일 공백으로 축소.
- **측정 지표**:
    - **WER (Word Error Rate)**: 전체 단어 수준의 오류율 (업계 표준).
    - **CER (Character Error Rate)**: 문자 수준 오류율 (복합어 근접 오류 탐지).
    - **KER (Keyword Error Rate) ★**: 플래그 지정된 `term`이 전사 결과에 **연속적이고 정확하게** 나타났는지 확인. (임상적 핵심 신호).
    - **SER (Sentence Error Rate)**: 문장 전체가 완벽한지 여부 (의사 체감 지표).

### 4. [Step 3d] 5개 섹션 리더보드 생성
결과를 다음 순서의 마크다운 리더보드로 작성합니다.
1. **Headline**: 전체 평균 WER, CER, KER, SER.
2. **KER by `entity_category`**: 약물, 시술, 해부학 등 카테고리별 성능 (배포 결정의 핵심).
3. **KER by `ipa_source`**: `merriam-webster` vs `magpie_g2p` 성능 차이 분석 (**가장 중요한 진단 지표**).
4. **KER by `noise_level`**: 소음 수준별(clean $\rightarrow$ 5dB) 강건성 확인.
5. **Per-term KER (Worst-first)**: 가장 많이 틀리는 용어 리스트 (Stage 4 미세조정 타겟).

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 요구사항
- **NeMo 포맷 매니페스트**: `term`, `entity_category`, `ipa_source` 등 임상 확장 필드가 포함되어야 함.
- **오디오 파일 존재 확인**: API 호출 전, 매니페스트에 기재된 모든 `.wav` 파일이 실제 경로에 존재하는지 사전 점검(Pre-flight check) 필수.
- **환경 설정**: `NVIDIA_API_KEY` export 및 `nvidia-riva-client`, `soundfile` 라이브러리 설치 완료 상태.

### 2. 데이터 흐름 고지 (Data Disclosure)
전사 시작 전, 다음 사항을 사용자에게 고지하십시오.
- **전송 데이터**: 매니페스트의 모든 오디오 클립(raw PCM)과 참조 텍스트가 **NVIDIA NVCF Parakeet/Nemotron ASR** 서버로 전송됩니다.
- **제한**: 전송되는 데이터는 Stage 2에서 생성된 **합성 데이터**여야 하며, 실제 환자 기록이나 PHI를 절대 전송하지 마십시오.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **KER 우선 원칙**: WER이 낮더라도 KER가 높다면 이는 임상적으로 위험한 상태입니다. 항상 KER를 헤드라인 지표로 보고하십시오.
2. **Strict-Contiguous KER**: `cefazolin` $\rightarrow$ `cefa zolin`과 같이 띄어쓰기가 발생한 경우- Miss로 처리합니다. (약국 조회 실패 등을 방지하기 위한 보수적 접근).
3. **IPA-Source 분석**: `merriam-webster` 행은 성공적이나 `magpie_g2p` 행만 실패한다면, 이는 모델의 문제가 아니라 **발음 힌트 부족(Coverage Gap)**입니다. 이 경우 미세조정이 아닌 **Stage 2의 IPA QA 단계로 회귀**시켜야 합니다.

### 🚫 주요 제한 사항 (Limitations)
- **언어 제한**: 기본적으로 영어(Latin script) 및 en-US 어휘 사전/정규화를 가정합니다.
- **단일 모델 평가**: 한 번의 실행에는 하나의 모델만 평가합니다. 모델 간 비교는 각각 실행 후 리더보드를 대조합니다.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **모든 행 KER=1** | 참조(ref)와 가설(hyp) 간의 정규화 불일치 | 4단계 정규화 프로세스(소문자, 구두점 제거 등)가 양측에 모두 적용되었는지 확인하십시오. |
| **KER=0이나 WER이 매우 높음** | 매니페스트-오디오 정렬 오류 (Mismatch) | 몇 가지 `(ref, hyp)` 쌍을 직접 대조하여 오디오와 텍스트가 일치하는지 확인하십시오. |
| **`magpie_g2p`만 성능 저하** | 발음 가이드 부족 (Pronunciation Gap) | `/digital-health-clinical-asr-build`의 Step 2d로 돌아가 IPA 오버라이드를 추가하십시오. **미세조정으로 해결하려 하지 마십시오.** |
| **`clean`은 성공, `snr_5db`는 실패** | 환경 강건성 부족 (Robustness Gap) | `/digital-health-clinical-asr-build`에서 노이즈 다양성을 확장하여 다시 구축하십시오. |

## ⏭️ 다음 단계 (의사결정 트리)

결과에 따라 다음과 같이 경로를 안내하십시오:

| 우선 순위 카테고리 KER | 추천 경로 | 비고 |
|---|---|---|
| **> 0.3** | `/digital-health-clinical-asr-finetune` | 매니페스트 100행 이상 시 미세조정 진행. |
| **0.1 – 0.3** | `/digital-health-clinical-asr-build` $\rightarrow$ Finetune | 첫 평가라면 용어 리스트 확장 우선, 이후 단계라면 미세조정 고려. |
| **< 0.1** | `/digital-health-clinical-asr-build` | 베이스라인 강력함. 더 어려운 용어/노이즈/음성을 추가하여 평가를 강화하십시오. |
| **IPA Gap 발생 시** | `/digital-health-clinical-asr-build` (Step 2d) | $\text{KER}_{MW} \ll \text{KER}_{G2P}$ 인 경우, 무조건 발음 교정 단계로 회귀하십시오. |
