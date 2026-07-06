---
title: "LSS vs BEVFormer — 카메라 기반 BEV 인지 비교"
type: litreview
tags: [litreview, BEV, perception, 자율주행, camera]
status: in-progress
aliases: [LSS vs BEVFormer, BEV 비교, Lift-Splat-Shoot BEVFormer]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# 🐦 LSS vs BEVFormer — 카메라 기반 BEV 인지 비교

> **한 줄 요약**
> 둘 다 여러 대의 카메라 영상을 **BEV(Bird's-Eye-View, 조감도)** 격자 특징으로 변환하는 방법. **LSS**는 픽셀별 깊이 분포를 예측해 3D로 "밀어 올리는(forward projection)" CNN 방식이고, **BEVFormer**는 BEV 쿼리가 이미지에서 특징을 "당겨오는(backward projection)" 트랜스포머 방식이다.

> **왜 정리하는가** — 카메라만으로 3D 공간을 인식하는 BEV 파이프라인은 [[HL FMA 2026]] 같은 소형 자율주행에서 LiDAR 의존도를 낮추는 핵심 접근. 현대 BEV 인지의 두 계보(depth-lift 계열 vs transformer 계열)의 원류를 비교한다.

---

## 0. BEV가 왜 필요한가
- **BEV(조감도)** = 차량을 위에서 내려다본 2D 평면 격자 표현. 각 셀이 지면 위 실제 위치(x, y)에 대응.
- 장점: ① 여러 카메라·센서를 **하나의 좌표계로 융합**하기 쉬움 ② 거리·크기가 왜곡 없이 보존돼 **경로 계획·제어**([[W3_Seminar]]의 Nav2·DWA·경로계획)로 바로 연결 ③ 시간에 따른 객체 추적/속도 추정에 유리.
- 핵심 난제: **2D 이미지 → 3D BEV 변환**(view transformation). 이미지에는 깊이 정보가 없어 이 변환을 어떻게 푸느냐가 방법을 가른다. → LSS와 BEVFormer는 이 문제에 대한 **정반대 해법**.

## 1. LSS (Lift, Splat, Shoot)
> Philion & Fidler, **ECCV 2020** (NVIDIA / Univ. of Toronto). 카메라 전용 BEV의 사실상 원조.

3단계 이름이 그대로 파이프라인:
- **Lift (들어올리기)** — 각 카메라 이미지의 픽셀마다
  - 미리 정한 **이산 깊이 구간(depth bins) D개**에 대한 **확률 분포**(softmax)를 예측 = "이 픽셀이 어느 깊이에 있을지" soft하게 추정.
  - 문맥 특징(context, C차원)과 깊이 분포(D차원)의 **외적(outer product)**으로, 픽셀 하나를 카메라 프러스텀(절두체)을 따라 뻗은 D개의 3D 점 특징으로 변환.
  - 깊이 GT 없이 학습 가능(깊이가 명시적이지만 **확률적·자기지도적**).
- **Splat (흩뿌리기)** — 카메라 내·외부 파라미터(intrinsics/extrinsics)로 모든 프러스텀 점을 **공유 BEV 격자**에 투영하고, 같은 셀에 떨어진 점들을 **pillar pooling(sum)**으로 합침. (cumulative-sum trick으로 효율화)
- **Shoot (쏘기)** — 원 논문에선 BEV 코스트맵 위로 **후보 궤적(template trajectory)**을 "쏘아" 모션 플래닝까지 수행. (지각+계획 통합)

- **투영 방향**: **Forward / Push (2D→3D)** — 픽셀을 3D로 밀어 올린 뒤 격자에 뿌림.
- **주 용도**: BEV 시맨틱 분할(맵·차량). 파생: BEVDet, **BEVDepth**(명시적 깊이 supervision 추가), BEVFusion, FIERY 등.

## 2. BEVFormer
> Li et al., **ECCV 2022** (Nanjing Univ. / Shanghai AI Lab 등). 트랜스포머 기반 멀티뷰 BEV의 대표작.

격자 모양의 **BEV 쿼리(BEV queries)**를 미리 두고, 두 종류의 attention으로 채운다:
- **Spatial Cross-Attention (SCA, 공간 교차 어텐션)**
  - 각 BEV 쿼리(격자 셀)를 높이 방향으로 몇 개의 **3D reference point**로 확장 → 카메라 파라미터로 각 이미지 뷰에 **투영** → 그 위치 주변에서 **deformable attention**으로 특징을 샘플링해 당겨옴.
  - 한 셀이 자기와 관련된 카메라 뷰들에서만 특징을 골라 취합 → 멀티뷰 융합.
- **Temporal Self-Attention (TSA, 시간 자기 어텐션)**
  - 현재 BEV 쿼리가 **직전 시점의 BEV 특징**을 참조(recurrent)해 과거 정보를 융합 → **속도 추정·가림(occlusion) 대응**에 강함.
- **Deformable Attention** 채택(Deformable DETR 계열)으로 전 픽셀 attention 대비 연산량 절감.

- **투영 방향**: **Backward / Pull (3D→2D)** — BEV 쿼리가 이미지에서 특징을 당겨옴. 명시적 깊이 예측 없음(3D reference point + attention이 대체).
- **주 용도**: 3D 객체 탐지 + 맵 분할. nuScenes에서 당시 카메라 전용 SOTA. 파생: BEVFormer v2, PolarFormer 등.
  - **성능(공식 저장소·논문 확인, 2026-07 검증)**:
    - **nuScenes test set**: **NDS 56.9%**(카메라 전용, 직전 최고 대비 +9.0점) — 강한 백본(V2-99) 기준 mAP 48.1%.
    - **validation** (백본별): BEVFormer-tiny(R50) 35.4/25.2 · small(R101-DCN) 47.9/37.0 · **base(R101-DCN) NDS 51.7% / mAP 41.6%**. *(NDS/mAP 순, 단위 %)*

## 3. 핵심 비교표

| 항목 | **LSS (Lift-Splat-Shoot)** | **BEVFormer** |
|------|----------------------------|----------------|
| 발표 | ECCV 2020 | ECCV 2022 |
| 계보 | Depth-lift 계열의 원조 | Transformer-BEV 계열의 대표 |
| 변환 방향 | **Forward / Push** (2D→3D) | **Backward / Pull** (3D→2D) |
| 깊이 처리 | **명시적** 깊이 분포(확률적) 예측 | **암시적** (3D ref. point + attention, 깊이 예측 없음) |
| 핵심 연산 | 외적 + splat pooling (CNN) | Spatial/Temporal deformable attention (Transformer) |
| 시간 정보 | 원 논문은 단일 프레임(없음) | **Temporal Self-Attention**으로 명시적 융합 |
| 대표 태스크 | BEV 시맨틱 분할 | 3D 객체 탐지 + 분할 |
| nuScenes 성능 | (분할 위주, 탐지 벤치 아님) | test **NDS 56.9%** / base val **51.7 NDS·41.6 mAP** |
| 강점 | 단순·경량·직관적, 깊이 GT 불필요 | 고정밀, 멀티뷰·시간 융합, 가림/속도에 강함 |
| 약점 | 깊이 부정확 시 품질 저하, 프러스텀 메모리 부담 | 구조 복잡, 연산·메모리 무거움, 학습 난이도↑ |
| 파생 | BEVDet, BEVDepth, BEVFusion, FIERY | BEVFormer v2, PolarFormer 등 |

## 4. 한 눈에 보는 직관
- **LSS** = "픽셀아, 네가 얼마나 멀리 있는지 확률로 말해줘 → 그 자리에 특징을 **뿌린다**." (깊이가 주인공)
- **BEVFormer** = "BEV 격자야, 네가 필요한 특징을 이미지 어디서든 **골라와**." (attention이 주인공, 깊이는 숨김)
- 그래서 LSS는 깊이 추정 품질에 성능이 직결되고(→ BEVDepth가 깊이 supervision을 더해 개선), BEVFormer는 attention으로 이 문제를 우회하되 연산 비용을 치른다.

## 5. 본 연구([[HL FMA 2026]])와의 관련성
- 1/5 스케일 차량은 연산 자원(Jetson 등)이 제한적 → **경량성**이 중요. 순수 BEVFormer는 무거울 수 있어, **LSS/BEVDepth 계열의 경량 depth-lift**가 현실적 출발점.
- 카메라만으로 BEV를 얻으면 [[LiDAR]] 의존을 낮추고, 얻은 BEV를 [[W3_Seminar]]의 **경로 계획(Nav2)·로컬 플래너(DWA/TEB)·PID 제어** 입력으로 바로 연결 가능.
- 학습 데이터는 [[KITTI]](단안 위주)보다 **멀티뷰+3D 라벨**이 있는 nuScenes 계열이 BEV 학습에 적합(→ [[W2_Seminar]] 향후계획의 "nuScenes 소개"와 연결).

## 6. 더 볼 것 (후속 정리 예정)
- 🔲 개념 노트 분리: `BEV` · `LSS` · `BEVFormer` · `deformable attention` · `BEVDepth`
- 🔲 논문 리뷰 스텁: LSS(ECCV'20) · BEVFormer(ECCV'22) → 본 문서를 요약본으로 링크
- ✅ 수치 검증(2026-07): BEVFormer nuScenes 수치 공식 저장소 대조 완료 → [출처](https://github.com/fundamentalvision/BEVFormer/blob/master/README.md)
- 🔲 LSS 분할 IoU(nuScenes/Lyft) 수치는 아직 미확인 — 원 논문 대조 필요

## 출처 (References)
**원 논문**
- LSS — J. Philion, S. Fidler, *"Lift, Splat, Shoot: Encoding Images from Arbitrary Camera Rigs by Implicitly Unprojecting to 3D,"* ECCV 2020. https://arxiv.org/abs/2008.05711
- BEVFormer — Z. Li et al., *"BEVFormer: Learning Bird's-Eye-View Representation from Multi-Camera Images via Spatiotemporal Transformers,"* ECCV 2022. https://arxiv.org/abs/2203.17270

**수치 검증** (2026-07-06 확인)
- BEVFormer 공식 저장소 Model Zoo (nuScenes NDS/mAP, 백본별) — https://github.com/fundamentalvision/BEVFormer/blob/master/README.md
- 참고 서베이 — *"Surround-View Vision-based 3D Detection for Autonomous Driving: A Survey,"* arXiv:2302.06650. https://arxiv.org/pdf/2302.06650

## 관련 노트
- [[LiDAR]] · [[KITTI]] · [[YOLO]] — 인지 관련 개념
- [[W2_Seminar]] (KITTI·nuScenes) · [[W3_Seminar]] (경로 계획·제어)
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
