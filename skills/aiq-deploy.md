# 🎴 Skill Card: aiq-deploy (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `aiq-deploy`
- **도메인**: NVIDIA AI-Q Blueprint Infrastructure
- **설명**: NVIDIA AI-Q Blueprint 서버의 설치, 배포, 실행, 검증, 트러블슈팅 및 중단을 관리하는 인프라 운영 스킬입니다.
- **핵심 목표**: `aiq-research` 스킬이 사용할 수 있도록 로컬 또는 자체 호스팅 환경에서 AI-Q 서버를 건강하게 배포하고, 검증된 `AIQ_SERVER_URL`을 제공하는 것입니다.

## ⚙️ 배포 경로 및 실행 방법 (Deployment Paths)

사용자의 요구사항에 따라 다음 네 가지 배포 모드 중 하나를 선택합니다.

### 1. Agent Skill Backend (백엔드 전용 - 권장)
`aiq-research`와 같은 에이전트 스킬이 사용할 백엔드 서비스만 실행합니다. (브라우저 UI 제외)
- **배포 방법 (Docker Compose)**:
  ```bash
  # .env 파일 준비
  test -f deploy/.env || cp deploy/.env.example deploy/.env
  
  # 백엔드 전용 배포 실행
  cd deploy/compose
  BUILD_TARGET=release docker compose --env-file ../.env -f docker-compose.yaml config --quiet
  BUILD_TARGET=release docker compose --env-file ../.env -f docker-compose.yaml up -d --build aiq-agent
  ```
- **특이사항**: `REQUIRE_AUTH=false` 설정 시 인증 없이 사용 가능 (로컬 단일 사용자 환경 전용).

### 2. UI Mode (브라우저 앱 포함)
백엔드와 프론트엔드 UI를 모두 실행하여 브라우저에서 AI-Q를 사용합니다.
- **배포 방법 (Docker Compose)**:
  ```bash
  cd deploy/compose
  docker compose --env-file ../.env -f docker-compose.yaml config --quiet
  docker compose --env-file ../.env -f docker-compose.yaml up -d --build
  ```
- **포트**: 백엔드 `8000`, 프론트엔드 `3000`.

### 3. CLI Mode (터미널 전용)
인터랙티브 터미널 기반의 AI-Q를 실행합니다. (상세 내용은 `references/terminal-cli.md` 참조)

### 4. Kubernetes/Helm Mode (클러스터 배포)
`kubectl` 및 `Helm`을 사용하여 K8s 클러스터에 배포합니다. (상세 내용은 `references/kubernetes-enlm.md` 참조)

---

## 🛠️ 사전 체크리스트 및 환경 설정 (Pre-flight Checks)

### 1. 필수 런타임 확인
- **Docker**: Docker Engine 및 Docker Compose v2.
- **Python**: 3.11+ 및 `uv`.
- **Node.js**: 20+ 및 `npm` (UI 모드 시).
- **K8s**: `kubectl` 1.28+, `Helm` 3.12+ (Helm 모드 시).

### 2. 포트 충돌 확인
배포 전 다음 포트들이 사용 중인지 확인하십시오.
```bash
for port in 8000 5432 3000; do
  if lsof -nP -iTCP:$port -sTCP:LISTEN >/dev/null 2>&1; then
    echo "port $port is already in use"
  else
    echo "port $port is free"
  fi
done
```
- `8000`: AI-Q Backend
- `5432`: PostgreSQL (DB)
- `3000`: AI-Q Frontend UI

### 3. 보안 및 비밀값 설정
- **`.env` 보호**: 비밀값이 깃에 올라가지 않도록 반드시 확인하십시오.
  ```bash
  git check-ignore deploy/.env
  ```
- **필수 키**: `NVIDIA_API_KEY` 및 검색 제공자 키(`TAVILY_API_KEY`, `SERPER_API_KEY` 또는 `EXA_API_KEY`)가 `deploy/.env`에 설정되어 있어야 합니다.

---

## 🚨 핵심 준수 규칙 (Critical Rules)

1. **비밀값 노출 금지**: 채팅창에 API 키를 직접 붙여넣지 마십시오. 반드시 `deploy/.env` 파일을 통해 설정하십시오.
2. **`.env` 덮어쓰기 금지**: 기존에 `deploy/.env` 파일이 존재하면 `cp` 명령어로 덮어쓰지 마십시오.
3. **버전 호환성**: 본 스킬은 **AI-Q Blueprint v2.1.0** 기준입니다. 메이저 버전이 다를 경우 호환되지 않을 수 있습니다.
4. **상태 검증 후 핸드오프**: 서버를 띄운 후 반드시 `/health` 엔드포인트를 확인하고, 검증된 `AIQ_SERVER_URL`을 `aiq-research`에 전달하십시오.

---

## 🔍 트러블슈팅 및 검증 (Troubleshooting & Validation)

### 1. 기본 건강 상태 검증
서버 기동 후 다음 명령어로 확인합니다.
```bash
curl -sf http://localhost:8000/health
```

### 2. 런타임 상태 확인 (Docker Compose)
```bash
# 컨테이너 실행 상태 확인
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E 'aiq-agent|aiq-postgres'

# DB 연결 확인
docker exec aiq-postgres pg_isready -U aiq -d aiq_jobs
docker exec aiq-postgres pg_isready -U aiq -d aiq_checkpoints
```

### 3. 주요 이슈 해결
| 증상 | 원인 | 해결책 |
|---|---|---|
| **포트 8000 사용 중** | 다른 서비스가 포트를 점유 | `lsof -nP -iTCP:8000`으로 프로세스 확인 후 종료 또는 `.env`에서 `PORT=8100`으로 변경 |
| **인증 실패 / 권한 없음** | API 키 누락 또는 잘못된 키 | `deploy/.env` 내 `NVIDIA_API_KEY` 및 검색 API 키 재확인 |
| **백엔드는 건강하나 research 실패** | 잘못된 config 파일 사용 | `configs/config_web_default_llamaindex.yml` 사용 여부 확인 및 재시작 |
| **DB 연결 실패** | PostgreSQL 컨테이너 미기동 | `docker compose logs aiq-postgres`로 로그 확인 후 재시작 |

## 🔗 참조 (References)
- **GitHub Repo**: https://github.com/NVIDIA-AI-Blueprints/aiq
- **내부 가이드**:
  - 환경 및 비밀값: `references/env-and-secrets.md`
  - Docker Compose 상세: `references/docker-compose.md`
  - 기본 검증: `references/validation.md`
  - 트러블슈팅: `references/troubleshooting.md`
  - 중단 및 정리: `references/shutdown.md`
