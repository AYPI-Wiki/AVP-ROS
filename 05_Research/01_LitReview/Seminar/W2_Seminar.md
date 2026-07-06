---
title: "W2 세미나 — 통신 모델 · 센서 데이터 처리 (2주차)"
type: seminar-week
tags: [seminar, ROS2, AVP-ROS]
week: 2
date: 2026-06-23
presenters: [이준하, 윤성웅, 육시우, 최민규]
status: done
---

← [[00_Seminar_Home]] | [[Schedule]] | [[00_Home]] | [[MOC_AVP-ROS]]

# 📅 W2 세미나 — 통신 모델 · 센서 데이터 처리 (2주차)

> **일시** 2026-06-23
> **발표** 이준하 · 윤성웅 · 육시우 · 최민규 (4인 발표)
> **주제** 노드/토픽 통신 심화, ROS2↔AUTOSAR 통신 대응, 센서(Camera/LiDAR/IMU) 데이터 처리, KITTI 데이터셋, turtlesim 기반 CLI 실습(기본 명령어·Service·Topic·Action), 그리고 출전 대회 소개

원본 발표자료(`_sources/seminar/`):
- [[2026-06-23_이준하_노드구조 및 토픽통신.pdf]]
- [[2026-06-23_윤성웅_ROS2 통신과 KITTI 데이터셋.pdf]]
- [[2026-06-23_육시우_ROS2 센서 데이터 처리.pdf]]
- [[2026-06-23_최민규_기본 명령어, Service, Topic, Action.pdf]]

---

## 1. 노드 구조 & 토픽 통신 (이준하)

- **[[Node]] 복습**: ROS2 최소 단위 SW, 독립 프로세스. 분산 구조(모듈화·확장성·재사용성·병렬·안정성). 패키지 = 노드+설정파일 묶음.
- **[[Topic]] 통신 — 작동 원리**:
  - 노드 간 데이터를 주고받는 가장 기본적인 **단방향** 통신, Publisher/Subscriber 모델.
  - Publisher: Topic Name으로 메시지를 지속 발행 / Subscriber: Topic Name 구독 후 **콜백**으로 처리.
  - **1:N, N:1, N:N** 통신 지원 — 한 토픽을 여러 노드가 발행/구독 가능.
- **AVP 시뮬레이션 노드 그래프** (H-Mobility-Autonomous-Advanced-Course-Simulation):
  ```
  Gazebo --scan--> Lidar_processor --Lidar_processed--> Obstacle detector --Obstacle_info--┐
  Gazebo --image--> Yolov8_node --detections--> Lane extractor --Lane_info--> Path planner --Path_result--> Motion planner --control--> Simulation sender --> Gazebo
                                 --detections--> Traffic detector --Traffic_info--------------------------------┘
  ```
  - 패키지: `camera_perception_pkg`(YOLOv8·Lane·TrafficLight), `lidar_perception_pkg`, `decision_making_pkg`(Path·Motion planner), `interfaces_pkg`(커스텀 메시지), `simulation_pkg`(Gazebo, ego/obstacle/traffic light 로드).

## 2. ROS2 통신 ↔ AUTOSAR 통신 (윤성웅)

ROS2 통신 3종을 AUTOSAR 인터페이스에 대응시켜 정리:

| ROS2 | 성격 | Classic AUTOSAR | Adaptive AUTOSAR (ara::com) |
|------|------|-----------------|------------------------------|
| **[[Topic]]** | 단방향 Pub/Sub (data만) | SRI (Sender-Receiver, data+event+mode) | Event |
| **[[Service]]** | 양방향 요청/응답 (RPC) | CSI (Client-Server) | Method (Future로 비동기) |
| **[[Action]]** | 복합 (목표+피드백+취소) | 단일 대응 없음 → CSI(goal/result/cancel)+SRI(feedback/status) | Method+Event+취소 Method |
| **[[Parameter]]** | 노드 튜닝 손잡이/설정 저장소 | — | Field (상태값+get/set/notify) |

- Action 내부 구성: Goal/Result/Cancel = Service, Feedback/Status = Topic. → 장기 실행 작업용 상위 계층 패턴.

## 3. 센서 데이터 처리 (육시우)

