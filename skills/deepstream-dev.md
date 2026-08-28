# 🎴 Skill Card: deepstream-dev (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `deepstream-dev`
- **도메인**: DeepStream / Core Development
- **설명**: NVIDIA DeepStream SDK를 사용하여 고성능 AI 비디오 분석 파이프라인을 설계, 구현 및 최적화하기 위한 엔지니어링 가이드입니다. `pyservicemaker` API를 통한 파이프라인 제어, TensorRT 추론 통합, GStreamer 기반의 제로-카피 메모리 관리 및 메타데이터 처리를 다룹니다.
- **핵심 목표**: NVMM(NVIDIA Video Memory Manager)을 활용한 GPU 가속 파이프라인을 구축하고, 다중 스트림 추론-트래킹-시각화-메시징으로 이어지는 End-to-End 워크플로우를 최적화하는 것입니다.

## 🎯 적용 시점 (When to Use)
- RTSP, HTTP, 로컬 파일 등 다양한 소스로부터 비디오 스트림을 입력받아 AI 분석 파이프라인을 구축할 때.
- TensorRT 기반의 객체 검출(Detection), 분류(Classification), 세그멘테이션 모델을 파이프라인에 통합할 때.
- 다중 객체 트래킹(Multi-Object Tracking)을 통해 객체 ID를 유지하고 경로를 추적해야 할 때.
- 분석 결과(Bounding Box, Label, Metadata)를 OSD(On-Screen Display)로 시각화하거나 Kafka/Cloud로 전송해야 할 때.
- Jetson(ARM64) 및 x86_64 플랫폼 모두에서 동작하는 최적화된 비디오 처리 앱을 개발할 때.

---

## 🏗️ 파이프라인 아키텍처 및 컴포넌트 (Pipeline Architecture)

### 1. 표준 파이프라인 흐름 (Standard Flow)
`Source` $\rightarrow$ `Stream Muxer` $\rightarrow$ `Inference (GIE)` $\rightarrow$ `[Tracker]` $\rightarrow$ `OSD` $\rightarrow$ `Renderer`

| 단계 | 역할 | 핵심 엘리먼트 | 필수 여부 |
| :--- | :--- | :--- | :--- |
| **Source** | 비디오 입력 (RTSP, File, Camera) | `nvurisrcbin` (권장), `nvmultiurisrcbin` | 필수 |
| **Stream Muxer** | 다중 스트림을 배치(Batch) 처리로 통합 | `nvstreammux` | 필수 |
| **Inference** | TensorRT 모델 추론 실행 | `nvinfer`, `nvinferserver` | 필수 |
| **Tracker** | 객체 간 ID 매칭 및 경로 추적 | `nvtracker` | 선택 (요청 시 추가) |
| **OSD** | 경계 상자, 라벨, 텍스트 오버레이 렌더링 | `nvosdbin` | 필수 (시각화 필요 시) |
| **Renderer** | 최종 영상 출력 또는 파일 저장 | `nveglglessink`, `nv3dsink`, `filesink` | 필수 |

### 2. 메모리 및 데이터 모델 (Memory Model)
- **NVMM (NVIDIA Video Memory Manager)**: 제로-카피(Zero-copy) GPU 버퍼 전송을 위해 사용합니다.
- **Caps-string**: GPU 메모리 사용 시 `video/x-raw(memory:NVMM), format=NV12`와 같은 형식을 사용합니다.
- **메타데이터 구조**: `.frame_items` 및 `.object_items` 반복자(Iterator)를 사용하여 프레임 및 객체 단위 정보를 처리합니다. (주의: `len()` 사용 불가, 1회성 소비)

---

## 🛠️ 핵심 구현 규칙 (Critical Implementation Rules)

### 1. 소스 및 싱크 설정 (Sources & Sinks)
- **`nvurisrcbin` 우선 사용**: RTSP, HTTP, 로컬 파일(`file://`)을 투명하게 처리합니다.
- **RTSP/Live 소스 최적화**: `nvstreammux`에서 `live-source=1`, Sink 엘리먼트에서 `sync=0`을 설정하여 지연 시간을 최소화하십시오.
- **플랫폼별 Sink 선택**:
  - **Jetson (aarch64)**: `nv3dsink` 사용.
  - **x86_64**: `nveglglessink` 사용.
- **Tee/Dynamic Source 교착 상태 방지**: 파이프라인에 `tee` 분기나 동적 소스가 포함된 경우, **모든 Sink 엘리먼트에 `async=0`**을 설정해야 PAUSED 상태에서 멈추지 않습니다.

### 2. 추론 설정 (`nvinfer` & TensorRT)
- **동적 ONNX 모델 처리**: 입력 형상이 가변적인 모델(예: YOLOv8 dynamic)의 경우, 설정 파일에 `infer-dims=C;H;W` (예: `infer-dims=3;640;640`)를 반드시 명시하십시오.
- **YOLO 버전별 출력 처리**:
  - **v8 / v11**: Pre-NMS 텐서 출력 $\rightarrow$ `cluster-mode: 2` (DeepStream NMS 사용).
  - **v10 / v26+**: Post-NMS 결과 출력 $\rightarrow$ `cluster-mode: 4` (NMS 생략).
- **설정 파일 형식**: YAML 사용 시 `property:` 섹션을 사용하고, INI 사용 시 `[property]` 섹션을 사용하십시오.

### 3. Python 및 환경 설정 (Environment)
- **`pyservicemaker` 가상환경 설치**: 가상환경(venv) 사용 시 `pyservicemaker`가 누락되어 `ModuleNotFoundError`가 발생합니다. 다음 명령어로 직접 설치하십시오.
  ```bash
  pip install /opt/nvidia/deepstream/deepstream/service-maker/python/pyservicemaker*.whl pyyaml
  ```
- **버퍼 복제 (Buffer Cloning)**: 비동기 처리 시 반드시 `.clone()`을 호출하여 버퍼를 복제하십시오.

---

## 🔍 트러블슈팅 가이드 (Troubleshooting)

| 증상 | 원인 | 해결 방법 |
| :--- | :--- | :--- |
| **파이프라인이 PAUSED 상태에서 멈춤** | Tee 또는 동적 소스 사용 시 Sink의 `async` 설정 누락 | 모든 Sink 엘리먼트에 `async=0` 설정 |
| **`setDimensions` 에러 / 엔진 빌드 실패** | 동적 ONNX 모델의 입력 차원 미지정 | `infer-dims=C;H;W` 설정 추가 |
| **객체 박스가 45°/135° 회전되어 나타남** | 모델 출력(Post-NMS)과 `cluster-mode` 불일치 | YOLOv10+인 경우 `cluster-mode: 4`로 변경 |
| **`RuntimeError: Probe failure`** | Sink 엘리먼트에 FPS 측정 프로브 부착 시도 | 프로브를 `nvinfer` 또는 `nvosdbin`에 부착 |
| **`iterator has no len()`** | 메타데이터 반복자에 `len()` 사용 | 반복문을 통해 개수를 직접 계산 |

## 📚 관련 참조 문서
- **GStreamer 플러그인**: `references/gstreamer_plugins.md`
- **Service Maker API**: `references/service_maker_api.md`
- **추론 설정 상세**: `references/nvinfer_config.md`
- **트래커 설정**: `references/tracker_config.md`
- **환경 구축/Docker**: `references/docker_containers.md`
