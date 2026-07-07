---
title: "ParkGaussian — 서라운드뷰 3DGS 기반 자율주차 장면 복원 (CVPR 2026)"
type: concept
tags: [concept, rendering, 3D복원, 자율주차, 자율주행]
status: in-progress
aliases: [ParkGaussian, 파크가우시안, ParkRecon3D]
paper: "arXiv:2601.01386"
venue: "CVPR 2026"
repo: "https://github.com/wm-research/ParkGaussian"
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# ParkGaussian

> **한 줄 정의**
> 서라운드뷰 **피시아이(어안) 카메라** 영상에 [[3DGS|3D Gaussian Splatting]]을 적용해 **주차 장면을 고품질 3D 복원**하는 최초의 프레임워크. (Xiaobao Wei et al., **CVPR 2026**, arXiv:2601.01386, 2026-01)

> ✅ **출처 검증 완료 (2026-07-07)** — 발표자료의 "ParkGaussian" 트랙은 실재하는 CVPR 2026 논문으로 확인. [[W3_Seminar]] §7.2/7.4의 L4 퓨처 브릿지 소개와 일치.

## 핵심 개념 (논문 기준)
- **문제 설정**: 주차는 **밀집된 주차 슬롯**과 **GPS 음영(실내/지하)** 환경이라는 고유 난제. → 카메라 서라운드뷰만으로 정밀 3D 복원 목표.
- **입력**: 4개 서라운드뷰 **피시아이** 카메라(캘리브레이션된 외부 파라미터) → 강한 왜곡 하에서 안정적 splatting을 위해 **UT 기반 투영(UT-based projection)** 사용.
- **Slot-aware 복원 전략**: 기존 주차 인지(주차슬롯 검출) 방법을 활용해 **슬롯 영역의 합성 품질**을 강화 → downstream 인지 일관성 보존.
- **벤치마크 ParkRecon3D**: 주차 장면 복원 전용 최초 벤치마크(서라운드뷰 피시아이 + dense 주차슬롯 주석). SOTA 복원 품질 달성.
- **저자**: Xiaobao Wei, Zhangjie Ye, Yuxiang Gu 외 (wm-research).

## 실습 가능성 (⚠️ 2026-07-07 검증)
| 항목 | 상태 |
|------|------|
| 논문 | ✅ 공개 (arXiv:2601.01386, CVPR 2026) |
| **코드** | ❌ **미공개** — repo는 프로젝트 페이지만. README: *"Code and datasets will be released soon"* (2026-02-21) |
| 데이터셋 ParkRecon3D | 🟡 HuggingFace `pipa83165/ParkRecon3D` 공개(2026-03-20), **라이선스 미표기** |
| 요구사양 | ❌ 문서화 없음 (일반적으로 CUDA NVIDIA GPU·고VRAM 필요) |
- **결론**: 코드 미공개로 **현재 직접 재현/실습 불가**. 논문 정독 + 데이터 구조 확인까지만 가능. 실습은 **코드 릴리스 후**로 보류.
- Gaussian Splatting 기법 자체 실습이 목적이면 원조 3DGS(`graphdeco-inria/gaussian-splatting`, 코드 공개)가 현실적 대안.

## 본 연구에서의 역할
- [[HL FMA 2026]] 자율주차 미션의 **장기 로드맵** 후보 — 카메라 중심 BEV/Occupancy 인지에 차세대 렌더링을 접목. 서라운드뷰 피시아이·주차슬롯 특화라 대회 도메인과 정합성 높음.
- 단, **단기 대회 준비(9월 본선)에는 부적합** — 미공개 코드·고사양 GPU·SOTA 난이도. 우선순위는 [[LSS_실행_Runbook|LSS]]·BEVFormer 실습이 먼저.

## 할 일
- [x] 원출처(논문/저장소) 확인 — arXiv:2601.01386 / wm-research/ParkGaussian (2026-07-07)
- [ ] 코드 릴리스 여부 주기적 확인 (repo watch)
- [ ] 논문 정독 후 리뷰 스텁 작성 (`05_Research/01_LitReview`)
- [ ] ParkRecon3D 데이터 구조·라이선스 확인
- [ ] [[3DGS]] 대비 주차 특화 차별점(피시아이 UT 투영·slot-aware) 정리

## 관련 노트
- [[3DGS]] — 기반 기술
- [[NeRF]] · [[Occupancy]] · [[BEV]] — 인접 개념
- [[W3_Seminar]] · [[HL FMA 2026]] · [[MOC_AVP-ROS]]
