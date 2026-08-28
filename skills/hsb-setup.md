---
license: Apache-2.0
name: hsb-setup
description: >
  Expert guide for bringing up the NVIDIA Holoscan Sensor Bridge (HSB) demo
  end to end: prove the remote devkit and sensor topology, detect the host
  platform, prepare host networking and Docker, build/run the correct demo
  container, verify board connectivity and FPGA compatibility, then cleanly
  hand the devkit back to the operator.
---

# Holoscan Sensor Bridge Demo Bring-Up (Ultra-Detailed)

Use this skill for the **interactive, end-to-end first bring-up** of the Holoscan Sensor Bridge (HSB) demo environment. It covers the devkit, the attached Sensor Bridge board, host prerequisites, the HSB repository, the demo container, and first connectivity validation.

> **Execution boundary:** This workflow has remote side effects: it can connect through SSH, change devkit network/Docker configuration, clone or refresh a repository, build native/container artifacts, and start applications. **Do not start it automatically.** Run it only after an explicit user request and phase-by-phase approval. `--y` does not bypass the mandatory token-budget gate.

## 1. Entry Gates — Complete in Order

### 1.1 Resolve and display execution inputs
Before connecting to a devkit or changing anything, resolve and show these settings (redact secrets):

| Input | Meaning | Requirement |
| --- | --- | --- |
| `SSH_TARGET` | Remote devkit login, such as `{user}@{host}` | Required unless running directly on the devkit. |
| `REMOTE_ROOT` | Remote directory for the clone and build | Required unless running directly on the devkit. |
| `REMOTE_SUDO` | `sudo`, `sudo -n`, or empty | Defaults to `sudo` when unspecified. |
| `REMOTE_SSH_OPTS` | Additional SSH options | Optional. |
| `HSB_PLATFORM` | Platform hint | Optional; hardware detection can replace it. |
| `HSB_REPO` | Repository source | Defaults to public `https://github.com/nvidia-holoscan/holoscan-sensor-bridge.git`; an explicit `--repo` wins. |

Stop and ask the operator if required remote inputs are missing. For an SSH repository, warn when no SSH key is configured and give setup guidance rather than attempting an unauthenticated clone.

### 1.2 Present the phase plan and await acknowledgement
The complete workflow is deliberately gated:

0. **Token-budget preflight** — confirm sufficient session budget before a remote command or devkit change.
1. **Platform and repository** — confirm topology, establish SSH if needed, detect platform, clone/refresh the repository, and read its user guide.
2. **Host prerequisites** — validate host dependencies, networking, Docker access, and display/network prerequisites.
3. **Native CLI build** — **AGX Thor only**; skip for all other platforms.
4. **Demo build and proof** — build/run the matching container, ping the board, and verify FPGA compatibility.
5. **Issues report** — summarize failures/repairs and offer Markdown export.
6. **Clean handoff** — stop applications, exit the container, and leave the operator at the repository root.

After every non-final phase, summarize what ran, result, issues, and the next action; then wait for `Proceed to Phase <N+1>?`. Concise summaries are the default; `--verbose` may show complete command output.

### 1.3 Token-budget preflight (Phase 0)
Run this **before** SSH, repository clone, package/configuration checks, builds, reboot, or any devkit change.

- Reserve at least **280,000 tokens** for IGX Orin, AGX Orin, or DGX Spark; reserve **340,000 tokens** for AGX Thor.
- Add **60,000 tokens** when verbose output, a custom repository, SSH-key remediation, reboot recovery, or substantial troubleshooting is expected; choose the larger estimate while platform remains unknown.
- Verify remaining session/account allowance using an authoritative mechanism. If it cannot be verified, or is below the estimate, stop before Phase 1.
- Display the estimate, basis, safety margin, observed availability (or `unverified`), and a PASS/FAIL result. The operator must intentionally confirm that the available budget is at least the estimate before continuing.

## 2. Platform Contract and Build Selection

### 2.1 Confirm physical topology before software work
Confirm that the selected devkit is connected to the HSB board, the board is powered, outside network access is available, and the devkit has the supported OS. After resolving the platform, read the cloned repository's user guide and show a simple devkit-to-sensor topology for the operator to confirm.

### 2.2 Detect hardware; do not trust a stale platform hint
On the devkit, inspect `/sys/class/dmi/id/product_name` and compare it with `HSB_PLATFORM` using case-insensitive matching:

| Product name contains | Effective platform | Additional decision |
| --- | --- | --- |
| `IGX Orin` | IGX Orin | Must distinguish iGPU from dGPU OS/configuration. |
| `AGX Orin` | AGX Orin | No additional GPU branch. |
| `AGX Thor` | AGX Thor | Enables the native CLI build phase. |
| `DGX Spark` | DGX Spark | No additional GPU branch. |

If hardware detection is recognized and `HSB_PLATFORM` is empty or disagrees, update the session value to match hardware and explicitly notify the operator. If detection fails/unrecognized but a platform hint exists, keep the hint and warn; if both are absent, ask the minimum platform question.

