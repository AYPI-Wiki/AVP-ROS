---
title: "W3 세미나 — 서비스 구현 · 경로 계획과 제어 (3주차)"
type: seminar-week
tags: [seminar, ROS2, AVP-ROS]
week: 3
date: 2026-06-30
presenters: [이준하, 육시우]
status: done
---

← [[00_Seminar_Home]] | [[Schedule]] | [[00_Home]] | [[MOC_AVP-ROS]]

# 📅 W3 세미나 — 서비스 구현 · 경로 계획과 제어 (3주차)

> **일시** 2026-06-30
> **발표** 이준하 · 육시우 (2인 발표)
> **주제** 커스텀 인터페이스(msg/srv/action)와 Service 통신 구현, 전통적 SfM+MVS 3D 재구성, 경로 계획 알고리즘(Dijkstra·A*), ROS2 Nav2 구조, 로컬 경로 계획(DWA·TEB), PID 제어

> ⚠️ **계획 대비 변경** — [[Schedule]] 상 3주차는 "응용 1(package·node, 서비스/액션 서버)"였으나, 실제 발표는 서비스 구현에 더해 SfM+MVS(3D 재구성)와 경로 계획·PID 제어(원래 5·6주차 실전 파트)까지 앞당겨 다룸.

원본 발표자료(`_sources/seminar/`):
- [[2026-06-30_이준하_ROS2_서비스.pdf]]
- [[2026-06-30_육시우_경로_계획과_제어.pdf]]

---

## 1. 커스텀 인터페이스 & Service 통신 구현 (이준하)

### 커스텀 메시지·서비스의 필요성
- 표준 메시지(`std_msgs` 등)만으로는 의미 있는 데이터를 묶어 표현하기 어려움 → **커스텀 인터페이스**로 도메인 특화 데이터 구조를 직접 정의.
- **IDL로 한 번 정의하면 Python·C++ 등 언어별 코드가 자동 생성** → 서로 다른 언어로 작성한 노드끼리도 같은 메시지 교환 가능.

### ROS2 인터페이스 3종
| 인터페이스 | 파일 | 성격 | 대응 통신 |
|-----------|------|------|-----------|
| 메시지 | `.msg` | 단방향·비동기 (응답 없음) | [[Topic]] |
| 서비스 | `.srv` | 요청/응답, 동기 | [[Service]] |
| 액션 | `.action` | 목표·피드백·결과, 장기 작업 | [[Action]] |
- 빌드 시 **`rosidl`**이 각 인터페이스를 언어별 소스코드로 변환.

### [[Topic]] vs [[Service]]
| 구분 | Topic | Service |
|------|-------|---------|
| 통신 방식 | 단방향 (1→N) | 양방향 (1:1) |
| 동기성 | 비동기 (연속 게시) | 동기 (요청 후 응답 대기) |
| 인터페이스 | `.msg` | `.srv` (`---`로 요청·응답 구분) |
| 사용 예 | 센서 데이터 스트림 | 값 계산·상태 변경 |

### 커스텀 `.srv` 정의 & 빌드
- `.srv`는 `---` 위가 **요청(Request)**, 아래가 **응답(Response)**. `srv/` 폴더에 저장(예: `srv/AddTwoInts.srv`).
  ```
  int64 a
  int64 b
  ---
  int64 sum
  ```
- 패키지 생성 & 빌드:
  ```bash
  ros2 pkg create --build-type ament_cmake cpp_srvcli \
    --dependencies rclcpp example_interfaces   # 의존성 자동 추가
  colcon build --packages-select cpp_srvcli
  source install/setup.bash                     # rosidl 빌드로 .srv → 코드 변환
  ```

### Service 통신 동작 원리 & 구현
- 클라이언트가 요청 → 서버가 처리 후 응답 반환 (**동기 1:1**).
- **서버**: `create_service(타입, 이름, 콜백)`로 등록 → 콜백`(request, response)`에서 처리 → `spin()`으로 대기.
- **클라이언트**: `create_client(타입, 이름)` → `wait_for_service`로 준비 확인 → `call_async()`로 비동기 호출 → `future`로 응답 수신.
- Python(`rclpy`)·C++(`rclcpp`) 구현 절차 동일.
- 실행 & CLI 테스트:
  ```bash
  ros2 run cpp_srvcli server
  ros2 run cpp_srvcli client 2 3        # a=2, b=3 → sum=5 응답
  ros2 service list / ros2 service call /add_two_ints ... / ros2 interface show
  ```

