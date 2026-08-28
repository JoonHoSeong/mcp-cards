# 🎴 Skill Card: cudaq-guide (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cudaq-guide`
- **도메인**: Quantum Computing / CUDA-Q
- **설명**: CUDA-Q 플랫폼의 설치부터 테스트 프로그램 작성, GPU 가속 시뮬레이션, 실제 QPU 하드웨어 연결 및 양자 애플리케이션 탐색까지 안내하는 종합 온보딩 가이드입니다.
- **핵심 목표**: 사용자가 자신의 환경(OS, 하드웨어)에 맞는 최적의 CUDA-Q 설정 경로를 찾고, 양자-고전 하이브리드 커널을 성공적으로 빌드 및 실행하도록 돕는 것입니다.

## ⚙️ 핵심 워크플로우 및 가이드라인

### 1. 설치 프로세스 (Installation)
사용자의 환경에 따라 다음 세 가지 경로 중 하나를 추천합니다.
- **Python 경로 (기본)**: `pip install cudaq` (Python 3.10+ 필수).
  - **Linux**: CUDA Toolkit 설치 시 GPU 가속 가능.
  - **macOS (Apple Silicon)**: CPU 시뮬레이션(`qpp-cpu`)만 가능.
- **C++ 경로**: `nvq++` 컴파일러 사용. Linux/WSL 환경 필수.
- **Cloud 경로 (Brev)**: NVIDIA Application Hub에서 워크스페이스 생성 후 SSH 접속. 모든 툴킷이 사전 설치되어 있음.
- **검증**: 설치 후 반드시 **Bell State** 예제를 실행하여 결과가 `{ 00:~500 11:~500 }` 근처로 나오는지 확인해야 합니다.

### 2. 테스트 프로그램 작성 (Kernel Development)
CUDA-Q 커널 작성 시 반드시 준수해야 할 기술적 제약 사항입니다.
- **커널 정의**: `@cudaq.kernel` 또는 `__qpu__` 데코레이터를 사용하여 양자 커널을 정의합니다. 이는 Quake MLIR로 컴파일됩니다.
- **주요 API**:
  - `cudaq.qvector(N)`: N개의 큐비트를 $|0\rangle$ 상태로 할당.
  - `cudaq.sample()`: 큐비트를 측정하여 비트스트링 히스토그램(`SampleResult`) 반환.
  - `cudaq.run()`: 고전적 값을 반환하며, 설정된 `shots_count`만큼 실행.
  - `cudaq.observe()`: 스핀 연산자에 대한 기대값 $\langle H \rangle$ 계산.
- **⚠️ 중요 제한**: 커널 내부에서는 **제한된 Python 서브셋**만 사용 가능합니다. **NumPy 및 SciPy는 커널 내부에서 사용할 수 없으며**, 반드시 커널 외부의 고전적 전/후처리 단계에서 사용해야 합니다.

### 3. GPU 가속 시뮬레이션 (GPU Simulation)
회로의 크기와 목적에 따라 최적의 타겟을 선택합니다.
- **`nvidia` (기본)**: 단일 GPU 상태 벡터 시뮬레이션 (약 30 큐비트까지).
- **`nvidia --target-option fp64`**: 고정밀도 계산이 필요한 화학/민감 관측치 계산 시 사용.
- **`nvidia --target-option mgpu`**: 여러 GPU의 메모리를 풀링하여 30 큐비트 이상의 대규모 회로 시뮬레이션 (MPI 필요).
- **`nvidia --target-option mqpu`**: 각 GPU를 가상 QPU로 매핑하여 수많은 독립 회로를 병렬 실행 (파라미터 스윕, VQE 그래디언트 계산 시 최적).
- **`tensornet`**: 얽힘이 적거나 얕은 회로, 혹은 상태 벡터 시뮬레이션이 불가능한 초거대 큐비트 수의 경우 사용.

### 4. 실제 QPU 연결 (Hardware Access)
QPU 연결은 [기술 선택 $\rightarrow$ 제공자 선택 $\rightarrow$ 가이드 안내]의 2단계 대화 프로세스로 진행합니다.
- **기술군**:
  - **이온 트랩 (Ion Trap)**: IonQ, Quantinuum.
  - **초전도 (Superconducting)**: IQM, OQC, Anyon, TII, QCI.
  - **중성 원자 (Neutral Atom)**: Infleqtion, QuEra, Pasqal.
  - **클라우드 플랫폼**: AWS Braket, Scaleway.
- **실행 전략**:
  - 하드웨어 제출 전 반드시 `emulate=True`로 로컬 테스트를 수행하십시오.
  - 비동기 제출(`sample_async`, `observe_async`)을 사용하여 블로킹을 방지하십시오.
  - **보안**: API 토큰은 절대 코드에 하드코딩하지 말고 **환경 변수**로 관리하십시오.

### 5. 병렬화 전략 (Parallelization)
- **메모리 확장**: 단일 회로가 너무 클 때 $\rightarrow$ `mgpu` 타겟 사용.
- **처리량 확장**: 많은 독립 회로를 동시에 돌릴 때 $\rightarrow$ `mqpu` 타겟 사용.
- **해밀토니안 분산**: 대규모 해밀토니안 기대값 계산 시 $\rightarrow$ `mqpu` + `execution=cudaq.parallel.thread` (단일 노드) 또는 `.mpi` (멀티 노드).

---

## 🛠️ 사전 체크리스트 및 환경 설정

- **최소 요구사항**: Python 3.10+.
- **OS별 제약**: GPU 시뮬레이션은 Linux(x86_64, ARM64)에서만 가능하며, macOS는 CPU 전용입니다.
- **필수 툴**: GPU 타겟 사용 시 CUDA Toolkit 및 `nvidia-smi` 확인 필수.

---

## 🚨 핵심 준수 규칙 (Critical Rules)

1. **커널 제약 준수**: 커널 내부에서 NumPy/SciPy 호출 시 컴파일 에러가 발생합니다. 반드시 외부로 분리하십시오.
2. **타겟 선택 최적화**: 큐비트 수 $\rightarrow$ 메모리 $\rightarrow$ 병렬성 순으로 검토하여 `nvidia` $\rightarrow$ `mgpu` $\rightarrow$ `mqpu` 순으로 타겟을 결정하십시오.
3. **자격 증명 보안**: QPU 제공자 토큰은 반드시 쉘 세션의 환경 변수로 export 하여 사용하십시오.
4. **검증 우선**: 모든 설치 및 설정 변경 후에는 Bell State 커널의 출력값이 기대치와 일치하는지 확인하십시오.

---

## 🔍 트러블슈팅 가이드

| 증상 | 원인 | 해결책 |
|---|---|---|
| **`pip install` 후 Import 에러** | Python 버전 미달 또는 OS 미지원 | Python 3.10+ 확인 및 Linux/macOS 환경인지 확인 |
| **GPU 미검출 (No GPU detected)** | CUDA Toolkit 미설치 또는 드라이버 문제 | `nvidia-smi` 작동 확인 후 `qpp-cpu`로 폴백하여 기능 테스트 |
| **커널 컴파일 에러** | 커널 내 비지원 Python 구문 사용 | `@cudaq.kernel` 내부에서 NumPy/SciPy 사용 여부 확인 및 제거 |
| **QPU 제출 실패** | 자격 증명 누락 또는 만료 | 환경 변수 설정 확인 및 제공자 포털에서 토큰 유효성 검사 |
| **`mgpu` 실행 에러** | MPI 환경 미구성 | MPI 라이브러리 설치 및 `mpirun` 설정 확인 |
