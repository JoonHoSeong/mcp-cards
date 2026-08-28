# 🎴 Skill Card: cupynumeric-migration-readiness (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cupynumeric-migration-readiness`
- **도메인**: cuPyNumeric / Pre-Migration Assessment
- **설명**: 기존 NumPy 기반 코드를 cuPyNumeric으로 포팅하기 **전**에, 해당 코드가 GPU/분산 환경에서 실제로 성능 확장(Scaling)이 가능할지, 혹은 심각한 병목이나 호환성 문제가 있을지를 정적으로 분석하는 진단 가이드입니다.
- **핵심 목표**: 무작정 포팅을 시작하기 전에 소스 코드를 분석하여 `READY`, `LIGHT REFACTOR`, `SIGNIFICANT REFACTOR`, `NOT RECOMMENDED` 중 하나의 판정을 내리고, 구체적인 리팩토링 포인트와 레시피를 제공함으로써 엔지니어링 리소스 낭비를 방지하는 것입니다.

## 🎯 적용 시점 (When to Use)
- NumPy 코드를 GPU로 옮기려는데, 이것이 정말 성능 이득이 있을지 판단이 필요할 때.
- 포팅 작업에 들어가기 전, 어떤 부분의 코드를 수정해야 cuPyNumeric의 분산 실행 모델(Legate)에 최적화될지 계획을 세워야 할 때.
- GPU 가속을 방해하는 NumPy 안티 패턴(Anti-patterns)을 식별하고 제거하고 싶을 때.

---

## ⚙️ 진단 프로세스 (Assessment Workflow)

### 단계 1: 컨텍스트 수집 및 기본 가정 설정
코드 스캔 전, 다음 기본값(Defaults)을 적용하고 사용자의 특이사항을 확인합니다.
- **데이터 규모**: 기본 3,000만 ~ 5,000만 개 요소. (1,000만 개 이상 시 단일 GPU 가속, 1억 개 이상 시 멀티 GPU 가속 효과 기대).
- **타겟 하드웨어**: 기본 1~4개 GPU, 단일 노드.
- **주요 연산 패턴**: 스텐실(Stencil), GEMM, 몬테카를로, 리덕션 등.

### 단계 2: API 지원 매니페스트 확인
`assets/api-support.md`를 참조하여 코드에서 사용 중인 NumPy API의 지원 수준을 확인합니다.
- `✓✓`: 멀티 GPU 지원 (최적).
- `✓`: 구현되었으나 단일 GPU/CPU 전용.
- `🟡`: 부분 지원 (노트 확인 필요).
- `✗`: 미구현. 핫패스(Hot-path)에서 사용 시 마이그레이션 차단 요소(Blocker)가 됩니다.

### 단계 3: 시맨틱 코드 분석 (Semantic Analysis)
단순한 키워드 검색이 아니라, 데이터의 흐름과 루프 구조를 분석하여 다음 패턴을 식별합니다.

#### 🟢 확장 가능 패턴 (SCALES - 유지)
- 벡터화된 요소별 연산 (Vectorized elementwise ops).
- 리덕션 (Reductions), Matmul / Einsum.
- `np.where`, 대규모 스텐실 슬라이싱 (`arr[1:-1, 1:-1]`).
- `out=` 파라미터를 통한 메모리 재사용.

#### 🔴 확장 차단 패턴 (BLOCKS - 제거 필수)
- **엘리먼트 루프**: `for i in range(n): arr[i] = ...` (가장 치명적).
- **스칼라 동기화**: 핫루프 내부에서 `.item()`, `float()`, `int()` 등을 호출하여 GPU$\rightarrow$CPU 동기화를 유발하는 경우.
- **리덕션 조건문**: `while np.max(err) > tol:`와 같이 루프마다 전체 배열을 리덕션하여 조건을 판단하는 경우.
- **Python 내장 함수 사용**: `sum(arr)`, `max(arr)` 등 (NumPy 함수 `np.sum` 등을 사용해야 함).
- **`mpi4py` 혼용**: cuPyNumeric/Legate의 분산 모델과 충돌하는 수동 MPI 통신 코드.

