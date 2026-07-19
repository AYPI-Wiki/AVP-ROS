---
title: "HL FMA 2026 — 결정: 코드 저장소 구조"
type: decision
tags: [project, 대회, AVP-ROS, 개발전략, 저장소, ADR]
status: proposed
date: 2026-07-19
owner: 이준하
parent: "[[HL FMA 2026 - 개발 전략]]"
---

← [[00_Home]] | [[MOC_AVP-ROS]] | [[HL FMA 2026 - 개발 전략|🛠️ 상위: 개발 전략]]

# 🗂️ 결정: 코드 저장소 구조

> [[HL FMA 2026 - 개발 전략#10. 다음 할 일 — 미결정 결정 액션|개발 전략 §10]]의 **크리티컬 패스 1번** 항목 정리.
> **담당** 이준하(인프라) · **기한** ~7.14 (⚠️ **경과** — D+5) · **상태** 🟡 제안(팀 승인 대기)
> 이 결정은 **개발 착수의 전제**다. 리포가 없으면 P0 셋업(차선추종 PoC·시뮬 환경)을 시작할 수 없다.

---

## 0. 결정 요약 (TL;DR)

| 질문 | 권고 결정 | 근거 |
|------|-----------|------|
| ① 볼트와 별개 개발 리포? | ✅ **별개 리포 신설** | 문서(볼트) ↔ 코드 라이프사이클·도구·무시대상이 다름 |
| ② 리포 내부 구조? | ✅ **단일 colcon 워크스페이스 모노레포** | 4인·10주·미션 간 원자적 변경 → 멀티리포는 과함 |
| ③ 호스팅 | 🔲 **GitHub (조직 계정 권장)** — 확인 필요 | 무료 org·PR·Actions. 계정 주체만 팀 확정 |
| ④ 브랜치 전략 | ✅ **trunk 기반 + 단기 feature 브랜치 + PR** | 소규모·단기 팀에 GitFlow는 과함 |
| ⑤ 대용량(rosbag·모델) | ✅ **리포에서 제외** + 외부 스토리지/LFS | 저장소 비대화·clone 지연 방지 |

> **한 줄 결론**: **볼트는 문서/연구 전용으로 두고, `avp_ros_ws`(가칭) colcon 모노레포를 GitHub에 새로 판다.** rosbag·학습 모델은 리포 밖.

---

## 1. 배경 / 맥락

- 현재 `AVP-ROS`는 **Obsidian 볼트**(문서·논문·세미나·연구 노트)이자 git 저장소다. 여기엔 아직 **ROS2 실행 코드가 없다.**
- 곧 P0(7.7~7.28)에서 **ROS2 워크스페이스 스캐폴딩·센서 드라이버·차선추종 PoC**를 시작한다([[HL FMA 2026 - 개발 전략#5. 개발 로드맵|전략 §5]]). → 코드를 **어디에, 어떤 구조로** 둘지 먼저 정해야 한다.
- 제약: **4인 팀 · 본선까지 ~9주(D-63) · Jetson Orin Nano · ROS2 Humble(colcon)**.
- 역할([[HL FMA 2026 - 개발 전략#6. 역할 분담|전략 §6]])이 인지/계획·제어/인프라로 나뉘어 있어 **패키지 경계 = 역할 경계**로 잡으면 병렬 작업이 쉬워진다.

---

## 2. 결정할 질문 (2 계층)

1. **계층 1 — 볼트 분리 여부**: ROS2 코드를 이 볼트 안에 둘 것인가, 별개 리포로 뺄 것인가?
2. **계층 2 — 코드 리포 내부 구조**: (분리 시) 단일 워크스페이스(모노) vs 패키지별 멀티리포? 브랜치·대용량 파일·CI는?

---

## 3. 옵션 비교 — 계층 1 (볼트 분리)

| 기준 | A. 별개 리포 ✅ | B. 볼트에 코드 포함 | C. 볼트 모노(docs/+src/) |
|------|:--:|:--:|:--:|
| 라이프사이클 분리(문서 vs 코드) | ✅ 명확 | 🔴 뒤섞임 | 🟡 폴더로만 분리 |
| `.gitignore` 상충(볼트 vs build/install/log) | ✅ 각자 | 🔴 충돌 | 🟡 경로별 관리 |
| CI(colcon build·test) 적용 | ✅ 깔끔 | 🔴 문서 커밋마다 트리거 | 🟡 path 필터 필요 |
| Obsidian 성능(코드/빌드산출물 인덱싱) | ✅ 영향 없음 | 🔴 볼트 비대·인덱싱 부담 | 🟡 제외설정 필요 |
| 커밋 히스토리 가독성 | ✅ 코드/문서 분리 | 🔴 섞임 | 🟡 섞임 |
| 신규 팀원 온보딩(코드만 clone) | ✅ 쉬움 | 🔴 볼트 통째 | 🟡 sparse 필요 |
| 초기 설정 비용 | 🟡 리포 1개 신설 | ✅ 없음 | 🟡 구조 설계 |

> **판정: A(별개 리포).** 볼트는 이미 wikilink·Obsidian 설정·연구 자산으로 최적화돼 있고, ROS2 코드는 colcon 빌드 산출물(`build/ install/ log/`)·CI·바이너리를 동반한다. 둘의 무시 규칙·도구·기여 흐름이 근본적으로 달라 **섞으면 양쪽 모두 나빠진다.**

---

## 4. 권고안 상세 — 별개 리포 + colcon 모노레포

### 4.1 리포 레이아웃 (`avp_ros_ws`, 가칭)

```
avp_ros_ws/                     # colcon 워크스페이스 = git 루트
├─ src/
│  ├─ avp_bringup/              # launch·파라미터·전체 기동 (이준하)
│  ├─ avp_msgs/                 # 커스텀 interface(msg/srv/action) (이준하)
│  ├─ avp_drivers/              # camera·lidar·imu·actuator(VESC/MCU) (최민규)
│  ├─ avp_perception/           # lane·trafficlight·stopline·parking_slot·obstacle (최민규)
│  ├─ avp_localization/         # ekf(robot_localization) 오도메트리 융합 (육시우/이준하)
│  ├─ avp_planning/             # mission_manager(FSM)·path_planner(pure pursuit) (육시우)
│  ├─ avp_control/              # pid 종·횡·hill-hold (육시우)
│  ├─ avp_description/          # URDF/xacro·TF (이준하)
│  └─ avp_sim/                  # 시뮬레이터 연동·코스 리소스 (이준하)
├─ config/                      # 튜닝 파라미터 yaml (미션별 세트)
├─ docs/                        # 리포 로컬 문서(README·빌드가이드) — 연구노트는 볼트
├─ .github/workflows/           # CI (colcon build·test·lint)
├─ .gitignore
└─ README.md
```

- **`avp_` 접두사 메타패키지 묶음** = §1 노드 그래프를 그대로 패키지로 사상. 미션 노드(`hill_stop`·`traffic_fsm`·`parking_t`…)는 `avp_planning` 내부 서브모듈/노드로.
- `avp_msgs` **단일 인터페이스 패키지**로 분리 → 순환 의존 방지, 팀 간 계약(contract) 한 곳.

### 4.2 패키지 ↔ 역할 매핑 (병렬 작업 경계)

| 팀원 | 주 패키지 | 근거(§6) |
|------|-----------|----------|
| 최민규(인지) | `avp_perception`, `avp_drivers` | 비전·센서 처리 |
| 육시우(계획·제어) | `avp_planning`, `avp_control`, (`avp_localization`) | 경로·제어·주차 궤적 |
| 이준하(인프라) | `avp_bringup`, `avp_msgs`, `avp_sim`, `avp_description`, CI | 통신·상태머신 골격·로깅·빌드 |
| 윤성웅(총괄) | 인터페이스 정의·통합·`config` 리뷰 | 파트 간 계약·통합 일정 |

> 패키지 경계 = 역할 경계 → **머지 충돌 최소화**. 공유 계약(`avp_msgs`) 변경만 총괄 리뷰 필수.

### 4.3 `.gitignore` (코드 리포용, 볼트와 별개)

```gitignore
# colcon 빌드 산출물
build/
install/
log/
# 데이터·모델 (리포 제외, §4.5)
*.bag
*.db3            # ros2 bag (sqlite3)
*.mcap
*.pt *.pth *.onnx *.engine
data/ rosbags/ models/
# 파이썬·OS
__pycache__/ *.pyc
.DS_Store Thumbs.db
```

### 4.4 브랜치 전략

- **`main` 보호** + **단기 `feat/<pkg>-<요약>` 브랜치 → PR → 스쿼시 머지.**
- PR은 **CI 통과(colcon build) 필수**, 리뷰어 1인(관련 파트 or 총괄).
- **릴리스 태그 규칙** — `v1.0`을 스캐폴딩 베이스라인으로 삼고, 이후 [[HL FMA 2026 - 개발 전략#5. 개발 로드맵|로드맵 §5]] Phase 완료 시점마다 마이너 태깅. 본선 제출 버전만 메이저 승격(`v2.0`).

  | 태그 | 시점 (로드맵 Phase) | 의미 |
  |------|---------------------|------|
  | `v1.0` | — | ✅ **초기 스캐폴딩**(colcon 모노레포, 2026-07-19) |
  | `v1.1-poc` | P0 셋업 | 차선추종 PoC·센서 드라이버 |
  | `v1.2-complete` | P1 완주 기반 | 7미션 상태 전이 동작(저속 무감점) |
  | `v1.3-precision` | P2 정밀 미션 | T자·평행 주차·요철·좌회전 |
  | `v1.4-robust` | P3 강건성 | 오도메트리 브리징·폴백·우천 |
  | `v2.0-race` | P4 속도 최적화 | 랩타임 단축·**본선 제출 버전** |

  > 계획 변경으로 `v1.0`이 스캐폴딩에 선점됨(원안: `v0.1-poc`→`v1.0-complete`→`v2.0-race`). 위 표가 현재 확정 규칙.
- GitFlow(develop/release/hotfix)는 4인·9주엔 **과함** → trunk 기반이 적합.

### 4.5 대용량 파일(rosbag·학습 모델) 정책

- **리포에 커밋 금지** — rosbag은 수 GB, 모델은 바이너리라 clone·CI를 망가뜨린다.
- 저장 위치(택1, §미결정): 팀 공유 드라이브 / NAS / (필요 시) **Git LFS**.
- 리포엔 **경로·다운로드 스크립트·체크섬**만. rosbag 재현은 오프라인 튜닝의 핵심([[HL FMA 2026 - 개발 전략#4. 공통 인프라|전략 §4-5]])이므로 **명명 규칙·인덱스 문서**를 볼트에 둔다.

### 4.6 CI (경량)

- GitHub Actions: **push/PR 시 `colcon build` + `colcon test` + ament_lint.**
- Humble 컨테이너(`ros:humble`) 기반 1개 잡으로 시작 → 무거워지면 캐시·매트릭스 확장.
- 목표는 "**빌드 깨짐 조기 발견**"이지 완벽한 파이프라인이 아님(P0 단계).

### 4.7 볼트 ↔ 코드 리포 연결

- 볼트(문서/연구/런북) = **"무엇을·왜·배운 것"**, 코드 리포 = **"어떻게(실행)"**.
- 볼트 노트에서 코드 리포 **URL·경로를 상대 참조**(예: 런북에서 `avp_ros_ws` 링크). 코드 리포 README에서 볼트 핵심 문서로 역링크.
- [[LSS_실행_Runbook]]·[[BEVFormer_실행_Runbook]] 같은 **연구 트랙 코드**는 대회 코드와 무관 → 별도(또는 각 upstream fork)로 두고 대회 리포에 섞지 않음.

---

## 5. 미결정 / 확인 필요 (팀 승인 시 함께 결정)

- ✅ **호스팅 계정 주체**: GitHub 조직 **`AUTONOMOUS-PCC-Inc`** 확정. 리포: [`avp_ros_ws`](https://github.com/AUTONOMOUS-PCC-Inc/avp_ros_ws).
- ✅ **리포 공개/비공개**: **private** 확정 (2026-07-19).
- ✅ **리포 이름**: **`avp_ros_ws`** 확정 (로컬 생성 완료 2026-07-19).
- 🔲 **대용량 저장소**: 공유 드라이브 vs NAS vs Git LFS 중 택1.
- 🔲 **ROS 배포판 최종 고정**: Humble(전략 권장) 확정 여부 → `.github` CI 이미지에 직결.

---

## 6. 액션 아이템

- [x] **로컬 리포 생성** — `C:\Users\asus\Documents\GitHub\avp_ros_ws`, 이름=`avp_ros_ws` 확정, colcon 모노레포 스켈레톤 초기 커밋 완료 (2026-07-19)
- [x] **§4.1 스켈레톤**: `src/avp_*` 9개 패키지 플레이스홀더 + `.gitignore` + README + `docs/STRUCTURE.md` + CI(`ci.yml`) — 커밋 `0411730`
- [x] **GitHub 원격(private) 생성 + push** — `AUTONOMOUS-PCC-Inc/avp_ros_ws` (조직, private), `main` push 완료, CI 자동 실행 (2026-07-19)
- [ ] **(이준하)** 팀 4인 협업자 초대 (조직 멤버/리포 권한) — ~7.21
- [ ] **(이준하)** `main` 브랜치 보호 규칙 설정(직접 push 금지·PR 필수) — ~7.23
- [ ] **(전원)** §5 미결정 5건 슬랙/회의서 확정 → 이 문서 상태 `proposed → accepted`, [[HL FMA 2026 - 개발 전략#9. 결정 현황|전략 §9]] "결정됨"으로 이동
- [ ] **(윤성웅)** `avp_msgs` 초안 인터페이스(미션 상태·제어 명령·주차 액션) 계약 정의 — ~7.28
- [ ] **(전원)** 각자 clone → `colcon build` 통과 확인(개발환경 표준화) — ~7.28(설명회 전)

---

## 관련 노트
- [[HL FMA 2026 - 개발 전략]] — §9 결정 현황 · §10 다음 할 일 (본 항목의 상위)
- [[HL FMA 2026]] — 대회 개요·규정·일정
- [[Workspace]] · [[Node]] — colcon 워크스페이스·패키지 개념
- [[MOC_AVP-ROS]]
