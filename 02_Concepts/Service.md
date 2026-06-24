---
title: "Service — 요청/응답(RPC) 양방향 통신"
type: concept
tags: [concept, ROS2, 통신]
status: in-progress
aliases: [서비스, ROS2 Service, Client, Server]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Service

> **한 줄 정의**
> 클라이언트의 요청에 서버가 응답하는 양방향 통신(함수 호출/RPC). "부메랑"처럼 요청을 보내고 응답을 받는다.

## 핵심 개념
- **구조**: Service Client(요청) ↔ Service Server(요청 받고 응답).
- **호출 방식**:
  - **동기 호출**: 응답이 올 때까지 대기 → 응답 지연 시 노드 전체가 멈출 위험(DeadLock).
  - **비동기 호출**: Future 객체를 받아두고 응답이 오면 콜백 처리 → 블로킹 방지, **권장**.
- **용도**: 짧고 빠르게 끝나는 일회성 명령/질의 — 그리퍼 열기/닫기(SetBool), 맵 저장(SaveMap), [[Parameter]] 변경, 배터리 잔량 질의, 카메라 캘리브레이션 정보 가져오기.
- **AUTOSAR 대응**: Classic의 CSI(Client-Server-Interface), Adaptive의 Method.

## 본 연구에서의 역할
- 1회성 설정·질의에 사용. [[Action]]의 Goal/Result/Cancel 내부도 Service 기반.

## 관련 노트
- [[Node]] · [[Topic]] · [[Action]] · [[Parameter]]
- [[W2_Seminar]]
