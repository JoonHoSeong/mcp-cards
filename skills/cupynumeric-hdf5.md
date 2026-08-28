# 🎴 Skill Card: cupynumeric-hdf5 (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cupynumeric-hdf5`
- **도메인**: cuPyNumeric / Distributed I/O
- **설명**: 대규모 cuPyNumeric 배열을 HDF5(.h5, .hdf5) 파일로 저장하거나 읽어들이는 분산 I/O 가이드입니다. Legate의 병렬 HDF5 I/O 엔진을 사용하여 모든 랭크가 자신의 타일을 동시에 읽고 쓰므로, 단일 프로세스를 통한 병목 현상을 원천적으로 제거합니다.
- **핵심 목표**: `legate.io.hdf5` API를 사용하여 분산 배열을 단일 HDF5 파일로 효율적으로 저장하고, 대용량 파일을 청크 단위로 읽어들여 메모리 효율성을 극대화하며, GPUDirect Storage(GDS)를 통해 디스크-GPU 간 데이터 전송 속도를 최적화하는 것입니다.

## 🎯 적용 시점 (When to Use)
- cuPyNumeric 분산 배열을 HPC 파이프라인이나 다른 분석 도구에서 사용할 수 있도록 단일 `.h5` 파일로 저장해야 할 때.
- 너무 커서 메모리에 한 번에 올릴 수 없는 대용량 HDF5 데이터셋을 청크(Chunk) 단위로 읽어 처리해야 할 때.
- GPU 메모리로 직접 데이터를 로드하여 호스트 메모리 복사 오버헤드를 줄이고 속도를 높여야 할 때 (GDS 활용).

---

## ⚙️ API 구현 및 호출 워크플로우

### 1. 사전 준비
`legate.io.hdf5` 모듈은 내부적으로 `h5py`를 사용하므로, 반드시 먼저 설치되어 있어야 합니다.
```bash
conda install -c conda-forge h5py
```

### 2. 기본 I/O 작업 (Round Trip)
`to_file`과 `from_file`을 사용하여 데이터를 저장하고 복구합니다.

- **데이터 저장 (`to_file`)**:
  - cuPyNumeric 배열을 직접 전달합니다. 별도의 변환이 필요 없습니다.
  - **중요**: Legate I/O는 비동기적으로 작동합니다. 외부 도구(h5py 등)로 파일을 열기 전에 반드시 실행 펜스(Execution Fence)를 설정해야 합니다.
  ```python
  import cupynumeric as cn
  from legate.core import get_legate_runtime
  from legate.io.hdf5 import to_file

  a = cn.arange(1000, dtype=cn.float32).reshape(10, 100)
  to_file(array=a, path="data.h5", dataset_name="/dataset1")
  get_legate_runtime().issue_execution_fence(block=True) # 필수: 쓰기 완료 대기
  ```

- **데이터 로드 (`from_file`)**:
  - `from_file`은 Legate `LogicalArray`를 반환하므로, cuPyNumeric 배열로 사용하려면 `cn.asarray()`로 브릿징해야 합니다.
  ```python
  from legate.io.hdf5 import from_file
  b = cn.asarray(from_file("data.h5", dataset_name="/dataset1"))
  ```

### 3. 대용량 파일 청크 읽기 (`from_file_batched`)
전체 파일을 메모리에 올리지 않고 `chunk_size` 단위로 읽어 처리합니다.
```python
from legate.io.hdf5 import from_file_batched
import h5py

with h5py.File("big_data.h5", "r") as f:
    shape, dtype = f["data"].shape, f["data"].dtype

out = cn.empty(shape, dtype=dtype)
for chunk, (r0, c0) in from_file_batched("big_data.h5", "data", chunk_size=(4096, 4096)):
    # 각 청크의 실제 shape과 offset(r0, c0)을 사용하여 배치
    out[r0:r0 + chunk.shape[0], c0:c0 + chunk.shape[1]] = cn.asarray(chunk)
```

---

## 🚀 성능 최적화: GPUDirect Storage (GDS)

GPU 메모리로 직접 데이터를 읽어들일 때, 기본 POSIX 경로를 사용하면 128MB의 작은 ZCMEM 스테이징 버퍼 제한 때문에 대용량 배열 로드 시 크래시가 발생할 수 있습니다. 이를 해결하기 위해 GDS VFD(Virtual File Driver)를 활성화해야 합니다.

### 설정 방법
런칭 전에 환경 변수를 설정하거나 `legate` 드라이버 옵션을 사용하십시오.
```bash
# 방법 1: 환경 변수 설정
export LEGATE_IO_USE_VFD_GDS=1

# 방법 2: legate 런처 옵션 사용
legate --io-use-vfd-gds my_script.py
```

### GDS 활용 가이드
- **효과**: ZCMEM 버퍼 제한을 제거하고 디스크에서 GPU 메모리로 직접 데이터를 전송하여 속도를 획기적으로 높입니다.
- **호환성**: 실제 GPUDirect Storage 지원 스토리지가 없더라도 `cuFile`의 호환 모드(Compatibility Mode)를 통해 작동하며, 여전히 ZCMEM 크래시를 방지하므로 **GPU 읽기 작업 시 항상 `=1`로 설정하는 것을 권장**합니다.
- **확인**: 실행 로그에서 `H5FD__gds_open: Successfully opened file w/GDS VFD` 문구를 확인하십시오.

---

## ⚠️ 주의사항 및 트러블슈팅 (Pitfalls & Troubleshooting)

### 1. 주요 제약 사항
- **가상 데이터셋 (VDS)**: `to_file`은 내부적으로 VDS를 생성합니다. 각 랭크가 자신의 타일을 쓰고, 파일은 이를 하나의 논리적 데이터셋으로 보여줍니다.
- **덮어쓰기 주의**: `to_file`은 대상 경로의 파일을 파괴적으로 덮어씁니다.
- **경로 설정**: `path`는 반드시 파일명까지 포함된 전체 경로여야 합니다. 디렉토리 경로를 전달하면 `ValueError`가 발생합니다.
- **경계 조건**: `from_file_batched` 사용 시, 전체 크기가 `chunk_size`로 나누어 떨어지지 않는 마지막 청크는 크기가 더 작을 수 있습니다. 반드시 `chunk.shape`를 사용하여 배치하십시오.

### 2. 흔한 에러 해결법
- **`ModuleNotFoundError: No module named h5py`**: `conda install -c conda-forge h5py`를 실행하십시오.
- **파일이 비어 있거나 잘림**: `to_file` 호출 후 `issue_execution_fence(block=True)`를 호출했는지 확인하십시오.
- **GPU 로드 중 Abort/Crash**: 배열 크기가 128MB를 초과하는지 확인하고 `LEGATE_IO_USE_VFD_GDS=1`을 설정하십시오.

---

## 📚 관련 인터페이스 및 스킬
- **`cupynumeric-parallel-data-load`**: HDF5 외의 다른 포맷(Parquet, Raw-Binary, Custom Sharding)을 이용한 병렬 데이터 로딩 가이드입니다.
- **`cupynumeric-install`**: `h5py` 설치 및 cuPyNumeric 환경 구축 가이드입니다.
- **`cupynumeric-migration-readiness`**: NumPy의 `np.save/load`에서 cuPyNumeric의 분산 HDF5 I/O로 전환할 때의 고려사항을 다룹니다.