### Camera + [[OpenCV]]
- OpenCV = 오픈소스 컴퓨터 비전 라이브러리 (이미지 처리·객체 검출·특징점 매칭·모션 추적·캘리브레이션). YOLO/PyTorch 전처리에 활용.
- ROS2 카메라 영상은 `sensor_msgs/msg/Image` (header/height·width/encoding/data). **cv_bridge**로 cv2 이미지 ↔ ROS Image 변환.
- 실습: `camera_subscriber_pkg` 생성 → `/camera/image_raw` 구독 → `imgmsg_to_cv2`로 변환 후 `cv2.imshow`. (⚠️ 영상 출력은 실패 — 노트북/카메라 연동 문제 추정)

### [[LiDAR]] (`sensor_msgs/LaserScan`)
- LiDAR = 레이저 반사로 거리 측정. 필드: `angle_min/max/increment`, `ranges`(거리 배열), `intensities`.
- 확장 실습: `min_distance = np.min(ranges)` → `if min_distance < 0.5: warn(...)` → 실제론 속도 제어 토픽을 강제 조정해 급정거에 활용.

### [[IMU]] (`sensor_msgs/Imu`)
- IMU = 기울기·회전·가속도 측정. 필드: `orientation`(Quaternion), `angular_velocity`(Vector3), `linear_acceleration`(Vector3).

## 4. KITTI 데이터셋 (윤성웅)

- **KITTI** = 2012년 개발된 대표 자율주행 데이터셋 (cvlibs.net/datasets/kitti). 카메라+Velodyne LiDAR+GPS/IMU.
- 데이터 구분: **RAW(synced+rectified)** = 날짜별 drive, GPS/IMU 포함 / **Odometry** = 시퀀스(00~21), 6-DoF pose(00~10), GPS/IMU 없음.
- 실습: rosbag(`kitti2bag`)으로 RViz에서 LiDAR 포인트클라우드+카메라 영상 시각화.
- KITTI 활용 자율주행 **VLA 논문**: ① Reasoning-VLA (2025, Chain-of-Causation) ② Impromptu-VLA (2025, 비구조 시나리오 4분류) ③ CoVLA (2025).

## 5. 🏁 출전 대회 — HL FMA 2026 (육시우)

> 이 세미나/볼트의 실제 목표 대회. 상세는 [[HL FMA 2026]] 참고.

- **HL FMA (Future Mobility Award) 2026** 자율주행 경진대회. 주최: HL만도·HL클레무브·한국도로교통공단 / 주관: 한라대 SW중심대학사업단.
- 우리 출전 부문: **HL FMA [1/5]** (전국 대학생, 팀당 6인 이하, AI 자율주행 1/5 스케일 차량).
- 일정: 참가접수 6.4~7.12 · 대회설명회 7.28 · **자율연습 9.5~9.6 · 예선 9.19 · 본선 9.20**.
- 차량 규정: 1/5 스케일, 전장 1500·전폭 800·전고 550mm 이내, 24V 이하, 센서(LiDAR·카메라·IMU·초음파) 자유[300만원 이하], 연산장치(Jetson 등) 자유. 모델 3종(T870/T8 sports·F8·기타) 중 택1.
- 평가: 자동차 운전면허시험 기준 + 특정 미션. 참가자 상호 점검으로 공정성 확보.

---

## 6. turtlesim 기반 CLI 실습 — 기본 명령어 · Service · Topic · Action (최민규)

> 1주차 계획상 "2주차 입문" 핵심. `turtlesim`을 대상으로 ROS2 CLI를 직접 두드려 보며 4대 통신/실행 단위를 손에 익히는 실습 파트.

### 기본 터미널 명령어 — 환경 설정
- **`.bashrc`**: 새 터미널을 열 때마다 위→아래로 자동 실행되는 홈 디렉토리 숨김 설정 파일.
  - 매번 `source /opt/ros/humble/setup.bash`를 치는 대신 `.bashrc` 맨 아래에 적어두면 터미널 열 때 ROS2가 자동 준비됨. (`code ~/.bashrc`로 편집)
- **`alias`**: 긴 명령어를 짧은 단어로 등록. `alias 이름="값"` (등호 양옆 붙여쓰기 필수).
  - 예: `alias humble="source /opt/ros/humble/setup.bash; echo \"ROS2 humble is activated\""` → `humble` 입력만으로 ROS2 활성화 (환경 충돌 방지용).

