---
title: "Parameter — 노드의 튜닝 손잡이 / 설정 저장소"
type: concept
tags: [concept, ROS2]
status: in-progress
aliases: [파라미터, ROS2 Parameter]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Parameter

> **한 줄 정의**
> 노드가 자신의 동작을 조정하기 위해 보유하는 이름이 붙은 타입값. 한마디로 튜닝 손잡이/설정 저장소.

## 핵심 개념
- 노드는 `declare_parameter("name", default)`로 파라미터를 선언하고 `.value`로 조회.
- 코드 수정 없이 동작을 바꿀 수 있는 설정값 — 예: 모델 경로, 추론 디바이스(cpu/cuda), 임계값(threshold).
- **AUTOSAR 대응**: Adaptive의 Field (상태값 + get/set/notify), latched topic 성격.

## 본 연구에서의 역할
- 시뮬레이션 `Yolov8Node`에서 `model`(.pt 경로), `device`, `threshold` 등을 파라미터로 설정. 모델 경로를 절대경로로 지정해 오류 해결한 사례. [[W1_Seminar]]

## 관련 노트
- [[Node]] · [[Service]]
- [[W2_Seminar]]
