---
title: "BEV (Bird's-Eye-View) — 조감도 표현·인지"
type: concept
tags: [concept, BEV, perception, 자율주행]
status: in-progress
aliases: [BEV, Bird's-Eye-View, 조감도, 버드아이뷰]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# BEV (Bird's-Eye-View)

> **한 줄 정의**
> 여러 센서(카메라·LiDAR) 입력을 **지면을 위에서 내려다본 2D 격자**(top-down grid)로 통합해 표현하는 방식. 자율주행 인지·계획의 공용 좌표계.

## 핵심 개념
- **왜 BEV인가**: 원근 왜곡이 없어 **거리·크기가 실제 스케일로 보존**됨 → 여러 카메라/센서를 하나의 좌표계로 융합하기 쉽고, 경로 계획·제어가 바로 얹힌다(맵 좌표와 정렬).
- **표현 형태**: 자차(ego) 중심 격자. 각 셀이 지면의 실제 위치(예: 0.5m 해상도)를 담당하고, 셀마다 특징 벡터를 채운다.
- **핵심 난제 = view transformation**: 2D 원근 이미지(perspective)를 3D BEV로 올리는 변환. 이를 푸는 두 계열:
  - **Forward (2D→3D, push)**: 픽셀 깊이를 추정해 특징을 밀어올림 → [[LSS]] 계열.
  - **Backward (3D→2D, pull)**: BEV 쿼리가 attention으로 이미지 특징을 당겨옴 → [[BEVFormer]] 계열.
- **입력 모달리티**: 카메라 전용 · LiDAR 전용 · 멀티모달 융합(BEVFusion). LiDAR는 이미 3D라 BEV 투영이 자연스럽다.
- **다운스트림 태스크**: BEV 시맨틱 분할(맵·차선·주행가능영역) · 3D 객체 탐지 · 모션 예측 · E2E 계획.

## 본 연구에서의 역할
- [[HL FMA 2026]] 인지 스택의 **공용 중간표현** 후보. 카메라(+LiDAR/IMU) 입력 → BEV → 경로 계획·제어로 이어지는 파이프라인의 허브.
- 구체적 구현 두 축이 [[LSS]](경량·forward) / [[BEVFormer]](정확·backward). Jetson 소형 차량 제약상 경량 계열이 현실적 출발점.
- BEV 격자는 [[W3_Seminar]]의 경로 계획(Nav2)·제어 입력으로 직접 연결.
- 심층 비교·수치·계보는 → [[BEV_LSS_vs_BEVFormer]].

## 관련 노트
- [[LSS]] · [[BEVFormer]] — BEV 생성 두 계열(forward / backward)
- [[BEVDepth]] — 깊이 supervision으로 forward BEV 개선
- [[deformable-attention]] — backward BEV의 핵심 연산
- [[BEV_LSS_vs_BEVFormer]] — 심층 비교
- [[LiDAR]] · [[KITTI]] — 인지 관련
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
