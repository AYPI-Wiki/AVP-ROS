---
title: "IMU — 관성 측정 장치"
type: concept
tags: [concept, 센서]
status: in-progress
aliases: [관성측정장치, IMU, Inertial Measurement Unit]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# IMU

> **한 줄 정의**
> Inertial Measurement Unit. 로봇의 기울기·회전·가속도 등을 감지하는 관성 측정 센서.

## 핵심 개념
- **측정 데이터**: 선형 가속도(m/s², x·y·z) · 각속도(rad/s, 회전 속도) · 자세 orientation(quaternion).
- **ROS2 메시지**: `sensor_msgs/Imu`
  | 필드 | 타입 | 설명 |
  |------|------|------|
  | `orientation` | `geometry_msgs/Quaternion` | 자세(회전) |
  | `angular_velocity` | `geometry_msgs/Vector3` | 각속도 |
  | `linear_acceleration` | `geometry_msgs/Vector3` | 선형 가속도 |
- 시각화 시 선형 가속도·각속도 그래프로 로봇의 움직임을 데이터화해 확인.

## 본 연구에서의 역할
- 차량 자세·움직임 추정에 사용. 대회 차량 허용 센서. [[W2_Seminar]] · [[HL FMA 2026]]

## 관련 노트
- [[LiDAR]] · [[OpenCV]]
- [[W2_Seminar]]
