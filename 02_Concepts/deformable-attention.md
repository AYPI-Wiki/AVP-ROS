---
title: "Deformable Attention — 희소 샘플링 어텐션"
type: concept
tags: [concept, transformer, attention, perception, 자율주행]
status: in-progress
aliases: [deformable attention, Deformable Attention, 디포머블 어텐션, Deformable DETR]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Deformable Attention

> **한 줄 정의**
> 전체 위치를 다 보는 대신, 각 쿼리마다 **소수의 참조점 주변 몇 개 지점만 골라(offset) 샘플링**해 attention을 계산하는 방식. 계산량을 O(N²)→선형 수준으로 낮춘다. (Deformable DETR, Zhu et al., ICLR 2021)

## 핵심 개념
- **동기**: 표준 attention은 모든 key와 상호작용 → 고해상도 이미지 특징에서 연산·메모리 폭발 + 수렴 느림.
- **동작**:
  - 쿼리마다 **reference point**(기준 위치) 설정.
  - 네트워크가 그 주변의 **sampling offset**(어디를 볼지)과 **attention weight**(얼마나 볼지)를 직접 예측.
  - 참조점 + offset 위치의 특징만 **쌍선형 보간**으로 뽑아 가중합. → key 전체 순회 없음.
- **효과**: 계산량이 특징맵 크기에 **선형**, 작은 물체·멀티스케일에 강함, DETR류 수렴 속도 대폭 개선.
- **BEV와의 연결**: [[BEVFormer]]의 **Spatial Cross-Attention**이 곧 deformable attention — BEV 쿼리를 이미지에 투영한 **reference point 주변만** 샘플링해 특징을 당겨온다(backward projection). Temporal Self-Attention에도 사용.

## 본 연구에서의 역할
- [[BEVFormer]]류 backward BEV가 **"무겁지만 그나마 감당 가능"**한 이유의 핵심 = 전역 attention이 아닌 희소 샘플링 덕분.
- [[HL FMA 2026]] 관점: 그래도 deformable 연산은 Jetson에서 부담 → forward 계열([[LSS]]/[[BEVDepth]]) 대비 실시간성 trade-off 판단 근거.
- 개념적으로 CNN의 **deformable convolution**(가변 수용영역)을 attention으로 옮긴 것.

## 관련 노트
- [[BEVFormer]] — 이 연산을 view transformation에 사용
- [[BEV]] · [[LSS]] · [[BEVDepth]] — BEV 계열 비교
- [[BEV_LSS_vs_BEVFormer]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
