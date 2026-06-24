---
title: "Workspace — ROS2 소스·빌드 관리 디렉터리"
type: concept
tags: [concept, ROS2, 빌드]
status: in-progress
aliases: [워크스페이스, ROS2 Workspace, colcon, ament]
---

← [[00_Home]] | [[MOC_AVP-ROS]]

# Workspace

> **한 줄 정의**
> 하나의 폴더를 지정해 소스 코드와 빌드된 결과물을 관리하는 디렉터리 공간.

## 핵심 개념
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
# echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc  # 자동화
```

- **ROS1 vs ROS2 빌드**: `catkin`/`catkin_make` → **`ament_cmake`/`ament_python` + `colcon build`**.
- 클라이언트 라이브러리: Python `rospy`→**`rclpy`**, C++ `roscpp`→**`rclcpp`**. `package.xml`은 ROS2에서 `format="3"`.

## 본 연구에서의 역할
- 시뮬레이션·실습 패키지를 빌드·실행하는 작업 환경. `ros2_ws` 등. [[W1_Seminar]]

## 관련 노트
- [[Node]]
- [[W1_Seminar]] · [[W2_Seminar]]