### 기본 명령어 — Node 실행
- **[[Node]]** = ROS 세계에서 실행 가능한 최소 단위(일 하나를 맡는 실행 단위). 실행: `ros2 run <PKG> <Node>`.
- turtlesim 실습: 설치 `sudo apt install ros-humble-turtlesim` → 실행 `ros2 run turtlesim turtlesim_node`.
- 노드 확인: `ros2 node list` (→ `/turtlesim`), `ros2 node info /turtlesim` (Sub/Pub·Service·Action 목록 일괄 확인).

### [[Service]] — 요청/응답 (1:1)
- Client가 Server에 **요청(request)** → Server가 수행 후 **응답(response)**.
- 조회: `ros2 service list` / 타입 `ros2 service type <srv>` / 정의 `ros2 interface show <type>`.
  - `srv` 정의는 `---`로 request/response 구분. 예) `TeleportAbsolute` = request `float32 x,y,theta` / response 없음. (**ROS는 각도를 radian 선호**)
- 호출: `ros2 service call <서비스명> <타입> "<값>"`
  ```bash
  ros2 service call /turtle1/teleport_absolute turtlesim/srv/TeleportAbsolute "{x: 2, y: 2, theta: 1.57}"
  ros2 service call /reset std_srvs/srv/Empty {}              # 위치 초기화
  ros2 service call /spawn turtlesim/srv/Spawn "{x: 1, y: 1, theta: 0, name: ''}"  # 거북이 추가
  ```
  - spawn 후 `service list` 재확인 시 `/turtle2/...` 서비스가 추가 생성 → 거북이별 개별 제어 가능.

### [[Topic]] — 단방향 연속 스트림 (Pub/Sub)
- 대상 없이 Publish하는 노드와, 그 토픽을 구독하는 노드들. 토픽 = **이름 붙은 통로**, 여러 노드가 동시에 붙을 수 있음.
- 조회: `ros2 topic list` (`-v` Pub/Sub 분리, `-t` 타입 표기) / `ros2 topic type <topic>` / `ros2 topic info <topic>` (타입·Publisher/Subscription count).
- 구독: `ros2 topic echo /turtle1/pose` (`--once` 1회만). `rqt_graph`로 `/turtlesim → /turtle1/pose → 터미널` 흐름 확인.
- 발행: `ros2 topic pub`
  ```bash
  ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist \
    "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"
  ```
  - `/turtle1/pose` = `turtlesim/msg/Pose` (x,y,theta + linear/angular velocity).
  - `/turtle1/cmd_vel` = `geometry_msgs/msg/Twist` = linear(이동) + angular(회전) 3D 벡터. **turtlesim은 `linear.x`·`angular.z`만 사용.**

### [[Action]] — 목표 + 피드백 + 결과 (장기 작업)
- Server가 action 수행, Client가 목표 요청 → 중간과정을 **feedback**으로 흘려보내며 최종 **result** 보고.
- 내부 구성: **목표·결과·취소 = Service**(수락/거부 1:1 응답 필요), **피드백·상태 = Topic**(응답 없이 계속 발송).
- 실습: `turtlesim_node` + `turtle_teleop_key`로 화살표 키 주행 → `ros2 action list -t`로 `/turtle1/rotate_absolute [turtlesim/action/RotateAbsolute]` 확인.
  - `interface show`상 3구분: 목표 `theta` / 결과 `delta` / 진행 `remaining` (모두 radian).
- 목표 전송: `ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 3.14}"` → `Goal accepted` → `Result: delta...` → `status: SUCCEEDED`.

---

## 7. 향후 계획 (윤성웅 제안)
1. 토픽 프로그래밍
2. 차량용 데이터셋 소개 (KITTI, nuScenes 등)
3. 논문 리뷰 — NeRF (Representing Scenes as Neural Radiance Fields)

## 토론 / 질문
- (세미나 중 나온 질문·논의 기록)

## 파생 (개념 노트로 분리)
- ✅ 생성됨: [[OpenCV]] · [[LiDAR]] · [[IMU]] · [[KITTI]] · [[Parameter]]
- 🔲 예정: `cv_bridge` (개념 노트 미생성) · VLA 논문 3종 → `01_Papers/`로
- [[HL FMA 2026]] (대회)

## 관련 노트
- [[W1_Seminar]]
- [[00_Seminar_Home]]
- [[Schedule]]
