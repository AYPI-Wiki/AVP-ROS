---
title: "Topic — 노드 간 단방향 Pub/Sub 통신"
type: concept
tags: [concept, ROS2, 통신]
status: in-progress
aliases: [토픽, ROS2 Topic, Publisher, Subscriber, Pub/Sub]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Topic

> **한 줄 정의**
> 노드 간에 데이터를 주고받는 가장 기본적인 **단방향** 통신 방식. Publisher/Subscriber 모델을 따르며 익명·비동기 연속 스트림이다.

## 핵심 개념
- **작동 원리**: Publisher가 Topic Name으로 메시지를 지속 발행 → Subscriber가 Topic Name을 구독하고 **콜백 함수**로 데이터 처리.
- **통신 구조**: **1:N, N:1, N:N** — 하나의 토픽에 여러 Publisher/Subscriber 연결 가능. 누가 보내고 받는지 서로 모르는 익명 통신.
- **용도**: 값이 빠르게 갱신되는 데이터에 최적 — 카메라 이미지(30Hz), 라이다 스캔(10Hz), `/cmd_vel`(속도), [[IMU]](100Hz+), TF(좌표 변환).
- **명령어**: `ros2 topic list` / `ros2 topic echo /<topic>` / `ros2 topic info /<topic>`.
- **AUTOSAR 대응**: Classic의 SRI(Sender-Receiver-Interface), Adaptive의 Event. (ROS Topic은 `data`만, SRI는 `data+event+mode`)

## 본 연구에서의 역할
- AVP 시뮬레이션 노드 간 데이터 흐름의 기본 — `detections`, `yolov8_lane_info`, `scan`, `control` 등이 모두 토픽. [[W2_Seminar]]

## 관련 노트
- [[Node]] · [[Service]] · [[Action]]
- [[W1_Seminar]] · [[W2_Seminar]]
