---
title: "NeRF (Neural Radiance Fields) — 신경망 암시적 장면 표현·뷰 합성"
type: concept
tags: [concept, rendering, 3D복원, 자율주행]
status: in-progress
aliases: [NeRF, Neural Radiance Fields, 신경 방사 필드]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# NeRF (Neural Radiance Fields)

> **한 줄 정의**
> 3D 좌표 `(x,y,z)`와 시선 방향 `(θ,φ)`를 입력받아 **색(RGB)과 밀도(σ)**를 출력하는 MLP로 장면을 **암시적으로** 표현하고, 볼륨 렌더링으로 새로운 시점 이미지를 합성하는 기법. (Mildenhall et al., ECCV 2020)

## 핵심 개념
- **암시적 표현**: 장면을 신경망 가중치에 저장 → 좌표를 질의하면 색·밀도를 반환. 볼륨 렌더링(광선 상 샘플 적분)으로 픽셀 색 계산, 사진과의 오차를 역전파.
- **강점**: 매우 고품질·연속적 뷰 합성, 콤팩트한 표현. **약점**: 학습·렌더링 **느림**, 장면마다 재학습, 동적/대규모 장면에 확장 난제(→ Instant-NGP·Mip-NeRF 등으로 개선).
- **vs [[3DGS]]**: NeRF=암시적·느림·고품질 ↔ 3DGS=명시적·실시간. 최근 자율주행 실시간 요구에는 3DGS 쪽이 부상.
- **자율주행 연계**: 정적 장면 복원, 새로운 시점 데이터 증강, [[Occupancy]]·시뮬레이션용 장면 재구성.

## 본 연구에서의 역할
- [[W3_Seminar]] §7.2의 **L4 퓨처 브릿지** — GPU 활용 3DGS/NeRF 기반 예측 확장의 한 축.
- [[HL FMA 2026]] 주차장 3D 복원·시뮬레이션 데이터 증강의 이론적 배경. 다만 실시간성 한계로 온보드보다는 **오프라인 맵 생성/데이터 증강**에 적합.
- 전통적 [[W3_Seminar#2. 전통적 SfM + MVS|SfM+MVS]] → 학습 기반 복원으로 넘어가는 계보의 출발점.

## 관련 노트
- [[3DGS]] — 명시적·실시간 대안(최근 주류)
- [[ParkGaussian]] — 3DGS/NeRF 계열 주차장 복원
- [[Occupancy]] — 렌더링 기반 3D 인지 백엔드
- [[W3_Seminar]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
