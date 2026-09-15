# IROL ROS 2 장비·가상환경 운영 구조

[전체 안내로 돌아가기](./README.md)

## 목적

IROL 실험 PC에서는 **장비 연결이 검증된 ROS 환경을 최대한 안정적으로 유지**하면서, 직접 개발한 ROS 패키지와 프로젝트별 Python dependency는 독립된 가상환경으로 분리한다.

프로젝트마다 달라지는 카메라·로봇·센서 설정은 환경을 새로 만들지 않고 **CLI 실행 옵션과 ROS parameter**로 관리한다.

핵심 목표는 완벽한 격리 자체가 아니라, **한 프로젝트의 dependency 변경 때문에 이미 검증된 장비 연결 환경을 다시 수정하지 않도록 하는 것**이다.

---

## 1. 기본 운영 구조

IROL 실험 PC의 공통 기반은 다음과 같이 유지한다.

```text
Ubuntu 22.04
└── ROS 2 Humble
    ├── 검증된 Binary ROS packages
    ├── 직접 개발한 ROS packages + 장비별 Python venv
    └── Project별 Python venv
```

운영 관점에서는 다음 세 계층을 분리해서 생각한다.

1. **공통 시스템 계층**: Ubuntu 22.04 + ROS 2 Humble
2. **장비 연결 계층**: 검증된 Binary ROS package 또는 장비별 custom ROS package
3. **연구 프로젝트 계층**: 프로젝트별 Python venv와 연구 코드

---

## 2. 하드웨어 ROS 패키지는 Binary 우선

가능한 장비는 공식 또는 실험실에서 검증된 **Binary ROS package**를 우선 사용한다.

현재 대상 장비는 다음과 같다.

- UR5e
- Robotiq 2F-85
- OnRobot Vacuum Gripper
- Intel RealSense
- Helios Camera
- F/T Sensor *(도입 예정)*
- Robot Hand *(도입 예정)*

Binary를 우선하는 이유는 단순히 설치가 편하기 때문이 아니다. Source build 중심으로 구성하면 문제가 발생했을 때 다음 항목을 모두 확인해야 할 수 있다.

- package version
- build configuration
- CMake
- SDK
- dependency
- network
- controller setting
- launch parameter

연구실에서는 Python dependency 충돌보다 **실제 장비 연결 오류를 다시 해결하는 비용이 더 큰 경우가 많다.** 따라서 이미 실제 장비에서 검증된 연결 경로를 최대한 보존한다.

> **운영 원칙:** Binary package가 존재하고 실제 장비에서 검증되었다면 해당 경로를 기본으로 사용한다. Source build는 Binary가 없거나 직접 수정·개발이 필요한 경우에만 사용한다.

---

## 3. 직접 개발한 ROS Package는 장비별 독립 가상환경 사용

현재 직접 개발하여 사용하는 ROS package는 다음과 같이 관리한다.

```text
optitrack_env
└── OptiTrack ROS Package

lucid_env
└── LUCID Vision ROS Package
```

OptiTrack과 LUCID Vision은 하나의 공통 venv로 묶지 않고 **각각 독립된 Python 가상환경**으로 관리한다.

이 구조의 목적은 다음과 같다.

- OptiTrack dependency 변경이 LUCID 환경에 영향을 주지 않도록 한다.
- LUCID SDK 또는 pip package 업데이트가 OptiTrack에 영향을 주지 않도록 한다.
- 각 장비 환경을 독립적으로 재현하고 점검할 수 있도록 한다.
- 문제가 발생했을 때 원인 범위를 해당 장비 환경으로 제한한다.

---

## 4. Python 버전 정책

ROS와 직접 통신하는 Python 환경은 **ROS가 사용하는 Python minor version과 동일하게 맞춘다.**

Ubuntu 22.04 + ROS 2 Humble 기준에서는 기본적으로 Python 3.10 계열을 사용한다.

```text
ROS 2 Humble
└── Python 3.10
    ├── optitrack_env     → Python 3.10
    ├── lucid_env         → Python 3.10
    ├── project_A_env     → Python 3.10
    └── project_B_env     → Python 3.10
```

