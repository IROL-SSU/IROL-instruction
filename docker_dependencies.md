# Docker 및 Dependencies 문제 안내

이 문서는 하나의 컴퓨터에서 여러 AI·Robotics 프로젝트를 개발할 때 발생하는 Dependencies 충돌과, 이를 Docker·Conda·uv 등의 가상환경으로 분리하려 할 때 생기는 문제를 설명합니다.

Dependencies 문제는 프로젝트를 다른 컴퓨터로 옮길 때만 발생하는 문제가 아닙니다. 연구실에서는 같은 컴퓨터에 프로젝트 A와 프로젝트 B를 함께 설치했을 때 더 자주 발생합니다. 두 프로젝트가 서로 다른 버전의 Python 패키지나 시스템 라이브러리를 요구하면, 한 프로젝트를 위해 변경한 환경 때문에 다른 프로젝트가 실행되지 않을 수 있습니다.

일반적인 AI 프로젝트에서는 학습·추론 프레임워크만 Conda나 uv 환경으로 분리하면 이 문제를 상당 부분 피할 수 있습니다. 그러나 ROS가 연결되면 ROS 배포판, Ubuntu의 시스템 Python, `apt`로 설치한 바이너리 패키지, 장치 권한, 네트워크, GPU 및 GUI까지 함께 고려해야 합니다. 따라서 Python 가상환경 하나만 만드는 것으로 전체 시스템이 완전히 분리되지는 않습니다.

## 1. Dependencies 충돌의 핵심 원인

**프로젝트 A를 설치한 뒤 기존 프로젝트 B가 실행되지 않는 경우**

먼저 마지막으로 설치하거나 변경한 패키지를 확인합니다. 프로젝트 A와 B가 같은 Python 인터프리터와 같은 전역 패키지 경로를 사용한다면, 한쪽에서 패키지를 설치하거나 버전을 변경한 작업이 다른 쪽에도 그대로 적용됩니다.

예를 들어 프로젝트 A는 특정 라이브러리의 이전 버전을 요구하고 프로젝트 B는 최신 버전을 요구할 수 있습니다. 두 버전을 같은 위치에 동시에 설치할 수 없다면 어느 한쪽에서 import 오류, API 변경 오류 또는 실행 중 동작 오류가 발생합니다. 이 문제는 컴퓨터의 성능 부족이 아니라 두 프로젝트가 하나의 Dependencies 집합을 공유하기 때문에 발생합니다.

이 경우에는 다음 순서로 대응합니다.

1. 각 프로젝트가 요구하는 운영체제, ROS 배포판, Python 인터프리터 및 주요 패키지 버전을 따로 정리합니다.
2. 두 프로젝트가 모두 사용할 수 있는 공통 버전이 있는지 먼저 확인합니다.
3. 공통 버전이 없다면 두 프로젝트가 같은 패키지 경로를 참조하지 않도록 실행 환경과 환경 변수를 분리합니다.
4. 어떤 패키지를 불러오는지 확실하지 않다면 실제 오류 메시지와 import 경로를 확인합니다.

가능한 조합은 프로젝트마다 다르므로 모든 충돌을 해결하는 하나의 명령은 없습니다. 버전 충돌, 바이너리 호환성, 환경 변수 오염 등 실제 오류의 원인을 확인하고 그때그때 대응해야 합니다. 해결한 뒤에는 사용한 버전과 실행 순서를 기록하여 같은 문제가 반복되지 않도록 합니다.

**다른 컴퓨터로 프로젝트를 옮긴 뒤 실행되지 않는 경우**

새 컴퓨터에 필요한 패키지가 없거나 버전이 다르면 동일한 문제가 발생할 수 있습니다. 이 경우 lock file, 환경 명세, Docker image와 같은 방법으로 기존 실행 환경을 재현할 수 있습니다.

그러나 환경을 다른 컴퓨터로 옮기는 문제만 해결해도, 한 컴퓨터에서 여러 프로젝트를 동시에 관리하는 문제는 남습니다. 프로젝트마다 서로 다른 Dependencies를 요구한다면 각 프로젝트가 어느 Python과 패키지 경로를 사용하는지 분리해야 합니다. 이 문서에서 중점적으로 다루는 문제도 이와 같은 동일 컴퓨터 내부의 충돌입니다.

## 2. 일반적인 Python 가상환경과 ROS 환경의 차이

