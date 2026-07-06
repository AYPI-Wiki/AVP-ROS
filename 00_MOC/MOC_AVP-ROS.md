---
title: "MOC — AVP-ROS"
type: MOC
tags: [MOC, AVP-ROS]
---

← [[00_Home]]

# 🗺️ MOC — AVP-ROS

> **연구 한 줄 정의**
> ROS2 기반 자율주행 시스템 학습·구현 → [[HL FMA 2026]] 경진대회 출전.

## 1️⃣ 핵심 개념
- [[Node]] · [[Topic]] · [[Service]] · [[Action]] · [[Parameter]] — ROS2 구성·통신
- [[DDS]] — ROS2 통신 미들웨어
- [[OpenCV]] · [[LiDAR]] · [[IMU]] — 센서 데이터 처리
- [[YOLO]] — 객체 탐지
- [[BEV]] — 조감도 표현 (상위 개념)
- [[LSS]] · [[BEVFormer]] — 카메라 기반 BEV 인지 (forward / backward)
- [[BEVDepth]] — 깊이 supervision으로 forward BEV 개선 · [[deformable-attention]] — backward BEV 핵심 연산

## 2️⃣ 주요 논문 / 데이터셋
- [[KITTI]] — 자율주행 데이터셋
- VLA 논문 3종 (Reasoning-VLA · Impromptu-VLA · CoVLA, 2025)
- [[BEV_LSS_vs_BEVFormer]] — 카메라 기반 BEV 인지 비교 (LSS vs BEVFormer)

## 3️⃣ 프로젝트 / 실험
- [[HL FMA 2026]] — 🏁 출전 목표 대회
- H-Mobility-Autonomous-Advanced-Course-Simulation — 자율주행 시뮬레이션
- [[LSS_실행_Runbook]] — 🛠️ LSS 실행 런북 (Ubuntu 22.04, 시각화→학습) · ✅ 검증
- [[BEVFormer_실행_Runbook]] — 🛠️ BEVFormer 실행 런북 (Ubuntu 22.04, 평가→시각화→학습) · 🚧 draft

## 4️⃣ 작업 흐름
- [[05_Research]] — LitReview → Idea → Experiment → Writing → Submission
- [[00_Seminar_Home]] — 🤖 ROS2 기초 세미나 (6주) · [[Schedule]]
