# Ubuntu 22.04 설치

[전체 안내로 돌아가기](../README.md)

실험 PC에 Ubuntu 22.04 Desktop을 설치하고, NVIDIA 드라이버와 ROS 2를 설치할 수 있는 상태까지 준비합니다.

> **문서 상태:** 담당자 경험과 Ubuntu 공식 문서를 바탕으로 작성한 초안입니다. 실제 신규 설치 검증이 필요합니다.

## 설치 정책

이 저장소에서는 다음 선택을 기준으로 합니다.

| 항목 | 선택 |
| --- | --- |
| Ubuntu | Ubuntu 22.04 LTS Desktop |
| 설치 구성 | Normal installation |
| 설치 중 업데이트 | 인터넷 연결 시 선택 |
| Third-party software | 선택하지 않음 |
| 디스크 구성 | 자동 구성 사용하지 않음 |
| 파티션 | 수동 설정 |
| NVIDIA 드라이버 | Ubuntu 설치 완료 후 별도 설치 |

Windows를 유지하는 경우에는 이 문서와 함께 [Windows 듀얼 부팅 설치](./dual-boot.md)를 따릅니다.

## 1. 설치 전 준비

다음을 준비합니다.

- 12GB 이상의 USB 메모리
- 인터넷에 연결된 PC
- Ubuntu를 설치할 실험 PC
- 실험 PC와 USB에 있는 중요 데이터의 백업

> **경고:** 설치 USB를 만드는 과정은 USB의 데이터를 모두 삭제합니다. 파티션을 잘못 선택하면 Windows나 기존 실험 데이터가 삭제될 수 있습니다.

Windows를 유지해야 한다면 Ubuntu 설치 USB로 부팅하기 전에 [듀얼 부팅 문서](./dual-boot.md)의 Windows 준비 단계를 먼저 완료합니다.

## 2. Ubuntu 22.04 ISO 다운로드

Ubuntu 공식 릴리스 페이지에서 Ubuntu 22.04 Desktop의 최신 point release ISO를 받습니다.

