# 🎴 Skill Card: amc-setup-calibration-stack (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `amc-setup-calibration-stack`
- **도메인**: NVIDIA AI-IoT / AutoMagicCalib (AMC)
- **설명**: NGC 릴리스 이미지를 사용하여 AutoMagicCalib 마이크로서비스(MS)와 웹 UI를 Docker Compose로 배포하고 실행하는 설정 스킬입니다.
- **핵심 목표**: Docker 환경 구축 $\rightarrow$ NGC 인증 $\rightarrow$ VGGT 모델 확보 $\rightarrow$ 네트워크/권한 설정 $\rightarrow$ 서비스 런칭 $\rightarrow$ 상태 검증으로 이어지는 전체 스택의 무결한 배포입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 시스템 수준의 권한(sudo, docker)과 외부 리소스(NGC, HuggingFace) 접근이 필요하므로, 단계별 체크리스트 준수가 필수적입니다.

### 1. [Step 0] 환경 전제 조건 검증 (Pre-flight)
서비스 런칭 전, 런타임 환경이 준비되었는지 확인합니다.
- **Docker 무권한 실행 확인**: `docker ps`를 실행하여 `sudo` 없이 작동하는지 확인합니다.
    - **실패 시**: `sudo usermod -aG docker $USER && newgrp docker` 실행 후 재확인합니다.
- **레포지토리 확보**: `auto-magic-calib` 레포지토리가 로컬에 있어야 합니다.
    - **경로 우선순위**: 현재 디렉토리 $\rightarrow$ `tools/auto-magic-calib` $\rightarrow$ `DEEPSTREAM_REPO_ROOT` $\rightarrow$ `~/auto-magic-calib`.
    - **없을 경우**: 사용자 확인 후 `https://github.com/NVIDIA-AI-IOT/auto-magic-calib`에서 클론합니다. (절대 silently clone 하지 마십시오).
- **Python venv 준비**: VGGT 다운로드를 위해 `pip`와 `huggingface_hub`가 설치된 venv가 필요합니다.

### 2. [Step 1] NGC 인증 (Authentication)
NGC 릴리스 이미지에 접근하기 위해 인증을 수행합니다.
- **동작**: 사용자로부터 NGC API Key를 입력받아 `docker login nvcr.io --username '$oauthtoken' --password-stdin`를 실행합니다.
- **검증**: `✓ NGC authentication complete` 메시지를 확인합니다.

### 3. [Step 2] VGGT 모델 확보 (Optional but Recommended)
고정밀 캘리브레이션을 위한 VGGT-1B-Commercial 모델을 다운로드합니다.
- **필수 절차**: 
    1. HuggingFace에서 `facebook/VGGT-1B-Commercial` 라이선스 동의.
    2. HF Read Token 확보.
- **다운로드**: `hf` CLI를 사용하여 `models/vggt/` 경로에 다운로드합니다. (`HF_TOKEN` 환경변수 사용으로 보안 유지).
- **주의**: 다운로드는 반드시 `chown` 작업(Step 4) **이전**에 수행해야 합니다.

### 4. [Step 3] Docker Compose 네트워크 및 환경 설정
컨테이너 간 통신 및 호스트 접근을 위해 `.env` 파일을 구성합니다.
- **포트 자동 할당**:
    - **Backend (MS)**: 8000-8009 범위 내 빈 포트 탐색.
    - **Frontend (UI)**: 5000-5009 범위 내 빈 포트 탐색.
- **HOST_IP 설정**: `hostname -I`를 통해 실제 네트워크 IP를 추출하여 설정합니다. (`localhost` 사용 금지 $\rightarrow$ UI 컨테이너가 호스트를 통해 MS에 접근해야 함).
- **파일 보안**: `.env` 파일 생성 후 `chmod 600`을 적용하여 API Key 등 민감 정보 유출을 방지합니다.

### 5. [Step 4] 디렉토리 권한 설정 (Ownership)
AMC 컨테이너는 내부적으로 **UID/GID 1000** 사용자로 실행됩니다.
- **대상**: `$REPO_ROOT/projects` 및 `$REPO_ROOT/models`.
- **동작**: `sudo chown 1000:1000 -R projects models` 실행.
- **필수 확인**: 이 작업은 `projects/`와 `models/` 디렉토리가 존재해야 하며, 사용자에게 명시적으로 알린 후 실행합니다.

