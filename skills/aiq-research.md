# 🎴 Skill Card: aiq-research (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `aiq-research`
- **도메인**: Research Agents (AI-Q Deep Research)
- **설명**: Reachable NVIDIA AI-Q Blueprint 백엔드를 통해 딥 리서치(Deep Research) 또는 AI-Q 리서치 쿼리를 실행하고 결과를 분석하는 스킬입니다.
- **핵심 목표**: `scripts/aiq.py` 헬퍼 스크립트를 사용하여 AI-Q 서버에 연구 요청을 보내고, 비동기 작업(Job)을 폴링하여 최종 리서치 보고서를 성공적으로 추출하는 것입니다.

## ⚙️ 리서치 수행 프로세스 (Research Pipeline)

리서치 요청은 다음 단계에 따라 엄격하게 수행됩니다.

### 단계 1: 백엔드 엔드포인트 확인 (Resolve Backend)
쿼리를 보내기 전, 대상 서버가 응답하는지 먼저 확인합니다.
- **기본 경로**: `http://localhost:8000` (또는 `AIQ_SERVER_URL` 환경 변수 값)
- **검증 명령어**:
  ```bash
  python3 $SKILL_DIR/scripts/aiq.py health
  ```
- **실패 시 대응**:
  - 백엔드가 없으면 사용자에게 URL을 묻거나, `aiq-deploy` 스킬로 핸드오프하여 서버를 먼저 배포합니다.
  - `401/403` 에러 발생 시, 본 스킬은 인증을 관리하지 않음을 알리고 환경 설정을 요청합니다.

### 단계 2: 리서치 요청 전송 (Submit Request)
사용자 쿼리를 서버로 전송합니다. 전송 전 반드시 대상 URL을 명시하여 사용자의 신뢰를 확인합니다.
- **실행 명령어**:
  ```bash
  python3 $SKILL_DIR/scripts/aiq.py chat \"<USER_QUESTION>\"
  ```
- **응답 유형**:
  - **즉시 응답**: 일반 JSON 결과가 반환되면 즉시 사용자에게 제시합니다.
  - **비동기 응답**: `{\"status\": \"deep_research_running\", \"job_id\": \"<JOB_ID>\"}` 형태의 응답이 오면 **단계 3(폴링)**으로 진입합니다.

### 단계 3: 비동기 작업 폴링 (Poll Async Jobs)
`job_id`가 반환된 경우, 작업이 완료될 때까지 상태를 확인합니다.
- **폴링 명령어**:
  ```bash
  python3 $SKILL_DIR/scripts/aiq.py research_poll <JOB_ID>
  ```
- **결과 처리**: 최종 리포트 JSON이 반환되면 인용구와 소스 URL을 유지한 채 사용자에게 제시합니다.

### 단계 4: 보고서 사후 처리 및 저장 (Post-processing)
완료된 보고서를 로컬 파일로 저장하거나 아티팩트를 다운로드합니다.
- **휴대용 리포트 생성** (`report.md` 및 `artifacts/` 폴더):
  ```bash
  python3 $SKILL_DIR/scripts/aiq.py report <JOB_ID> --out-dir ./my-report
  ```
- **아티팩트(차트, CSV 등) 개별 다운로드**:
  ```bash
  python3 $SKILL_DIR/scripts/aiq.py artifacts <JOB_ID> --download-dir ./aiq-artifacts
  ```

---

## 🛠️ 사전 체크리스트 및 환경 설정 (Pre-flight Checks)

### 1. 필수 요구사항
- **Python**: 3.11 이상 (`python3` 명령어로 실행 가능해야 함).
- **네트워크**: AI-Q 백엔드 URL(기본 `http://localhost:8000`)에 대한 접근 권한.
- **환경 변수**: 백엔드가 기본 포트가 아닐 경우 `AIQ_SERVER_URL` 설정 필요.

### 2. 유효성 확인
리서치 요청 전 다음 명령어가 성공하는지 확인하십시오.
```bash
python3 $SKILL_DIR/scripts/aiq.py health
python3 $SKILL_DIR/scripts/aiq.py agents
```
- `agents` 명령어가 실패하면 백엔드가 API-enabled 설정이 아니거나 버전이 호환되지 않는 것입니다.

---

## 🚨 핵심 준수 규칙 (Critical Rules)

1. **엔드포인트 신뢰 확인**: 비로컬(non-local) URL로 쿼리를 보내기 전, 반드시 사용자에게 해당 URL이 신뢰할 수 있는 곳인지 확인받으십시오.
2. **비밀값 전송 금지**: 쿼리 텍스트에 API 키, 쿠키, 베어러 토큰 등 비밀값을 절대 포함하지 마십시오.
3. **인용 유지**: 리포트 제시 시 모든 인용(Citations)과 소스 URL을 절대 생략하거나 수정하지 마십시오.
4. **자동 재시도 금지**: 작업 상태가 `failed`, `failure`, `cancelled`인 경우 자동 재시도하지 말고 에러를 보고한 뒤 사용자의 판단을 기다리십시오.

---

## 🔍 트러블슈팅 및 명령어 가이드 (Troubleshooting)

### 1. 주요 스크립트 API 레퍼런스
| 스크립트 명령어 | 목적 | 주요 인자 |
|---|---|---|
| `aiq.py health` | 서버 응답 확인 | 없음 |
| `aiq.py chat` | 쿼리 전송 (동기/비동기) | `<query>` |
| `aiq.py agents` | 사용 가능한 에이전트 타입 목록 확인 | 없음 |
| `aiq.py research` | 비동기 요청 $\rightarrow$ 폴링 $\rightarrow$ 리포트 출력까지 한 번에 수행 | `<query> [agent_type]` |
| `aiq.py research_poll` | 기존 작업의 완료 대기 및 결과 수신 | `<job_id>` |
| `aiq.py status` | 작업 상태 및 아티팩트 확인 | `<job_id>` |
| `aiq.py report` | 최종 리포트 가져오기 / 내보내기 | `<job_id> [--out-dir DIR]` |
| `aiq.py artifacts` | 아티팩트 목록 확인 및 다운로드 | `<job_id> [--download-dir DIR]` |
| `aiq.py cancel` | 실행 중인 작업 강제 종료 | `<job_id>` |

### 2. 주요 이슈 해결
| 증상 | 원인 | 해결책 |
|---|---|---|
| **Connection Refused** | AI-Q 백엔드 미구동 또는 포트 불일치 | `aiq-deploy`를 통해 서버를 띄우거나 `AIQ_SERVER_URL` 재설정 |
| **HTTP 401/403** | 백엔드 인증 설정 활성화됨 | 인증이 비활성화된 백엔드를 사용하거나 전용 인증 스킬 사용 |
| **Job stuck in 'running'** | 서버 측 처리 지연 또는 폴링 랙 | `aiq.py status <JOB_ID>`로 실제 서버 상태 확인 |
| **Artifacts missing** | 아티팩트 생성 실패 또는 경로 오류 | `aiq.py artifacts <JOB_ID>`로 사용 가능한 목록 확인 |

## 🔗 참조 (References)
- **GitHub Repo**: https://github.com/NVIDIA-AI-Blueprints/aiq
- **인프라 배포 가이드**: `../aiq-deploy/SKILL.md`
- **헬퍼 스크립트 소스**: `scripts/aiq.py`