- [Ubuntu 22.04 LTS 공식 다운로드](https://releases.ubuntu.com/22.04/)

파일 이름은 다음 형태입니다. 세부 버전은 다운로드 시점에 따라 달라질 수 있습니다.

```text
ubuntu-22.04.x-desktop-amd64.iso
```

`amd64`가 포함된 Desktop 이미지를 사용합니다. Ubuntu 24.04 또는 다른 버전의 이미지를 사용하지 않습니다.

## 3. 설치 USB 만들기

Windows PC에서 Rufus를 사용합니다.

1. [Rufus 공식 사이트](https://rufus.ie/)에서 Rufus를 받습니다.
2. 설치할 USB를 연결합니다.
3. `Device`에서 연결한 USB를 선택합니다.
4. `Boot selection`에서 다운로드한 Ubuntu 22.04 ISO를 선택합니다.
5. UEFI 시스템을 기준으로 `Partition scheme`은 `GPT`, `Target system`은 `UEFI`를 선택합니다.
6. 선택한 USB가 맞는지 다시 확인하고 `START`를 누릅니다.
7. ISO Image mode와 DD Image mode 중 선택을 요구하면 먼저 권장되는 ISO Image mode를 사용합니다.
8. 완료될 때까지 USB를 분리하지 않습니다.

> USB 장치명과 용량을 확인하지 않은 상태에서 `START`를 누르지 않습니다.

## 4. 설치 USB로 부팅

1. 설치 USB를 실험 PC에 연결합니다.
2. PC를 켜거나 재부팅합니다.
3. 부팅 직후 Boot Menu 키를 반복해서 누릅니다.
4. 목록에서 `UEFI`가 표시된 USB를 선택합니다.
5. Ubuntu 메뉴에서 `Try or Install Ubuntu`를 선택합니다.
6. 설치 화면이 나타나면 `Install Ubuntu`를 선택합니다.

Boot Menu 키는 메인보드 제조사에 따라 `F12`, `F11`, `F10`, `Esc` 또는 다른 키일 수 있습니다. 정확한 키는 PC 또는 메인보드 모델을 확인한 뒤 기록해야 합니다.

## 5. Ubuntu 설치 옵션 선택

설치 화면에서 다음과 같이 선택합니다.

1. 언어를 선택합니다.
2. 키보드 레이아웃을 선택합니다.
3. 인터넷에 연결합니다.
4. `Normal installation`을 선택합니다.
5. 설치 중 업데이트 다운로드는 인터넷 연결 시 선택합니다.
6. 그래픽·Wi-Fi 하드웨어 및 미디어 형식을 위한 third-party software는 **선택하지 않습니다**.

NVIDIA 드라이버는 설치 화면에서 함께 설치하지 않고, Ubuntu 설치가 끝난 뒤 [NVIDIA 드라이버 문서](./nvidia-driver.md)에 따라 설치합니다.

## 6. 파티션 수동 설정

`Installation type` 화면에서는 자동 디스크 구성을 선택하지 않습니다.

- Windows를 유지하는 듀얼 부팅이면 `Erase disk and install Ubuntu`를 선택하지 않습니다.
- 수동 파티션 설정을 의미하는 `Something else`를 선택합니다.
- [Windows 듀얼 부팅 설치](./dual-boot.md)의 파티션 절차를 따릅니다.

> **중단 조건:** 설치 대상 디스크, EFI 파티션, Ubuntu에 사용할 할당되지 않은 공간을 구분할 수 없다면 설치를 진행하지 않습니다.

전체 디스크를 Ubuntu에 사용하는 경우의 팀 표준 파티션 구성은 아직 확인되지 않았습니다. 해당 절차를 작성하기 전에는 자동으로 파티션을 생성하거나 기존 파티션을 삭제하지 않습니다.

## 7. 사용자와 시간대 설정

설치 화면에서 다음 값을 입력합니다.

| 항목 | 값 |
| --- | --- |
| 사용자 이름 | 팀 규칙 확인 필요 |
| 컴퓨터 이름 | 팀 규칙 확인 필요 |
| 비밀번호 | 저장소에 기록하지 않음 |
| 시간대 | Asia/Seoul |
| 로그인 방식 | 비밀번호 입력 필요 |

설치 요약 화면에서 디스크 변경 내용을 다시 확인한 뒤 설치를 시작합니다.

## 8. 재부팅

1. 설치 완료 메시지가 나타나면 재부팅합니다.
2. USB를 제거하라는 메시지가 나타나면 USB를 제거합니다.
3. `Enter`를 눌러 재부팅을 계속합니다.
4. 생성한 사용자로 로그인합니다.

듀얼 부팅 환경에서는 재부팅 후 Ubuntu와 Windows가 모두 부팅되는지 [듀얼 부팅 문서](./dual-boot.md)에 따라 확인합니다.

## 9. 최초 업데이트

`Ctrl` + `Alt` + `T`로 터미널을 열고 실행합니다.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install curl wget git build-essential software-properties-common -y
sudo reboot
```

재부팅 후 Ubuntu 버전을 확인합니다.

```bash
cat /etc/os-release
uname -m
```

다음 값이 포함되어야 합니다.

```text
VERSION_ID="22.04"
VERSION_CODENAME=jammy
x86_64
```

인터넷 연결을 확인합니다.

```bash
ping -c 4 archive.ubuntu.com
```

응답이 수신되면 기본 Ubuntu 설치가 완료된 것입니다.

## 10. 다음 단계

다음 순서로 진행합니다.

1. [NVIDIA 드라이버 설치](./nvidia-driver.md)
2. [ROS 2 Humble 설치](../ros/README.md)

## 완료 체크리스트

- [ ] 중요 데이터를 백업했다.
- [ ] Ubuntu 22.04 Desktop ISO를 사용했다.
- [ ] UEFI 모드로 설치 USB를 부팅했다.
- [ ] Normal installation을 선택했다.
- [ ] Third-party software를 선택하지 않았다.
- [ ] 자동 파티션 설정을 사용하지 않았다.
- [ ] 설치 전 디스크 변경 내용을 확인했다.
- [ ] Ubuntu 22.04로 정상 부팅했다.
- [ ] 초기 업데이트를 완료했다.
- [ ] 인터넷 연결을 확인했다.

## 참고

- [Ubuntu Desktop 공식 설치 안내](https://ubuntu.com/tutorials/install-ubuntu-desktop)
- [Ubuntu 부팅 USB 제작 안내](https://ubuntu.com/desktop/docs/en/latest/how-to/create-a-bootable-usb-stick/)
