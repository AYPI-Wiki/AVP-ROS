---
title: "KITTI — 대표 자율주행 데이터셋"
type: concept
tags: [concept, dataset, 자율주행]
status: in-progress
aliases: [KITTI, KITTI dataset, 키티]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# KITTI

> **한 줄 정의**
> 2012년 개발된 대표적인 자율주행 데이터셋. 카메라 + Velodyne LiDAR + GPS/IMU로 수집. (cvlibs.net/datasets/kitti)

## 핵심 개념
- **데이터 구분**:
  | | RAW (synced+rectified) | Odometry |
  |---|---|---|
  | 성격 | 원본 센서 데이터 모음 | RAW를 SLAM 평가용 재가공 |
  | 단위 | 날짜별 drive | 시퀀스 번호(00~21) |
  | 센서 | 카메라+Velodyne+GPS/IMU(OXTS) | 카메라+Velodyne (GPS/IMU 없음) |
  | GT | OXTS 기반 Pose | 6-DoF pose(00~10만 제공) |
- 실습: `kitti2bag` rosbag으로 RViz에서 LiDAR 포인트클라우드 + 카메라 영상 시각화.
- **활용 VLA 논문(2025)**: Reasoning-VLA(Chain-of-Causation), Impromptu-VLA(비구조 시나리오 4분류), CoVLA. → `01_Papers/`로 정리 예정.

## 본 연구에서의 역할
- 자율주행 인지/VLA 모델 학습·평가용 데이터셋 후보. [[W2_Seminar]]

## 관련 노트
- [[YOLO]] · [[LiDAR]] · [[IMU]]
- [[W2_Seminar]]
