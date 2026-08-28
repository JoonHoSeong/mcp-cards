# 🎴 Skill Card: digital-health-clinical-asr-setup (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `digital-health-clinical-asr-setup`
- **도메인**: NVIDIA Healthcare / Clinical ASR Flywheel
- **설명**: Clinical ASR 플라이휠(Flywheel)의 **Stage 1 (Setup)** 단계입니다. NVCF 및 Merriam-Webster 데이터 공개 고지, API 키 확인, 의존성 설치 및 TTS $\rightarrow$ ASR 스모크 테스트를 통해 환경을 부트스트랩합니다.
- **핵심 목표**: 사용자가 보유한 `NVIDIA_API_KEY`로 NVIDIA의 호스팅 음성 스택(Magpie TTS $\rightarrow$ Parakeet/Nemotron ASR)에 성공적으로 도달할 수 있음을 증명하고, 이후 단계인 `/digital-health-clinical-asr-build`로 진행할 수 있는 상태를 만드는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 설치 스크립트가 따로 없으며, 아래의 3단계 검증 프로세스 자체가 실행 절차입니다.

### 1. 환경 설정 및 의존성 설치
먼저 전용 가상환경을 구축하고 필수 라이브러리를 설치합니다.
\`\`\`bash
python3 -m venv .venv
source .venv/bin/activate
pip install nvidia-riva-client pandas soundfile requests
# (선택 사항) 평가를 위한 jiwer 설치
pip install jiwer
\`\`\`

### 2. API 키 검증 (보안 준수)
`NVIDIA_API_KEY`가 쉘에 설정되어 있는지 확인합니다. 보안을 위해 실제 값은 절대 출력하지 않고 길이만 확인합니다.
\`\`\`bash
export NVIDIA_API_KEY=nvapi-... # build.nvidia.com에서 발급
test -n "$NVIDIA_API_KEY" && echo "NVIDIA_API_KEY len=${#NVIDIA_API_KEY}"
\`\`\`
*정상적인 키의 길이는 보통 70자 이상입니다.*

### 3. 호스팅 NVCF 스택 스모크 테스트 (핵심 게이트)
실제 임상 문장이 TTS $\rightarrow$ ASR 라운드트립을 성공적으로 수행하는지 확인합니다.

**구현 코드 (Python):**
\`\`\`python
import wave, tempfile
import riva.client

NVCF_HOST = "grpc.nvcf.nvidia.com:443"
MAGPIE_FUNCTION_ID    = "877104f7-e885-42b9-8de8-f6e4c6303969"   # Magpie TTS
PARAKEET_FUNCTION_ID  = "d3fe9151-442b-4204-a70d-5fcc597fd610"   # Parakeet TDT 0.6B v2

def auth_for(function_id: str, api_key: str) -> riva.client.Auth:
    return riva.client.Auth(
        use_ssl=True, uri=NVCF_HOST,
        metadata_args=[["function-id", function_id], ["authorization", f"Bearer {api_key}"]],
    )

def smoke_test(api_key: str):
    # 1. TTS 생성: "The patient was prescribed cefazolin."
    tts = riva.client.SpeechSynthesisService(auth_for(MAGPIE_FUNCTION_ID, api_key))
    pcm = b"".join(c.audio for c in tts.synthesize_online(
        text="The patient was prescribed cefazolin.",
        voice_name="Magpie-Multilingual.EN-US.Mia",
        language_code="en-US", sample_rate_hz=16000,
    ))
    with tempfile.NamedTemporaryFile(suffix=".wav", delete=False) as f:
        with wave.open(f, "wb") as w:
            w.setnchannels(1); w.setsampwidth(2); w.setframerate(16000); w.writeframes(pcm)
        wav_path = f.name

    # 2. ASR 전사
    asr = riva.client.ASRService(auth_for(PARAKEET_FUNCTION_ID, api_key))
    with open(wav_path, "rb") as f:
        audio_bytes = f.read()
    config = riva.client.RecognitionConfig(
        encoding=riva.client.AudioEncoding.LINEAR_PCM,
        sample_rate_hertz=16000, language_code="en-US",
        max_alternatives=1, enable_automatic_punctuation=True,
    )
    response = asr.offline_recognize(audio_bytes, config)
    transcript = response.results[0].alternatives[0].transcript if response.results else ""
    return transcript
\`\`\`

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 요구사항
| 항목 | 필수 여부 | 용도 | 확보 방법 |
|---|---|---|---|
| `NVIDIA_API_KEY` | **필수** | NVCF 기반 Magpie TTS / Parakeet ASR 접근 | <https://build.nvidia.com> |
| Python $\ge$ 3.10 | **필수** | NeMo 클라이언트 및 매니페스트 도구 실행 | `python3 --version` |
| 필수 라이브러리 | **필수** | `nvidia-riva-client`, `pandas`, `soundfile`, `requests` | `pip install ...` |
| `DICTIONARY_API_KEY`| 선택 | Merriam-Webster 의학 사전 조회 (Stage 2) | <https://dictionaryapi.com> |

### 2. 데이터 흐름 고지 (Data Disclosure)
본 플라이휠 사용 전, 다음 데이터 전송 경로를 반드시 확인하고 승인해야 합니다.
| 서비스 | 전송 데이터 | 시점 | 호스팅 주체 |
|---|---|---|---|
| **NVIDIA NVCF** | 합성 텍스트 및 전사 대상 오디오 파일 | Stage 2 TTS 및 Stage 3 ASR 호출 시 | NVIDIA (build.nvidia.com 약관) |
| **Merriam-Webster** | 개별 임상 용어 (약물명, 해부학 용어 등) | Stage 2 IPA 태깅 시 | Merriam-Webster |

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **PHI 입력 금지**: 본 플라이휠은 큐레이션된 용어 리스트 기반의 **합성 데이터**를 위해 설계되었습니다. **실제 환자 전사 기록, 녹음된 임상 오디오, 또는 PHI를 절대 입력하지 마십시오.**
2. **API 키 보안**: `NVIDIA_API_KEY`를 절대 `echo`, `print` 하거나 로그에 남기지 마십시오. 오직 길이 확인(`len`)만 허용됩니다.
3. **명시적 인자 전달**: 레시피 코드 내에서 `os.environ`을 직접 읽지 말고, 반드시 외부 하네스에서 전달받은 `api_key` 인자를 사용하십시오.

### 🚫 주요 제한 사항 (Limitations)
- **범위 제한**: 본 스킬은 환경 준비 상태만 확인합니다. 용어 리스트의 적절성이나 발음 교정은 `/digital-health-clinical-asr-build`에서 결정합니다.
- **언어 제한**: 기본적으로 `en-US` (Magpie) 기준입니다. 다른 로케일은 별도의 음소 세트가 필요합니다.
- **배포 가정**: 호스팅된 NVCF 환경을 가정합니다. 자체 호스팅 Riva NIM 설정은 Stage 4에서 다룹니다.

---

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **`len=0` 또는 변수 미설정** | `NVIDIA_API_KEY`가 현재 쉘에 export 되지 않음 | `export NVIDIA_API_KEY=...` 실행 후 재확인하십시오. |
| **`401 Unauthorized`** | 키 값이 잘못되었거나 만료됨 | <https://build.nvidia.com>에서 새 키를 발급받으십시오. |
| **`grpc.RpcError: function not found`** | NVCF 카탈로그의 Function ID가 변경됨 | `/digital-health-clinical-asr-eval` 스킬의 최신 ID 리스트를 확인하여 상수를 업데이트하십시오. |
| **`ModuleNotFoundError: riva.client`** | 가상환경 미활성화 또는 설치 누락 | `source .venv/bin/activate` 후 `pip install nvidia-riva-client`를 실행하십시오. |

## ⏭️ 다음 단계
Stage 1 성공 후 반드시 다음 단계로 안내하십시오:
- **다음 경로**: `/digital-health-clinical-asr-build` (전문 분야 인터뷰, 용어 큐레이션, IPA 태깅 및 NeMo 매니페스트 합성)
- **핵심 지표**: Stage 3에서 측정하게 될 **KER (Keyword Error Rate)**가 본 플라이휠의 최종 성공 척도임을 명시하십시오.
