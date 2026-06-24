---
title: "OpenCV — 오픈소스 컴퓨터 비전 라이브러리"
type: concept
tags: [concept, vision, 센서]
status: in-progress
aliases: [cv2, OpenCV, cv_bridge]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# OpenCV

> **한 줄 정의**
> 컴퓨터가 시각 정보(사진·동영상)를 이해·분석하도록 돕는 오픈소스 컴퓨터 비전 및 머신러닝 라이브러리.

## 핵심 개념
- **기능**: 이미지 처리/변환(크기·회전·필터·색공간), 객체 검출·인식, 특징점 추출·매칭, 모션 분석·객체 추적, 카메라 캘리브레이션·차선 인식.
- **활용**: TensorFlow/PyTorch/[[YOLO]] 등 AI 모델의 **전처리**로 사용 — 카메라 영상을 OpenCV로 전처리 후 모델에 입력.
- **ROS2 연동 — cv_bridge**: ROS2 카메라 영상은 `sensor_msgs/msg/Image`(header/height·width/encoding/data) 형식. OpenCV `cv2` 이미지와 형식이 달라 **`cv_bridge`** 패키지로 변환 필요.
  - `imgmsg_to_cv2(data, "bgr8")` / `cv2_to_imgmsg(img, "mono8")`.

## 본 연구에서의 역할
- 카메라 인지 파이프라인의 기반 — 차선 마스크 에지 추출, ROI 처리 등. `camera_subscriber_pkg` 실습. [[W2_Seminar]]
- ⚠️ 실습 중 영상 출력 실패(노트북/카메라 연동 문제 추정) 기록.

## 관련 노트
- [[YOLO]] · [[LiDAR]] · [[IMU]]
- [[W2_Seminar]]