#### 🟡 리팩토링 필요 패턴 (REFACTOR - 수정 권장)
- **루프 내 할당**: 루프 내부에서 매번 새로운 배열을 생성하는 경우 $\rightarrow$ 루프 밖으로 할당 이동.
- **반복적 결합**: 루프 내에서 `vstack`, `hstack`, `concatenate`를 반복 호출하는 경우.
- **`np.nonzero()` 기반 인덱싱**: 결과 크기가 가변적인 인덱싱 $\rightarrow$ 마스킹 연산으로 대체.

---

## ⚖️ 판정 프레임워크 (Verdict Framework)

분석 결과를 바탕으로 다음 순서에 따라 최종 판정을 내립니다. (먼저 매칭되는 것이 우선함)

| 판정 | 조건 | 권장 조치 |
|---|---|---|
| **NOT RECOMMENDED** | **Gate 4 실패**: 데이터 구조가 Sparse/Graph/ML/Sequential인 경우 <br> **Gate 2 실패**: 배열 크기가 GPU당 65,536개 미만인 경우 | 구조적 재설계 후 재검토 또는 타 런타임 고려 |
| **SIGNIFICANT REFACTOR** | **mpi4py 사용** 또는 핫패스에 **다수의 BLOCKS** 패턴이 존재하는 경우 | 모듈당 1~3주 정도의 리팩토링 기간 산정 필요 |
| **LIGHT REFACTOR** | 소수의 단순한 BLOCKS 패턴 또는 기계적인 REFACTOR 패턴만 존재하는 경우 | 제공된 레시피를 적용하여 `READY` 상태로 전환 |
| **READY** | BLOCKS가 없고 REFACTOR 패턴이 거의 없는 경우 | 즉시 import 교체 및 벤치마크 수행 |

---

## 📝 최종 보고서 구조 (Report Structure)

진단 결과는 반드시 다음 8개 섹션을 모두 포함하여 보고해야 합니다.
1. **Verdict**: 한 문장으로 요약된 최종 판정.
2. **What works (SCALES)**: 성능 확장이 기대되는 코드 라인 및 근거.
3. **What blocks (BLOCKS)**: 확장 차단 요소, 해당 라인, 관련 ID (예: [R104]), 리팩토링 레시피.
4. **What is fixable (REFACTOR)**: 수정 가능한 비효율 패턴과 대응 레시피.
5. **Compatibility / Cost notes (INFO)**: SciPy 경계, 단일 장치 전용 API 등 주의사항.
6. **API support gaps**: 매니페스트 기준 미구현 API 목록.
7. **Decision-framework summary**: Gate 1~6 통과 여부 (Pass/Fail/Uncertain).
8. **Recommended next steps**: 우선 적용할 레시피 및 cuPyNumeric Doctor 실행 시점 안내.

## 🛠️ 사후 검증 (Runtime Validation)
정적 분석 후 리팩토링을 완료했다면, 반드시 **cuPyNumeric Doctor**를 실행하여 런타임 상의 숨은 병목(Scalar item access, Advanced indexing 등)을 최종 확인하도록 안내하십시오.
```bash
CUPYNUMERIC_DOCTOR=1 CUPYNUMERIC_DOCTOR_FORMAT=json CUPYNUMERIC_DOCTOR_FILENAME=doctor-report.json legate --gpus 1 main.py
```

---

## 📚 관련 인터페이스 및 스킬
- **`cupynumeric-install`**: 진단 후 실제 포팅을 위한 환경 구축 가이드입니다.
- **`cupynumeric-hdf5`**: 대규모 데이터셋의 분산 I/O 최적화 가이드입니다.
