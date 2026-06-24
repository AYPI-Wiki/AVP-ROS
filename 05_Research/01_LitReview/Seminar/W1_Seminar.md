---
title: "W1 세미나 — ROS2 기본 개념 (1주차)"
type: seminar-week
tags: [seminar, ROS2, AVP-ROS]
week: 1
date: 2026-05-26
presenters: [윤성웅, 이준하, 육시우, 최민규]
status: done
---

← [[00_Seminar_Home]] | [[Schedule]] | [[00_Home]] | [[MOC_AVP-ROS]]

# 📅 W1 세미나 — ROS2 기본 개념 (1주차)

> **일시** 2026-05-26
> **발표** 윤성웅 · 이준하 · 육시우 · 최민규 (4인 각자 발표)
> **주제** ROS2 개요, 노드/통신 모델, DDS, 워크스페이스, 그리고 자율주행 시뮬레이션 실습

원본 발표자료(`_sources/seminar/`):
- [[2026-05-26_이준하_ROS2 개요 및 노드 개념.pdf]]
- [[2026-05-26_육시우_세미나 1회차 ROS2 기본 개념.pdf]]
- [[2026-05-26_윤성웅_ROS2와 DDS.pdf]]
- [[2026-05-26_최민규_AVP Seminar 1주차.pdf]]

---

## 1. ROS2 개요

- **ROS2 (Robot Operating System 2)**: 로봇 소프트웨어 개발을 위한 오픈소스 미들웨어/프레임워크. HW와 SW를 통합해 로봇 개발을 쉽게 만든다.
- 역사: 2007년 Stanford STAIR 프로젝트의 **Switchyard**(Morgan Quigley)에서 출발.
- 핵심 구성요소: **[[Node]]**(노드), **[[Topic]]**(토픽), **[[Service]]**(서비스), **[[Action]]**(액션).
- 각 노드는 **독립적인 프로세스**로 동작하고, **[[DDS]]**(Data Distribution Service)로 다양한 환경에서 안정적 통신을 보장.

### ROS1 → ROS2 아키텍처 변화 (윤성웅)
| 항목 | ROS1 (과거의 한계) | ROS2 (현재 표준) |
|------|-------------------|------------------|
| 네트워크 아키텍처 | 중앙 집중형 네임 서버 (`roscore`) | P2P 분산형 검색 |
| 전송 프로토콜 | 맞춤형 TCP/UDP | 국제 표준 **DDS** |
| 프로세스 구조 | 프로세스당 단일 노드 | 프로세스당 다중 노드 |
| 임베디드 지원 | 제한적(`rosserial`) | 상용 수준(micro-ROS) |
| 상태 관리 | 없음 | 라이프사이클(Lifecycle) 노드 |

- ROS1의 한계: **단일 장애점(SPOF)** — `roscore` 다운 시 전체 마비 · 불안정 네트워크에서 생존력 부족 · 보안 취약.

---

## 2. 통신 모델 — Topic / Service / Action

| 방식 | 패턴 | 특징 | 용도 |
|------|------|------|------|
| **[[Topic]]** | 비동기 연속 스트림 (Pub/Sub) | 익명 N:N, 빠른 갱신에 최적 | 카메라 이미지(30Hz), 라이다 스캔(10Hz), `/cmd_vel`, IMU, TF |
| **[[Service]]** | 요청·응답 (부메랑) | 동기/비동기 호출 | 짧은 일회성 명령·질의 (그리퍼 열기, 맵 저장, 파라미터 변경, 배터리 잔량) |
| **[[Action]]** | 목표 지향 + 피드백 | Goal/Feedback/Result, 취소 가능 | 오래 걸리는 작업 (Nav2 주행, MoveIt2 팔 제어) |

- **Publisher/Subscriber**: Topic 이름 기준으로 비동기 통신. Publisher=송신, Subscriber=수신.
- **Callback 함수**: 특정 이벤트(메시지 도착 등) 발생 시 자동 실행되는 함수.
- 서비스 호출 시 **동기 호출은 DeadLock 위험** → **비동기(Future + 콜백) 권장** (블로킹 방지).
- 주요 Topic 명령어: `ros2 topic list` / `ros2 topic echo /<topic>` / `ros2 topic info /<topic>`.

---

## 3. 노드(Node) 개념 (이준하)

- 노드 = ROS2를 구성하는 **최소 단위 소프트웨어**, 연산을 수행하는 독립 프로세스. 로봇은 수많은 노드의 집합.
- **Monolithic 방식 vs ROS2 방식**: 단일 SW가 모든 HW를 제어하는 대신, 하드웨어(LiDAR/Camera/IMU/Motor…)마다 독립 노드가 Topic으로 통신.
- 분산 구조의 이점: 모듈화 · 확장성 · 재사용성 · 병렬 처리 · 안정성(한 노드 문제가 전체로 전파 안 됨).
- **패키지(Package)** = 관련 노드와 설정 파일을 담는 폴더: 소스코드(Python/C++), 설정파일(`package.xml`, `setup.py`, `CMakeLists.txt`), 의존성, launch 파일, 메시지 정의.

---

## 4. 워크스페이스 & 빌드 (육시우)

