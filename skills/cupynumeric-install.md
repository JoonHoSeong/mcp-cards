# 🎴 Skill Card: cupynumeric-install (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cupynumeric-install`
- **도메인**: cuPyNumeric / Installation & Verification
- **설명**: NVIDIA cuPyNumeric을 Python 환경에 설치하고, 단순히 패키지가 설치된 것을 넘어 실제로 GPU 가속이 작동하는지 검증하는 전체 프로세스 가이드입니다.
- **핵심 목표**: 사용자의 하드웨어 환경(GPU/CPU)과 패키지 매니저(Conda/Pip)에 맞는 최적의 설치 경로를 제공하고, `legate` 런처를 통해 GPU 활용도를 엄격하게 검증하는 것입니다.

## 🎯 적용 시점 (When to Use)
- 새로운 GPU 서버나 워크스테이션에 cuPyNumeric 환경을 구축해야 할 때.
- 설치 후 `ImportError`가 발생하거나, 결과는 맞지만 실제로는 CPU로 동작하고 있는 "Silent CPU Fallback" 현상을 진단하고 해결해야 할 때.
- Docker 컨테이너 이미지 빌드 시, 실제 GPU가 없는 빌드 환경에서 GPU 전용 바이너리를 강제 설치해야 할 때.

---

## ⚙️ 설치 및 검증 워크플로우 (Step-by-Step)

### 1. 사전 요구사항 확인 (Prerequisites)
설치 전 다음 항목을 반드시 확인하십시오.
- **GPU**: Compute Capability 7.0 이상 (Volta 아키텍처 이상). *CPU-only 모드도 지원합니다.*
- **CUDA**: 12.2 버전 이상 설치 필요.
- **OS**: Linux (x86_64, aarch64) 또는 Windows WSL2.
- **Python**: 3.11 ~ 3.14 버전.
- **Conda**: 24.1 버전 이상 (낮은 버전은 변종 선택 시 오류 발생 가능).

### 2. 설치 경로 선택 (Installation Paths)

#### 경로 A: Conda (권장 방식)
Conda는 의존성 관리가 뛰어나며 cuPyNumeric에서 공식적으로 권장하는 방식입니다.
- **신규 환경 생성**:
  ```bash
  conda create -n cupynumeric -c conda-forge -c legate cupynumeric
  conda activate cupynumeric
  ```
- **기존 환경에 추가**: `conda install -c conda-forge -c legate cupynumeric`

#### 경로 B: Pip (가상환경 방식)
- **환경 구축**:
  ```bash
  python -m venv .venv
  source .venv/bin/activate
  pip install nvidia-cupynumeric
  ```

#### 💡 특수 케이스: GPU 바이너리 강제 설치 (CONDA_OVERRIDE_CUDA)
빌드 서버처럼 GPU가 없는 환경에서 GPU용 cuPyNumeric을 설치해야 하는 경우, 환경 변수를 통해 CUDA 버전을 명시하십시오.
```bash
CONDA_OVERRIDE_CUDA="12.2" conda install -c conda-forge -c legate cupynumeric
```

---

## 🔍 엄격한 설치 검증 (Strict Verification)

단순한 `import cupynumeric` 성공은 설치 완료를 의미하지 않습니다. 다음 두 단계의 검증을 반드시 수행하십시오.

### 단계 1: 스모크 테스트 (정상 동작 확인)
`legate` 런처를 사용하여 기본 연산이 정확한 값으로 나오는지 확인합니다.
```bash
TMP=$(mktemp -d)
cat > "$TMP/smoke.py" <<EOF
import cupynumeric as np
a = np.arange(10)
b = np.ones((4, 4))
print("sum:", a.sum())            # 기대값: 45
print("matmul:", (b @ b).sum())   # 기대값: 64.0
EOF
legate "$TMP/smoke.py"
rm -rf "$TMP"
```
- **결과 해석**: `sum: 45`, `matmul: 64.0`이 출력되어야 합니다. `legate: command not found`가 나오면 환경 활성화(`conda activate`)가 되지 않은 것입니다.

