# 🎴 Skill Card: accelerated-computing-cudf (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `accelerated-computing-cudf`
- **도메인**: NVIDIA GPU Accelerated DataFrames (RAPIDS cuDF)
- **설명**: NVIDIA cuDF 및 dask-cuDF를 사용하여 pandas 워크로드를 GPU로 가속화하고, 대규모 ETL, 조인, 그룹화 및 I/O 작업을 최적화하는 공식 가이드입니다.
- **핵심 목표**: 사용자의 pandas 코드를 최소한의 마찰로 정확하고 빠른 GPU 코드로 전환하며, 데이터 크기와 가용 메모리에 맞는 최적의 경로(cudf.pandas $\rightarrow$ Explicit cuDF $\rightarrow$ dask-cuDF)를 선택하여 구현하는 것입니다.

## ⚙️ GPU 데이터프레임 구현의 세 가지 경로 (Three Paths)

사용자의 의도와 데이터 규모에 따라 다음 세 가지 경로 중 하나를 선택하여 구현합니다.

### 경로 1: `cudf.pandas` 가속기 (호환성 및 최소 변경)
가장 낮은 진입 장벽으로, 기존 pandas 코드를 거의 수정하지 않고 가속화할 때 사용합니다.
- **Jupyter/IPython**:
  ```python
  %load_ext cudf.pandas
  import pandas as pd # 이제 GPU 백엔드로 작동하며, 지원되지 않는 연산은 자동으로 CPU로 폴백됩니다.
  ```
- **스크립트 실행**:
  ```bash
  python -m cudf.pandas my_script.py
  ```
- **멀티프로세싱 사용 시**:
  ```python
  import cudf.pandas
  cudf.pandas.install() # pandas 임포트 및 Pool 생성 전에 반드시 호출해야 함
  from multiprocessing import Pool
  ```

### 경로 2: 명시적 cuDF API (최적화 및 제어)
전체 제어, 핫패스 최적화, 명확한 DataFrame 마이그레이션이 필요할 때 사용합니다.
```python
import cudf

# 1. 데이터를 GPU로 직접 로드 (가장 빠름)
df = cudf.read_parquet("data.parquet")

# 2. pandas와 유사한 API로 연산 수행
result = df.groupby("key")["value"].sum()
merged = df.merge(lookup, on="id", how="left")
filtered = df[df["amount"] > 1000]

# 3. 문자열 연산 (GPU 가속)
df["clean"] = df["name"].str.strip().str.lower()

# 4. 경계 지점에서만 pandas로 변환 (디스플레이, 플로팅, CPU 전용 라이브러리용)
final_pd_df = result.to_pandas()
```

### 경로 3: dask-cuDF (멀티 GPU 및 대규모 데이터)
데이터셋이 단일 GPU 메모리를 초과하는 경우 사용합니다.
```python
from dask_cuda import LocalCUDACluster
from dask.distributed import Client
import dask_cudf

# GPU당 워커 하나 생성, GPU 메모리 초과 시 호스트 RAM으로 스필(Spill) 허용
cluster = LocalCUDACluster(enable_cudf_spill=True)
client = Client(cluster)

# S3 등 분산 저장소에서 데이터 로드
ddf = dask_cudf.read_parquet("s3://bucket/data/*.parquet")
result = ddf.groupby("key").agg({"value": "sum"}).compute()
```

---

## 🛠️ 사전 체크리스트 및 환경 설정 (Pre-flight Checks)

### 1. 하드웨어 및 소프트웨어 요구사항
- **GPU**: NVIDIA Volta 아키텍처 이상 (CUDA 12), 또는 Turing 이상 (CUDA 13).
- **드라이버**: CUDA 12.2-12.9 (드라이버 535+), CUDA 13.0-13.1 (드라이버 580+).
- **Python**: 3.11 ~ 3.14.
- **데이터 규모**: 최소 **100K 행(rows) 이상**일 때 GPU 가속 효과가 뚜렷합니다. 그 미만은 전송 오버헤드가 더 클 수 있습니다.

