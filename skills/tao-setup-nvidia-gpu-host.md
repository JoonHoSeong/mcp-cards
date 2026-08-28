# 🎴 Skill Card: tao-setup-nvidia-gpu-host (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-setup-nvidia-gpu-host`
- **도메인**: TAO / Platform Setup
- **설명**: TAO Toolkit이 GPU 백엔드(Docker, local-Docker, Kubernetes)에서 정상 작동하기 위해 필요한 호스트 런타임(Driver, CUDA, Container Toolkit)을 검사하고 설치하는 자동화 스킬입니다.
- **핵심 목표**: 모든 TAO 워크플로우의 전제 조건인 **NVIDIA GPU Runtime Standard**를 강제하여, 환경 불일치로 인한 학습/추론 실패를 원천 차단하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **초기 환경 구축**: 새로운 GPU 서버에 TAO Toolkit을 설치하기 전.
- **런타임 검증**: `nvidia-smi`는 작동하지만 Docker 내부에서 GPU 접근이 안 되는 경우.
- **K8s 워커 노드 준비**: Kubernetes 클러스터의 각 GPU 워커 노드에 동일한 드라이버/CUDA 스택을 배포해야 할 때.
- **버전 업데이트**: NVIDIA 드라이버나 CUDA Toolkit 버전을 TAO 요구 사양(예: Driver 580, CUDA 13.0)으로 업데이트해야 할 때.

---

## 🏗️ 표준 런타임 스택 (TAO Standard Stack)

본 스킬은 다음의 핀(Pinned) 버전을 표준으로 강제합니다:

| 구성 요소 | 표준 버전 | 비고 |
| :--- | :--- | :--- |
| **NVIDIA Driver** | Branch `580` | Open Kernel Module 권장 |
| **CUDA Toolkit** | `13.0` | `cuda-toolkit-13-0` 패키지 |
| **Container Toolkit** | `1.19.0` | Docker/K8s GPU 가속 필수 구성 요소 |
| **Docker Engine** | 최신 안정 버전 | `docker.io` (Debian), `moby-engine` (RHEL) |

---

## 🛠️ 구현 가이드 및 워크플로우

### 1. 실행 모드 (Execution Modes)
- **검사 모드 (`--check-only`)**: **(권장)** 시스템을 변경하지 않고 현재 설치 상태만 확인합니다. 모든 배포 파이프라인의 첫 단계로 실행되어야 합니다.
- **설치 모드 (`--install`)**: 사용자 승인 후 부족한 패키지를 자동으로 설치하고 설정을 변경합니다.
- **비대화형 모드 (`--yes`)**: 에이전트나 CI/CD 환경에서 사용할 때, `Continue? [y/N]` 프롬프트를 자동으로 수락합니다.

### 2. 배포 패밀리별 자동화 매트릭스
| 패밀리 | 대상 OS | 패키지 관리자 | 특이 사항 |
| :--- | :--- | :--- | :--- |
| **Debian** | Ubuntu 22.04/24.04, Debian 12 | `apt-get` | `cuda-keyring` 및 `.list` 파일 추가 |
| **RHEL** | RHEL/Rocky/AlmaLinux 9, Fedora | `dnf` / `yum` | `cuda-<distro>.repo` 추가, `moby-engine` 우선 |
| **SUSE** | openSUSE Leap 15, SLES 15 | `zypper` | `.repo` 파일 기반 설치 |
| **기타** | Arch, Alpine, NixOS 등 | N/A | 수동 설치 가이드 URL 제공 후 종료 |

### 3. 표준 검증 시퀀스 (Verification)
설치 완료 후 다음 명령어를 통해 런타임 상태를 최종 확인합니다:
```bash
# 1. 드라이버 및 CUDA 버전 확인 (Driver 580.x, CUDA 13.0 예상)
nvidia-smi

# 2. NVCC 컴파일러 버전 확인 (release 13.0 예상)
/usr/local/cuda-13.0/bin/nvcc --version

# 3. Docker 런타임 설정 확인 (nvidia 런타임 포함 여부)
docker info --format '{{json .Runtimes}}' | grep nvidia

# 4. 실제 GPU 컨테이너 구동 테스트
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

---

## 🔍 트러블슈팅 및 핵심 제약 (Constraints)

### 1. 치명적 함정 및 해결책
- **Docker 권한 문제**: 스크립트가 사용자를 `docker` 그룹에 추가하지만, **현재 쉘 세션에는 즉시 반영되지 않습니다.** $ightarrow$ `newgrp docker` 명령어를 실행하거나 로그아웃 후 재로그인하십시오.
- **Secure Boot 충돌**: 드라이버 설치 후 `nvidia-smi`가 실패하는 경우, Secure Boot가 활성화되어 MOK(Machine Owner Key) 등록이 필요할 수 있습니다.
- **K8s GPU 용량 미검출**: 노드 런타임이 준비되었음에도 `nvidia.com/gpu` 할당량이 0인 경우, NVIDIA GPU Operator를 설치하십시오:
  ```bash
  helm install --wait gpu-operator -n gpu-operator --create-namespace nvidia/gpu-operator
  ```

### 2. 절대 금지 사항 (Anti-Patterns)
- **사전 승인 없는 `--install`**: 시스템 전역 설정을 변경하므로 반드시 `--check-only` 결과를 사용자에게 보고하고 승인을 받은 후 실행하십시오.
- **버전 임의 변경**: TAO Toolkit 버전과 맞지 않는 CUDA 버전을 설치하면 런타임에 `Cuda Error: invalid device function` 등이 발생할 수 있습니다.

## 📚 관련 참조 스킬
- **TAO 전체 흐름**: [`tao-run-platform`](../tao-run-platform/SKILL.md)
- **모델 학습 시작**: [`tao-train-image-classification`](../tao-train-image-classification/SKILL.md)
