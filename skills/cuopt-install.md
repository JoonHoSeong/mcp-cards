# 🎴 Skill Card: cuOpt Install (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cuopt-install`
- **도메인**: NVIDIA cuOpt (Optimization)
- **설명**: Python, C, 또는 REST Server 환경에서 cuOpt를 설치하고 검증하는 스킬입니다.
- **핵심 목표**: 사용자의 하드웨어(GPU) 및 소프트웨어(CUDA) 환경에 최적화된 설치 경로를 제공하고, 설치 후 API가 정상적으로 동작하는지 검증하는 것입니다.

## ⚙️ 구현 및 실행 가이드

### 1. [Step 1] 환경 진단 및 요구사항 확인 (Pre-flight Check)
설치 전, 하드웨어와 드라이버 호환성을 반드시 확인해야 합니다.
- **GPU 요구사항**: NVIDIA Compute Capability $\ge 7.0$ (Volta 이상).
    - **지원**: V100, A100, H100, RTX 20xx/30xx/40xx.
    - **미지원**: GTX 10xx (Pascal) 및 그 이하.
- **CUDA 버전 확인**: `nvcc --version` 또는 `nvidia-smi`를 통해 설치된 CUDA 버전을 확인합니다.
    - cuOpt 패키지의 접미사(`cu12`, `cu13`)가 런타임 CUDA 버전과 일치해야 합니다.
- **드라이버 확인**: 설치하려는 CUDA 버전과 호환되는 NVIDIA 드라이버가 설치되어 있어야 합니다.

### 2. [Step 2] 설치 경로 선택 (Interface Selection)
사용자의 목적에 따라 다음 세 가지 경로 중 하나를 선택합니다. **(주의: Python과 C 패키지를 중복 설치하지 마십시오. CUDA/패키지 충돌이 발생할 수 있습니다.)**

#### 경로 A: Python API (`cuopt-cuXX`)
가장 일반적인 사용 경로이며, 설치 시 C 라이브러리(`libcuopt-cuXX`)가 의존성으로 함께 설치됩니다.
- **pip 설치**:
    - CUDA 13.x: `pip install --extra-index-url=https://pypi.nvidia.com cuopt-cu13`
    - CUDA 12.x: `pip install --extra-index-url=https://pypi.nvidia.com 'cuopt-cu12==26.2.*'`
- **conda 설치**: `conda install -c rapidsai -c conda-forge -c nvidia cuopt`
- **검증**:
    ```python
    import cuopt
    from cuopt import routing
    dm = routing.DataModel(n_locations=3, n_fleet=1, n_orders=2)
    print(cuopt.__version__)
    ```

#### 경로 B: C API (`libcuopt-cuXX`)
Python 없이 C/C++ 환경에서만 사용하려는 경우 선택합니다.
- **pip 설치**:
    - CUDA 13.x: `pip install --extra-index-url=https://pypi.nvidia.com libcuopt-cu13`
    - CUDA 12.x: `pip install --extra-index-url=https://pypi.nvidia.com 'libcuopt-cu12==26.2.*'`
- **conda 설치**: `conda install -c rapidsai -c conda-forge -c nvidia libcuopt`
- **검증**: 헤더 파일(`cuopt_c.h`) 및 공유 라이브러리(`libcuopt.so`) 경로 확인.

#### 경로 C: Server (REST API)
언어에 상관없이 HTTP 통신으로 cuOpt 기능을 사용하려는 경우 선택합니다.
- **pip 설치**: `pip install --extra-index-url=https://pypi.nvidia.com cuopt-server-cu12 cuopt-sh-client`
- **conda 설치**: `conda install -c rapidsai -c conda-forge -c nvidia cuopt-server cuopt-sh-client`
- **Docker 설치 (가장 권장)**:
    ```bash
    docker pull nvidia/cuopt:latest-cuda12.9-py3.13
    docker run --gpus all -it --rm -p 8000:8000 nvidia/cuopt:latest-cuda12.9-py3.13
    ```
- **검증**:
    ```bash
    # 서버 실행 후 헬스체크
    curl -s http://localhost:8000/cuopt/health | jq .
    ```

---

## 🛠️ 사전 체크리스트

- [ ] GPU Compute Capability 7.0 이상 확인.
- [ ] `nvidia-smi`를 통해 CUDA 런타임 버전 확인.
- [ ] `pip` 또는 `conda` 환경이 활성화되어 있는지 확인.
- [ ] (Server 설치 시) 8000번 포트 사용 가능 여부 확인.

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **버전 일치**: 반드시 CUDA 버전과 패키지 접미사(`cu12` $\leftrightarrow$ `cu12`)를 일치시키십시오. 불일치 시 런타임 에러가 발생합니다.
2. **중복 설치 금지**: `cuopt-cuXX`와 `libcuopt-cuXX`를 동시에 다른 방식으로 설치(예: 하나는 pip, 하나는 conda)하지 마십시오.
3. **의존성 이해**: `cuopt-cuXX` (Python) 설치 시 `libcuopt-cuXX` (C)가 자동으로 포함되지만, 그 반대는 성립하지 않습니다.

### 🚫 주요 제한 사항 (Limitations)
- **Pascal 아키텍처 미지원**: GTX 10xx 시리즈에서는 cuOpt를 실행할 수 없습니다.
- **OS 제약**: 공식적으로 Linux 환경을 지원합니다.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| `No module named 'cuopt'` | 설치 실패 또는 Python 인터프리터 불일치 | `pip list \| grep cuopt`로 설치 여부를 확인하고 `which python`으로 환경을 점검하십시오. |
| `CUDA not available` / 런타임 에러 | CUDA 버전 불일치 | `nvcc --version`과 설치된 패키지의 `cuXX` 접미사가 일치하는지 확인하십시오. |
| `docker run` 중 GPU 인식 실패 | NVIDIA Container Toolkit 미설치 | `nvidia-container-toolkit`이 설치되어 있고 `--gpus all` 옵션을 사용했는지 확인하십시오. |
| 서버 헬스체크 응답 없음 | 포트 충돌 또는 서버 미기동 | `netstat -tulpn \| grep 8000`으로 포트 점유 상태를 확인하고 로그를 분석하십시오. |

## 🚀 다음 단계
설치가 완료되었다면, 해결하려는 문제 유형에 맞는 스킬로 이동하십시오:
- **경로 최적화 (VRP, TSP)**: `/cuopt-routing-api-python`
- **수치 최적화 (LP, MILP, QP)**: `cuopt-numerical-optimization-api`
- **소스 코드 수정 및 기여**: `/cuopt-developer`