반면 각 환경에 `pip`로 설치되는 Python package의 버전은 서로 달라도 된다.

```text
optitrack_env
├── Python 3.10
└── NumPy / 기타 dependency A

lucid_env
├── Python 3.10
└── NumPy / OpenCV / Vendor dependency B

project_A_env
├── Python 3.10
└── PyTorch / YOLO / NumPy / 기타 dependency C
```

각 프로세스는 Python 객체를 직접 공유하는 것이 아니라 ROS topic, service, action 등의 메시지 인터페이스를 통해 통신한다. 따라서 **환경 간 pip dependency 버전을 동일하게 맞출 필요는 없다.**

단, 하나의 가상환경 내부에 설치된 패키지끼리는 서로 호환되어야 한다.

Python 가상환경과 ROS 2 바이너리 호환성에 대한 기술적인 설명은 [ROS 2에서 여러 Python 가상환경으로 의존성 분리하기](./ros2_python_virtual_environments.md)를 참고한다.

---

## 5. Project도 프로젝트별 독립 가상환경 사용

각 연구 프로젝트는 독립된 Python 가상환경을 가진다.

```text
project_A_env
├── Python 3.10
├── PyTorch
├── YOLO
├── NumPy
└── Project A code

project_B_env
├── Python 3.10
├── 다른 PyTorch / NumPy 구성
└── Project B code
```

목적은 Project A의 dependency를 변경하거나 업데이트했을 때 Project B 또는 장비 연결 환경이 영향을 받지 않도록 하는 것이다.

프로젝트별 venv에는 해당 연구에서 필요한 알고리즘 패키지만 설치하고, 검증된 장비 연결 환경은 가능한 한 수정하지 않는다.

---

## 6. 장비 Node와 Project Node는 ROS로 통신

장비 Node와 Project Node 사이의 경계는 ROS interface로 둔다.

### RealSense 예시

```text
RealSense Binary Node
        │
        │ sensor_msgs/Image
        ▼
      ROS 2
        │
        ▼
Project A Node
(project_A_env)
```

### OptiTrack 예시

```text
OptiTrack Node
(optitrack_env)
        │
        │ Pose / TF
        ▼
      ROS 2
        │
        ▼
Project A Node
(project_A_env)
```

이 구조에서는 RealSense 내부 환경, OptiTrack 환경, Project 환경에서 사용하는 NumPy/OpenCV 등의 버전이 서로 달라도 ROS 메시지를 통해 통신할 수 있다.

환경 간 연결 기준은 Python package 버전이 아니라 다음과 같은 **ROS interface 호환성**이다.

- Topic name
- Message type
- Service interface
- Action interface
- TF frame
- QoS 설정

---

## 7. 프로젝트별 장비 설정은 CLI와 ROS Parameter로 관리

같은 장비를 사용하더라도 프로젝트마다 카메라 파라미터나 로봇 설정이 달라질 수 있다.

예를 들어 다음과 같은 차이가 있을 수 있다.

```text
Project A
├── RealSense exposure = 100
├── FPS = 30
└── UR5e end-effector = Robotiq 2F-85
```

```text
Project B
├── RealSense exposure = 300
├── FPS = 15
└── UR5e end-effector = OnRobot Vacuum
```

이 차이를 위해 별도의 RealSense 환경이나 UR5e 환경을 새로 만들지 않는다.

**동일한 검증된 Binary package를 사용하고, 프로젝트 실행 시 필요한 parameter만 CLI에서 다르게 전달한다.**

개념 예시는 다음과 같다.

```bash
ros2 launch <realsense_package> <launch_file> \
  exposure:=100 \
  fps:=30
```

다른 프로젝트에서는 다음처럼 실행할 수 있다.

```bash
ros2 launch <realsense_package> <launch_file> \
  exposure:=300 \
  fps:=15
```

> 위 명령은 구조를 설명하기 위한 예시다. 실제 package name, launch file, parameter name은 장비별 실행 문서에서 검증된 값을 사용한다.

### 관리 대상별 기준

