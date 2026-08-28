---
name: doca-pcc-counters
description: Arm and read fixed firmware/hardware PCC (Programmable Congestion Control) diagnostic counters on ConnectX/BlueField devices via the pcc_counters.sh script.
version: 2.1.0
license: Apache-2.0
metadata:
  author: NVIDIA AI-Q Blueprint Team
  tags:
    - doca
    - pcc
    - congestion-control
    - diagnostics
    - networking
  domain: networking-diagnostics
---

# 🎴 Skill Card: doca-pcc-counters (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `doca-pcc-counters`
- **도메인**: DOCA Networking Diagnostics
- **설명**: ConnectX 및 BlueField 장치의 펌웨어/하드웨어 PCC(Programmable Congestion Control) 진단 카운터(CNP, RTT, WRED-drop 등)를 `pcc_counters.sh` 스크립트를 통해 활성화(`set`)하고 읽는(`query`) 스킬입니다.
- **핵심 목표**: 장치의 혼잡 제어 상태를 정밀하게 진단하기 위해 하드웨어 수준의 카운터 데이터를 정확하게 캡처하고 분석하는 것입니다.

## 🚀 실행 파이프라인 (Execution Pipeline)

### 1. 환경 전제 조건 확인 (Pre-flight Check)
리서치 전, 다음 사항이 충족되었는지 확인합니다.
- **설치 확인**: DOCA/MFT가 설치되어 있고 `pcc_counters.sh` 스크립트가 도구 디렉토리에 존재하는지 확인합니다.
- **권한 및 마운트**: `root` 권한(sudo)이 필요하며, `debugfs`가 마운트되어 있어야 합니다 (`mount | grep debugfs`).
- **장치 인식**: `sudo mst start` 후 `sudo mst status -v`를 통해 대상 NIC의 정확한 mst 장치 경로(예: `/dev/mst/mt41692_pciconf0`)를 확보합니다.

### 2. 카운터 활성화 (Arming - `set`)
`query`를 수행하기 전, 반드시 어떤 카운터를 수집할지 장치에 프로그래밍해야 합니다.
- **명령어**: `sudo ./pcc_counters.sh set <mst-device-path>`
- **동작**: 장치를 PCI 주소로 변환한 후, `/sys/kernel/debug/mlx5/<pci>/diag_cnt/` 인터페이스에 고정된 카운터 ID 리스트와 샘플링 파라미터를 기록합니다.
- **주의**: 이는 특권 권한의 `debugfs` 쓰기 작업입니다. 실행 전 반드시 대상 장치와 PCI BDF를 확인하고 사용자의 승인을 받으십시오.

### 3. 카운터 읽기 (Reading - `query`)
활성화된 카운터의 현재 값을 읽어옵니다.
- **명령어**: `sudo ./pcc_counters.sh query <mst-device-path>`
- **동작**: `diag_cnt/dump`를 읽어 `Counter: <NAME> Value: <n>` 형식으로 출력합니다.
- **데이터 처리**: 출력된 카운터 라인을 그대로 인용(Quote)하며, 임의로 수정하거나 요약하지 않습니다.

### 4. 진단 및 분석 (Diagnosis & Analysis)
- **상태 비교**: 변화 전 `query` $\rightarrow$ 제어 변수 변경 $\rightarrow$ 변화 후 `query` 순으로 실행하여 델타(Delta)를 분석합니다.
- **제로 값 해석**: 카운터가 0인 경우, 도구 오류가 아니라 실제로 해당 이벤트(혼잡, RTT 요청 등)가 발생하지 않았을 가능성이 높으므로 실제 트래픽 존재 여부를 먼저 확인합니다.

---

## 🛠️ 상세 구현 가이드 (Operational Deep-Dive)