### 2. 메모리 관리 설정
- **Spill 활성화**: OOM(Out of Memory) 방지를 위해 GPU 메모리 부족 시 호스트 RAM을 사용하도록 설정합니다.
  ```python
  import cudf
  cudf.set_option("spill", True)
  ```
- **RMM Pool 할당자**: 빈번한 메모리 할당/해제 오버헤드를 줄이기 위해 사용합니다.
  ```python
  import rmm
  rmm.set_current_device_resource(rmm.mr.CudaAsyncMemoryResource())
  # 주의: 모든 cuDF 연산이 시작되기 전에 호출되어야 함
  ```

---

## 🚨 핵심 준수 규칙 (Critical Rules)

1. **경계 지점에서 변환**: `.to_pandas()`, `.values`, `.numpy()` 호출은 최종 출력 단계에서만 수행하십시오. 중간 ETL 단계에서는 데이터를 GPU에 유지해야 합니다.
2. **Float32 우선 사용**: `float64` 연산은 GPU에서 더 느립니다. 정밀도가 허용하는 한 조기에 `float32`로 캐스팅하십시오.
3. **시맨틱 검증**: Null 처리, 조인, 시계열 연산 시 소규모 pandas 레퍼런스 경로를 만들어 shape, labels, null count를 비교 검증하십시오.
4. **메모리 전략 선택**:
   - GPU 여유 공간 > 데이터셋 2배 $\rightarrow$ **Single GPU cuDF**
   - GPU 여유 공간 1~2배 $\rightarrow$ **cuDF + Spill 활성화**
   - 데이터셋 > GPU 메모리 $\rightarrow$ **dask-cuDF**

---

## 🔍 트러블슈팅 및 세부 주의사항 (Troubleshooting & Nuances)

### 1. 성능 및 결과 이슈
| 증상 | 원인 | 해결책 |
|---|---|---|
| **pandas 대비 속도 저하** | 데이터 규모가 너무 작음 (<100K rows) | 더 큰 데이터셋으로 벤치마크 수행 |
| **잦은 CPU 폴백** | `cudf.pandas` 사용 중 미지원 연산 발생 | `%%cudf.pandas.profile`로 확인 후 명시적 cuDF API로 리팩토링 |
| **CUDA Out of Memory** | GPU 메모리 부족 | `cudf.set_option("spill", True)` 설정 또는 dask-cuDF 전환 |
| **pandas와 결과값 상이** | Null/NaN 처리 방식 차이 | cuDF는 nullable dtype을 사용함. `to_pandas(nullable=True)`로 비교 |
| **정렬 결과 상이** | cuDF 정렬은 기본적으로 unstable | pandas와 동일한 순서가 필요하면 `stable=True` 옵션 사용 |

### 2. API 제약 및 워크아라운드
- **RE2 정규표현식**: cuDF는 RE2를 사용하므로 Lookahead/Lookbehind(`(?=...)`)나 Backreferences(`\\1`)를 지원하지 않습니다.
- **미지원 연산**: `read_html`, `read_excel`, `to_sql` 등은 지원되지 않습니다. pandas로 처리 후 `cudf.from_pandas()`로 가져오십시오.
- **CuPy 인터옵**: `.values` 호출 시 NumPy가 아닌 **CuPy array**가 반환됩니다. NumPy가 필요하면 `.to_numpy()`를 사용하십시오.

## 🔗 참조 (References)
- **cuDF 공식 문서**: https://docs.rapids.ai/api/cudf/stable/
- **dask-cuDF API**: https://docs.rapids.ai/api/dask-cudf/stable/api/
- **GitHub**: https://github.com/rapidsai/cudf
- **내부 가이드**: `references/cudf-pandas-accelerator.md`, `references/api-patterns.md`, `references/dask-cudf-patterns.md`
