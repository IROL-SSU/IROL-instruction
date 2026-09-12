# 학습된 RL 모델 실행

이 문서는 IsaacLab에서 정의한 강화학습 환경을 등록하고, RSL-RL을 이용하여 모델을 학습하거나 학습된 모델을 실행하는 기본 절차를 설명합니다.

## 1. 학습 환경 준비

[Sweeping-Policy-DRL-Sim-Train][1]과 같이 학습 환경이 정의된 패키지가 필요합니다. 이 패키지는 실행 프로그램이나 별도의 설치 도구가 아니라 환경, 로봇, 관측값, 행동, 보상 및 학습 설정을 정의한 순수한 코드 베이스이며, [IsaacLab][2]을 의존성으로 요구합니다.

먼저 [IsaacLab Installation Guide][3]에 따라 Isaac Sim과 IsaacLab이 정상적으로 실행되는 Conda 환경을 구성합니다. 현재 학습 환경 패키지에는 이 환경 외에 기본적으로 요구되는 별도의 종속성 패키지가 없습니다. 이후 기능을 추가하면서 필요한 Python 패키지가 생길 수는 있으나, 현재는 별도의 `requirements.txt`나 추가 설치 절차가 있다고 가정하지 않습니다.

IsaacLab과 학습 환경의 버전이 서로 호환되는지 확인한 후 작업합니다. 기본적으로 호환되나, IsaacLab Installation Guide에 포함된 Recommend 방법을 사용하는 것을 권장합니다. 이 권장 설정은 버전에 따라 달라질 수 있습니다.

## 2. 학습 환경 등록

학습 환경을 enroll하는 방법은 여러 가지가 있으나, 기본적으로는 Python이 학습 환경 패키지의 경로를 참조할 수 있도록 경로 의존성을 주입하는 방식으로 처리합니다. 핵심은 환경 등록 코드가 들어 있는 `__init__.py`를 `train.py`와 `play.py`가 import할 수 있게 하는 것입니다. 해당 파일이 import되면 `gymnasium.register()`가 실행되고, 환경에 정의된 고유한 Task ID가 Gym registry에 등록됩니다.

권장 방식은 학습 환경 패키지를 editable mode로 설치하는 것입니다. 다음 명령은 실제 패키지 구조에 맞는 디렉터리에서 실행합니다.

```bash
conda activate isaaclab  # 실제로 생성한 Conda 환경 이름으로 변경
python -m pip install -e /absolute/path/to/Sweeping-Policy-DRL-Sim-Train
```

`pip install -e`는 심볼릭 링크 또는 이에 준하는 editable path 정보로 소스 경로를 현재 Python 환경에 연결하므로, 코드를 수정할 때마다 패키지를 다시 복사하여 설치할 필요가 없습니다. 패키지 구조상 저장소 루트가 아니라 `source/PACKAGE_NAME` 등의 하위 디렉터리에 `pyproject.toml` 또는 `setup.py`가 있다면 해당 디렉터리를 지정합니다.

editable installation을 사용할 수 없는 경우에는 학습 스크립트가 실행되기 전에 패키지의 절대경로를 `PYTHONPATH` 또는 `sys.path`에 추가하고, 환경 등록 모듈을 명시적으로 import할 수 있습니다.

```bash
export PYTHONPATH=/absolute/path/to/Sweeping-Policy-DRL-Sim-Train/source:${PYTHONPATH}
```

```python
import importlib
import sys

sys.path.insert(0, "/absolute/path/to/Sweeping-Policy-DRL-Sim-Train/source")
importlib.import_module("ENVIRONMENT_PACKAGE")  # 실제 패키지 이름으로 변경
```

이 과정은 특수한 학습 절차가 아니라 Python의 import 경로를 맞추는 코드 수준의 문제입니다. 경로를 추가했더라도 환경 등록 모듈을 import하지 않으면 Task ID가 registry에 등록되지 않을 수 있습니다.

## 3. Task ID 확인

train과 play에서 가장 중요한 인자는 `--task`에 전달하는 Task ID입니다. Task ID는 학습 환경 패키지가 `gymnasium.register(id=...)`로 등록한 환경의 Unique Key를 의미합니다. 클래스 이름, 패키지 이름 또는 체크포인트 파일 이름을 임의로 입력하는 값이 아닙니다.

예를 들어 환경 등록 코드가 다음과 같다면 Task ID는 `IROL-Sweeping-v0`입니다.

