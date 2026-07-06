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

## 5. View Transformation 심층 비교 (Forward vs Backward)

### 5-1. 둘 다 "view transformation" 모듈
- **View transformation(뷰 변환)** = 원근뷰(이미지, PV) 특징을 **BEV로 바꾸는 단계** 그 자체. LSS·BEVFormer **둘 다 이걸 수행**. 서베이의 표준 분류:

| 분류 | 방식 | 대표 |
|------|------|------|
| **Geometry-based (forward, 2D→3D)** | 깊이 추정으로 밀어올림(push) | **LSS** · BEVDet · BEVDepth · CaDDN |
| **Transformer/Attention-based (backward, 3D→2D)** | BEV 쿼리가 attention으로 당겨옴(pull) | **BEVFormer** · DETR3D · PETR |

> ⚠️ **용어 혼동 주의** — "transformer"가 두 뜻으로 쓰임:
> ① **view *transformer*** = 뷰를 변환하는 *모듈*(PV→BEV 기능) ② **Transformer** = attention 쓰는 *신경망 아키텍처*.
> - LSS = 기하(lift-splat)로 변환 → view transformer지만 Transformer 아키텍처는 **안 씀**.
> - BEVFormer = **Transformer 아키텍처로 변환** → view transformer이면서 Transformer도 씀.
> - 참고: BEVDet 구조도의 "view transformer" 컴포넌트는 관례상 **LSS식 lift-splat을 좁게** 지칭하기도 함.

### 5-2. Forward vs Backward 장단점
| 축 | **Forward (LSS)** | **Backward (BEVFormer)** |
|----|-------------------|--------------------------|
| 깊이 | ✅ 명시적(확률분포)·supervision 가능(BEVDepth) | ❌ 명시적 깊이 없음, attention이 대체 |
| 정확도 상한 | ⚠️ **깊이 추정 품질에 종속**(단안 깊이 ill-posed) | ✅ 깊이 병목 회피 → 벤치 정확도 대체로 높음 |
| BEV 격자 채움 | ⚠️ splat이라 **원거리 희박·구멍/중복** | ✅ 모든 칸이 쿼리 → **균일·조밀** |
| 집약 | 픽셀 독립 lift, 전역 추론 없음 | 칸이 여러 뷰·픽셀 능동 취합(문맥적) |
| 연산/메모리 | ✅ 단순·경량·모듈화 쉬움 | ❌ 무겁고 느림·학습 난이도↑ |
| 시간 융합 | 원 논문 없음(별도 설계) | ✅ Temporal Self-Attention |
| 센서 융합 | ✅ 명시적 3D 점 → **LiDAR 정렬 쉬움**(BEVFusion) | ⚠️ 3D 점 없어 추가 설계 필요 |
| 데이터 요구 | 상대적으로 적음 | 트랜스포머라 **data-hungry** |

- **깊이 오류의 성격**: Forward는 깊이를 **한 번 커밋** → 틀리면 특징이 엉뚱한 칸에 박혀 복구 불가. Backward는 깊이 커밋 없이 ray를 따라 여러 3D reference point로 훑어 attention → **깊이 모호성에 강함**(대신 기하가 암시적).
- **커버리지**: Forward는 hole·중복 발생, Backward는 격자 전체가 쿼리라 빠짐없이 채워짐(원거리 유리).

### 5-3. 그래서 나온 결론 — Hybrid
두 방식이 **상보적**이라, **FB-BEV**(Forward-Backward BEV, ICCV 2023)처럼 forward(명시적 깊이·희소 BEV) + backward(빈 곳 채움)를 **결합**한 방법이 등장 → occupancy 챌린지(FB-OCC) 우승. 한쪽의 약점을 다른 쪽이 메움.

## 6. E2E 관점에서의 위치 · BEV 인코더 계보

### "E2E(end-to-end)"의 두 가지 의미
- **의미 A — E2E로 *학습*** (미분가능 단일 네트워크): **LSS·BEVFormer 둘 다 해당.** 이미지→BEV→태스크 헤드가 끊김 없이 한 번에 학습됨.
- **의미 B — E2E *주행*** (센서→계획·제어 한 모델): **둘 다 아님.** 둘은 **인지(Perception) 모듈**. → 이 둘을 부품으로 쓰는 더 큰 시스템이 의미 B를 담당.

### 인지 → 간이 계획 → 완전 E2E 스펙트럼
```
BEVFormer          LSS                       UniAD
(순수 인지)   →   (인지 + 간이 계획)   →   (인지+예측+계획 = 완전 E2E)
```
- **BEVFormer** = 순수 BEV 인지기(3D 탐지·분할). 계획 없음.
- **LSS** = 인지 + **"Shoot"**. Shoot은 *계획 전 단계가 아니라* **간이 계획** — 미리 정한 후보 궤적(templates)을 BEV 코스트맵에 얹어 비용이 최소인 궤적 하나를 **고름**(생성이 아닌 선택). 다른 에이전트 미래 예측 등은 없음.
- **UniAD** (CVPR 2023 Best Paper, OpenDriveLab) = 인지·예측·계획을 **한 네트워크**로 묶어 최종 계획을 목표로 함께 학습하는 **정통 E2E 주행** 모델.