### 단계 2: GPU 실제 활용 검증 (GPU-Usage Check)
**매우 중요**: CPU-variant가 설치되어도 단계 1은 통과합니다. 실제 GPU가 사용되는지 확인해야 합니다.

#### 방법 A: GPU 강제 런칭 테스트
```bash
TMP=$(mktemp -d)
cat > "$TMP/check.py" <<EOF
import cupynumeric as np
print(np.ones((4096, 4096)).sum())
EOF
legate --gpus 1 "$TMP/check.py"
rm -rf "$TMP"
```
- **결과 해석**: `16777216.0`이 출력되면 성공입니다. `CUDA driver` 또는 `no GPUs available` 에러가 발생하면 CPU-variant가 설치된 것이므로 재설치가 필요합니다.

#### 방법 B: 실시간 GPU 유틸라이제이션 모니터링
연산 루프를 돌리면서 `nvidia-smi`를 통해 메모리 점유율과 GPU 사용률을 확인합니다.
```bash
# 1. GPU 부하 스크립트 실행 (백그라운드)
TMPDIR_GPU=$(mktemp -d)
SCRIPT="$TMPDIR_GPU/gpu_check.py"
cat > "$SCRIPT" <<EOF
import cupynumeric as np, time
a = np.ones((10000, 10000))
deadline = time.time() + 20
while time.time() < deadline:
    b = a @ a
    _ = float(b.sum()) # 동기화 강제
EOF
legate --gpus 1 "$SCRIPT" &
WORKLOAD=$!

# 2. 1초 간격으로 10번 GPU 상태 샘플링
sleep 5
for _ in $(seq 10); do
  nvidia-smi --query-gpu=utilization.gpu,memory.used --format=csv,noheader
  sleep 1
done
wait "$WORKLOAD"
rm -rf "$TMPDIR_GPU"
```
- **판정 기준**: `memory.used`가 GiB 단위로 상승하고 `utilization.gpu`가 유의미하게 상승해야 합니다. 모두 0 근처라면 CPU-variant가 설치된 것입니다.

---

## ⚠️ 주의사항 및 트러블슈팅 (Pitfalls & Troubleshooting)

### 1. 주요 제약 및 금지 사항
- **Conda-Pip 혼용 금지**: 한 환경에서 두 매니저를 섞어 쓰면 바이너리 충돌로 `ImportError`가 발생합니다. 전환 시 반드시 `pip uninstall` 또는 `conda remove` 후 재설치하십시오.
- **하드웨어 제한**: Pascal(GTX 10xx, P100) 이전 세대는 지원하지 않습니다. Volta(V100, RTX 20xx) 이상이어야 합니다.
- **런처 사용**: 멀티 GPU 사용 시 `python script.py` 대신 `legate --gpus N script.py`를 사용하십시오.

### 2. 흔한 에러 해결법
- **`ModuleNotFoundError`**: `which python`과 `pip list`를 실행하여 현재 쉘의 파이썬 경로와 설치된 패키지 경로가 일치하는지 확인하십시오.
- **`ImportError` (CUDA 관련)**: CPU-variant가 설치되었거나 CUDA 버전이 맞지 않는 경우입니다. `CONDA_OVERRIDE_CUDA`를 사용하여 재설치하십시오.
- **노트북에서 NumPy보다 느림**: 작은 데이터셋의 경우 Legate의 태스크 오버헤드 때문에 느릴 수 있습니다. 이는 정상이며, 대규모 데이터셋에서 성능 이점이 나타납니다.

---

## 📚 관련 인터페이스 및 스킬
- **`cupynumeric-hdf5`**: 설치 후 대규모 데이터를 효율적으로 로드하기 위한 HDF5 인터페이스 가이드입니다.
- **`cupynumeric-parallel-data-load`**: 분산 환경에서 데이터 로딩 성능을 최적화하는 방법입니다.
- **`cupynumeric-migration-readiness`**: 기존 NumPy 코드를 cuPyNumeric으로 마이그레이션할 때의 호환성 체크리스트입니다.