```python
gym.register(
    id="IROL-Sweeping-v0",
    entry_point="<ENVIRONMENT_PACKAGE>.env:SweepingEnv",
    kwargs={
        "env_cfg_entry_point": "<ENVIRONMENT_PACKAGE>.env:SweepingEnvCfg",
        "rsl_rl_cfg_entry_point": "<ENVIRONMENT_PACKAGE>.agents:SweepingPPORunnerCfg",
    },
)
```

사용할 Task ID는 학습 환경 패키지의 `__init__.py` 또는 환경 등록 코드를 확인하여 지정합니다. 동일한 ID를 중복 등록하지 않으며, train과 play에서는 원칙적으로 동일한 환경과 설정에 대응하는 Task ID를 사용합니다.

## 4. 모델 학습

[IsaacLab 저장소의 `scripts/reinforcement_learning/rsl_rl`][4]에 있는 `train.py`를 사용합니다. 다음 명령은 IsaacLab 저장소 루트에서 실행합니다.

```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py \
  --task "TASK_ID_DEFINED_BY_ENVIRONMENT" \
  --num_envs 64 \
  --headless
```

`TASK_ID_DEFINED_BY_ENVIRONMENT`에는 학습 환경에 등록된 실제 Task ID를 입력합니다. `--num_envs`와 `--headless` 등의 인자는 장비 자원과 디버깅 목적에 따라 조정할 수 있습니다. 학습 결과와 체크포인트는 RSL-RL agent configuration에 정의된 experiment name을 기준으로 IsaacLab 실행 위치의 `logs/rsl_rl/` 아래에 생성됩니다.

학습 창을 확인해야 할 때에는 `--headless`를 제외할 수 있습니다. 다만 렌더링을 활성화하면 학습 속도와 GPU 메모리 사용량에 영향을 줄 수 있습니다.

## 5. 학습된 모델 실행

시뮬레이션에서 학습된 모델을 확인할 때에는 같은 디렉터리의 `play.py`를 사용하고, 학습에 사용한 Task ID와 체크포인트를 지정합니다.

```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/play.py \
  --task "TASK_ID_DEFINED_BY_ENVIRONMENT" \
  --checkpoint /absolute/path/to/model.pt \
  --num_envs 1
```

체크포인트 경로를 지정하지 않으면 agent configuration의 `load_run`과 `load_checkpoint` 설정을 기준으로 파일을 찾으므로, 실행할 모델을 명확히 선택해야 할 때에는 `--checkpoint`에 절대경로를 지정합니다. Task ID, 환경 configuration, agent configuration 및 체크포인트의 관측·행동 차원이 서로 일치해야 합니다.

현재 IsaacLab의 RSL-RL `play.py`는 체크포인트를 불러온 뒤 inference policy를 실행하며, 같은 체크포인트 디렉터리의 `exported/` 아래에 TorchScript 형식의 `policy.pt`와 ONNX 형식의 `policy.onnx`도 생성합니다. 생성 여부와 파일 경로는 사용하는 IsaacLab 및 RSL-RL 버전에 따라 확인합니다.

## 6. 실제 로봇으로의 전이

위의 train과 play는 IsaacLab에 등록된 환경에서 정책을 학습하고 재생하는 절차입니다. `play.py`를 실행하는 것만으로 실제 로봇에 정책이 전이되거나 로봇 제어 프로그램이 생성되는 것은 아닙니다.

실제 로봇에서 모델을 실행하려면 별도의 제어 코드에서 학습된 체크포인트를 PyTorch로 load하거나, TorchScript 또는 ONNX 등의 형식으로 변환한 모델을 불러와 inference를 수행합니다. 이후 실제 센서와 로봇 상태를 학습 때와 동일한 순서, 단위, 좌표계 및 정규화 방식의 observation으로 구성하고, 모델의 action을 실제 로봇 명령으로 변환하여 전달합니다.

따라서 Sim-to-Real 실행에 별도의 특수한 등록 과정이 요구되는 것은 아닙니다. 모델 로딩, observation 전처리, action 후처리 및 로봇 통신을 구현하는 일반적인 code-level development가 필요합니다. 다만 실제 장비에서는 시뮬레이션과 다른 주기, 지연, 제한값 및 안전 조건이 존재하므로, action clipping, 비상 정지 및 통신 실패 처리를 포함한 안전 계층을 검증한 후 적용합니다.

[1]: https://github.com/IROL-SSU/Sweeping-Policy-DRL-Sim-Train.git "Sweeping-Policy-DRL-Sim-Train"
[2]: https://github.com/isaac-sim/IsaacLab.git "IsaacLab"
[3]: https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/index.html "IsaacLab Installation Guide"
[4]: https://github.com/isaac-sim/IsaacLab/tree/main/scripts/reinforcement_learning/rsl_rl "IsaacLab RSL-RL scripts"
