# 🎴 Skill Card: cuOpt Routing — Python API (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cuopt-routing-api-python`
- **도메인**: NVIDIA cuOpt / Vehicle Routing Problem (VRP)
- **설명**: cuOpt의 Python API를 사용하여 VRP, TSP, PDP와 같은 경로 최적화 문제를 정의하고 해결하는 스킬입니다.
- **핵심 목표**: `DataModel`을 통해 위치, 차량(Fleet), 주문(Orders), 제약 조건(Constraints)을 정확히 정의하고, `Solve` 함수를 호출하여 최적의 경로 솔루션을 도출하는 것입니다.

## ⚙️ 구현 및 실행 가이드

본 스킬은 **Python 전용**입니다. cuOpt의 Routing 기능은 C API를 제공하지 않으므로 반드시 Python 환경에서 실행해야 합니다.

### 1. [Step 1] 문제 정의 및 데이터 모델링 (Data Modeling)
해결하려는 문제의 특성에 따라 `routing.DataModel`을 설정합니다.
- **기본 구성**:
    - `n_locations`: 전체 지점 수 (디포 포함).
    - `n_fleet`: 사용 가능한 차량 수.
    - `n_orders`: 방문해야 할 주문/작업 수.
- **필수 데이터 입력**:
    - **Cost Matrix**: 지점 간의 비용(거리/시간) 행렬. 반드시 `float32` 타입의 `cudf.DataFrame`으로 입력하십시오.
      ```python
      dm.add_cost_matrix(cost_matrix.astype("float32"))
      ```
    - **Order Locations**: 각 주문이 위치한 지점 인덱스. `int32` 타입의 `cudf.Series`를 사용합니다.
      ```python
      dm.set_order_locations(cudf.Series([1, 2, 3], dtype="int32"))
      ```

### 2. [Step 2] 제약 조건 추가 (Adding Constraints)
문제의 복잡도에 따라 다음 제약 조건을 선택적으로 추가합니다.

#### A. 시간 윈도우 (Time Windows) 및 서비스 시간
- **이동 시간 설정**: `dm.add_transit_time_matrix(transit_time_matrix)` (필수: 시간 윈도우 사용 시).
- **주문 시간 제한**: `dm.set_order_time_windows(earliest_series, latest_series)`.
- **서비스 시간**: `dm.set_order_service_times(service_times)` (정지 시 소요 시간).
- **차량 시간 제한**: `dm.set_vehicle_time_windows(earliest_start, latest_return)`.

#### B. 용량 제한 (Capacities)
- **차원 정의**: `dm.add_capacity_dimension(name, demand_series, capacity_series)`.
    - `name`: "weight", "volume" 등 차원 이름.
    - `demand_series`: 주문별 수요량.
    - `capacity_series`: 차량별 최대 수용량.

#### C. 특수 경로 제약
- **Pickup-Delivery Pairs (PDP)**: `dm.set_pickup_delivery_pairs(pickup_indices, delivery_indices)`.
- **우선순위/선행 조건 (Precedence)**: `dm.add_order_precedence(node_id, preceding_nodes)`.
- **차량 시작/종료 지점**: `dm.set_vehicle_locations(start_locations, end_locations)`.

### 3. [Step 3] 솔루션 실행 및 설정 (Solving)
- **솔버 설정 (`SolverSettings`)**:
    - `set_time_limit(seconds)`: 최대 탐색 시간 설정.
    - `set_verbose_mode(True)`: 실행 과정 로그 출력.
    - `set_error_logging_mode(True)`: 에러 상세 로그 활성화.
- **실행**: `solution = routing.Solve(dm, settings)`.

### 4. [Step 4] 결과 분석 및 검증 (Solution Analysis)
- **상태 확인**: `solution.get_status()`
    - `0`: SUCCESS (최적 또는 준최적해 발견).
    - `1`: FAIL / `2`: TIMEOUT / `3`: EMPTY.
- **데이터 추출**:
    - `solution.get_route()`: 상세 경로 데이터프레임 반환.
    - `solution.get_total_objective()`: 전체 목적 함수 값(총 비용) 반환.
    - `solution.get_infeasible_orders()`: 제약 조건으로 인해 방문하지 못한 주문 목록 확인.

---

## 🛠️ 데이터 타입 가이드 (Crucial)

cuOpt는 GPU 가속을 위해 엄격한 데이터 타입을 요구합니다. 타입 불일치 시 silent error가 발생하거나 런타임에 크래시가 발생할 수 있습니다.

| 데이터 항목 | 권장 타입 | Python/cuDF 예시 |
|---|---|---|
| Cost/Transit Matrix | `float32` | `.astype("float32")` |
| Location Indices | `int32` | `dtype="int32"` |
| Demand/Capacity | `int32` | `dtype="int32"` |
| Time Windows | `float32` | `.astype("float32")` |

---

## 🚨 핵심 준수 규칙 및 제한 사항

### ⚠️ 절대 준수 규칙
1. **명시적 타입 캐스팅**: 모든 입력 행렬과 시리즈에 대해 `.astype("float32")` 또는 `dtype="int32"`를 명시적으로 적용하십시오.
2. **시간 행렬 필수**: 시간 윈도우(`Time Windows`) 제약을 사용할 때는 반드시 `add_transit_time_matrix()`를 먼저 호출해야 합니다.
3. **인덱스 정합성**: `DataModel`에 입력하는 모든 인덱스는 $0$부터 `n_locations - 1` 범위 내에 있어야 합니다.

### 🚫 주요 제한 사항 (Limitations)
- **Python 전용**: cuOpt Routing 기능은 C API를 지원하지 않습니다.
- **메모리 사용량**: 매우 큰 Cost Matrix(예: 지점 수 $> 10,000$)의 경우 GPU 메모리(VRAM) 부족이 발생할 수 있습니다.

## ❓ 트러블슈팅

| 문제 현상 | 원인 | 해결 방법 |
|---|---|---|
| **Empty solution / Status 3** | 제약 조건이 너무 엄격함 | 시간 윈도우 범위를 넓히거나 차량 대수(`n_fleet`)를 늘리십시오. |
| **Infeasible orders 발생** | 용량 초과 또는 시간 부족 | `get_infeasible_orders()`로 대상 주문을 확인하고 차량 용량을 늘리거나 경로를 조정하십시오. |
| **상태는 SUCCESS나 결과가 이상함** | Cost Matrix 비대칭/오류 | 비용 행렬이 대칭인지, 거리/시간 값이 현실적인지 확인하십시오. |
| **`compute_waypoint_sequence` 호출 후 데이터 변경** | In-place 수정 특성 | 이 함수는 `route_df`의 `location` 컬럼을 직접 수정합니다. 원본 인덱스가 필요하면 `route_df.copy()`를 전달하십시오. |
| **런타임 에러 (Cuda Error)** | 데이터 타입 불일치 | 입력 데이터가 `float32` 또는 `int32`인지 다시 확인하십시오. |

## 🚀 관련 스킬
- **cuOpt 설치**: `/cuopt-install`
- **수치 최적화 (LP/MILP)**: `cuopt-numerical-optimization-api`
- **내부 로직 수정 및 기여**: `/cuopt-developer`
