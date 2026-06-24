---
title: "LiDAR — 레이저 거리 측정 센서"
type: concept
tags: [concept, 센서]
status: in-progress
aliases: [라이다, LiDAR, LaserScan, Light Detection and Ranging]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# LiDAR

> **한 줄 정의**
> Light Detection and Ranging. 레이저를 발사해 반사된 신호로 거리를 측정하는 센서.

## 핵심 개념
- **ROS2 메시지**: `sensor_msgs/LaserScan`
  | 필드 | 설명 |
  |------|------|
  | `angle_min` / `angle_max` | 시작/종료 각도(라디안) |
  | `angle_increment` | 스캔 간격 |
  | `ranges` | 거리 데이터 배열 |
  | `intensities` | 반사 강도(선택) |
- 결과값은 극좌표 그래프로 표시. `ranges`는 360도 전 방향 수백 개 거리값 배열.
- **활용 예**: `min_distance = np.min(ranges)` → `if min_distance < 0.5: warn(...)` → 실제론 속도 제어 토픽을 조정해 급정거에 활용.

## 본 연구에서의 역할
- `lidar_perception_pkg`에서 장애물 인식(Obstacle detector)에 사용. 대회 차량 허용 센서. [[W2_Seminar]] · [[HL FMA 2026]]

## 관련 노트
- [[IMU]] · [[OpenCV]] · [[Topic]]
- [[W2_Seminar]]
