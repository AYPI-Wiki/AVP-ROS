---
title: "3D Occupancy — 복셀 점유 예측 (카메라 기반 3D 인지)"
type: concept
tags: [concept, BEV, Occupancy, perception, 자율주행, camera]
status: in-progress
aliases: [Occupancy, 3D Occupancy, 점유, 복셀 점유, Occupancy Grid, OCC]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# 3D Occupancy (복셀 점유)

> **한 줄 정의**
> 공간을 **복셀(voxel) 격자**로 나눠 각 칸이 **점유/비점유(또는 시맨틱 클래스)**인지를 예측하는 3D 인지 표현. [[BEV]]가 2D 평면으로 눌러 표현하는 것을 **높이(Z)까지 살려** 3D로 확장한 것.

## 핵심 개념
- **BEV vs Occupancy**: BEV는 Z축을 눌러 top-down 격자로 표현 → 오버행(육교·나뭇가지)·비정형 장애물에 약함. Occupancy는 `(X,Y,Z)` 복셀마다 점유 여부를 예측 → **임의 형태 장애물**도 표현.
- **출력 형태**: 각 복셀에 대해 (1) 점유 확률, (2) 시맨틱 클래스(차량·보행자·도로·주차구획 등), (3) 필요 시 flow(동적 점유)까지.
- **Ground Truth**: [[LiDAR]] 점군을 복셀화해 **GT 자동 생성**(예: VLP-16으로 복셀 점유 라벨링) → 카메라 단독 예측을 라이다로 **지도학습(supervised)**. ([[W3_Seminar]] §7.4)
- **학습 기반 파이프라인**: 다중 카메라 → BEV/복셀 특징 → Occupancy 헤드. [[LSS]]식 forward lifting이나 [[BEVFormer]]식 backward attention 위에 Occupancy 헤드를 얹는 구조가 일반적.
- **대표 흐름**: MonoScene · TPVFormer · OccFormer · SurroundOcc → nuScenes **Occ3D** 벤치마크로 수렴.

## 본 연구에서의 역할
- [[HL FMA 2026]] **자율주차** 미션의 핵심 표현: 주차면 점유 여부를 복셀 단위로 직접 판단 → **빈 주차공간 감지 + 실시간 맵 업데이트**. ([[W3_Seminar]] §7.3)
- 카메라 중심 인지로 고가 LiDAR 의존을 낮춰 **300만원·24V 센서 규정** 하에서 유리.
- GT 자동화(VLP-16급 소형 LiDAR)로 라벨링 비용 절감 → 1/5 스케일 차량 데이터셋 구축에 현실적.
- 렌더링 확장: [[3DGS]] · [[NeRF]] 기반 Occupancy 예측, [[ParkGaussian]] 트랙과 연계.

## 관련 노트
- [[BEV]] — 2D 조감도 표현(높이 눌림), Occupancy의 전 단계
- [[LSS]] · [[BEVFormer]] — Occupancy 헤드가 얹히는 BEV 인코더
- [[3DGS]] · [[NeRF]] · [[ParkGaussian]] — 차세대 렌더링 기반 예측
- [[LiDAR]] — GT 자동 생성 소스
- [[W3_Seminar]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
