# 🎴 Skill Card: cupynumeric-parallel-data-load (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `cupynumeric-parallel-data-load`
- **도메인**: cuPyNumeric / Distributed Data Loading
- **설명**: 단일 파일 로더(`cupynumeric.load`, `legate.io.hdf5`)로 처리할 수 없는 샤딩된(Sharded) 다중 파일 데이터셋(sharded .npy, Parquet, Raw Binary, Custom Layout 등)을 분산 cuPyNumeric 배열로 병렬 로드하는 고급 기법입니다.
- **핵심 목표**: Legate의 `@task`를 수동으로 설계하고 런처를 통해 실행함으로써, 데이터 읽기 작업을 모든 GPU/CPU 프로세서에 균등하게 분산시켜 I/O 병목을 제거하고 로딩 속도를 극대화하는 것입니다.

## 🎯 적용 시점 (When to Use)
- **빌트인 로더가 없을 때**: Parquet/Arrow, Raw Binary, 또는 특수한 커스텀 레이아웃의 샤딩된 파일을 로드해야 할 때.
- **성능 최적화**: `np.concatenate([read(f) for f in files])`와 같은 순차적 로딩 방식을 GPU 병렬 로딩으로 대체하여 속도를 높이고 싶을 때.
- **유연한 데이터 구조**: 각 샤드(파일)마다 행(row) 수가 서로 다른 불균일한 데이터셋을 하나의 거대한 분산 배열로 통합해야 할 때.

---

## ⚙️ 병렬 로드 아키텍처 및 워크플로우

### 1. 데이터 레이아웃 가정 및 메타데이터 수집 (Discovery)
데이터가 공유 파일 시스템에 인덱싱 가능한 형태로 존재한다고 가정합니다.
- **프로세스**: 모든 샤드 파일의 헤더만 읽어(`mmap_mode="r"`) 전체 행 수, 데이터 타입(`dtype`), 그리고 trailing axes(axis 0을 제외한 나머지 차원)를 수집합니다.
- **누적 오프셋 테이블**: 각 파일의 행 수를 기반으로 `cum_rows = np.cumsum([0] + per_file_rows)` 테이블을 생성합니다. 이는 나중에 각 태스크가 어떤 파일의 어느 범위를 읽어야 할지 결정하는 지도가 됩니다.

### 2. 출력 저장소 할당 (Allocation)
수집된 전체 크기를 바탕으로 cuPyNumeric 빈 배열을 생성합니다.
```python
import cupynumeric as cn
total_shape = (total_rows,) + trailing_shape
out = cn.empty(total_shape, dtype=dtype)
```

### 3. 프로세서 기반 타일링 (Tiling)
파일 개수가 아니라 **사용 가능한 프로세서 수**에 맞춰 데이터를 분할합니다.
- **타일 크기 계산**: `tile_rows = ceil(total_rows / num_processors)`
- **파티셔닝**: `as_logical_array(out).data.partition_by_tiling((tile_rows,) + trailing_shape)`
- **효과**: 파일 수가 10개라도 GPU가 100개라면, 데이터를 더 잘게 쪼개어 100개의 GPU가 동시에 읽게 하여 효율을 높입니다.

### 4. 리프 태스크 설계 및 수동 실행 (Leaf Task & Manual Launch)
가장 핵심적인 단계로, 각 프로세서에서 실행될 `@task`를 정의합니다.

- **태스크 내부 로직**:
    1. **뷰 생성**: `VariantCode`에 따라 GPU면 `cp.from_dlpack(dst)`, CPU면 `np.asarray(dst)`를 통해 로컬 타일의 뷰를 생성합니다.
    2. **범위 계산**: 자신의 태스크 인덱스($t$)를 이용해 글로벌 행 범위 `[row_start, row_end)`를 계산합니다.
    3. **파일 매핑**: `bisect`를 사용하여 `cum_rows` 테이블에서 해당 범위와 겹치는 파일들을 찾습니다.
    4. **부분 읽기 및 복사**: 겹치는 각 파일의 슬라이스만 읽어 `np.ascontiguousarray`로 변환 후 목적지 뷰에 복사합니다.

- **수동 실행**: `runtime.create_manual_task`를 통해 정의한 파티션 크기에 맞는 런칭 도메인을 설정하고 실행합니다.

---

## 🛠️ 포맷별 리더 구현 (Custom Readers)

태스크 내부의 "파일 읽기" 부분만 변경하여 다양한 포맷을 지원할 수 있습니다.

| 포맷 | 태스크 내부 리더 구현 방식 |
|---|---|
| **`.npy`** | `np.ascontiguousarray(np.load(p, mmap_mode="r")[file_lo:file_hi])` |
| **Raw Binary** | `arr = np.memmap(p, dtype=DTYPE, mode="r", shape=...); host = np.ascontiguousarray(arr[file_lo:file_hi])` |
| **HDF5** | `with h5py.File(p, "r") as f: host = np.ascontiguousarray(f["data"][file_lo:file_hi])` |
| **Parquet** | `tbl = pq.read_table(p).slice(file_lo, file_hi - file_lo); host = tbl.to_pandas().values` |

---

## ⚠️ 핵심 제약 및 주의사항 (Hard Constraints)

### 1. 런타임 호출 금지 (The `cn.asarray` Trap)
- **금지**: `@task` 내부에서 `cn.asarray(store)`나 `cn_dst[slice] = host_np`와 같이 cuPyNumeric의 상위 런타임 API를 호출하면 `LEGION API USAGE EXCEPTION`이 발생하며 크래시됩니다.
- **해결**: 반드시 `cupy`, `torch`, `numpy` 같은 **제3자 라이브러리**를 통해 DLPack 캡슐을 직접 소비하십시오.

### 2. 데이터 일관성
- 모든 샤드는 반드시 **동일한 `dtype`과 trailing axes**를 가져야 합니다. (axis 0인 행 수만 달라도 무방함).

### 3. 메모리 연속성
- `mmap` 뷰는 항상 C-contiguous하지 않을 수 있습니다. `.set()` 또는 numpy 쓰기 전에 반드시 `np.ascontiguousarray()`로 감싸야 합니다.

### 4. 공유 파일 시스템
- 멀티 노드 실행 시, `SHARD_DIR`은 모든 노드에서 접근 가능한 **공유 파일 시스템**(NFS, Lustre 등)에 위치해야 합니다.

---

## 📚 관련 인터페이스 및 스킬
- **`legate.io.hdf5.from_file`**: 단일 HDF5 파일 로드 시 사용 (본 스킬보다 우선순위 높음).
- **`cupynumeric.load`**: 단일 `.npy` 파일 로드 시 사용 (본 스킬보다 우선순위 높음).
- **`cupynumeric-install`**: Legate 런처 및 cuPyNumeric 환경 설정 가이드입니다.