**다른 AI 프로젝트에서는 Conda나 uv가 잘 동작했는데 ROS 프로젝트에서는 오류가 발생하는 경우**

학습이나 추론만 수행하는 프로젝트는 대부분 PyTorch, NumPy, Transformers와 같은 Python 기반 프레임워크만 분리하면 됩니다. 이 경우 Conda나 uv로 Python 인터프리터와 패키지를 프로젝트별로 나누는 방식이 효과적입니다.

반면 `apt`로 설치한 ROS 2 바이너리는 해당 ROS 배포판이 지원하는 Ubuntu의 시스템 Python을 기준으로 빌드되어 있습니다. ROS 공식 문서도 미리 빌드된 ROS 바이너리를 사용할 때에는 가상환경의 Python 인터프리터가 빌드에 사용된 시스템 인터프리터와 일치해야 한다고 안내합니다. Conda는 별도의 Python 인터프리터를 사용하는 경우가 많으므로 ROS 바이너리와 호환되지 않을 가능성이 큽니다. (ROS 2 Python 패키지 안내)

따라서 연구실의 기본 ROS 구성에서는 `/usr/bin/python3`를 사용하는 ROS 환경을 기준으로 두고, Conda의 Python과 한 프로세스 안에서 혼용하지 않습니다. ROS Python 노드만이 아니라 ROS CLI, 빌드 결과, C++ 라이브러리, middleware 및 환경 설정도 함께 연결되어 있기 때문에 Python 패키지만 분리해서 문제가 모두 해결된다고 생각해서는 안 됩니다.

가상환경별 특징은 다음과 같습니다.

| 방법 | 주로 분리하는 대상 | ROS와 함께 사용할 때 확인할 점 |
| --- | --- | --- |
| `venv`·uv | Python 패키지와 프로젝트 환경 | 시스템 Python을 기반으로 만들었는지, ROS 바이너리와 인터프리터가 일치하는지 확인해야 합니다. |
| Conda | Python 인터프리터, Python 패키지 및 일부 네이티브 라이브러리 | Conda의 Python이 ROS 바이너리의 시스템 Python과 다르면 import와 바이너리 호환성 문제가 발생할 수 있습니다. |
| Docker | 사용자 공간의 파일 시스템, 패키지, 프로세스 및 네트워크 공간 | USB, GPU, 네트워크, GUI 등 호스트의 실제 자원을 별도로 연결해야 합니다. |

즉, Conda나 uv가 잘못된 도구라는 뜻은 아닙니다. ROS와 직접 연결되지 않는 학습·추론 코드를 분리하는 데에는 적합하지만, ROS가 같은 프로세스의 Python 모듈과 인터프리터를 직접 사용하기 시작하면 단순한 가상환경 분리가 어려워집니다.

## 3. Docker Container를 사용할 때 발생하는 문제

Docker는 프로그램과 Dependencies를 image 안에 고정하여 프로젝트 간 충돌을 줄일 수 있습니다. 그러나 container는 기본적으로 호스트의 파일 시스템, 장치, 네트워크 및 화면 표시 환경과 격리됩니다. 센서와 로봇을 직접 연결하는 개발에서는 이 격리를 해제하기 위한 설정이 추가로 필요합니다.

### 3.1 USB 및 Serial Port 문제

**호스트에서는 USB 센서가 보이지만 container 안에서는 보이지 않는 경우**

호스트의 일반 파일을 공유할 때에는 bind mount나 volume을 사용하지만, USB·Serial 장치는 보통 `--device`와 같은 device mapping으로 접근 권한을 부여합니다. Docker container는 기본적으로 호스트 장치에 접근할 수 없기 때문입니다. (Docker Container 실행 문서)

어떤 장치를 사용할지는 대체로 container를 생성하는 시점의 실행 설정 또는 Docker Compose 설정에 기록합니다. 센서를 추가하거나 장치 경로가 달라졌는데 기존 설정에 해당 장치가 없다면, 설정을 수정한 뒤 container를 다시 생성해야 할 수 있습니다. 따라서 센서 구성이 자주 바뀌는 개발 단계에서는 장치를 변경할 때마다 container 구성을 함께 수정하고 재생성해야 하는 불편이 생깁니다.

다음 순서로 대응합니다.