## 2. 전통적 SfM + MVS — 이미지 기반 3D 재구성 (이준하)

여러 장의 2D 사진에서 카메라 포즈와 3D 형상을 복원하는 파이프라인:

```
입력 이미지 → SfM (COLMAP) → Dense MVS → 메시 (Poisson)
```

- **SfM(Structure from Motion)** — 희소(sparse) 복원:
  - 특징 검출·매칭(SIFT 등) → **증분식 등록**으로 카메라 내·외부 파라미터(포즈) 추정 → 삼각측량으로 희소 3D 점 생성 → **Bundle Adjustment**(재투영 오차 최소화, 전역 최적화).
  - 출력: 카메라 포즈 + 희소 점군 → MVS 입력.
- **Dense MVS** — 조밀(dense) 복원:
  - 왜곡 보정(Undistortion) → **PatchMatch Stereo**(이미지별 깊이맵·법선맵, 신뢰 시드에서 깊이 가설 전파) → **Stereo Fusion**(일관성 검사 후 3D 융합, `fused.ply`).
  - 연산량 커 **GPU(CUDA) 가속 강력 권장**.
- **Poisson 메시 재구성**:
  - 조밀 점군의 위치+법선으로 연속 표면 복원. **Screened Poisson**은 빈 영역도 메워 watertight 표면에 적합.
  - 주요 파라미터: `depth`(옥트리 최대 깊이 — 클수록 디테일↑·메모리↑), `point_weight`(점 보간 가중치). 대안: Delaunay meshing(CGAL) + QEM 단순화·텍스처 매핑.
- **데이터 흐름 요약**: `이미지 → database.db`(feature_extractor·matcher) → `sparse/`(mapper+BA) → 보정 이미지 → depth/normal maps(image_undistorter→patch_match_stereo) → `fused.ply`(stereo_fusion) → `meshed.ply`(poisson_mesher).

## 3. 경로 계획 알고리즘 — 그래프 탐색 (육시우)

- **경로 계획** = 로봇이 시작점(Start)→목표점(Goal)까지 **충돌 없이 가장 효율적인** 경로를 찾는 과정.
  - 전역(Global): 전체 지도 기반 최적 경로 탐색 / 지역(Local): 주변 장애물 회피 중심 실시간 경로 보정.
- 경로 계획 문제는 보통 **그래프**로 모델링: 노드(위치)·엣지(연결)·비용(거리/시간).

### Dijkstra
- 모든 노드까지의 **최소 비용 경로** 탐색. BFS 기반, 누적 비용 최소화.
- 절차: ① 시작 노드 거리 0, 나머지 ∞ → ② 미방문 중 최소 거리 노드 선택 → ③ 이웃 거리 갱신(더 작으면 갱신) → ④ 전 노드 방문까지 반복.
- 장점: 음이 아닌 비용에서 **항상 최적 경로 보장** / 단점: 목표가 멀거나 불필요한 탐색 많음.

### A*
- Dijkstra + **휴리스틱**(목표 방향 추정치). 목적지 방향을 우선 탐색.
- `f(n) = g(n) + h(n)` — `g`: 시작점→현재 실제 비용(Dijkstra 개념), `h`: 현재→목표 추정 비용(휴리스틱).

| 항목 | Dijkstra | A* |
|------|----------|-----|
| 탐색 기준 | 누적 거리 | 거리 + 휴리스틱 |
| 속도 | 느림 | 빠름 |
| 정확도 | 항상 최적 | 휴리스틱에 따라 달라짐 |
| 응용 | 지도 전체 탐색 | 자율주행·SLAM·게임 AI |

## 4. ROS2 Navigation Stack (Nav2) 구조 (육시우)

로봇이 지도 상에서 스스로 **위치 추정 → 경로 계획 → 목표 이동**하도록 하는 ROS2 패키지.

