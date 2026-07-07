---
title: "3DGS (3D Gaussian Splatting) — 명시적 가우시안 기반 실시간 3D 복원·렌더링"
type: concept
tags: [concept, rendering, 3D복원, 자율주행, NeRF]
status: in-progress
aliases: [3DGS, 3D Gaussian Splatting, 가우시안 스플래팅]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# 3DGS (3D Gaussian Splatting)

> **한 줄 정의**
> 장면을 수백만 개의 **3D 가우시안**(위치·공분산·색·불투명도)으로 표현하고, 이를 화면에 **스플래팅(투영·혼합)**해 실시간 렌더링하는 명시적 3D 복원 기법. (Kerbl et al., SIGGRAPH 2023)

## 핵심 개념
- **명시적 표현**: [[NeRF]]가 좌표→색·밀도를 **암시적 MLP**로 학습하는 것과 달리, 3DGS는 가우시안 집합을 **직접 저장**·최적화 → 학습·렌더링이 훨씬 빠름(실시간 60+ FPS).
- **파이프라인**: SfM 점군으로 초기화 → 미분 가능 **타일 기반 래스터라이저**로 렌더 → 사진과의 오차를 역전파해 가우시안 파라미터 최적화 + adaptive densify/prune.
- **장점**: 고품질·실시간·편집 용이(명시적). **단점**: 메모리 큼, 동적 장면·희소 뷰에 취약(연구 진행 중).
- **자율주행 연계**: 장면을 가우시안으로 복원 → 새로운 시점 합성, [[Occupancy]] 예측 백엔드, 시뮬레이션용 디지털 트윈에 활용.

## 본 연구에서의 역할
- [[W3_Seminar]] §7.2/7.4의 **L4 퓨처 브릿지** — GPU 성능 발전에 따라 3DGS 기반 Occupancy 예측 도입, [[ParkGaussian]] 트랙과 합류.
- 주차장 3D 모델을 고품질·실시간 복원 → [[HL FMA 2026]] 자율주차 맵 생성·시뮬레이션의 차세대 후보.
- [[NeRF]] 대비 **실시간성**이 강점이라 온보드/근실시간 요구에 더 현실적.

## 관련 노트
- [[NeRF]] — 암시적 표현 대비 개념(3DGS는 명시적)
- [[ParkGaussian]] — 주차장 특화 3DGS 파이프라인
- [[Occupancy]] — 렌더링 기반 점유 예측 백엔드
- [[BEV]] · [[LSS]] — 카메라 기반 3D 인지 계보
- [[W3_Seminar]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