1. 호스트에서 센서가 정상적으로 인식되는지 먼저 확인합니다.
2. container에 필요한 장치만 명시적으로 mapping했는지 확인합니다.
3. 장치의 읽기·쓰기 권한과 container 내부 사용자의 권한을 확인합니다.
4. 센서 구성이 바뀌었다면 Docker Compose 또는 실행 설정을 수정하고 container를 재생성합니다.

문제를 빠르게 피하기 위해 `--privileged`로 호스트의 모든 장치와 넓은 권한을 넘기는 방법은 사용 범위와 보안 영향을 충분히 검토해야 합니다. 특정 장치만 필요한 경우에는 필요한 자원만 명시하는 구성을 우선합니다.

### 3.2 GPU 문제

**호스트에서는 GPU를 사용하지만 container 안에서는 CUDA 또는 GPU를 찾지 못하는 경우**

Docker container는 NVIDIA GPU를 기본으로 할당받지 않습니다. 호스트에 NVIDIA Driver와 NVIDIA Container Toolkit/Runtime을 구성하고 container 실행 시 GPU 접근을 허용해야 합니다. NVIDIA Container Toolkit은 Docker가 호스트의 NVIDIA GPU와 필요한 드라이버 라이브러리를 container에 연결할 수 있게 합니다. (NVIDIA Container Toolkit)

GPU 연결 자체가 다소 불안정하며, Robotics 학습 환경에서는 다음 구성 요소의 버전이 모두 맞아야 하므로 네이티브 실행보다 문제를 확인할 지점이 늘어납니다.

- 호스트 NVIDIA Driver
- NVIDIA Container Toolkit/Runtime
- CUDA를 사용하는 container image
- Isaac Sim 및 IsaacLab 버전
- PyTorch와 학습·추론 코드

학습 또는 추론 중 GPU 오류가 발생하면 container 내부의 Python 패키지만 반복해서 재설치하지 않습니다. 먼저 호스트 GPU가 정상인지 확인하고, 그다음 container에서 GPU가 노출되는지, 마지막으로 Isaac Sim·IsaacLab과 CUDA 구성의 호환성을 확인할 필요가 있습니다.

### 3.3 Network 문제

**container에서 로봇이나 센서에 연결되지 않거나 인터넷을 사용할 수 없는 경우**

Docker의 네트워크 모드는 목적에 따라 동작이 다릅니다. (Docker Network Driver 안내)

- `bridge`는 기본적인 container 네트워크로, 가상 Ethernet interface와 NAT를 사용합니다. 일반적인 외부 통신에는 편리하지만, 로봇 검색에 필요한 broadcast·multicast나 외부 장치의 직접 접근에서 추가 설정이 필요할 수 있습니다.
- `host`는 container가 호스트의 네트워크를 직접 사용하게 합니다. 네트워크 격리는 줄어들지만 ROS 통신이나 호스트와 동일한 interface가 필요한 경우에는 구성이 단순해질 수 있습니다.

학교망에서는 bridge 모드에서는 다른 NET 기반 센서와 연결이 불가능해지며, host 모드에서는 네트워크를 할당 받지 못하는 이슈가 있습니다.

### 3.4 CPU 가상화 문제

**Docker를 설치하기 위해 BIOS 가상화를 켠 뒤 IsaacLab 오류가 발생하는 경우**

Docker를 사용하려면 BIOS/UEFI의 hardware virtualization이 필요합니다. 해당 장비에서 Docker와 IsaacLab의 구성이 충돌한다면, IsaacLab은 검증된 Ubuntu native 환경에서 실행하고 Docker 사용 범위를 별도로 제한할 필요가 있습니다.

### 3.5 Display 문제

**container에서 Isaac Sim이나 RViz를 실행했지만 창이 나타나지 않는 경우**

Linux Desktop의 GUI는 주로 X11의 X Server를 통해 표시되며, 최신 환경에서는 Wayland를 사용할 수도 있습니다. Docker container는 호스트의 Display Server에 기본적으로 접근할 수 없습니다. GUI를 표시하려면 `DISPLAY` 환경, X11 socket, 인증 정보와 접근 권한 등을 container에 전달해야 합니다. 제공된 Docker GUI 실행 참고 자료도 X Server 권한과 X11 socket, GPU 연결을 함께 설명합니다.

