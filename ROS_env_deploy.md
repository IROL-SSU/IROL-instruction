# 서로 다른 컴퓨터 간 ROS 환경 이식 및 빌드

컴퓨터 A에서 사용하던 ROS 개발 환경을 컴퓨터 B로 이식하고 다시 빌드하는 작업은 크게 어렵지 않다. 기반 환경의 버전을 동일하게 맞춘 뒤 Python 의존성과 ROS 의존성을 각각 표준 도구로 설치하면 대부분의 환경을 복제할 수 있다.

## 전제 조건

컴퓨터 A와 컴퓨터 B는 다음 환경이 서로 동일해야 한다.

- OS 종류 및 버전
- 커널 버전
- ROS 버전 및 배포판

기반 환경의 버전이 다르면 같은 의존성을 설치하더라도 바이너리 호환성 문제나 빌드 오류가 발생할 수 있다. 따라서 이식 작업을 시작하기 전에 다음과 같은 명령으로 두 컴퓨터의 환경을 비교하는 것이 좋다.

이 문서는 다음 수준의 하드웨어 환경을 기준으로 한다.

- Intel Core i9-9900K
- 16 GB 이상 RAM
- NVIDIA GeForce RTX 4090

위 사양이 반드시 필요한 것은 아니다. 다만 기준 환경과 하드웨어 사양이나 구성이 크게 다른 경우에는 장치 드라이버 또는 펌웨어의 호환 문제로 빌드나 실행 과정에서 오류가 발생할 가능성이 있다.

```bash
# OS 버전 확인
cat /etc/os-release

# 커널 버전 확인
uname -r

# ROS 배포판 확인
echo $ROS_DISTRO
```

## Python Dependencies

컴퓨터 A에서 현재 Python 환경에 설치된 패키지와 버전을 `requirements.txt` 파일로 저장한다.

```bash
python3 -m pip freeze > requirements.txt
```

프로젝트에서 가상환경을 사용하고 있다면 가상환경을 활성화한 상태에서 실행해야 프로젝트와 관계없는 패키지가 포함되는 것을 줄일 수 있다.

생성한 `requirements.txt`를 컴퓨터 B로 전달한 뒤 다음 명령을 실행한다.

```bash
python3 -m pip install -r requirements.txt
```

설치 과정에서 발생하는 의존성 오류는 대부분 패키지 버전이나 Python 버전의 차이에서 발생한다. 에러 로그에서 설치에 실패한 패키지와 요구 버전을 확인하면 비교적 쉽게 원인을 찾을 수 있다. 필요한 경우 충돌하는 패키지의 버전을 조정한 뒤 다시 설치한다.

## ROS Dependencies

ROS 패키지에 필요한 시스템 의존성은 `rosdep install`을 통해 대부분 해결할 수 있다. 컴퓨터 B에서 ROS 환경을 불러온 뒤 ROS 워크스페이스의 루트 디렉터리에서 다음 명령을 실행한다.

```bash
source /opt/ros/<ros-distro>/setup.bash
cd <ros-workspace>

rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

`rosdep`이 초기화되지 않은 컴퓨터라면 최초 한 번 다음 명령을 실행해야 한다.

```bash
sudo rosdep init
rosdep update
```

`rosdep`이 의존성을 정상적으로 찾으려면 각 ROS 패키지의 `package.xml`에 의존성이 올바르게 선언되어 있어야 한다.

## 별도로 설치해야 하는 의존성

`pip`와 `rosdep`만으로 해결되지 않는 의존성도 간혹 존재한다. 이러한 경우에는 사용할 패키지의 `README`와 공식 설치 문서를 직접 확인해야 한다.

일부 패키지는 다음과 같은 추가 설치를 요구할 수 있다.

- `apt`로 설치해야 하는 서드파티 프로그램
- 하드웨어 제조사 또는 공급업체에서 제공하는 전용 SDK
- 소스 코드를 내려받아 직접 빌드해야 하는 외부 라이브러리
- 별도로 설정해야 하는 환경 변수 또는 라이브러리 경로

컴퓨터 A에 `apt`로 설치된 모든 패키지를 그대로 복제하는 방법은 권장하지 않는다. 전체 패키지 목록에는 현재 ROS 프로젝트와 관계없는 프로그램이 너무 많이 포함될 수 있기 때문이다. 따라서 각 패키지의 `README`를 기준으로 실제 필요한 항목만 확인하여 설치하는 편이 관리와 재현성 측면에서 적절하다.

## 빌드 및 확인

모든 의존성을 설치한 뒤 컴퓨터 B에서 워크스페이스를 빌드한다.

ROS 1 워크스페이스의 예시는 다음과 같다.

```bash
source /opt/ros/<ros-distro>/setup.bash
cd <ros-workspace>

catkin_make
source devel/setup.bash
```

ROS 2 워크스페이스의 예시는 다음과 같다.

```bash
source /opt/ros/<ros-distro>/setup.bash
cd <ros-workspace>

colcon build --symlink-install
source install/setup.bash
```

빌드가 완료되면 주요 노드 또는 launch 파일을 실행하여 다음 항목을 확인한다.

- 패키지가 정상적으로 탐색되는지
- 메시지와 서비스 타입을 정상적으로 불러오는지
- 필요한 장치와 연결되는지
- 런타임 라이브러리와 환경 변수가 올바르게 설정되었는지

## 결론

OS 및 커널 버전과 ROS 버전이 동일하다면 서로 다른 컴퓨터 간의 ROS 환경 이식과 빌드는 어렵지 않다. Python 의존성은 `pip freeze`와 `requirements.txt`로 복제하고, ROS 의존성은 `rosdep install`로 설치하면 대부분의 환경을 재현할 수 있다.

이 두 가지 방법으로 해결되지 않는 항목은 대체로 패키지별 `README`에 별도 설치 방법이 안내되어 있다. 에러 로그와 공식 설치 문서를 확인하여 필요한 프로그램이나 SDK만 추가로 설치하면 된다.