| 관리 대상 | 관리 방법 |
| --- | --- |
| 장비 ↔ ROS 연결 | 검증된 Binary ROS package |
| 직접 개발 ROS package의 Python dependency | 장비별 독립 venv |
| 연구 알고리즘의 Python dependency | Project별 독립 venv |
| Exposure, FPS, 해상도, Tool 선택 등 | Project별 CLI / ROS parameter |
| 실행 순서와 정상 동작 확인법 | Project 내부 Markdown 문서 |

---

## 8. Project 내부 Markdown을 실행 매뉴얼로 사용

각 Project repository에는 실제 실험 환경을 재현할 수 있는 Markdown 실행 문서를 둔다.

권장 구조는 다음과 같다.

```text
Project-A/
├── README.md
├── RUN.md
└── ...
```

`RUN.md`에는 신규 인원이 **복사해서 실행할 수 있는 수준**으로 실제 명령과 검증 방법을 기록한다.

### Required Hardware

- 사용 Robot
- 사용 End-effector
- 사용 Camera / Sensor

### Required Environment

- Project venv 이름
- 필요한 custom ROS venv
- Python version

### Start Commands

실제 실행 순서대로 작성한다.

1. UR5e 실행 CLI
2. Gripper 실행 CLI
3. RealSense / Helios 실행 CLI와 실제 사용 parameter
4. OptiTrack / LUCID 실행 방법
5. Project 환경 활성화
6. Project Node 실행

### Expected ROS Interfaces

예:

```text
/joint_states
/camera/color/image_raw
/camera/depth/...
/optitrack/...
```

### Validation

다음 내용을 포함한다.

- 각 장비 실행 후 확인할 topic / service / action
- 정상 데이터가 들어오는지 확인하는 명령
- TF 또는 frame 확인 방법
- 로봇 동작 전 확인 사항

### Shutdown

- 종료 순서
- 로봇 및 센서 종료 시 주의사항

이 문서는 프로젝트의 **실험 실행 기준서**로 사용한다. 신규 인원은 기존 담당자의 기억에 의존하지 않고 `RUN.md`의 명령과 검증 절차를 따라 동일한 실험 환경을 재현할 수 있어야 한다.

---

## 9. 전체 구조

```text
               IROL Experiment PC

          Ubuntu 22.04 + ROS 2 Humble
                     │
      ┌──────────────┴──────────────┐
      │                             │
Binary ROS Packages          Custom ROS Packages
      │                             │
UR5e                          optitrack_env
Robotiq 2F-85                 └─ OptiTrack Node
OnRobot Vacuum
RealSense                     lucid_env
Helios                        └─ LUCID Vision Node
F/T Sensor                          │
Robot Hand                          │
      │                             │
      └─────────── ROS 2 ───────────┘
                     │
                     ▼
               Project venv
            PyTorch / YOLO / VLM
            NumPy / OpenCV / etc.
```

정리하면, **장비 연결 환경은 안정성을 우선하고 연구 코드 환경은 유연성을 우선한다.** 두 영역의 경계는 ROS interface로 유지한다.

---

## 10. 최종 운영 원칙

1. **Ubuntu 22.04 + ROS 2 Humble을 공통 기반으로 유지한다.**
2. **하드웨어 ROS package는 가능한 한 검증된 Binary를 우선 사용한다.**
3. **직접 개발한 OptiTrack과 LUCID Vision ROS package는 각각 별도 venv로 격리한다.**
4. **ROS와 직접 통신하는 Python 환경은 ROS와 동일한 Python minor version을 사용한다.**
5. **pip dependency는 각 venv 안에서 독립적으로 관리한다.**
6. **각 Project도 독립된 venv를 사용한다.**
7. **장비와 Project 사이의 통신은 ROS topic / service / action으로 수행한다.**
8. **프로젝트마다 다른 카메라 파라미터와 end-effector 설정은 CLI 또는 ROS parameter로 전달한다.**
9. **각 Project의 실제 CLI, 실행 순서, 사용 장비, 정상 확인 절차는 Project 내부 Markdown에 기록한다.**
10. **새 프로젝트 때문에 검증된 장비 연결 환경을 다시 수정하지 않는 것을 최우선 운영 원칙으로 한다.**
