# 연구실 소프트웨어 및 장비 설치·운용 유의사항

설치 및 설정 변경 전에는 사용 중인 장비와 소프트웨어의 버전을 확인하고, 해당 버전의 공식 문서를 참고해야 합니다. 기존 데이터와 설정은 사전에 백업하며, 공용 장비의 연결 구성과 로봇의 안전 설정은 담당자 확인 없이 변경하지 않습니다.

## 1. Ubuntu

**파티션 설정**
Ubuntu 설치 시에는 설치 대상 디스크와 파티션을 반드시 확인합니다. 설치용으로 확보한 빈 공간 또는 파티션을 사용하고, 기존 운영체제나 연구 데이터가 저장된 영역을 실수로 삭제하거나 포맷하지 않도록 주의합니다.

**NVIDIA 그래픽 드라이버 및 CUDA·cuDNN**
그래픽 드라이버와 필요한 CUDA·cuDNN은 GPU 사양, 운영체제, 사용 프로그램의 요구사항을 확인한 후 설치합니다. 자동으로 추천되는 버전이나 최신 버전을 그대로 선택하지 말고, 구성 요소 간 호환성을 검토해야 합니다. 특히 Isaac Sim·Isaac Lab은 사용하는 버전에 맞는 드라이버 요구사항을 확인해야 하며, 지원되지 않는 조합에서는 정상적으로 실행되지 않을 수 있습니다. ([NVIDIA Docs][1])

**USB 장치 접근 권한**
USB 장치는 연결만으로 통신 준비가 완료되는 것은 아닙니다. 특히 `/dev/ttyUSB*`, `/dev/ttyACM*`로 인식되는 시리얼 장치에서 접근 오류가 발생하면 사용자에게 필요한 읽기·쓰기 권한이 있는지 확인합니다. 권한 설정은 장치 소유 그룹, `dialout` 그룹, 장치별 `udev` 규칙 등을 통해 관리하며, `chmod 777`로 모든 사용자에게 광범위한 권한을 부여하는 방식을 기본 절차로 사용하지 않습니다. ([Arduino 도움 센터][2])

**여러 USB 장치의 동시 사용**
여러 센서를 동시에 연결한 상태에서 통신이 불안정하면 포트와 허브의 전원 공급, 대역폭, 케이블 상태를 함께 점검합니다. 연결 포트를 변경하거나 센서를 하나씩 연결하여 문제가 발생하는 구성을 확인합니다.

## 2. ROS

**운영체제와 ROS 버전**
Ubuntu 버전에 대응하는 ROS 배포판을 설치합니다. 예를 들어 Ubuntu 22.04에서는 ROS 2 Humble의 공식 설치 절차를 사용할 수 있습니다. ([ROS Docs][3])

**실행 환경 설정**
ROS 설치 후에는 사용할 터미널에서 환경 설정 파일을 불러와야 합니다. ROS 2 Humble을 사용하는 경우 다음 명령을 실행합니다.

```bash
source /opt/ros/humble/setup.bash
```

Bash 기반 터미널에서 이를 자동으로 적용하려면 `~/.bashrc`에 해당 명령을 추가합니다. 자체 워크스페이스를 사용하는 경우에는 그 워크스페이스의 환경 설정도 불러와야 합니다. 여러 ROS 배포판이나 워크스페이스의 설정을 동시에 적용하지 않도록 주의합니다. ([ROS Docs][4])

**워크스페이스 및 패키지 관리**
프로젝트별로 워크스페이스를 분리하고, 패키지 소스는 각 워크스페이스의 `src/` 디렉터리에 배치하는 것을 원칙으로 합니다. `apt`로 설치한 패키지와 동일한 이름의 패키지를 소스로 추가할 때에는 의도적인 대체인지 확인해야 합니다. 불필요한 중복 구성을 피하고, 소스 버전으로 대체하는 경우에는 적용되는 워크스페이스와 환경 설정 순서를 명확히 관리합니다. ([ROS Docs][5])