ROS2 [[Workspace]] = 소스코드와 빌드 결과물을 관리하는 디렉터리 공간.

| 폴더 | 역할 |
|------|------|
| `src/` | 내가 작성한 소스코드·패키지 |
| `build/` | `colcon` 빌드 중간 산출물 |
| `install/` | 빌드 완료된 실행 파일 + `setup.bash` |
| `log/` | 빌드/실행 로그 |

핵심 명령어:
```bash
cd ~/my_robot_ws            # 워크스페이스 이동
colcon build               # 빌드
source install/setup.bash  # 환경 불러오기
# 자동화: echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

- 시각화 도구: **rqt**(`rqt_graph`로 노드·토픽 연결 그래프 확인) · **RViz2**(센서/로봇 3D 시각화) · **Gazebo**(물리 시뮬레이션).
- ROS1 vs ROS2 빌드: `catkin`/`catkin_make` → **`ament_cmake`/`ament_python` + `colcon build`**.
- 클라이언트 라이브러리: Python `rospy`→**`rclpy`**, C++ `roscpp`→**`rclcpp`**. `package.xml`은 ROS2에서 `format="3"`.

---

## 5. 자율주행 시뮬레이션 실습 (최민규)

H-Mobility Autonomous Advanced Course 시뮬레이션 기반 실습.

**src 패키지 구성**:
- `camera_perception_pkg` & `lidar_perception_pkg` — 차량의 "눈" (차선 검출, YOLO/라이다 장애물 인식)
- `decision_making_pkg` — 인식 데이터로 경로(Path)·모션 제어
- `interfaces_pkg` — 노드 간 메시지 형식 정의
- `simulation_pkg` — 제어 명령을 Gazebo 차량에 전달 (`driving_sim.launch.py`가 시작점)
- `debug_pkg` — RViz 모니터링

**예제 노드 `lane_info_extractor_node.py`** (→ [[Pub-Sub 예제 lane_info_extractor]] 참고):
```
[YOLOv8 노드] --DetectionArray--> [이 노드] --LaneInfo--> [제어 노드]
                                            --Image-----> [시각화/디버깅]
```
- `detections` 토픽 구독 → 콜백에서 차선 계산 → `yolov8_lane_info` + `roi_image` 토픽 발행.
- QoS 설정(RELIABLE / KEEP_LAST / VOLATILE), `rclpy.spin()`으로 메시지 대기 무한루프.

**[[YOLO]] 객체 탐지**:
- YOLO(You Only Look Once) = 이미지를 한 번만 보고 위치+클래스를 동시 출력하는 1-stage 모델 → 실시간 처리 가능. (R-CNN 등 2-stage 대비 빠름)
- 작동: 그리드 분할(SxS) → 셀별 책임 → 동시 예측(BBox+Confidence+Class) → NMS로 중복 박스 제거.
- `Yolov8Node`는 **LifecycleNode**(configure→activate→deactivate→shutdown)로 무거운 모델 로딩을 안전하게 처리.

**겪은 오류 & 해결**:
- 차가 정지 → 코드 내 유령 공백 제거.
- 차선 미인식·맵 이탈 → `model` 파라미터를 `sim.pt` **절대경로**로 지정.
- 차선 침범 → 스티어링을 ±3으로 낮춤.

**학습 모델(.pt) 제작**: 라벨링 500장 → Ultralytics 설치 → 데이터셋 8:2(train:val) → `data.yaml` 작성 → 학습. (라벨 포맷은 Detection=BBox만, **Segmentation=BBox+Mask** 차이 주의)

---

## 6. 향후 세미나 계획 (최민규 제안)

PinkLAB **"무작정 따라하는 ROS2"** (입문→응용→실전→심화) 기반 학습 계획:

원래 제안은 8주였으나, **세미나는 6주로 확정**(7·8주차 심화과정 제외). 최종 일정은 [[Schedule]] 참고.

| 주차 | 내용 |
|------|------|
| 1주차 | ROS2 개념 및 인지/학습 파트 공부 ← **(이번 주)** |
| 2주차 | 입문 (기본 명령어, 서비스, Topic, Action, 클래스) |
| 3주차 | 응용1 (package·node, 서비스/액션 서버 만들기) |
| 4주차 | 응용2 (파라미터, LOG, RQT, rosbag) |
| 5주차 | 실전1 (OpenCV, Optical Flow node) |
| 6주차 | 실전2 (ROS_DOMAIN_ID, PID·statemachine 주행 제어) |
| ~~7주차~~ | ~~심화1 (Gazebo 활용·실습)~~ — 제외 |
| ~~8주차~~ | ~~심화2 (SLAM, AMCL, Nav2)~~ — 제외 |

---

## 토론 / 질문
- (세미나 중 나온 질문·논의를 여기에 기록)

## 파생 (개념 노트로 분리)
- [[Node]] · [[Topic]] · [[Service]] · [[Action]] · [[DDS]] · [[Workspace]] · [[YOLO]]
- AI 보조 툴: NotebookLM, PaperBanana (발표자료 제작용, 윤성웅 소개)

## 관련 노트
- [[00_Seminar_Home]]
- [[Schedule]]