### 2.3 Container build mapping
Use the checked-out repository's current user guide as the authority. Unless it supersedes this mapping:

| Platform | Demo-container build mode |
| --- | --- |
| IGX Orin with dGPU OS/configuration | `sh docker/build.sh --dgpu` |
| IGX Orin iGPU | `sh docker/build.sh --igpu` |
| AGX Orin | `sh docker/build.sh --igpu` |
| AGX Thor | `sh docker/build.sh --igpu` |
| DGX Spark | `sh docker/build.sh --igpu` |

The input “IGX Orin” alone is insufficient: ask whether its OS/configuration is iGPU or dGPU before selecting the build. AGX Thor is the only supported branch that performs the native CLI build in Phase 3.

## 3. Execution Pipeline

### 3.1 Phase 1 — connect, reconcile platform, and prepare source

1. If running externally, establish the approved SSH connection. If the skill runs directly on the devkit, do not create an unnecessary remote hop.
2. Read the DMI product name, reconcile `HSB_PLATFORM`, and persist the effective selection in remote session state for later phases.
3. Clone the selected repository or refresh the existing checkout to the latest `main`; preserve the operator's custom source choice.
4. Read the repository's `docs/user_guide` before choosing host setup, container command, or FPGA workflow. Do not manufacture repository-specific paths or flags from memory.

### 3.2 Phase 2 — host prerequisite and network readiness

Validate the chosen platform's documented host configuration before building:

- Required system dependencies and Git LFS content.
- Docker access and any documented display/X11 prerequisite.
- The network route/interface needed for the Sensor Bridge board.
- Platform-specific host configuration from the checked-out user guide.

Use safe, targeted remediation only. For a transient failure, re-run the failed command once; then repair known prerequisites (for example Git LFS, Docker access, display access, or route), refresh repository/LFS state, and re-run **only the failed phase**. Explain the failed command, likely cause, attempted safe repair, and outcome rather than dumping raw logs.

### 3.3 Phase 3 — AGX Thor native CLI build only

For AGX Thor, follow the repository's documented native CLI/SIPL/FuSa build prerequisites before the container phase. For IGX Orin, AGX Orin, and DGX Spark, explicitly mark this phase skipped; never run a Thor-only build merely because the source tree contains it.

### 3.4 Phase 4 — build, run, and verify the demo

1. Build the demo container with the platform-selected mode from §2.3.
2. Start the documented demo workflow and monitor the application/container status.
3. Validate board connectivity by pinging **`192.168.0.2`**. If it fails, report evidence and ask whether the HSB board uses a different address before changing network assumptions.
4. Read the FPGA version at register **`0x80`** using the repository-documented access path. Compare it with the HSB host software version.
5. If FPGA and host software are incompatible, do not flash as an implicit repair. Explain the mismatch and hand off to `hsb-flash` for its separate destructive/update workflow.

### 3.5 Phases 5–6 — report and clean handoff

For every issue, record the symptom, failed command/evidence, root cause assessment, repair attempted, and result. Offer to export the final report as Markdown.

When setup is finished—or the operator asks to stop—shut down running applications, exit the demo container, and return terminal control at the repository root. Do not leave a background demo process holding device, network, or GPU resources without the operator's consent.

## 4. Failure Ladder and Safe Recovery Bound

| Order | Failure class | Safe response |
| --- | --- | --- |
| **1** | Missing remote inputs or unverified budget | Stop before Phase 1; ask for inputs or a verifiable budget result. |
| **2** | Physical topology, power, OS, or external-network issue | Ask the operator to correct the physical/platform prerequisite; do not paper over it with software changes. |
| **3** | Platform hint disagrees with detected DMI product name | Use the recognized hardware result, notify the operator, then use the matching branch. |
| **4** | Clone, Git LFS, SSH-key, Docker, display, or route prerequisite failure | Apply one documented/safe repair, then re-run only the affected phase. |
| **5** | Build/container failure | Preserve concise diagnostics, use the checked-out user guide, and avoid changing platform/build mode without evidence. |
| **6** | Board ping or FPGA-version failure | Ask about an alternate board IP; route an actual FPGA mismatch to `hsb-flash`; do not flash automatically. |

If the same phase remains blocked after the bounded repair sequence, stop with a concise diagnosis and copy-paste commands the operator may run. Do not restart the entire workflow or continuously retry remote actions.

## 5. Deferred Work and Related HSB Skills

- **FPGA programming or version remediation** → `hsb-flash`; it must be separately authorized.
- **Creating an HSB IP block/top or packetizer definitions** → `hsb-ip-create-top`, `hsb-ip-def`, and `hsb-ip-packetizer`.
- **Writing or running an HSB application** → `hsb-app`.
- **Validation beyond first connectivity** → `hsb-test`.
- **General Holoscan runtime/package setup not specific to the Sensor Bridge demo** → the applicable `holoscan-*` skill.
