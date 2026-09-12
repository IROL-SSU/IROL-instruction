# Windows와 Ubuntu 22.04 듀얼 부팅

[Ubuntu 설치로 돌아가기](./README.md)

Windows를 유지하면서 Ubuntu 22.04를 별도 파티션에 설치합니다. 파티션 크기와 실제 디스크 구성은 아직 팀 기준이 확정되지 않았으므로, 이 문서는 안전한 작업 순서와 중단 조건을 우선 정의합니다.

> **문서 상태:** 초안입니다. 실제 실험 PC의 디스크 구성과 파티션 크기를 확인한 뒤 보완해야 합니다.

## 중요 경고

이 과정에서 잘못된 디스크나 파티션을 선택하면 Windows, 복구 파티션 또는 실험 데이터가 삭제될 수 있습니다.

- 백업 없이 진행하지 않습니다.
- Windows 파티션은 Ubuntu 설치 화면에서 축소하지 않습니다.
- Windows의 디스크 관리에서 Ubuntu 공간을 먼저 확보합니다.
- 기존 EFI System Partition은 포맷하지 않습니다.
- 파티션을 구분할 수 없으면 설치를 중단합니다.

## 1. 현재 디스크 구성 확인

Windows에서 `Win` + `R`을 누르고 다음을 입력합니다.

```text
diskmgmt.msc
```

다음을 기록합니다.

| 확인 항목 | 기록 |
| --- | --- |
| Windows가 설치된 디스크 번호 | 확인 필요 |
| 디스크 전체 용량 | 확인 필요 |
| Windows `C:` 파티션 용량 | 확인 필요 |
| EFI System Partition 위치와 용량 | 확인 필요 |
| Ubuntu에 할당할 용량 | 확인 필요 |
| Windows와 Ubuntu가 같은 SSD인지 여부 | 확인 필요 |

화면을 캡처해 설치 중 비교할 수 있게 보관하는 것을 권장합니다. 복구 파티션과 EFI 파티션은 삭제하거나 축소하지 않습니다.

## 2. UEFI 모드 확인

Windows에서 `Win` + `R`을 누르고 다음을 입력합니다.

```text
msinfo32
```

`BIOS Mode`가 `UEFI`인지 확인합니다. 다른 값이면 이 문서대로 진행하지 않고 담당자에게 확인합니다.

## 3. BitLocker 확인

관리자 권한으로 명령 프롬프트 또는 PowerShell을 열고 실행합니다.

```powershell
manage-bde -status
```

Windows 파티션에 BitLocker가 활성화되어 있다면 다음을 수행합니다.

1. BitLocker 복구 키를 안전한 별도 위치에 보관합니다.
2. Windows 설정에서 BitLocker를 끕니다.
3. 암호 해독이 완전히 끝날 때까지 기다립니다.
4. `manage-bde -status`로 해제 상태를 다시 확인합니다.

BitLocker가 활성화된 상태에서는 Ubuntu 설치를 진행하지 않습니다.

## 4. Intel RST 확인

Ubuntu 설치 프로그램이 Intel RST 사용 경고를 표시하면 설치를 중단합니다.

> BIOS에서 저장장치 모드를 즉시 변경하지 않습니다. Windows 설정을 준비하지 않고 RST 또는 RAID를 AHCI로 변경하면 Windows가 부팅되지 않을 수 있습니다.

실험 PC에서 RST를 사용했는지와 안전한 변경 절차는 재검증이 필요합니다.

## 5. Windows 파티션 축소

Windows의 `Disk Management`에서 진행합니다.

1. Windows가 설치된 `C:` 파티션을 찾습니다.
2. `C:` 파티션을 마우스 오른쪽 버튼으로 누릅니다.
3. `Shrink Volume`을 선택합니다.
4. Ubuntu에 사용할 크기를 입력합니다.
5. 축소 후 생긴 공간이 `Unallocated`로 표시되는지 확인합니다.

Ubuntu용 NTFS 볼륨을 새로 만들지 않습니다. 공간을 `Unallocated` 상태로 둡니다.