### 6. [Step 5] 서비스 런칭 및 이미지 검증 (Fail-Fast)
무작정 `up` 하기 전, NGC 키의 이미지 접근 권한을 먼저 체크하여 시간 낭비를 줄입니다.
- **이미지 검증**: `docker compose config --images`로 추출한 모든 이미지에 대해 `docker manifest inspect`를 실행합니다.
- **런칭**: `docker compose up -d`를 통해 백그라운드 실행합니다.

### 7. [Step 6] 서비스 가동 상태 최종 검증 (Health Check)
- **MS 검증**: `http://localhost:${MS_PORT}/v1/ready` 호출 $\rightarrow$ `{"code": 0}` 응답 확인. (최대 120초 대기/24회 시도).
- **UI 검증**: `http://localhost:${UI_PORT}` 호출 $\rightarrow$ HTTP 200 응답 확인.
- **최종 결과**: `http://${HOST_IP}:${MS_PORT}` 및 `http://${HOST_IP}:${UI_PORT}` URL을 사용자에게 제공합니다.

---

## 🛠️ 사전 체크리스트 및 환경 설정

### 1. 필수 진입 게이트
- [ ] Docker & Docker Compose 설치 완료.
- [ ] NVIDIA Container Toolkit (nvidia-docker2) 설정 완료.
- [ ] 유효한 NGC API Key 보유.
- [ ] (옵션) HuggingFace 계정 및 VGGT 라이선스 동의 완료.

### 2. 컴퓨팅 요구사항
- **OS**: Ubuntu 20.04/22.04 권장.
- **GPU**: NVIDIA GPU + 최신 드라이버.
- **Storage**: VGGT 모델 포함 시 최소 10GB 이상의 여유 공간.

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **Silent Clone 금지**: 레포지토리가 없을 때 사용자 확인 없이 `git clone` 하지 마십시오. 특히 DeepStream 서브모듈 경로를 덮어쓰지 않도록 주의하십시오.
2. **HOST_IP 실측값 사용**: `.env`의 `HOST_IP`에 절대 `127.0.0.1`이나 `localhost`를 넣지 마십시오. UI 컨테이너 내부에서 호스트의 MS 포트로 라우팅하기 위해 실제 네트워크 IP가 필요합니다.
3. **권한 설정 순서**: `VGGT 다운로드` $\rightarrow$ `chown 1000:1000` 순서를 반드시 지키십시오. 순서가 바뀌면 다운로드 시 Permission Denied가 발생합니다.

### 🚫 주요 제한 사항 (Limitations)
- **포트 충돌**: 지정된 범위(8000-8009, 5000-5009) 외의 포트를 사용해야 하는 경우, `.env` 파일을 수동으로 수정한 후 런칭하십시오.
- **sudo Docker**: `sudo docker`를 사용하는 환경에서는 Compose 파일 내의 볼륨 권한 문제가 빈번하므로, 반드시 `usermod -aG docker` 설정을 권장합니다.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| `docker ps` Permission Denied | 사용자 그룹 설정 누락 | `sudo usermod -aG docker $USER && newgrp docker` 실행. |
| `docker login` 401/403 | NGC 키 만료 또는 오타 | 키를 다시 확인하고 `docker login`을 재수행하십시오. |
| `docker manifest inspect` 실패 | 이미지 접근 권한 부족 | 해당 이미지 네임스페이스에 접근 가능한 NGC 키를 사용하십시오. |
| VGGT 다운로드 실패 | HF 토큰 오류 또는 라이선스 미동의 | HF 설정에서 Read 토큰을 생성하고 라이선스 페이지에서 동의했는지 확인하십시오. |
| `/v1/ready` Timeout | 컨테이너 초기 구동 지연 또는 크래시 | `docker compose logs auto-magic-calib-ms`로 로그를 분석하십시오. |
| UI에서 Backend 연결 불가 | `HOST_IP` 설정 오류 | `.env` 파일의 `HOST_IP`가 실제 머신 IP인지 확인하십시오. |
| Projects/Models 쓰기 권한 오류 | UID/GID 불일치 | `sudo chown 1000:1000 -R projects models`를 다시 실행하십시오. |

## 🚀 다음 단계
스택 배포가 완료되면 다음 스킬을 통해 기능을 검증하십시오:
- **샘플 데이터 검증**: `/amc-run-sample-calibration`
- **커스텀 비디오 캘리브레이션**: `/amc-run-video-calibration`
- **실시간 RTSP 캘리브레이션**: `/amc-run-rtsp-calibration`
