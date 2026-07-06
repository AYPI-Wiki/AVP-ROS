---
title: "BEVDepth — 명시적 깊이 supervision 기반 BEV 인지"
type: concept
tags: [concept, BEV, perception, 자율주행, camera, depth]
status: in-progress
aliases: [BEVDepth, 베브뎁스]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# BEVDepth

> **한 줄 정의**
> [[LSS]]식 forward projection에서 **깊이 예측을 LiDAR로 명시적 supervision** 하고, 카메라 파라미터를 깊이 추정에 넣어 정확도를 끌어올린 카메라 BEV 3D 탐지 모델. (Li et al., AAAI 2023)

## 핵심 개념
- **문제의식**: LSS의 깊이 분포는 GT 없이(간접) 학습돼 부정확 → 저자들이 분석하니 "학습된 깊이가 실제 깊이와 잘 안 맞는데도 탐지가 되는" **불안정한** 상태였다. 깊이가 곧 BEV 위치 정확도.
- **Explicit Depth Supervision**: 학습 시 **LiDAR 포인트를 깊이 GT**로 투영해 깊이 예측을 직접 감독. (추론 시엔 LiDAR 불필요 = 카메라 전용 유지)
- **Camera-aware Depth**: 카메라 내·외부 파라미터(intrinsic/extrinsic)를 깊이 네트워크에 주입 → 카메라별 화각·설치 차이를 깊이 추정이 반영.
- **Depth Refinement**: 깊이 축을 따라 특징을 다듬는 모듈로 오차 누적 완화.
- **위치**: [[LSS]] → BEVDet → **BEVDepth**로 이어지는 forward/geometry 계열의 정확도 개선판. nuScenes에서 카메라 전용 3D 탐지 SOTA급 달성.

## 본 연구에서의 역할
- [[LSS]]의 최대 약점(부정확·불안정한 깊이)을 **적은 추가 비용**으로 보완 → 경량 forward 계열을 실전 정확도까지 끌어올리는 현실적 경로.
- 학습 때만 [[LiDAR]] GT가 필요하므로, LiDAR 있는 데이터셋(nuScenes/[[KITTI]])으로 학습 후 [[HL FMA 2026]] 차량에선 카메라 추론만 하는 구성이 가능.
- [[BEVFormer]](backward, 무거움)에 비해 Jetson 친화적이면서 정확도 격차를 줄이는 절충안.
- forward vs backward 큰 그림은 → [[BEV_LSS_vs_BEVFormer]].

## 관련 노트
- [[LSS]] — 원류(forward projection), 깊이가 암묵적
- [[BEV]] — 상위 개념
- [[BEVFormer]] — backward 계열 대비
- [[LiDAR]] — 깊이 supervision GT 소스
- [[BEV_LSS_vs_BEVFormer]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