Display 연결과 GPU 연결은 서로 다른 문제입니다. X11 forwarding이 되어 창을 만들 수 있더라도, NVIDIA GPU가 container에 연결되지 않았다면 Isaac Sim과 같은 GPU 기반 프로그램은 정상적으로 렌더링하지 못할 수 있습니다. 반대로 GPU는 보이지만 X Server 접근 권한이 없으면 GUI 창을 표시할 수 없습니다.

별도의 가상 Display Server를 만들면 단순한 GUI 프로그램은 실행할 수 있지만, 그 Display가 GPU rendering을 제공하지 않으면 Isaac Sim을 실질적으로 사용하기 어렵습니다. 다음 순서로 구분하여 확인합니다.

## 4. Docker가 Robotics 개발 트러블 슈팅

Docker는 Robotics에 사용할 수 없는 도구가 아닙니다. 실제로 ROS와 IsaacLab도 container 구성을 제공합니다. 다만 Docker의 주된 장점은 프로그램과 Dependencies가 정해진 상태에서 동일한 app을 반복 실행하고 배포하는 데 있습니다.

하지만, Robotics 개발 중에는 USB 센서가 추가되거나, 네트워크 interface가 바뀌거나, 로봇의 IP 구성이 변경되고, GPU와 GUI를 직접 확인해야 하는 일이 자주 발생합니다. 이때 container의 격리는 다음과 같은 추가 작업을 만듭니다.

- 새 USB 장치와 권한을 container 설정에 반영해야 합니다.
- GPU Driver와 NVIDIA Container Runtime의 호환성을 함께 관리해야 합니다.
- 로봇 통신 방식에 맞는 network mode를 다시 선택해야 합니다.
- X11·Wayland와 GPU rendering을 container에 연결해야 합니다.

따라서 센서와 네트워크 구성이 계속 바뀌는 초기 개발 단계에서는 Docker가 문제 해결 지점을 늘릴 수 있습니다. 반대로 하드웨어 구성과 Dependencies가 확정된 app의 배포, 재현 가능한 headless 학습, CI 작업에는 Docker가 유용할 수 있습니다. 연구실에서는 Docker 사용 여부를 관성적으로 결정하지 말고 현재 작업이 개발 단계인지, 구성이 고정된 배포 단계인지 구분해야 합니다.

## 5. 가상환경으로 Dependencies 문제를 완전히 해결하기 어려운 이유

**문제 1: 같은 컴퓨터에서 서로 다른 프로젝트가 같은 ROS 환경을 사용하는 경우**

ROS 프로젝트는 해당 배포판의 시스템 Python과 설치된 ROS 패키지를 함께 사용합니다. 프로젝트 A와 B가 서로 다른 Python Dependencies를 요구해도 두 프로젝트가 `/usr/bin/python3`와 같은 패키지 경로를 공유하면 충돌할 수 있습니다.

먼저 두 프로젝트를 동시에 만족하는 패키지 버전을 찾습니다. 불가능하다면 프로젝트별 ROS workspace와 실행 터미널을 분리하고, 각 프로젝트를 실행할 때 필요한 workspace만 source합니다. 환경 변수를 조정하여 다른 프로젝트의 패키지를 잘못 참조하지 않게 할 수도 있지만, 이미 설치된 바이너리와 Python 경로의 조합에 따라 오류 형태가 달라집니다. 결국 실제 code error와 import 경로를 확인하면서 대응해야 하며, 모든 프로젝트에 적용되는 단일 해법은 없습니다.

**문제 2: 문제 1을 해결하려고 가상환경을 도입하면 ROS 외부 자원 문제가 생기는 경우**

Conda와 uv는 Python 환경을 분리할 수 있지만, ROS 바이너리가 요구하는 시스템 Python과 다르면 호환성 문제가 생길 수 있습니다. Docker는 사용자 공간을 더 넓게 분리할 수 있지만 USB, GPU, network, Display를 다시 연결해야 합니다. 즉, 도구마다 격리할 수 있는 범위와 새로 발생하는 문제가 다릅니다.

다른 AI 프로젝트에서 Conda나 uv를 사용해 문제가 없었던 이유는 대개 학습·추론 framework만 가상환경 안에서 실행했기 때문입니다. ROS를 import하지 않고, 실제 센서·로봇 통신을 별도 프로세스가 담당한다면 이러한 분리가 가능합니다. 그러나 같은 프로세스가 ROS와 학습 framework를 모두 사용하면 ROS의 시스템 Python 조건과 AI package의 버전 조건을 동시에 만족해야 하므로 구성이 어려워집니다.