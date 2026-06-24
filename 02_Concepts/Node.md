---
title: "Node — ROS2를 구성하는 최소 단위 소프트웨어"
type: concept
tags: [concept, ROS2]
status: in-progress
aliases: [노드, ROS2 Node]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Node

> **한 줄 정의**
> ROS2를 구성하는 최소 단위의 소프트웨어로, 연산을 수행하는 독립 프로세스. 수많은 노드의 집합으로 로봇 시스템이 구성된다.

## 핵심 개념
- **Monolithic 방식 vs ROS2 방식**: 단일 SW가 모든 HW(LiDAR/Camera/IMU/Motor…)를 제어하는 대신, 하드웨어마다 독립 노드를 두고 [[Topic]]으로 통신.
- **분산 구조의 이점**: 모듈화(독립 개발·테스트) · 확장성(새 노드만 추가) · 재사용성 · 병렬 처리 · 안정성(한 노드 문제가 전체로 전파 안 됨).
- **주요 특성**: 독립성(독립 프로세스 실행) · 재사용성 · 통신 인터페이스([[Topic]]·[[Service]]·[[Action]]·[[Parameter]]로 외부와 상호작용).
- **패키지(Package)**: 관련 노드와 설정 파일을 담는 "폴더". 소스코드(Python/C++), 설정파일(`package.xml`·`setup.py`·`CMakeLists.txt`), 의존성, launch 파일, 메시지 정의 포함.

## 본 연구에서의 역할
- AVP 시뮬레이션은 인지(camera/lidar perception) → 판단(decision making) → 제어(simulation sender) 노드들의 그래프로 구성. [[W2_Seminar]]의 노드 그래프 참고.

## 관련 노트
- [[Topic]] · [[Service]] · [[Action]] · [[Parameter]]
- [[Workspace]] · [[DDS]]
- [[W1_Seminar]] · [[W2_Seminar]]
