# 🛠️ rag-blueprint (NVIDIA RAG Blueprint)

NVIDIA RAG Blueprint의 배포, 설정, 문제 해결 및 관리를 위한 마스터 제어 스킬입니다. Docker, Helm, Library 모드 등 다양한 환경에서 RAG 인프라의 모든 기능을 제어합니다.

## 📌 핵심 기능 (Capabilities)
- **배포 및 설치**: Docker Compose, Kubernetes(Helm)를 통한 RAG 인프라 전체 배포.
- **기능 설정 (Configure)**: 
    - VLM, NeMo Guardrails, Agentic RAG, Query Rewriting, Ingestion 등 개별 기능 활성화/비활성화 및 설정.
    - LLM, Embedding, Ranking 모델 교체 및 인프라 설정(포트, GPU 할당).
- **문제 해결 (Troubleshoot)**: 서비스 상태 진단, 오류 분석 및 복구.
- **종료 및 정리 (Shutdown)**: 서비스 중지 및 리소스 정리.

## ⚙️ 실행 워크플로우 (Execution Workflow)

### 1. 의도 분석 및 라우팅 (Intent Routing)
사용자 요청에 따라 다음 참조 문서로 즉시 라우팅합니다:
- **배포/설치** $\rightarrow$ `references/deploy.md`
- **설정 변경** $\rightarrow$ 하단 [기능별 설정 매핑] 참조
- **문제 해결/디버깅** $\rightarrow$ `references/troubleshoot.md`
- **종료/정리** $\rightarrow$ `references/shutdown.md`

### 2. 설정 변경 프로세스 (Configuration Flow)
설정 변경 시 반드시 다음 단계를 준수합니다:
1. **환경 탐색**: 현재 실행 중인 서비스(NIM, RAG 서버 등)와 배포 유형(Self-hosted, NVIDIA-hosted)을 자동 탐색합니다.
2. **설정 파일 식별**: 배포 유형에 따라 설정 파일 위치를 결정합니다.
    - Docker Self-hosted: `deploy/compose/.env`
    - Docker NVIDIA-hosted: `deploy/compose/nvdev.env`
    - K8s/Helm: `values.yaml`
    - Library mode: `notebooks/config.yaml`
3. **상태 확인**: 설정 파일의 값과 실제 실행 중인 서비스의 환경 변수를 비교하여 정합성을 확인합니다.
4. **리소스 검증**: 추가 GPU가 필요한 기능의 경우 `nvidia-smi`로 가용 자원을 확인합니다.
5. **변경 적용 및 재시작**: 설정 파일을 수정한 후 해당 서비스를 재시작합니다.
6. **최종 검증**: Health Check API 및 `docker ps`/`kubectl get pods`로 정상 작동을 확인합니다.

## 🗺️ 기능별 설정 매핑 (Feature Mapping)

| 기능 키워드 | 참조 문서 | 주요 설정 내용 |
| :--- | :--- | :--- |
| **VLM** | `references/configure/vlm.md` | VLM 임베딩, 이미지 캡셔닝 |
| **Guardrails** | `references/configure/guardrails.md` | NeMo Guardrails 설정 |
| **Agentic RAG** | `references/configure/agentic-rag.md` | 플래닝/실행 에이전트, 스트리밍 |
| **Query/Conversation** | `references/configure/query-and-conversation.md` | 쿼리 재작성, 분해, 멀티턴 |
| **Ingestion** | `references/configure/ingestion.md` | 텍스트/오디오 추출, Nemotron Parse, OCR |
| **Search/Retrieval** | `references/configure/search-and-retrieval.md` | 하이브리드 검색, 필터링, Reranker |
| **Models & Infra** | `references/configure/models-and-infrastructure.md` | 모델 교체, Vector DB, 포트/GPU 설정 |
| **Reasoning/Generation** | `references/configure/reasoning-and-generation.md` | Thinking mode, 프롬프트, 생성 파라미터 |
| **Summarization** | `references/configure/summarization.md` | 요약 기능 설정 |
| **Observability** | `references/configure/observability.md` | Tracing, Grafana, Prometheus |
| **Multimodal** | `references/configure/multimodal-query.md` | 이미지+텍스트 쿼리 |
| **Data Catalog** | `references/configure/data-catalog.md` | 컬렉션/문서 메타데이터 |
| **User Interface** | `references/configure/user-interface.md` | UI 설정, 추론 패널 |
| **API Reference** | `references/configure/api-reference.md` | 엔드포인트 및 스키마 |
| **Evaluation** | `references/configure/evaluation.md` | RAGAS 메트릭, `rag-eval` 스킬 연계 |
| **MCP** | `references/configure/mcp.md` | MCP 서버 및 클라이언트 설정 |
| **Migration** | `references/configure/migration.md` | 버전 업그레이드 |
| **Notebooks** | `references/configure/notebooks.md` | 노트북 설정 및 카탈로그 |

## ⚠️ 제약 및 주의사항
- **환경 의존성**: 본 스킬은 NVIDIA RAG Blueprint 저장소 기반의 운영 가이드만 제공합니다.
- **비밀 정보**: `NGC_API_KEY`와 같은 시크릿 값은 사용자 환경에서 제공되어야 합니다.
- **하드웨어 제한**: GPU 모델(B200, RTX 6000 등)에 따라 지원되지 않는 기능이 있으므로 `docs/support-matrix.md`를 반드시 참조하십시오.
