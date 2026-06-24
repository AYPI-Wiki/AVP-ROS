---
title: "YOLO — 실시간 1-stage 객체 탐지 모델"
type: concept
tags: [concept, vision, 딥러닝]
status: in-progress
aliases: [YOLO, YOLOv8, You Only Look Once]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# YOLO

> **한 줄 정의**
> You Only Look Once. 이미지를 한 번만 보고 물체의 위치와 종류를 동시에 찾아내는 1-stage 실시간 객체 탐지 모델.

## 핵심 개념
- **1-stage vs 2-stage**: R-CNN 등 2-stage(Region Proposal → 분류)는 정확하지만 느림. YOLO는 전체 이미지를 신경망에 1번 통과 → BBox+클래스 동시 출력 → 실시간(30fps+) 가능.
- **작동 원리**: 그리드 분할(SxS) → 셀별 책임(중심점이 든 셀이 탐지) → 동시 예측(BBox + Confidence + Class) → NMS(겹친 박스 중 최고 확률만 남김).
- **라벨 포맷**: Detection = BBox(축 정렬 사각형)만, **Segmentation = BBox + Mask**(정확한 영역).
- **Lifecycle 처리**: 무거운 모델 로딩을 위해 `Yolov8Node`는 LifecycleNode(configure→activate→deactivate→shutdown)로 안전하게 시작.

## 본 연구에서의 역할
- `camera_perception_pkg`의 핵심 — 차선/객체/신호등 검출(`detections` 발행). 라벨링 500장으로 `sim.pt` 학습 모델 제작. [[W1_Seminar]] · [[W2_Seminar]]

## 관련 노트
- [[OpenCV]] · [[KITTI]]
- [[W1_Seminar]] · [[W2_Seminar]]