### BEV 인코더로서의 활용 (둘 다 "부품"으로 쓰임)
두 방법 모두 **더 큰 시스템의 BEV 인코더(입력단)**로 재사용된다 — 이게 실제 산업/연구에서의 위치.

| BEV 인코더 | 방식 | 이걸 인코더로 쓰는 대표 상위 시스템 |
|-----------|------|-------------------------------------|
| **BEVFormer** | 트랜스포머, backward | **UniAD** (E2E 주행) |
| **LSS** (lift-splat) | forward | **BEVFusion**(카메라-LiDAR 융합 인지) · **ST-P3**(E2E 비전 주행) · BEVDet · BEVDepth · FIERY |

- **BEVFormer → UniAD**: UniAD가 BEVFormer식 BEV 인코더를 인지 프론트엔드로 사용.
- **LSS → ST-P3**: UniAD의 대칭. **ST-P3**(ECCV 2022)는 lift-splat 계열 BEV 변환을 인지단에 두고 인지→예측→계획을 수행하는 E2E 비전 주행 모델.
- **LSS → BEVFusion**: LSS의 lift-splat으로 카메라를 BEV로 인코딩 후 LiDAR BEV와 융합("LSS 패러다임을 멀티모달로 확장").
- 요약: **LSS의 lift-splat이 더 단순·모듈화**되어 있어 BEV 인코더 building block으로 채택 사례가 더 많은 편.

## 7. 본 연구([[HL FMA 2026]])와의 관련성
- 1/5 스케일 차량은 연산 자원(Jetson 등)이 제한적 → **경량성**이 중요. 순수 BEVFormer는 무거울 수 있어, **LSS/BEVDepth 계열의 경량 depth-lift**가 현실적 출발점.
- 카메라만으로 BEV를 얻으면 [[LiDAR]] 의존을 낮추고, 얻은 BEV를 [[W3_Seminar]]의 **경로 계획(Nav2)·로컬 플래너(DWA/TEB)·PID 제어** 입력으로 바로 연결 가능.
- 학습 데이터는 [[KITTI]](단안 위주)보다 **멀티뷰+3D 라벨**이 있는 nuScenes 계열이 BEV 학습에 적합(→ [[W2_Seminar]] 향후계획의 "nuScenes 소개"와 연결).

## 8. 더 볼 것 (후속 정리 예정)
- 개념 노트 분리: ✅ [[LSS]] · [[BEVFormer]] / 🔲 예정 `BEV` · `deformable attention` · `BEVDepth`
- 🔲 논문 리뷰 스텁: LSS(ECCV'20) · BEVFormer(ECCV'22) → 본 문서를 요약본으로 링크
- ✅ 수치 검증(2026-07): BEVFormer nuScenes 수치 공식 저장소 대조 완료 → [출처](https://github.com/fundamentalvision/BEVFormer/blob/master/README.md)
- 🔲 LSS 분할 IoU(nuScenes/Lyft) 수치는 아직 미확인 — 원 논문 대조 필요

## 출처 (References)
**원 논문**
- LSS — J. Philion, S. Fidler, *"Lift, Splat, Shoot: Encoding Images from Arbitrary Camera Rigs by Implicitly Unprojecting to 3D,"* ECCV 2020. https://arxiv.org/abs/2008.05711
- BEVFormer — Z. Li et al., *"BEVFormer: Learning Bird's-Eye-View Representation from Multi-Camera Images via Spatiotemporal Transformers,"* ECCV 2022. https://arxiv.org/abs/2203.17270

**BEV 인코더 활용 사례 (E2E 계보)**
- BEVFusion — Liu et al., NeurIPS 2022 (LSS 패러다임을 멀티모달 융합으로 확장) — https://arxiv.org/abs/2205.13790
- ST-P3 — Hu et al., ECCV 2022 (lift-splat 계열 BEV의 E2E 비전 주행) — https://arxiv.org/abs/2207.07601
- UniAD — Hu et al., CVPR 2023 Best Paper (BEVFormer 인코더 기반 완전 E2E 주행) — https://arxiv.org/abs/2212.10156
- FB-BEV — Li et al., ICCV 2023 (forward+backward 결합 하이브리드) — https://arxiv.org/abs/2308.02236

**수치 검증** (2026-07-06 확인)
- BEVFormer 공식 저장소 Model Zoo (nuScenes NDS/mAP, 백본별) — https://github.com/fundamentalvision/BEVFormer/blob/master/README.md
- 참고 서베이 — *"Surround-View Vision-based 3D Detection for Autonomous Driving: A Survey,"* arXiv:2302.06650. https://arxiv.org/pdf/2302.06650

## 관련 노트
- [[LiDAR]] · [[KITTI]] · [[YOLO]] — 인지 관련 개념
- [[W2_Seminar]] (KITTI·nuScenes) · [[W3_Seminar]] (경로 계획·제어)
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
