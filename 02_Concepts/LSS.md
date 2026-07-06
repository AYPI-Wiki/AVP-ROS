---
title: "LSS (Lift-Splat-Shoot) — 깊이 기반 카메라 BEV 인지"
type: concept
tags: [concept, BEV, perception, 자율주행, camera]
status: in-progress
aliases: [LSS, Lift-Splat-Shoot, 리프트스플랫슛]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# LSS (Lift, Splat, Shoot)

> **한 줄 정의**
> 픽셀별 **깊이 분포**를 예측해 이미지 특징을 3D로 들어올린 뒤 BEV 격자에 뿌리는 **카메라 전용** BEV 인지 방법. 카메라→BEV 변환의 사실상 원조. (Philion & Fidler, ECCV 2020)

## 핵심 개념
- 이름이 곧 3단계 파이프라인:
  - **Lift**: 각 픽셀에 대해 이산 깊이 구간(D bins)의 **확률 분포**(softmax)를 예측 → 문맥 특징(C)과 깊이(D)의 **외적**으로 프러스텀 3D 점 특징 생성. (깊이 GT 없이 학습 가능)
  - **Splat**: 카메라 파라미터로 3D 점을 **공유 BEV 격자**에 투영, 같은 칸끼리 pillar pooling(sum)으로 합침.
  - **Shoot**: BEV 코스트맵 위로 **후보 궤적(templates)**을 쏘아 최소 비용 경로를 **선택** = 간이 계획(생성 아닌 선택).
- **Forward projection (2D→3D, push)**: 깊이를 추정해 특징을 밀어 넣음 ↔ [[BEVFormer]]의 backward(pull)와 대비.
- **분류**: view transformation 중 **geometry 기반(forward)** 계열. **깊이가 명시적** → BEVDepth가 깊이 supervision 추가로 개선.
- **태스크**: BEV 시맨틱 분할(맵·차량) 위주. 파생: BEVDet · BEVDepth · **BEVFusion**(멀티모달 융합) · FIERY.

## 본 연구에서의 역할
- **경량·모듈화**되어 BEV 인코더 building block으로 채택 사례 多 → [[HL FMA 2026]]의 Jetson 소형 차량에 **현실적 출발점**.
- 명시적 3D 점을 만들어 [[LiDAR]]와 정렬이 쉬움(BEVFusion식 카메라-LiDAR 융합에 유리).
- **E2E 주행 계보**: ST-P3(ECCV 2022)가 lift-splat 계열 BEV를 인지 프론트엔드로 사용.
- 얻은 BEV는 [[W3_Seminar]]의 경로 계획(Nav2)·제어 입력으로 연결 가능. 실제 실행은 → [[LSS_실행_Runbook]].
- 이론·수치·장단점 비교 전체는 → [[BEV_LSS_vs_BEVFormer]].

## 관련 노트
- [[BEVFormer]] — backward projection 대비 개념
- [[BEV_LSS_vs_BEVFormer]] — 심층 비교(정의·수치·E2E 계보)
- [[LSS_실행_Runbook]] — 🛠️ 실제 실행 가이드 (Ubuntu 22.04)
- [[LiDAR]] · [[KITTI]] · [[YOLO]] — 인지 관련
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
