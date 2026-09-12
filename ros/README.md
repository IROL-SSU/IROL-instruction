# ROS 2 Humble 설치

[전체 안내로 돌아가기](../README.md)

Ubuntu 22.04에 ROS 2 Humble Desktop 전체 구성을 설치하고 `~/.bashrc`에 환경 설정을 추가합니다.

> 대상 환경: Ubuntu 22.04 (Jammy), ROS 2 Humble
>
> 문서 상태: 공식 문서 기반 초안, 실제 신규 설치 검증 필요

## 1. 설치 환경 확인

터미널을 열고 Ubuntu 버전을 확인합니다.

```bash
cat /etc/os-release
```

출력에서 아래 두 값을 확인합니다.

```text
VERSION_ID="22.04"
VERSION_CODENAME=jammy
```

다른 값이 나오면 아래 설치를 진행하지 않습니다.

## 2. UTF-8 로캘 설정

아래 명령을 순서대로 실행합니다.

```bash
sudo apt update
sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
locale
```

마지막 출력에 `UTF-8`이 포함되어야 합니다.

## 3. ROS 2 저장소 등록

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update
sudo apt install curl -y
```

ROS 2 저장소 설정 패키지의 최신 버전을 내려받아 설치합니다.

```bash
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```

## 4. ROS 2 Humble 설치

먼저 Ubuntu 패키지를 업데이트합니다.

```bash
sudo apt update
sudo apt upgrade -y
```

ROS 2 Humble Desktop과 개발 도구를 설치합니다.

```bash
sudo apt install ros-humble-desktop ros-dev-tools -y
```

## 5. `~/.bashrc` 설정

새 터미널을 열 때마다 ROS 2 Humble 환경이 자동으로 적용되도록 설정합니다.

```bash
grep -qxF 'source /opt/ros/humble/setup.bash' ~/.bashrc || echo 'source /opt/ros/humble/setup.bash' >> ~/.bashrc
source ~/.bashrc
```

설정을 확인합니다.

```bash
echo "$ROS_DISTRO"
which ros2
```

아래와 같이 출력되면 설치와 환경 설정이 완료된 것입니다.

```text
humble
/opt/ros/humble/bin/ros2
```

## 주의

이 설치는 Ubuntu 시스템 Python을 사용합니다. 설치 및 ROS 2 실행 중에는 Conda 환경을 활성화하지 않습니다.

## 참고

- [ROS 2 Humble 공식 Ubuntu 설치 문서](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)
