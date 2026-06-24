---
title: "DDS — ROS2의 표준 통신 미들웨어"
type: concept
tags: [concept, ROS2, 통신]
status: in-progress
aliases: [Data Distribution Service, 데이터 분산 서비스]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# DDS

> **한 줄 정의**
> Data Distribution Service. ROS2가 노드 간 통신에 사용하는 국제 표준 통신 규격으로, 다양한 환경에서 안정적인 통신을 보장한다.

## 핵심 개념
- ROS2는 ROS1의 중앙 집중형 `roscore` 대신 **DDS 기반 P2P 분산형 검색**을 사용 → 단일 장애점(SPOF) 제거.
- ROS1의 한계(SPOF, 불안정 네트워크 생존력 부족, 보안 취약)를 DDS 도입으로 개선.
- 프로세스당 다중 노드 지원, 임베디드(micro-ROS) 상용 수준 지원, Lifecycle 노드 등 ROS2 아키텍처의 기반.

## 본 연구에서의 역할
- ROS2를 선택하는 핵심 이유 중 하나 — 자율주행처럼 안정성이 중요한 실시간 분산 시스템에 적합. [[W1_Seminar]]

## 관련 노트
- [[Node]] · [[Topic]]
- [[W1_Seminar]]
