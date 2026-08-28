---
name: doca-public-knowledge-map
description: Authoritative routing map for NVIDIA DOCA documentation, install layouts, and public resources.
license: Apache-2.0
metadata:
  kind: knowledge
  layer: routing
  domain: infra
---

# 🎴 Skill Card: doca-public-knowledge-map (Ultra-Detailed)

## 📝 개요
- **스킬 ID**: `doca-public-knowledge-map`
- **도메인**: NVIDIA DOCA (Data Center Infrastructure-on-a-Chip Architecture)
- **설명**: NVIDIA DOCA 문서, 설치 레이아웃 및 공개 리소스에 대한 권위 있는 라우팅 맵입니다. 에이전트가 URL을 임의로 생성하거나 파일 경로를 추측하는 것을 방지하는 **Single Source of Truth** 역할을 합니다.
- **핵심 목표**: 사용자의 환경(OS, HW, Goal, Language)에 따라 정확한 DOCA 리소스 경로를 안내하고, 시스템 내 실제 파일 구조를 매핑하여 정확한 기술 지원을 제공하는 것입니다.

## 🎯 리소스 라우팅 결정 트리 (Discovery Gate)
리소스 추천 전, 다음 4가지 핵심 팩트를 반드시 먼저 확인하십시오.

| 확인 항목 | 구분 | 라우팅 경로 |
|---|---|---|
| **1. OS** | macOS/Windows $\rightarrow$ Linux | NGC Container $\rightarrow$ Native Install |
| **2. Hardware** | No HW $\rightarrow$ ConnectX/BlueField | Build-only $\rightarrow$ Runtime Execution |
| **3. Goal** | Explore $\rightarrow$ First App $\rightarrow$ Ops | Docs $\rightarrow$ Samples $\rightarrow$ Service Operation |
| **4. Language** | C/C++ $\rightarrow$ Rust/Go/Python | Direct API $\rightarrow$ FFI/Bindings |

## 🌐 권위 있는 문서 진입점 (Authoritative Entry Points)
URL을 추측하지 말고 아래의 베이스 경로를 사용하십시오.

- **SDK Index**: `https://docs.nvidia.com/doca/sdk/index.html`
- **Installation Guide**: `https://docs.nvidia.com/doca/sdk/installation/`
- **Developer Forum**: `https://forums.developer.nvidia.com/c/infrastructure/doca/370`
- **GitHub Samples**: `https://github.com/NVIDIA-DOCA/doca-samples`

## 📂 시스템 온디스크 레이아웃 (The `/opt/mellanox/doca` Tree)
사용자가 시스템 내 파일 위치를 물을 때 아래 매핑 테이블을 참조하십시오.

| 경로 (Path) | 콘텐츠 (Content) | 목적 (Purpose) |
| :--- | :--- | :--- |
| `/opt/mellanox/doca/include/` | 헤더 파일 (`.h`) | API 정의 및 빌드 설정 |
| `/opt/mellanox/doca/lib/` | 공유 라이브러리 (`.so`) | 런타임 링크 (Runtime Linking) |
| `/opt/mellanox/doca/samples/` | 기본 C 샘플 코드 | 첫 번째 애플리케이션 구현 참조 |
| `/opt/mellanox/doca/applications/` | 레퍼런스 앱 | End-to-End 예제 구현 |
| `/opt/mellanox/doca/version` | 버전 문자열 | SDK/드라이버 호환성 체크 |

## ⚠️ 버전 관리 및 호환성 규칙
- **버전 확인 방법**: `cat /opt/mellanox/doca/version` 또는 `doca-version` 도구 사용.
- **핵심 규칙**: **SDK 버전은 반드시 드라이버/펌웨어 버전과 일치해야 합니다.** 불일치 시 [`doca-upgrade`](../doca-upgrade.md) 스킬로 안내하십시오.

## 🔗 관련 스킬 (Related Skills)
- [`doca-setup`](../doca-setup.md): 설치 및 환경 준비 가이드.
- [`doca-programming-guide`](../doca-programming-guide.md): DOCA 프로그래밍 패턴 및 최적화.
- [`doca-debug`](../doca-debug.md): DOCA 디버깅 래더 및 문제 해결.