| 구성 요소 | 노드 | 역할 |
|-----------|------|------|
| Localization | `amcl` | Particle Filter 기반 위치 추정 |
| Global Planner | `nav2_planner` | 전역 경로 계산 (A*, Dijkstra) |
| Local Planner | `nav2_controller` | 장애물 회피 중심 실시간 경로 수정 |
| Costmap | `costmap_2d` | 이동 가능 영역·장애물 정보 지도화 |
| BT Navigator | `bt_navigator` | Behavior Tree 기반 자율주행 흐름 제어 |
| Lifecycle Manager | `lifecycle_manager` | Nav2 노드 상태 관리 |

- **Behavior Tree(BT) 기반 구조** — 순차 실행이 아닌 논리 트리로 동작:
  ```
  NavigateToPose
  ├── ComputePathToPose   # 경로 생성
  ├── FollowPath          # 경로 추종(이동)
  └── RecoveryBehavior    # 실패 시 복구(제자리 회전·후진 후 재계획)
  ```
- 주요 토픽/서비스: `/plan`(전역 경로), `/cmd_vel`(속도 명령), `/amcl_pose`(추정 위치), `/map`(SLAM/로드된 지도), `/costmap`(이동 비용 지도).

## 5. 로컬 경로 계획 — DWA · TEB (육시우)

- **로컬 경로** = 전역 경로와 달리 실시간 상황 변화에 대응하는 즉각적 경로. Local Planner 역할: 속도 명령(`cmd_vel`) 계산 + 충돌 없는 단기 경로 생성.
- **DWA (Dynamic Window Approach)**: 속도 공간 `(v, ω)`에서 가능한 조합을 탐색해 다음 순간 최적 속도 선택.
  - 입력: 현재 속도·목표 위치·장애물 / 출력: 최적 선속도 `v`·각속도 `ω`.
  - 평가 함수: `G(v,ω) = α·heading + β·distance + γ·velocity` (목표 방향 일치 / 장애물 거리 / 속도 크기).
- **TEB (Timed Elastic Band)**: 경로 전체를 "시간을 포함한 곡선"으로 모델링, 시간·속도·장애물 제약을 동시 최적화. DWA보다 부드럽고 정확, 실제 물리 제약 반영. 패키지 `teb_local_planner`.
- 설정: `nav2_params.yaml`의 `controller_server → ros__parameters → FollowPath → plugin` 수정으로 로컬 플래너 변경.

## 6. PID 제어 (육시우)

목표값(Setpoint)과 실제값(Feedback)의 차이인 **오차(error)**를 비례(P)·적분(I)·미분(D) 항으로 조합해 제어 입력 계산.

- **P (비례, 현재 오차)**: 오차에 비례해 즉각 반응 → 목표 근처로 빠르게 도달. 한계: 목표 근처에서 힘이 줄어 **정상상태 오차(steady-state error)** 잔존.
- **I (적분, 과거 오차 누적)**: 쌓인 미세 오차를 모두 더해 큰 힘 생성 → 장기적 오차 제거(목표값에 정확히 정착).
- **D (미분, 미래 변화 예측)**: 오차 변화율을 보고 미리 억제 → **오버슈트(Overshoot) 방지**, 댐퍼(완충) 역할로 부드러운 정착.

---

## 7. 향후 계획
- [[Schedule]] 기준 다음은 **W4 응용 2(파라미터·LOG·RQT·rosbag)** → 실제로는 이번 주에 실전 파트(경로·제어)를 일부 선행했으므로 순서 조정 논의 필요.

## 토론 / 질문
- (세미나 중 나온 질문·논의 기록)

## 파생 (개념 노트로 분리)
- ✅ 생성됨: [[Service]] · [[Topic]] · [[Action]] · [[Node]] · [[Parameter]]
- 🔲 예정(개념 노트 미생성): `SfM` · `MVS` · `COLMAP` · `Poisson 재구성` · `Nav2` · `Behavior Tree` · `Costmap` · `AMCL` · `Dijkstra` · `A*` · `DWA` · `TEB` · `PID`

## 관련 노트
- [[W2_Seminar]]
- [[00_Seminar_Home]]
- [[Schedule]]
