# IROL Instruction

연구실 로봇·센서·소프트웨어의 설치 및 시스템 구성 안내서입니다. 전체 구성을 확인한 뒤 각 항목의 설치 문서를 참고합니다.

> 현재는 문서 구조를 준비한 단계입니다. 버전, 설치 명령, 네트워크 설정 및 실제 동작 검증 결과는 확인 후 작성합니다.

## 전체 시스템 구성

### 구성 요소의 역할

| 구성 요소 | 문서에서 정리할 내용 | 확인할 사항 |
| --- | --- | --- |
| Ubuntu PC | 로봇·센서 연동과 실험 프로그램 실행 환경 | PC 사양, Ubuntu 버전, 실제 담당 기능 |
| Windows PC | OptiTrack 운용 및 실험 PC와 데이터 연계 | Windows·Motive 버전, 통신 방식 |
| UR 로봇 및 컨트롤러 | 로봇 연결, 상태 확인 및 제어 환경 | 정확한 모델, PolyScope 및 드라이버 버전 |
| Helios | 장치 인식과 데이터 취득 환경 | 제품 모델, SDK, 연결 방식 |
| OptiTrack | 추적 환경 및 데이터 송수신 설정 | 카메라 구성, 캘리브레이션, 수신 프로그램 |

그림에 함께 표시된 F/T 센서, 그리퍼, D435로 보이는 장치, Isaac 및 가상머신/PolyScope 관련 항목은 추가 확인 대상입니다. 현재 문서 범위는 아래 7개 항목입니다.

## 설치 문서

| 항목 | 안내 |
| --- | --- |
| [Ubuntu](./ubuntu/README.md) | 실험용 PC의 운영체제 설치 및 초기 설정 |
| [ROS](./ros/README.md) | 장비와 로봇 소프트웨어를 연동하기 위한 ROS 환경 준비 |
| [MoveIt](./moveit/README.md) | 로봇의 동작 계획 환경 설치 및 구성 |
| [Anaconda](./anaconda/README.md) | Python 가상환경과 실험용 패키지 관리 환경 준비 |
| [UR](./ur/README.md) | UR 로봇 사용을 위한 드라이버와 연결 환경 준비 |
| [Helios](./helios/README.md) | Helios 장비 사용을 위한 SDK 및 드라이버 환경 준비 |
| [OptiTrack](./optitrack/README.md) | OptiTrack 소프트웨어와 데이터 전달 환경 준비 |

## 저장소 구조

```text
IROL-instruction/
├── README.md
├── ubuntu/
│   └── README.md
├── ros/
│   └── README.md
├── moveit/
│   └── README.md
├── anaconda/
│   └── README.md
├── ur/
│   └── README.md
├── helios/
│   └── README.md
└── optitrack/
    └── README.md
```

## 설치 진행 순서 초안

실제 사용할 버전의 호환성을 먼저 확인한 뒤 순서를 확정합니다.

1. PC별 역할, 장비 모델 및 사용할 버전을 결정합니다.
2. Ubuntu PC의 운영체제와 기본 네트워크를 준비합니다.
3. 해당 환경에 맞는 ROS를 준비합니다.
4. UR 드라이버와 MoveIt의 호환성을 확인하고 로봇 연동 환경을 준비합니다.
5. Helios 및 OptiTrack을 각 장치의 사용 PC에 구성하고 데이터 수신을 확인합니다.
6. Anaconda가 필요한 실험 프로그램은 별도의 Python 환경을 구성합니다. ROS와의 환경 적용 범위도 기록합니다.
7. 장치별 동작 확인 후 시스템 전체의 연결과 실행 순서를 검증합니다.

Anaconda는 모든 구성 요소의 필수 선행 설치로 가정하지 않습니다. 장치별 설치는 대상 PC와 의존성에 따라 별도로 진행할 수 있습니다.

## 사용 버전 및 연결 정보

| 항목 | 값 |
| --- | --- |
| Ubuntu | 확인 필요 |
| ROS 1 / ROS 2 및 배포판 | 확인 필요 |
| MoveIt | 확인 필요 |
| Anaconda / Python | 확인 필요 |
| UR 모델 / PolyScope / 드라이버 | 확인 필요 |
| Helios 모델 / SDK | 확인 필요 |
| Windows / Motive / OptiTrack 구성 | 확인 필요 |
| PC 간 통신 방식 및 네트워크 구성 | 확인 필요 |

## 문서 작성 규칙

- 각 설치 문서는 사전 준비 → 설치 → 환경 설정 → 정상 동작 확인 → 문제 해결 순서로 작성합니다.
- 명령을 실행할 PC와 운영체제, 필요한 버전을 명시합니다.
- 설치 명령과 예상 출력은 실제 환경에서 검증한 후 추가합니다.
- 확인하지 않은 항목은 추측으로 채우지 않고 `확인 필요`로 남깁니다.
- 설정값이나 절차가 바뀌면 관련 문서와 이 페이지의 구성 정보를 함께 갱신합니다. 
