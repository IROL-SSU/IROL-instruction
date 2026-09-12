# NVIDIA 드라이버 설치

[Ubuntu 설치로 돌아가기](./README.md)

Ubuntu 22.04에서 실험 PC의 NVIDIA GeForce RTX 4090 드라이버를 설치합니다.

> **문서 상태:** 설치 흐름 초안입니다. 팀에서 사용할 드라이버 버전과 Isaac Sim/Lab 비호환 버전을 확인하기 전에는 실제 설치 명령을 실행하지 않습니다.

## 1. 설치 정책

- Ubuntu 설치 화면의 third-party software 옵션으로 드라이버를 설치하지 않습니다.
- Ubuntu 설치와 초기 업데이트를 먼저 완료합니다.
- 자동으로 최신 드라이버를 설치하지 않습니다.
- Isaac Sim/Lab과의 호환성을 확인한 정확한 버전을 선택합니다.
- 선택한 버전을 Ubuntu 패키지로 명시하여 설치합니다.

Isaac Sim/Lab 자체는 현재 실험 PC setup 문서의 범위가 아니지만, 향후 사용 가능성을 막는 드라이버 버전은 이 문서에 경고로 남깁니다.

## 2. GPU 인식 확인

```bash
lspci | grep -Ei 'VGA|3D|NVIDIA'
```

출력에 `NVIDIA`와 RTX 4090에 해당하는 장치가 보여야 합니다.

Ubuntu가 추천하는 드라이버 후보를 확인합니다.

```bash
sudo apt update
sudo apt install ubuntu-drivers-common -y
ubuntu-drivers devices
```

이 명령은 후보를 확인하기 위한 것입니다. 아직 `sudo ubuntu-drivers install`은 실행하지 않습니다.

## 3. 호환성 표 확인

설치 전에 아래 표의 `팀 사용 버전`과 비호환 정보를 확정해야 합니다.

| 항목 | 값 | 상태 |
| --- | --- | --- |
| GPU | NVIDIA GeForce RTX 4090 | 담당자 확인 |
| 팀 사용 NVIDIA 드라이버 | 확인 필요 | 설치 전 필수 확인 |
| 사용한 Isaac Sim 버전 | 확인 필요 | 담당자 경험 확인 필요 |
| 사용한 Isaac Lab 버전 | 확인 필요 | 담당자 경험 확인 필요 |
| 작동하지 않은 드라이버 버전 | 확인 필요 | 담당자 경험 확인 필요 |
| 정상 작동한 드라이버 버전 | 확인 필요 | 담당자 경험 확인 필요 |

> `확인 필요` 상태에서는 드라이버를 설치하지 않습니다. NVIDIA 드라이버와 Isaac의 호환성은 제품 버전에 따라 달라지므로 단순히 가장 높은 버전을 선택하지 않습니다.

## 4. Secure Boot 확인

```bash
sudo apt install mokutil -y
mokutil --sb-state
```

Secure Boot가 활성화되어 있으면 NVIDIA 커널 모듈 등록 과정에서 MOK 암호 설정과 재부팅 후 등록 화면이 나타날 수 있습니다. 팀의 Secure Boot 정책은 재검증이 필요합니다.

## 5. 확정된 버전 설치

팀에서 검증한 버전이 예를 들어 `XXX`라면 패키지 이름은 `nvidia-driver-XXX` 형태입니다. 아래의 `XXX`를 실제로 검증된 숫자로 바꾼 후에만 실행합니다.

```bash
sudo apt update
sudo apt install nvidia-driver-XXX
sudo reboot
```

> `XXX`를 그대로 입력하지 않습니다. 드라이버 버전을 확정하지 못했다면 여기서 중단합니다.

## 6. 설치 확인

재부팅 후 실행합니다.

```bash
nvidia-smi
```

다음을 확인합니다.

- GPU 이름이 NVIDIA GeForce RTX 4090입니다.
- `Driver Version`이 팀에서 선택한 버전입니다.
- 오류 없이 GPU 상태 표가 출력됩니다.

커널 모듈도 확인합니다.

```bash
lsmod | grep nvidia
```

`nvidia` 관련 모듈이 출력되어야 합니다.

## 7. 문제 발생 시 수집할 정보

드라이버를 임의로 제거하거나 다른 버전으로 바꾸기 전에 다음 출력을 저장하여 담당자 또는 AI agent에게 전달합니다.

```bash
cat /etc/os-release
uname -r
lspci | grep -i nvidia
nvidia-smi
ubuntu-drivers devices
mokutil --sb-state
dkms status
```

## 완료 체크리스트

- [ ] RTX 4090을 확인했다.
- [ ] 팀에서 사용할 드라이버 버전을 확정했다.
- [ ] Isaac Sim/Lab 비호환 버전을 확인했다.
- [ ] Secure Boot 상태를 확인했다.
- [ ] 확정된 드라이버 패키지를 설치했다.
- [ ] 재부팅 후 `nvidia-smi`가 정상 출력된다.
- [ ] 실제 드라이버 버전이 선택한 버전과 일치한다.

## 참고

- [Ubuntu 공식 추가 드라이버 안내](https://help.ubuntu.com/stable/ubuntu-help/hardware-driver.html)
- [NVIDIA Isaac Sim 기술 요구사항](https://docs.omniverse.nvidia.com/isaacsim/latest/common/technical-requirements.html)
