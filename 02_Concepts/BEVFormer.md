---
title: "BEVFormer — 트랜스포머 기반 멀티카메라 BEV 인지"
type: concept
tags: [concept, BEV, perception, 자율주행, transformer]
status: in-progress
aliases: [BEVFormer, 베브포머]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# BEVFormer

> **한 줄 정의**
> 여러 대의 카메라 이미지에서 **spatiotemporal transformer**로 BEV(조감도) 표현을 학습하는 **카메라 전용** 3D 인지 모델. (Li et al., ECCV 2022)

## 핵심 개념
- **BEV 쿼리(BEV queries)**: 격자 모양으로 미리 배치한 학습 쿼리. 각 쿼리가 BEV 한 칸(지면 위 실제 위치)을 담당.
- **Spatial Cross-Attention (SCA)** — *view transformation* 담당:
  - **Backward projection (3D→2D, pull)**: BEV 쿼리를 높이 방향 3D reference point로 확장 → 카메라 파라미터로 각 이미지 뷰에 투영 → **deformable attention**으로 특징을 당겨옴.
  - 명시적 깊이 예측 **없음** (attention이 대체) ↔ [[LSS]]의 forward projection과 대비.
- **Temporal Self-Attention (TSA)**: 직전 시점 BEV 특징을 참조(recurrent) → 속도 추정·가림(occlusion)에 강함.
- **태스크**: 3D 객체 탐지 + 맵 분할. **성능**: nuScenes test **NDS 56.9%**(카메라 전용, 당시 SOTA) / base(R101-DCN) val 51.7 NDS·41.6 mAP.
- **분류**: view transformation 중 **transformer/attention 기반(backward)** 계열. 파생: BEVFormer v2, PolarFormer 등.
- ⚠️ **용어 주의**: "Transformer 아키텍처"로 "view transformation(뷰 변환)"을 수행 — 두 의미 구분. → 상세 [[BEV_LSS_vs_BEVFormer]].

## 본 연구에서의 역할
- 카메라만으로 3D BEV를 얻는 인지 후보. 얻은 BEV는 [[W3_Seminar]]의 경로 계획(Nav2)·제어 입력으로 연결 가능.
- **E2E 주행 스택의 부품**: UniAD(CVPR 2023) 등이 BEVFormer식 BEV 인코더를 백본으로 사용.
- [[HL FMA 2026]] 관점: 정확도는 높지만 트랜스포머라 **연산·메모리 부담** → Jetson 소형 차량엔 무거울 수 있음. 경량 대안은 [[LSS]]/BEVDepth 계열.
- 이론·수치·장단점 비교 전체는 → [[BEV_LSS_vs_BEVFormer]].

## 관련 노트
- [[LSS]] — forward projection 대비 개념
- [[BEV_LSS_vs_BEVFormer]] — 심층 비교(정의·수치·E2E 계보)
- [[BEVFormer_실행_Runbook]] — 🛠️ 실제 실행 가이드 (Ubuntu 22.04)
- [[deformable-attention]] — Spatial Cross-Attention의 핵심 연산
- [[LiDAR]] · [[KITTI]] · [[YOLO]] — 인지 관련
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