### 1. 고정 카운터 셋 (Fixed Counter Set)
본 스크립트는 펌웨어에 내장된 다음 고정 카운터들을 보고합니다. (사용자 확장 불가)
- **CNP**: `PCC_CNP_COUNT` (혼잡 알림 패킷 수)
- **RTT Perf**: `MAD_RTT_PERF_CONT_REQ`, `MAD_RTT_PERF_CONT_RES`
- **WRED Drops**: `SX_EVENT_WRED_DROP`, `SX_RTT_EVENT_WRED_DROP`, `ACK_EVENT_WRED_DROP`, `CNP_EVENT_WRED_DROP`, `RTT_EVENT_WRED_DROP`
- **Events**: `HANDLED_SXW_EVENTS`, `HANDLED_RXT_EVENTS`
- **Port Specific**: `DROP_RTT_PORT[0|1]_REQ/RES`, `RTT_GEN_PORT[0|1]_REQ/RES`

### 2. 에러 분류 및 대응 (Error Taxonomy)
| 에러 메시지/증상 | 원인 | 해결책 |
|---|---|---|
| `ERROR: Bad Device` | mst 장치 경로 불일치 | `sudo mst status -v`로 정확한 경로 재확인 |
| `Bad Request: choose set or query` | 잘못된 첫 번째 인자 사용 | 오직 `set` 또는 `query`만 사용 가능 |
| 값이 0이거나 stale함 | `set` 명령 미실행 | 반드시 `set`을 먼저 실행한 후 `query` 수행 |
| 권한/접근 에러 | sudo 미사용 또는 debugfs 미마운트 | `sudo` 실행 및 `mount \| grep debugfs` 확인 |

---

## ⚠️ 제약 사항 및 주의 사항 (Gotchas)

- **실행 순서 엄수**: 항상 `set` $\rightarrow$ `query` 순서를 지켜야 합니다. `set` 없이 `query`를 하면 이전 설정값이나 빈 값이 출력됩니다.
- **독립성**: 이 스크립트는 **펌웨어/HW 진단 카운터**를 읽는 도구입니다. 커스텀 `doca-pcc` DPA 커널 로딩 여부와 관계없이 작동하며, 커스텀 알고리즘 제어는 `doca-pcc` 스킬에서 다룹니다.
- **고위험 결정**: 카운터 읽기 값에 기반한 혼잡 제어(CC) 파라미터 변경은 패브릭 전체의 안정성에 영향을 줄 수 있는 고위험 작업입니다. 분석 결과에 따른 설정 변경은 반드시 사용자의 도메인 분석 후 수행하십시오.
- **조작 금지**: 배포된 `pcc_counters.sh` 스크립트를 임의로 수정하거나, 존재하지 않는 플래그/서브커맨드를 발명하여 사용하지 마십시오.

---

## 🔍 진단 래더 (Diagnostic Ladder)

1. **스크립트 존재 확인**: `/opt/mellanox/doca` 하위에 스크립트가 있는가? $\rightarrow$ 없으면 `doca-setup`으로 라우팅.
2. **장치 경로 검증**: `mst status -v`에 출력된 경로와 입력 경로가 일치하는가? $\rightarrow$ 불일치 시 `Bad Device` 에러.
3. **권한/인터페이스 확인**: `sudo` 권한이 있는가? `debugfs`가 `/sys/kernel/debug`에 마운트 되었는가?
4. **활성화 상태 확인**: `set` 명령이 성공적으로 수행되었는가? $\rightarrow$ 미수행 시 0 또는 stale 값 출력.
5. **트래픽 유무 확인**: 실제 RoCE 트래픽이 흐르고 있는가? $\rightarrow$ 트래픽이 없으면 카운터 0은 정상.

## 🔗 코드 앵커 및 참조
- **핵심 스크립트**: `tools/pcc_counters/pcc_counters.sh`
- **관련 스킬**:
    - `doca-pcc`: 커스텀 CC 알고리즘 설계 및 로드 (별개 표면)
    - `doca-setup`: 환경 준비 및 mst/debugfs 설정
    - `doca-debug`: 교차 진단 래더
    - `doca-public-knowledge-map`: 공식 PCC 문서 라우팅
