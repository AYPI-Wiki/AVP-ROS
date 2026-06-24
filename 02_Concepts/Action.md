---
title: "Action — 목표 지향 + 피드백 + 취소 (장기 실행 작업)"
type: concept
tags: [concept, ROS2, 통신]
status: in-progress
aliases: [액션, ROS2 Action]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Action

> **한 줄 정의**
> 시간이 오래 걸리고 진행 상황·취소가 필요한 장기 실행(long-running) 작업을 위한 통신. 비동기 요청-응답 + 중간 피드백 스트림 + 취소를 한 묶음으로 표준화한 상위 계층 패턴.

## 핵심 개념
- **작동 방식**:
  - **Goal**: "이 좌표까지 가" 같은 명령.
  - **Feedback**: 진행 중 계속 들어오는 상태 보고 ("지금 50% 진행…").
  - **Result**: 작업 종료 시 최종 결과 ("성공"/"장애물로 실패").
- **내부 구성**: Goal/Result/Cancel = [[Service]] · Feedback/Status = [[Topic]].
- **용도**: Nav2 자율주행(NavigateToPose, 진행률 피드백·새 명령 시 취소), MoveIt2 로봇 팔 제어(MoveGroup), 그리퍼 잡기, 캘리브레이션 절차.
- **AUTOSAR 대응**: 단일 대응물 없음 → CSI(goal/result/cancel) + SRI(feedback/status) 조합. = 비동기 Method + Event 스트림 + 취소 Method.

## 본 연구에서의 역할
- 자율주행 경로 이동 등 장기 작업 제어에 활용 가능.

## 관련 노트
- [[Node]] · [[Topic]] · [[Service]]
- [[W2_Seminar]]