> **미확정:** Ubuntu에 할당할 표준 용량은 담당자 확인이 필요합니다.

## 6. Ubuntu USB를 UEFI 모드로 부팅

[Ubuntu 설치 문서](./README.md)에 따라 설치 USB를 만들고 Boot Menu에서 `UEFI`가 표시된 USB를 선택합니다.

같은 USB가 두 번 보인다면 `UEFI`가 붙은 항목을 선택합니다.

## 7. 수동 파티션 설정

Ubuntu 설치 화면에서 `Something else`를 선택합니다.

먼저 Windows에서 기록한 디스크 용량과 파티션 배치를 비교하여 다음을 찾습니다.

- 기존 EFI System Partition
- Windows NTFS 파티션
- Windows 복구 파티션
- Windows에서 확보한 `free space`

### 기존 EFI System Partition

다음과 같이 설정합니다.

| 항목 | 선택 |
| --- | --- |
| 파티션 종류 | 기존 EFI System Partition |
| 사용 방식 | EFI System Partition |
| 마운트 위치 | `/boot/efi` |
| Format | **선택하지 않음** |

EFI 파티션을 포맷하면 Windows 부팅 항목이 손상될 수 있습니다.

### Ubuntu 루트 파티션

Windows에서 확보한 `free space` 안에 생성합니다.

| 항목 | 선택 |
| --- | --- |
| 파일 시스템 | Ext4 journaling file system |
| 마운트 위치 | `/` |
| Format | 선택 |
| 용량 | 팀 표준 확인 필요 |

별도 `/home` 파티션과 swap 파티션 사용 여부는 아직 확정되지 않았습니다. 확정 전에는 임의의 표준값을 문서에 추가하지 않습니다.

### 부트로더 대상

`Device for boot loader installation`은 Ubuntu와 기존 EFI System Partition이 있는 실제 디스크를 선택해야 합니다. 파티션이 아닌 디스크 장치인지 확인합니다.

예시는 환경마다 달라질 수 있으므로 `/dev/nvme0n1` 같은 장치명을 그대로 따라 입력하지 않습니다.

## 8. 설치 전 최종 확인

설치 프로그램이 보여주는 변경 목록에서 다음을 확인합니다.

- 새로 포맷되는 파티션은 Ubuntu용으로 만든 Ext4 파티션뿐입니다.
- Windows NTFS 파티션은 포맷되지 않습니다.
- EFI System Partition은 포맷되지 않습니다.
- Windows 복구 파티션은 변경되지 않습니다.

하나라도 확신할 수 없다면 `Go Back` 또는 `Quit`을 선택합니다.

## 9. 설치 후 부팅 확인

설치 완료 후 재부팅하여 GRUB 메뉴에서 다음 항목을 각각 한 번 이상 부팅합니다.

1. Ubuntu
2. Windows Boot Manager

두 운영체제에서 중요한 파일과 인터넷 연결을 확인합니다.

Ubuntu에서 부팅 모드를 확인합니다.

```bash
test -d /sys/firmware/efi && echo "UEFI 부팅" || echo "Legacy 부팅"
```

`UEFI 부팅`이 출력되어야 합니다.

디스크와 마운트 상태를 기록합니다.

```bash
lsblk -o NAME,SIZE,FSTYPE,FSVER,MOUNTPOINTS
findmnt /boot/efi
```

## 완료 체크리스트

- [ ] Windows 데이터를 백업했다.
- [ ] UEFI 모드를 확인했다.
- [ ] BitLocker 상태를 확인하고 필요한 경우 완전히 해제했다.
- [ ] Windows에서 파티션을 축소했다.
- [ ] Ubuntu 공간을 `Unallocated`로 남겼다.
- [ ] Ubuntu USB를 UEFI 모드로 부팅했다.
- [ ] 기존 EFI 파티션을 포맷하지 않았다.
- [ ] Windows 및 복구 파티션을 변경하지 않았다.
- [ ] Ubuntu와 Windows가 모두 정상 부팅된다.

## 참고

- [Ubuntu Desktop 공식 설치 안내](https://ubuntu.com/tutorials/install-ubuntu-desktop)