**rosdep을 이용한 의존성 설치**
워크스페이스를 빌드하기 전에는 `rosdep`을 초기화하고 패키지 의존성을 설치합니다. `rosdep init`은 시스템에서 최초 한 번만 수행하며, 이미 초기화된 환경에서는 다시 실행하지 않고 인덱스만 갱신합니다. 이후 워크스페이스 루트에서 다음과 같이 `src/` 아래 패키지의 의존성을 확인하고 설치합니다.

```bash
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

의존성 설치를 생략한 채 빌드 오류를 소스 코드 문제로 판단하지 않습니다. 설치 전에는 사용 중인 ROS 배포판의 환경 설정이 올바르게 적용되어 있는지 확인합니다.

**빌드 오류 대응**
빌드가 실패하면 오류 메시지를 먼저 확인하고, 의존성 누락과 `package.xml`, `CMakeLists.txt`, `setup.py` 등의 설정을 점검합니다. 노드와 스크립트의 실행 항목이 올바르게 등록되어 있는지도 확인합니다. ([ROS Docs][6])

이전 빌드 결과나 캐시가 원인으로 의심되면 필요한 오류 로그를 보관한 뒤 `build/`, `install/`, `log/` 등 생성된 디렉터리를 정리하고 다시 빌드합니다. **`src/`를 제외한 모든 파일을 일괄 삭제하는 방식은 사용하지 않습니다.** 별도로 보관한 설정 파일과 데이터가 삭제되지 않도록 대상 경로를 확인해야 합니다.

필요한 실행 환경과 의존성이 갖춰진 Python 스크립트는 직접 실행하여 문제를 진단할 수도 있습니다. 다만 이는 임시 확인 방법이며, 패키지 구성이나 빌드 설정의 문제를 해결하는 절차를 대신하지는 않습니다.

**빌드 결과와 실제 실행 코드 확인**
빌드 성공 여부와 정상 실행 여부는 별도로 확인해야 합니다. 일반적인 ROS 2 패키지 실행에서는 `src/`의 원본 파일이 아니라 설치된 실행 파일과 리소스를 사용하므로, 수정한 내용이 실제 실행 환경에 반영되었는지 확인합니다. `--symlink-install`은 일부 소스 및 리소스의 변경 반영에 도움이 되지만, 모든 변경에서 재빌드가 불필요해지는 것은 아닙니다. ([ROS Docs][7])

**RViz 표시 문제와 좌표계**
토픽을 추가했는데 데이터가 표시되지 않으면 토픽 수신 상태, Display 설정, 메시지의 `frame_id`, RViz의 `Fixed Frame`을 확인합니다. 센서 좌표계와 표시 기준 좌표계가 다르면 두 좌표계 사이의 유효한 TF 변환이 필요합니다. 예를 들어 `world` 좌표계에서 센서 데이터를 확인하려면 실제 장착 관계에 맞는 변환을 제공해야 합니다. 단순히 프레임 이름만 변경하거나 임의의 변환을 넣어 문제를 해결하려고 해서는 안 됩니다. ([ROS Docs][8])

## 3. Anaconda

**ROS와 Python 환경의 분리**
ROS 환경을 먼저 구성하고 정상 동작을 확인한 후 Anaconda 환경을 구성하는 것을 기본 절차로 합니다. 다만 설치 순서만으로 충돌이 방지되지는 않습니다. ROS 설치 경로 아래에는 해당 배포판에서 사용하는 Python 패키지가 포함되어 있으며, 각 ROS 배포판은 대응하는 Ubuntu의 시스템 Python 인터프리터와 버전을 기준으로 구성됩니다. 따라서 `apt`로 설치한 ROS 바이너리와 Conda의 Python 인터프리터 및 라이브러리를 하나의 실행 환경에서 혼용하지 않습니다. ROS를 빌드하거나 실행하는 터미널에서는 Conda 환경을 활성화하지 않고, 실제로 선택된 Python 인터프리터와 라이브러리 경로를 확인합니다. ([ROS Docs][9])

Conda 환경의 프로그램과 ROS를 반드시 연동해야 하는 경우에는 두 환경을 별도 프로세스 또는 별도 호스트로 분리하고 TCP endpoint 등의 통신 인터페이스를 통해 연결할 수 있습니다. 다만 통신 계층과 장애 지점이 추가되므로 연구실의 기본 구성으로 권장하지 않으며, 불가피한 경우에만 인터페이스와 데이터 형식을 명확히 정의하여 사용합니다.

**프로젝트별 가상환경 관리**
프로젝트별로 독립적인 가상환경을 생성하여 사용합니다. 서로 다른 프로젝트의 라이브러리를 하나의 환경에 무분별하게 설치하지 않으며, 사용 중인 환경과 주요 라이브러리 버전을 명확히 관리합니다.

**설치 및 이용 조건 확인**
Anaconda의 무료 이용 범위와 유료 서비스는 적용 약관 및 학술 이용 조건을 확인하여 구분합니다. 설치 과정에 표시되는 유료 서비스 안내를 모두 필수 결제 항목이나 단순 후원 요청으로 판단하지 않습니다. ([Anaconda][10])

**저장 공간과 모델 캐시 관리**
사용하지 않는 가상환경과 불필요한 캐시는 주기적으로 정리합니다. 다만 Hugging Face 모델은 가상환경과 별도의 캐시 경로에 저장될 수 있으며, 같은 캐시 경로에 접근하는 환경에서는 재사용할 수 있습니다. 가상환경이 다르다는 이유만으로 모델을 반드시 다시 다운로드해야 하는 것은 아닙니다. 공유 캐시를 삭제하기 전에는 다른 프로젝트에서 사용 중인지 확인합니다. ([Hugging Face][11])

## 4. UR 로봇

**네트워크 연결 및 IP 설정**
로봇과 연결된 PC의 네트워크 인터페이스를 확인하고, 로봇과 제어 PC가 서로 통신할 수 있도록 IP와 네트워크 설정을 구성합니다. UR 로봇의 IP는 외부 호스트의 네트워크 구성에 맞추어 할당되거나 DHCP 서버를 통해 할당되므로, PolyScope의 IP 설정은 외부 네트워크에서 정한 주소 체계와 할당 값에 의존합니다. PolyScope에서 임의의 값을 먼저 정하지 않으며, DHCP 또는 고정 IP 사용 여부와 서브넷 구성을 확인한 후 적용합니다. **로봇의 IP와 제어 PC의 IP를 혼동하지 않도록 주의합니다.** ROS 드라이버 실행 시에는 로봇의 IP를 지정하고, 티치 펜던트의 External Control 설정에는 드라이버가 실행되는 PC의 IP를 입력합니다. 케이블을 분리하여 연결 인터페이스를 확인해야 한다면 로봇을 정지시킨 상태에서 수행합니다. ([유니버설 로봇 문서][12])

**External Control 프로그램 확인**
External Control URCap은 ROS bringup을 통해 ROS Controller로 로봇을 제어하는 구성에 사용합니다. 이 구성에서는 티치 펜던트의 프로그램에 External Control 노드가 포함되어 있어야 하며, 해당 프로그램이 실행 중인지 확인해야 합니다. 네트워크 연결이 정상인데 로봇이 동작하지 않으면 프로그램 실행 상태와 연결 설정을 함께 점검합니다. 확인되지 않은 프로그램 블록을 임의로 추가하지 않습니다. ([유니버설 로봇 문서][12])

**제어 방식 전환**
ROS bringup은 External Control URCap과 ROS Controller를 사용하는 제어에 활용하고, Remote Mode는 별도의 RTDE 기반 제어에 활용합니다. 두 방식을 전환하여 사용할 때에는 각 프로그램이 요구하는 로봇 실행 모드와 설정을 확인합니다. 기존 제어 프로그램의 종료 여부도 확인하여 의도하지 않은 동시 제어를 방지합니다. 

ROS bringup 패키지는 일반적인 운용에서는 코드 베이스를 직접 내려받아 빌드하기보다 해당 ROS 배포판에 맞는 바이너리 패키지로 설치하는 구성이 안정적입니다. 소스 수정이 반드시 필요한 경우에만 호환되는 브랜치와 의존성을 확인한 후 별도의 워크스페이스에서 빌드합니다.

**안전 설정 및 동작 제약**
로봇의 안전 제한과 소프트웨어의 동작 제약은 기능과 영향을 충분히 이해한 상태에서만 변경합니다. 관련 설정을 임의로 완화하거나 해제하지 않으며, 변경이 필요한 경우 담당자의 검토를 거칩니다.

**URCap 설치 및 업데이트**
URCap을 추가하거나 업데이트하기 전에는 로봇의 PolyScope 버전, 사용 중인 드라이버, 기존 URCap과의 호환성을 확인합니다. 최신 버전이라는 이유만으로 업데이트하지 않으며, 변경 전 기존 프로그램과 설치 설정을 백업합니다. ([유니버설 로봇 문서][12])

## 5. MoveIt

**설치 방식**
일반적인 사용에서는 해당 ROS 배포판에 맞는 `apt` 패키지 설치를 우선합니다. 소스 수정이나 특정 기능이 필요한 경우에는 공식 소스 빌드 절차를 따르고, 사용하는 브랜치와 의존성 버전을 확인합니다. GitHub에서 소스를 내려받는 방식 자체가 잘못된 것은 아니지만, 설치 구성을 직접 관리해야 한다는 점을 고려해야 합니다. ([MoveIt][13])

**Robot Description Package 및 구성 생성**
MoveIt 구성에는 대상 로봇의 URDF·SRDF와 관련 리소스를 제공하는 Robot Description Package가 의존성으로 필요합니다. 해당 패키지의 로봇 모델과 관절 구성이 실제 장비 및 드라이버 구성과 일치하는지 먼저 확인합니다. 

**문제 해결 및 문서 확인**
MoveIt에서 문제가 발생하면 공식 문서와 오류 로그를 바탕으로 설정과 동작 구조를 확인합니다. AI 코딩 도구가 제시한 코드나 설정을 검토 없이 적용하지 않으며, 사용자가 해당 설정의 역할과 변경 영향을 이해한 상태에서 수정합니다.

## 6. RealSense

**USB 연결 상태 확인**
USB 3.x 포트에 연결했더라도 실제로 어떤 USB 규격으로 인식되는지 확인합니다. RealSense Viewer에서 장치 정보와 영상 출력 상태를 확인하고, 필요한 해상도와 프레임률을 사용할 수 있는 연결 상태인지 점검합니다. USB 2.x로 인식되는 경우에는 사용할 수 있는 스트림 설정이 제한될 수 있습니다. ([RealSense Help Center][14])

**연결 오류 대응**
장치가 인식되지 않거나 연결이 반복적으로 끊기면 재연결, 다른 USB 포트 사용, 케이블 교체, PC 재부팅 순으로 기본 사항을 점검합니다. 케이블의 커넥터 형태만으로 USB 3.x 지원 여부를 판단하지 말고 사양을 확인합니다. 문제가 지속되면 SDK·펌웨어 조합과 오류 로그를 확인하여 원인을 좁혀갑니다.

**컬러 영상과 깊이 영상의 정렬**
컬러 영상과 깊이 영상을 함께 사용할 때에는 SDK의 정렬 기능을 적용합니다. 두 영상의 대응 관계를 단순히 일정한 픽셀 수만큼 이동시키는 방식으로 처리하지 않습니다. 예를 들어 깊이 영상을 컬러 영상 기준으로 사용할 때에는 `rs.align(rs.stream.color)`와 같은 기능을 활용할 수 있습니다. ([GitHub][15])

## 7. 그리퍼 및 로봇 핸드

### Robotiq

**물리적 연결 및 통신 구성**
로봇 말단의 툴 커넥터를 이용하는 구성에서는 그리퍼의 연결 상태와 통신 설정을 확인합니다. 티치 펜던트에서 필요한 URCap과 설정은 사용하는 제어 방식에 따라 확인해야 합니다. 특히 Robotiq URCap과 툴 통신 중계용 URCap의 조합은 호환성을 확인한 후 구성합니다. ([GitHub][16])

**전원 및 포트 설정**
ROS 드라이버에서 툴 전원을 설정하는 경우 `tool_voltage` 등의 항목을 확인하고, 그리퍼 매뉴얼에 명시된 정격 전압을 적용합니다. 툴 통신을 PC의 시리얼 장치로 중계하는 구성에서는 생성된 장치 경로와 Robotiq 실행 패키지의 포트 설정을 일치시켜야 합니다. bringup 실행만으로 필요한 전원과 통신 설정이 모두 완료되었다고 가정하지 않습니다. ([GitHub][16])

### OnRobot

로봇과 연동하는 방식 또는 별도 컨트롤러를 통해 PC에 연결하는 방식 중, 사용 중인 모델이 지원하는 구성을 확인합니다. 연구실에서 사용한 PC 연결 구성을 참고하되, 전원 공급장치와 통신 인터페이스는 해당 장비의 매뉴얼을 기준으로 확인합니다. 기억에 의존하여 배선하거나 전원을 연결하지 않습니다.

### 로봇 핸드

세부 설치 및 운용 절차는 장비 도입 후 검증 결과를 바탕으로 보완합니다. 초기 설정 시에는 전원, 통신 인터페이스, SDK, 구동 범위 및 안전한 초기 자세를 우선 확인합니다.

## 8. OptiTrack

**운용 PC 및 라이선스**
연구실의 Motive 운용용 Windows PC를 사용합니다. 설치할 Motive 버전이 보유 라이선스의 지원 범위에 포함되는지 확인하고, USB 라이선스 키가 정상적으로 연결되어 있는지 점검합니다.

**캘리브레이션 및 마커 설정**
측정 전에는 캘리브레이션 상태를 확인하고 필요한 경우 다시 수행합니다. 작업 편의를 위해 2인 수행을 권장하며, 구체적인 절차는 공식 매뉴얼과 교육 영상을 참고합니다.

캘리브레이션 후에는 측정 목적에 맞게 마커와 Rigid Body를 구성합니다. 연구실에서는 주로 Rigid Body 기반 추적을 사용하며, 개별 마커 데이터가 필요한 경우에는 해당 데이터의 출력 및 처리 방법을 별도로 확인합니다.

**네트워크·전원 연결 및 문제 해결**
PoE를 사용하는 카메라 구성에서는 이더넷 케이블이 데이터 통신과 전원 공급을 함께 담당하므로, 지정된 스위치와 연결 구성을 확인합니다. 케이블과 포트를 임의로 변경하지 않으며, 문제가 발생하면 연구실에 보관된 OptiTrack 연구노트와 기존 해결 사례를 우선 참고합니다.

**ROS 연동 네트워크 구성**
OptiTrack을 ROS와 연동할 때에는 Motive를 실행하는 Windows Host PC와 ROS를 실행하는 Host PC가 각각 필요하며, 두 PC를 동일한 Subnet Mask의 네트워크에 구성합니다. 연구실 구성에서는 Windows에서 TCP 방식으로 같은 서브넷의 ROS endpoint에 접근하므로, 양쪽 PC의 IP 주소, Subnet Mask, 방화벽 및 endpoint 포트를 함께 확인합니다. 서로 다른 서브넷에 연결하거나 주소만 임의로 변경하면 endpoint에 접근할 수 없으므로 기존 네트워크 구성을 먼저 확인합니다.

## 9. Helios

**SDK 및 측정 조건**
장비에 맞는 SDK를 설치하고 기본 영상·깊이 데이터의 수신 상태를 확인합니다. 렌즈와 측정 거리 조건을 고려하여 설치하며, 설치 위치를 변경하거나 이동하면서 사용할 경우에는 해당 조건에서 측정 품질을 별도로 검증합니다. 로봇이나 다른 센서와 데이터를 함께 사용하는 경우에는 필요한 좌표계 보정도 수행합니다.

**네트워크 및 전원 연결**
연구실에서는 장비용으로 지정된 PCIe 네트워크 카드의 포트를 사용합니다. 다만 연결 가능한 인터페이스와 전원 공급 방식은 Helios의 세부 모델 및 구성에 따라 다르므로, 다른 포트로 변경하기 전에는 해당 모델의 요구사항을 확인해야 합니다. 이더넷 연결 여부만으로 전원 공급까지 충족된다고 판단하지 않습니다. ([루시드 지원센터][17])

PCIe 카드는 메인보드의 확장 슬롯에 장착되는 장치입니다. 장비용 카드와 연결 포트를 구분하기 어려운 경우에는 임의로 연결하지 말고 담당자에게 확인합니다.

[1]: https://docs.nvidia.com/deeplearning/cudnn/installation/latest/prerequisites.html?utm_source=chatgpt.com "Prerequisites — NVIDIA cuDNN Installation"
[2]: https://support.arduino.cc/hc/en-us/articles/360016495679-Fix-port-access-on-Linux "Fix port access on Linux – Arduino Help Center"
[3]: https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html?utm_source=chatgpt.com "Ubuntu (deb packages) — ROS 2 Documentation"
[4]: https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html?utm_source=chatgpt.com "Configuring environment — ROS 2 Documentation: Humble ..."
[5]: https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html?utm_source=chatgpt.com "Creating a workspace — ROS 2 Documentation"
[6]: https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html?utm_source=chatgpt.com "Creating a package — ROS 2 Documentation: Humble documentation"
[7]: https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html?utm_source=chatgpt.com "Using colcon to build packages — ROS 2 Documentation"
[8]: https://docs.ros.org/en/humble/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html?utm_source=chatgpt.com "RViz User Guide — ROS 2 Documentation: Humble documentation"
[9]: https://docs.ros.org/en/humble/How-To-Guides/Using-Python-Packages.html?utm_source=chatgpt.com "Using Python Packages with ROS 2 - ROS documentation"
[10]: https://www.anaconda.com/legal/terms/terms-of-service?utm_source=chatgpt.com "Terms of Service - Anaconda"
[11]: https://huggingface.co/docs/huggingface_hub/en/package_reference/environment_variables?utm_source=chatgpt.com "Environment variables - Hugging Face"
[12]: https://docs.universal-robots.com/Universal_Robots_ROS2_Documentation/doc/ur_client_library/doc/setup/robot_setup.html?utm_source=chatgpt.com "Robot setup — Universal Robots ROS 2 Driver Documentation 0.1 ..."
[13]: https://moveit.ai/install-moveit2/binary/?utm_source=chatgpt.com "MoveIt 2 Binary Install"
[14]: https://support.realsenseai.com/hc/en-us/community/posts/15149975945363-realsense-SDK-on-windows-11?utm_source=chatgpt.com "realsense SDK on windows 11"
[15]: https://github.com/IntelRealSense/librealsense/blob/master/wrappers/python/examples/align-depth2color.py?utm_source=chatgpt.com "align-depth2color.py - realsenseai/librealsense - GitHub"
[16]: https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver/blob/main/ur_robot_driver/doc/setup_tool_communication.rst "Universal_Robots_ROS2_Driver/ur_robot_driver/doc/setup_tool_communication.rst at main · UniversalRobots/Universal_Robots_ROS2_Driver · GitHub"
[17]: https://support.thinklucid.com/getting-started/ "Getting Started with LUCID Cameras | LUCID Support & Help"
