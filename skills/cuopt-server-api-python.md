# 🎴 Skill Card: cuOpt Server API — Python (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cuopt-server-api-python`
- **도메인**: NVIDIA cuOpt / REST API / Deployment
- **설명**: cuOpt REST 서버를 배포하고, Python 클라이언트를 통해 경로 최적화(Routing) 및 수치 최적화(LP/MILP) 문제를 요청하고 처리하는 스킬입니다.
- **핵심 목표**: 서버의 헬스체크부터 요청 제출(POST), 상태 폴링(GET), 최종 솔루션 파싱까지의 전체 비동기 라이프사이클을 관리하는 것입니다.

## ⚙️ 구현 및 실행 가이드

### 1. [Step 1] 서버 배포 및 확인 (Deployment & Health Check)
cuOpt 서버는 기본적으로 8000번 포트를 사용하며, Docker를 통한 배포가 가장 권장됩니다.

- **Docker 실행**:
    ```bash
    docker run --gpus all -d -p 8000:8000 -e CUOPT_SERVER_PORT=8000 nvidia/cuopt:latest-cuda12.9-py3.13
    ```
- **상태 확인 (Health Check)**:
    ```bash
    curl http://localhost:8000/cuopt/health
    ```
    - 응답이 `{"status": "ok"}` 또는 유사한 정상 메시지여야 합니다.

### 2. [Step 2] API 요청 구조 (Request Lifecycle)
REST API는 비동기 방식으로 동작합니다. 요청을 보낸 즉시 결과가 나오지 않으며, `reqId`를 통해 결과를 추적해야 합니다.

#### A. 요청 제출 (POST `/cuopt/request`)
문제 정의(Payload)를 담아 POST 요청을 보냅니다.
- **Routing Payload 필수 항목**:
    - `cost_matrix_data`: 지점 간 비용 행렬.
    - `travel_time_matrix_data`: 지점 간 이동 시간 행렬.
    - `task_data`: 주문 지점, 수요, 시간 윈도우, 서비스 시간.
    - `fleet_data`: 차량 시작/종료 지점, 용량, 차량 시간 윈도우.
- **LP/MILP Payload**: CSR(Compressed Sparse Row) 형식의 행렬 데이터가 필요합니다.

#### B. 결과 폴링 (GET `/cuopt/solution/{reqId}`)
`reqId`를 사용하여 솔루션 상태를 확인합니다.
- **Polling Loop**: `status`가 `COMPLETED` 또는 `FAILED`가 될 때까지 반복 호출합니다.
- **응답 데이터**: 솔루션 상태가 `COMPLETED`가 되면 최적 경로, 총 목적 함수 값, 미방문 주문 목록 등이 포함된 JSON 응답을 받습니다.

### 3. [Step 3] Python 클라이언트 구현 예시
```python
import requests, time

SERVER = "http://localhost:8000"
HEADERS = {"Content-Type": "application/json", "CLIENT-VERSION": "custom"}

def solve_cuopt_rest(payload):
    # 1. 요청 제출
    r = requests.post(f"{SERVER}/cuopt/request", json=payload, headers=HEADERS)
    req_id = r.json()["reqId"]
    
    # 2. 폴링
    while True:
        res = requests.get(f"{SERVER}/cuopt/solution/{req_id}")
        data = res.json()
        if data.get("status") == "COMPLETED":
            return data
        elif data.get("status") == "FAILED":
            raise Exception(f"cuOpt Solve Failed: {data.get('error')}")
        time.sleep(1)

# Payload 예시 (Routing)
payload = {
    "cost_matrix_data": {"data": {"0": [[0,10,15],[10,0,12],[15,12,0]]}},
    "travel_time_matrix_data": {"data": {"0": [[0,10,15],[10,0,12],[15,12,0]]}},
    "task_data": {"task_locations": [1, 2], "demand": [[10, 20]], "task_time_windows": [[0,100],[0,100]], "service_times": [5, 5]},
    "fleet_data": {"vehicle_locations": [[0, 0]], "capacities": [[50]], "vehicle_time_windows": [[0, 200]]},
    "solver_config": {"time_limit": 5}
}
print(solve_cuopt_rest(payload))
```

---

## 🛠️ REST API 명칭 및 데이터 매핑 (Crucial)

Python API와 REST API는 사용하는 용어가 서로 다릅니다. 매핑 테이블을 참조하여 Payload를 작성하십시오.

| Python API 용어 | REST API 용어 | 비고 |
|---|---|---|
| `order_locations` | `task_locations` | REST에서는 'task'라는 용어 사용 |
| `set_order_time_windows()` | `task_time_windows` | |
| `service_times` | `service_times` | |
| `transit_time_matrix_data` | `travel_time_matrix_data` | **절대 주의**: REST에서는 `travel_` 명칭 사용 |
| `capacities` (per vehicle) | `capacities` (per dimension) | REST에서는 `[[50, 50]]` 형태로 차원별 용량 정의 |

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **명칭 불일치 주의**: REST API 요청 시 `transit_time_matrix_data`라는 필드명을 사용하면 **422 Unprocessable Entity** 에러가 발생합니다. 반드시 `travel_time_matrix_data`를 사용하십시오.
2. **용량 정의 방식**: 차량별 용량이 아닌, 차원별 용량을 리스트의 리스트 형태로 제출하십시오 (예: `[[50, 50]]`).
3. **비동기 처리**: 요청 후 즉시 결과를 기대하지 마십시오. 반드시 `reqId`를 통한 폴링 로직을 구현하십시오.

### 🚫 주요 제한 사항 (Limitations)
- **Symmetry**: 기본적으로 비용 행렬은 대칭임을 가정하나, 비대칭 행렬도 지원합니다.
- **QP 미지원**: REST API를 통해서는 QP(이차 최적화) 문제를 해결할 수 없습니다. 오직 Routing, LP, MILP만 가능합니다.
- **포트 충돌**: 기본 8000번 포트가 사용 중인 경우, 서버 실행 시 `--port` 옵션으로 변경하십시오.

## ❓ 트러블슈팅

| 에러 현상 | 원인 | 해결 방법 |
|---|---|---|
| **HTTP 422** | Payload 필드명 오류 | OpenAPI 스펙(`/cuopt.yaml`)을 확인하여 필드명이 정확한지(`travel_` vs `transit_`) 확인하십시오. |
| **HTTP 404** | 잘못된 `reqId` 또는 서버 재시작 | `reqId`가 정확한지 확인하고, 서버가 재시작되어 세션 데이터가 사라졌는지 확인하십시오. |
| **HTTP 500** | 서버 내부 런타임 에러 (CUDA/VRAM) | 서버 로그를 확인하고, GPU 메모리가 부족하지 않은지 `nvidia-smi`로 점검하십시오. |
| **결과가 `FAILED`** | 문제 정의 오류 (Infeasible) | 제약 조건이 너무 엄격하지 않은지 확인하고, `fleet_data`의 차량 대수를 늘려보십시오. |

## 🚀 관련 스킬
- **cuOpt 설치**: `/cuopt-install`
- **Routing Python API**: `/cuopt-routing-api-python`
- **수치 최적화 정식화**: `/cuopt-numerical-optimization-formulation`
