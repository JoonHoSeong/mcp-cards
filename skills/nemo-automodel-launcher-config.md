---
name: nemo-automodel-launcher-config
description: Configure NeMo AutoModel job launches for interactive runs, Slurm clusters, and SkyPilot cloud execution.
version: 0.1.1
license: Apache-2.0
metadata:
  author: NVIDIA
  tags:
    - nemo-automodel
    - launcher-config
  domain: infra
---

# 🎴 Skill Card: nemo-automodel-launcher-config (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `nemo-automodel-launcher-config`
- **도메인**: NeMo AutoModel Infrastructure
- **설명**: NeMo AutoModel의 작업 실행(Job Launch) 설정을 관리하는 가이드입니다. 대화형 실행(Interactive), Slurm HPC 클러스터, 그리고 SkyPilot 클라우드 실행의 세 가지 모드를 지원하며, 각 환경에 맞는 리소스 할당, 컨테이너 설정 및 프로파일링 구성을 다룹니다.
- **핵심 목표**: 사용자의 실행 환경에 최적화된 런처 YAML 설정을 생성하고, 멀티노드 통신을 위한 랑데부(Rendezvous) 설정 및 인프라 제약 사항을 해결하여 학습 작업을 안정적으로 제출하는 것입니다.

## 🚀 실행 모드 결정 트리 (Launcher Selection)

| 환경 (Environment) | 추천 모드 | 핵심 도구 | 특징 |
|---|---|---|---|
| **단일 노드 / 개발 및 디버깅** | **Interactive** | `torchrun` | 설정 파일에 `slurm:` 또는 `skypilot:` 섹션이 없으면 자동 선택 |
| **HPC 클러스터 (On-prem)** | **Slurm** | `sbatch` / `Enroot` | `SlurmConfig` 기반 SBATCH 스크립트 자동 생성, 컨테이너 마운트 관리 |
| **퍼블릭 클라우드 (AWS/GCP/Azure)** | **SkyPilot** | `sky` CLI | 클라우드 무관(Agnostic) 실행, Spot 인스턴스 최적화, 자동 프로비저닝 |

---

## 🛠️ 상세 설정 가이드 (Execution Pipeline)

### 1. Slurm 구성 (HPC Cluster)
Slurm 모드는 `SlurmConfig` 데이터클래스를 통해 SBATCH 스크립트를 생성합니다.

**핵심 YAML 템플릿:**
```yaml
slurm:
  job_name: llm_finetune
  nodes: 2
  ntasks_per_node: 8
  time: "04:00:00"
  account: my_account
  partition: batch
  container_image: nvcr.io/nvidia/nemo:dev
  hf_home: ~/.cache/huggingface
  extra_mounts:
    - source: /data
      dest: /data
  env_vars:
    HF_TOKEN: "${HF_TOKEN}"
  master_port: 13742
  nsys_enabled: true # Nsight Systems 프로파일링 활성화
```
- **World Size 계산**: `WORLD_SIZE = nodes * ntasks_per_node`로 자동 결정됩니다.
- **통신 설정**: `MASTER_ADDR`와 `MASTER_PORT`가 자동으로 설정되어 멀티노드 랑데부를 수행합니다.

### 2. SkyPilot 구성 (Cloud)
`SkyPilotConfig`를 통해 클라우드 인스턴스 사양과 리전, 비용 최적화를 설정합니다.

**핵심 YAML 템플릿:**
```yaml
skypilot:
  cloud: aws
  accelerators: "H100:8"
  num_nodes: 2
  use_spot: true # 스팟 인스턴스 사용 (비용 절감)
  disk_size: 200
  region: us-east-1
  setup: "pip install nemo-automodel"
  env_vars:
    HF_TOKEN: "${HF_TOKEN}"
```
- **Spot 인스턴스 전략**: `use_spot: true` 설정 시 선점(Preemption) 가능성이 높으므로, `step_scheduler.checkpoint_interval`을 짧게 설정하고 `restore_from.path`를 통해 복구하는 전략이 필수적입니다.

### 3. Nsight Systems (nsys) 프로파일링
- **설정**: Slurm 섹션 내 `nsys_enabled: true` 지정.
- **동작**: 런처가 학습 커맨드를 `nsys profile`로 랩핑하여 실행하며, `.nsys-rep` 리포트 파일을 생성합니다.
- **주의**: 오버헤드와 대용량 아티팩트가 발생하므로 **진단 목적으로만 짧게 실행**하고, 실제 프로덕션 학습 시에는 반드시 비활성화하십시오.

---

## ⚠️ 제약 사항 및 주의 사항 (Gotchas)

- **포트 충돌 (Port Collisions)**: 기본 `master_port` (13742)가 동일 노드의 다른 작업에서 사용 중인 경우, 반드시 다른 포트로 변경하여 연결 실패를 방지하십시오.
- **컨테이너 마운트 (Mount Failures)**: `extra_mounts`의 `source` 경로는 할당된 **모든 노드**에 동일하게 존재해야 합니다. 경로 누락 시 컨테이너 시작 단계에서 실패합니다.
- **시간 제한 vs 비동기 체크포인트**: Slurm의 `time` 제한이 너무 타이트하면, 비동기 체크포인트 쓰기 작업이 완료되기 전에 프로세스가 종료되어 **체크포인트 파일이 손상(Corrupted)**될 수 있습니다. 최소 5~10분의 여유 시간을 두십시오.
- **환경 변수 구문**: YAML 내에서 `${VAR}` 구문을 사용해야 쉘 변수가 확장됩니다. 단순 변수명은 확장되지 않습니다.

## 🔍 진단 래더 (Diagnostic Ladder)

| 증상 | 체크포인트 | 해결책 |
|---|---|---|
| **Job Submission Failure** | YAML 섹션 이름 확인 | `slurm:` 또는 `skypilot:` 오타 확인 및 필수 필드 누락 체크 |
| **Multi-node Hang** | `master_port` 및 네트워크 확인 | 포트 충돌 확인 및 `MASTER_ADDR` 도달 가능성 체크 |
| **Container Startup Error** | `extra_mounts` 경로 확인 | 모든 계산 노드에 해당 소스 경로가 존재하는지 검증 |
| **Spot Job Loss** | 체크포인트 주기 확인 | `checkpoint_interval`을 단축하고 `restore_from` 경로 설정 확인 |

## 🔗 코드 앵커 (Code Anchors)
- `components/launcher/slurm/config.py`: `SlurmConfig` 데이터클래스 및 `VolumeMapping` 정의.
- `components/launcher/slurm/template.py`: SBATCH 스크립트 템플릿 생성 로직.
- `components/launcher/skypilot/config.py`: `SkyPilotConfig` 정의.
- `_cli/app.py`: CLI 진입점 및 런처 루팅 로직.
