# 🎴 Skill Card: digital-health-clinical-asr-build (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `digital-health-clinical-asr-build`
- **도메인**: NVIDIA Healthcare / Clinical ASR Flywheel
- **설명**: Clinical ASR 플라이휠의 **Stage 2 (Build the benchmark)** 단계입니다. 임상 용어를 큐레이션하고, IPA(국제 음성 기호) 태깅을 거쳐, 최종적으로 NeMo 포맷의 매니페스트와 합성 오디오 데이터를 생성합니다.
- **핵심 목표**: 임상 전문가의 도메인 지식을 반영한 벤치마크 데이터셋을 구축하여, Stage 3에서 **KER (Keyword Error Rate)**를 정밀하게 측정할 수 있는 기반을 만드는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 단계는 매우 대화 중심적이며, 각 단계 사이에 명확한 게이트(Gate)가 존재하는 프로세스입니다.

### 1. [Step 2a] 전문 분야 인터뷰 및 용어 추출 (`term_seed.csv`)
사용자와의 인터뷰를 통해 4~10개의 핵심 용어를 선정합니다.
- **인터뷰 순서**: 전문 분야/워크플로우 확인 $\rightarrow$ ASR 실패 사례 분석 $\rightarrow$ 일상적 용어 vs 고난도 용어 구분.
- **결과물**: `term_seed.csv` 생성.
- **허용 카테고리 (Fixed Vocabulary)**: `drug`, `procedure`, `anatomy`, `condition`, `lab`, `role` (이 외의 카테고리는 매핑하거나 거부해야 함).

### 2. [Step 2b] 문장 생성 (`/data-designer` 연동)
추출된 용어를 자연스러운 임상 문장에 임베딩합니다.
- **방법**: `/data-designer` 스킬에 요청하여 용어당 3~5개의 문장 변체 생성.
- **컨텍스트 타입**: `dictation`, `handoff`, `chart_note`, `history`.
- **fallback**: `/data-designer` 사용 불가 시 4가지 템플릿을 이용한 기계적 치환 수행 (이 경우 매니페스트에 태그 표시).

### 3. [Step 2c] 2단계 IPA 태깅 파이프라인 (품질 핵심)
모든 용어는 다음 우선순위에 따라 발음 기호를 할당받습니다.
1. **Override**: `pronunciation_overrides.csv`에 정의된 검증된 IPA (최우선).
2. **Merriam-Webster**: MW 사전의 respelling을 IPA로 변환 (태그: `merriam-webster`).
3. **Magpie G2P**: 위 두 단계 실패 시 Magpie의 신경망 G2P 사용 (태그: `magpie_g2p`).

### 4. [Step 2d] QA 모드 합성 및 청음 검증 (Fail-Closed Gate)
전체 합성을 진행하기 전, 용어당 1개의 샘플 오디오를 생성하여 사용자의 확인을 받습니다.
- **필수 절차**: 사용자가 직접 오디오를 듣고(`afplay` 등) 판정을 내려야 하며, 단순히 "승인" 버튼을 누르는 것으로는 게이트를 통과할 수 없습니다.
- **IPA 보정**: `magpie_g2p`로 판정된 용어에 대해 접미사 패턴(예: `-mycin`, `-prazole`)을 적용한 IPA 후보를 제안하고, Magpie의 음소 인벤토리에 존재하는지 검증 후 `pronunciation_overrides.csv`에 추가합니다.

### 5. [Step 2e] 전체 벤치마크 생성 (Cartesian Product)
모든 발음이 확정되면 다음의 조합으로 전체 데이터셋을 생성합니다.
- **조합**: `|용어|` $\times$ `|음성(Voice)|` $\times$ `|노이즈 레벨(clean, 15dB, 5dB)|` $\times$ `|컨텍스트 타입|`.
- **결과물**: `manifest.jsonl` (NeMo 포맷 + 임상 확장 필드) 및 `audio/*.wav` 파일들.

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 선행 조건
- **`/digital-health-clinical-asr-setup` 완료**: `NVIDIA_API_KEY` 설정 및 Python 의존성 설치 완료 상태여야 함.
- **연동 스킬**: `/read-aloud` (TTS 합성) 및 `/data-designer` (문장 생성) 접근 가능 여부 확인.

### 2. 데이터 흐름 고지 (Data Disclosure)
작업 시작 전, 다음 외부 서비스로 데이터가 전송됨을 사용자에게 고지해야 합니다.
- **Merriam-Webster API**: 용어 조회를 위한 HTTP 요청.
- **NVIDIA NVCF Magpie TTS**: 문장 합성을 위한 gRPC 호출.
- **주의**: 모든 전송 데이터는 **비식별 합성 데이터(Non-PHI)**여야 하며, 실제 환자 기록이나 PHI를 절대 전송하지 마십시오.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **HITL 청음 게이트**: Step 2d의 사용자 청음 검증 없이 Stage 3로 진입하는 것을 엄격히 금지합니다. 잘못된 발음 데이터는 Stage 3의 KER 신호를 오염시킵니다.
2. **카테고리 고정**: `entity_category`는 정해진 6가지 값만 사용하십시오. 이는 후속 분석 및 리더보드 생성의 기준이 됩니다.
3. **합성 데이터 전용**: 본 스킬은 큐레이션된 리스트 기반의 벤치마크용입니다. 실제 전사 데이터나 PHI를 처리하지 마십시오.

### 🚫 주요 제한 사항 (Limitations)
- **영어 전용**: 기본적으로 Magpie `en-US` 음소 인벤토리를 기준으로 검증합니다.
- **NVCF 속도 제한**: 100행 이상의 대규모 작업 시 `RESOURCE_EXHAUSTED` 에러가 발생할 수 있으며, 이 경우 지수 백오프(Exponential Backoff)를 통해 재시도해야 합니다.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **`RESOURCE_EXHAUSTED`** | NVCF API 호출 속도 제한 초과 | 누락된 행만 선별하여 재실행하거나, `/read-aloud`의 백오프 설정을 확인하십시오. |
| **모든 행이 `magpie_g2p`로 태깅됨** | MW API 키 오류 또는 네트워크 차단 | `DICTIONARY_API_KEY` 설정을 재확인하거나, HTML 스크래핑 경로(Path B)의 도달 가능성을 확인하십시오. |
| **IPA 적용 후에도 발음이 이상함** | IPA가 Magpie 인벤토리에 없거나 SSML 문법 오류 | `magpie_validates_ipa` 레시피를 통해 음소 존재 여부를 먼저 확인하고 SSML 래핑 구조를 점검하십시오. |
| **문장이 너무 정형화됨** | `/data-designer` 프롬프트 부족 | 프롬프트에 구체적인 임상 예시(Few-shot)를 추가하여 다시 생성하십시오. |

## ⏭️ 다음 단계
Stage 2가 완료되면 다음 경로로 안내하십시오:
- **다음 경로**: `/digital-health-clinical-asr-eval` (생성된 매니페스트를 전사하고 WER/CER/KER/SER를 측정하여 리더보드 생성)
- **핵심 지표**: 본 단계의 노력이 Stage 3의 **KER (Keyword Error Rate)** 개선으로 이어짐을 강조하십시오.
