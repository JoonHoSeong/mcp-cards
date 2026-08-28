# 🎴 Skill Card: tao-run-platform (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `tao-run-platform`
- **도메인**: TAO / Execution SDK (Platform Management)
- **설명**: TAO Toolkit의 작업을 GPU 가속 플랫폼(Brev, SLURM, Local Docker, Kubernetes)에 제출하고 모니터링하는 Python SDK 기반의 실행 계층입니다. 단순한 `docker run`을 넘어, 작업 핸들링, S3 I/O 래핑, 다중 노드 분산 학습 및 플랫폼 전용 기능을 제공합니다.
- **핵심 목표**: 복잡한 인프라(클러스터, 클라우드 인스턴스) 상에서 TAO 학습/추론 작업을 추상화된 인터페이스로 제출하고, 그 상태와 로그를 실시간으로 추적하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **작업 추적 필요**: `Job` 핸들을 통해 작업 상태(Pending $ightarrow$ Running $ightarrow$ Complete/Error)를 폴링하고 로그를 스트리밍하고 싶을 때.
- **S3 데이터 통합**: 입력 데이터 다운로드와 결과 업로드를 SDK 수준에서 자동으로 처리하고 싶을 때.
- **분산 학습 실행**: SLURM이나 Kubernetes 환경에서 다중 GPU/다중 노드 분산 학습을 설정해야 할 때.
- **플랫폼 전용 기능 활용**: Brev 인스턴스 재사용, SLURM 큐 관리, K8s Job 생성 등 플랫폼 특화 기능을 사용할 때.

---

## 🏗️ 플랫폼 아키텍처 및 SDK 구조

### 1. 지원 플랫폼 및 필수 자격 증명 (Credentials)
SDK는 환경 변수를 통해 자격 증명을 읽으며, 플랫폼별로 요구 사항이 다릅니다.

| 플랫폼 | 필수 환경 변수 | 선택 사항 | 특이 사항 |
| :--- | :--- | :--- | :--- |
| **Brev** | (기본 `brev login` 사용 가능) | `BREV_API_TOKEN` | 클라우드 GPU 인스턴스 기반 |
| **S3 I/O** | `S3_BUCKET_NAME`, `ACCESS_KEY`, `SECRET_KEY` | `S3_ENDPOINT_URL`, `CLOUD_REGION` | 모든 플랫폼 공통 S3 래핑 시 필요 |
| **Container** | `NGC_KEY` | `HF_TOKEN` | NVIDIA NGC / HuggingFace 이미지 풀링용 |
| **SLURM** | `SLURM_USER`, `SLURM_HOSTNAME` | — | HPC 클러스터 전용 |

### 2. 핵심 API 인터페이스 (Universal SDK Shape)
모든 플랫폼 SDK는 동일한 추상화 인터페이스를 구현하여 일관된 제어를 제공합니다.

- `create_job(image, command, gpu_count=1, ...)` $ightarrow$ `Job` 객체 반환
- `get_job_status(job_id)` $ightarrow$ `JobStatus` (Pending, Running, Complete, Error, Canceled)
- `get_job_logs(job_id, tail=N)` $ightarrow$ 최신 로그 텍스트 반환
- `cancel_job(job_id)` $ightarrow$ 작업 강제 종료
- `get_failure_analysis(job_id)` $ightarrow$ `ERR_PROGRAM`, `ERR_INFRA` 등 근본 원인 분석 결과

---

## 🛠️ 구현 가이드 및 핵심 워크플로우

### 1. 작업 제출 프로세스 (Submission Flow)
단순한 명령 실행이 아니라, SDK는 내부적으로 **`build_entrypoint`** 과정을 거쳐 컨테이너 명령을 생성합니다.

1. **이미지 결정**: `resolve_container_image()`를 통해 `versions.yaml` 기반의 최적 이미지 URI 결정.
2. **엔트리포인트 생성**:
    - `script_runner` 런타임을 베이스64 형태로 인라인 삽입.
    - 입력 데이터(S3/HF/NGC) $ightarrow$ 로컬 경로로 변환 및 다운로드 로직 추가.
    - 결과 데이터 $ightarrow$ S3 업로드 로직 추가.
3. **작업 생성**: `sdk.create_job()`을 호출하여 플랫폼에 제출.

### 2. 데이터 흐름 및 결과 저장 (Output Resolution)
결과 저장소는 런타임 환경 변수에 따라 동적으로 결정됩니다.

| 환경 변수 | 결과 저장 경로 | 특이 사항 |
| :--- | :--- | :--- |
| `TAO_RESULTS_ROOT` 설정됨 | `{TAO_RESULTS_ROOT}/<job_id>/<key>/` | Lustre, PVC, Bind Mount 등 영구 저장소 |
| `S3_BUCKET_NAME` 설정됨 | `s3://{bucket}/results/<job_id>/<key>/` | 네트워크 기반 자동 업로드 (S3 fallback) |
| 둘 다 없음 | `/results/<job_id>/<key>/` | 컨테이너 내 임시 저장 (종료 후 소멸 - 경고 발생) |

### 3. 스펙(Spec) 구성의 핵심: 중첩 딕셔너리 (Nested Dicts)
가장 흔한 오류는 스펙을 평면적인(flat) 키-값 쌍으로 작성하는 것입니다. **반드시 모델 컨테이너가 기대하는 중첩 구조를 그대로 반영해야 합니다.**

- **❌ 잘못된 예**: `{"section.subsection.key": "value"}` (문자열 키로 인식되어 무시됨)
- **✅ 올바른 예**: `{"section": {"subsection": {"key": "value"}}}`

---

## 🔍 모니터링 및 트러블슈팅 (Monitoring & Debugging)

### 1. 실시간 폴링 패턴 (Polling Pattern)
인터랙티브 환경에서는 작업이 종료될 때까지 일정 간격(기본 5분)으로 상태를 확인합니다.
- **상태 변화**: `Pending` $ightarrow$ `Running` $ightarrow$ `Complete` (성공) 또는 `Error` / `Canceled` (실패).
- **주의**: 작업이 `Pending` 상태에서 오래 머물더라도(큐 대기), 사용자가 중단하기 전까지는 폴링을 계속 유지해야 합니다.

### 2. 실패 분석 매트릭스
`get_failure_analysis()` 결과에 따른 대응 전략입니다.
- **`ERR_PROGRAM`**: 모델 코드 또는 설정 파일(Spec) 오류 $ightarrow$ 로그 확인 후 스펙 수정.
- **`ERR_INFRA`**: GPU 드라이버 충돌, OOM(Out of Memory), 네트워크 단절 $ightarrow$ GPU 수 증설 또는 인스턴스 재시작.

## 📚 관련 참조 스킬
- **런타임 준비**: [`tao-setup-nvidia-gpu-host`](../tao-setup-nvidia-gpu-host/SKILL.md)
- **자동화 학습**: [`tao-run-automl`](../tao-run-automl/SKILL.md) (AutoML 정책이 활성화된 모델의 경우 기본 경로)
- **플랫폼 세부 사항**: `references/platform-notes.md` (Brev, SLURM, K8s 특이사항)
