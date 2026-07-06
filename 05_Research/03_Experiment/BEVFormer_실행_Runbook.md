---
title: "BEVFormer 실행 런북 — Ubuntu 22.04"
type: runbook
tags: [runbook, experiment, BEVFormer, BEV, transformer, 자율주행]
status: draft   # ⚠️ 아직 실제 머신 미검증. BEVFormer는 의존성이 매우 까다로움 — 각 Phase 공식 문서 대조 필수.
target_env: "Ubuntu 22.04 + NVIDIA GPU (≥8GB, tiny 기준)"
created: 2026-07-07
last_run: null
---

← [[00_Home]] | [[MOC_AVP-ROS]] | [[BEV_LSS_vs_BEVFormer]]

# 🛠️ BEVFormer 실행 런북 — Ubuntu 22.04

> **목적** [[BEV_LSS_vs_BEVFormer]]에서 정리한 **BEVFormer**(backward projection, [[deformable-attention]] 기반)를 실제로 돌려본다.
> **순서** ① 사전학습 가중치로 **정량 평가(NDS/mAP)** → ② 결과 **시각화(카메라+BEV)** → ③ tiny로 **짧은 학습**.
> **대상 머신** Ubuntu 22.04 + NVIDIA GPU. **tiny 기준 8GB**면 평가 가능, base/small은 VRAM·데이터가 훨씬 큼.
> ⚠️ **LSS 런북과 결정적 차이**: BEVFormer는 `mmcv-full 1.4.0 / mmdet 2.14 / mmdet3d 0.17.1` 등 **구버전 핀 + CUDA 컴파일**이 필요해 설치가 어렵다. **Phase 1이 전체의 8할.** → [[LSS_실행_Runbook]]보다 시간·실패 확률이 크다.

---

## 🤖 이 문서를 Claude Code로 실행하는 법 (에이전트 지침)

> 우분투 머신에서 Claude Code를 열고:
> **"`BEVFormer_실행_Runbook.md`를 열어서 Phase 0부터 순서대로 실행해줘. 🚧 수동 게이트에서 멈추고, 버전 핀은 절대 임의로 올리지 마."**

**에이전트 실행 규칙:**
1. **Phase 0 → 5 순서대로.** 각 Phase 끝의 ✅ **검증(Check)** 통과가 다음 진행 조건.
2. 🚧 **수동 게이트(MANUAL GATE)** = 사람이 로그인/다운로드해야 하는 단계. **반드시 멈추고** 사용자에게 요청.
3. ⛔ **버전을 마음대로 올리지 말 것.** mmcv/mmdet/mmdet3d/torch 조합은 서로 강하게 결합돼 있어, 하나만 바꿔도 컴파일이 깨진다. 실패 시 §트러블슈팅부터.
4. 같은 오류 2회 이상이면 멈추고 보고. 경로 변수(`$BEV`, `$NUSCENES`, `$CKPT`)를 실제 절대경로로 채워 사용.
5. **드라이버는 건드리지 말 것.** CUDA 11.1 컴파일은 conda `cudatoolkit`으로 처리하며, 시스템 NVIDIA 드라이버 재설치는 하지 않는다.

---

## 📋 사전 준비물 (사람이 미리 확인)
- **NVIDIA 드라이버 설치됨** (`nvidia-smi` 동작). 최신 드라이버여도 conda cudatoolkit=11.1로 빌드하므로 OK.
- **gcc/g++** 및 빌드 도구 (`sudo apt install build-essential ninja-build`) — mmcv/mmdet3d 오퍼레이터 컴파일에 필요.
- **nuScenes 계정** (무료) — https://www.nuscenes.org.
- 디스크 여유 — **mini 경로 ~20GB** (mini 4GB + CAN bus + map + 코드/모델/컴파일 캐시). full trainval은 ~**350GB+**(권장 안 함, 아래는 mini 기준).
- 인터넷 연결(가중치·백본 다운로드).

> **왜 tiny + mini인가**: 원 논문 학습은 8×A100. 여기 목표는 "**돌아간다 + BEV가 나온다**" 확인이므로 가장 가벼운 **bevformer_tiny**를 **nuScenes mini**로 돌린다. 사전학습 가중치는 full로 학습된 것이라, mini 평가 수치는 논문값과 다르다(정상).

---

## Phase 0 — 환경 점검
```bash
nvidia-smi                      # GPU/VRAM 기록 (tiny 평가 ~ 수 GB, tiny 학습 8GB 권장)
conda --version || echo "conda 없음 → miniconda 설치 필요"
git --version
gcc --version || echo "build-essential 필요"
```
✅ **Check**: `nvidia-smi`가 GPU 표시 + `conda`/`git`/`gcc` 버전 출력.

> conda 없으면 (LSS 런북과 동일):
> ```bash
> wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
> bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
> source $HOME/miniconda3/bin/activate
> ```
> 빌드 도구 없으면: `sudo apt update && sudo apt install -y build-essential ninja-build`

---

## Phase 1 — 환경·의존성 (★ 가장 어려움, 버전 핀 엄수)

> BEVFormer 공식 `docs/install.md` 조합을 그대로 따른다. **순서와 버전을 바꾸지 말 것.**

```bash
export WORK=$HOME/bevformer_practice
mkdir -p $WORK && cd $WORK

# 1) conda 환경 (Python 3.8 고정)
conda create -y -n bevformer -c conda-forge --override-channels python=3.8
conda activate bevformer
python -m ensurepip --upgrade        # conda-forge 최소 env pip 부트스트랩 (§T11)
# ⚠️ 이후 모든 설치는 `python -m pip`. ROS 오염 머신은 매 세션 `unset PYTHONPATH` (§T8)

# 2) PyTorch 1.9.1 + cudatoolkit 11.1 (conda로 CUDA 런타임까지)
conda install -y pytorch==1.9.1 torchvision==0.10.1 cudatoolkit=11.1 \
  -c pytorch -c conda-forge

# 3) mmcv-full 1.4.0 (torch1.9/cu111 prebuilt wheel — 컴파일 회피의 핵심)
python -m pip install mmcv-full==1.4.0 \
  -f https://download.openmmlab.com/mmcv/dist/cu111/torch1.9.0/index.html

# 4) mmdet / mmsegmentation (핀 고정)
python -m pip install mmdet==2.14.0 mmsegmentation==0.14.1

# 5) mmdet3d 0.17.1 (소스 빌드 — CUDA 오퍼레이터 컴파일 발생, 수 분 소요)
cd $WORK
git clone https://github.com/open-mmlab/mmdetection3d.git
cd mmdetection3d && git checkout v0.17.1
python -m pip install -v -e .        # 실패 시 §T2/§T3

# 6) BEVFormer 부가 의존성 (핀 — numpy/setuptools 특히 민감)
python -m pip install einops fvcore seaborn iopath==0.1.9 timm==0.6.13 \
  typing-extensions==4.5.0 numpy==1.19.5 matplotlib==3.5.2 numba==0.48.0 \
  pandas==1.4.4 scikit-image==0.19.3 setuptools==59.5.0

# 7) BEVFormer 코드
cd $WORK
git clone https://github.com/fundamentalvision/BEVFormer.git
export BEV=$WORK/BEVFormer
mkdir -p $BEV/ckpts
```
✅ **Check**:
```bash
python -c "import torch; print('torch', torch.__version__, 'cuda', torch.cuda.is_available())"   # 1.9.1, cuda True
python -c "import mmcv, mmdet, mmseg, mmdet3d; print('mm ok', mmcv.__version__, mmdet.__version__, mmdet3d.__version__)"
# → mmcv 1.4.0 / mmdet 2.14.0 / mmdet3d 0.17.1
```
> ⚠️ 위 import가 하나라도 실패하면 다음으로 넘어가지 말 것. 대부분의 실패가 여기서 발생(§트러블슈팅 T1~T5).

---

## Phase 2 — 데이터셋 · 가중치 준비

### 🚧 MANUAL GATE 2a — nuScenes mini + 확장 (사람이 직접)
> 로그인 필요 → 자동화 불가. 사용자가 https://www.nuscenes.org 로그인 후 **Downloads**에서:
1. **Full dataset (v1.0) → Mini** = `v1.0-mini.tgz` (~4GB)
2. **CAN bus expansion** = `can_bus.zip` ← ⚠️ **BEVFormer 필수** (시계열 BEV가 자차 pose를 CAN bus에서 읽음). 없으면 학습·평가 불가.
3. **Map expansion (v1.3)** = `nuScenes-map-expansion-v1.3.zip` (시각화용)
4. 세 파일을 `$HOME/Downloads/`에 둔다.

에이전트는 파일 준비되면 배치(BEVFormer는 `data/nuscenes` 기본 경로 가정):
```bash
export NUSCENES=$BEV/data/nuscenes
mkdir -p $NUSCENES
tar -xf $HOME/Downloads/v1.0-mini.tgz -C $NUSCENES
# CAN bus → data/can_bus (create_data의 --canbus가 가리키는 위치)
unzip -o $HOME/Downloads/can_bus.zip -d $BEV/data/
# map 확장 → maps/ 아래 병합
unzip -o $HOME/Downloads/nuScenes-map-expansion-v1.3.zip -d $NUSCENES/maps/
```
목표 구조:
```
$BEV/data/
├── can_bus/                     # ← CAN bus 확장 (필수)
└── nuscenes/
    ├── maps/ samples/ sweeps/   # 센서 데이터 + 맵
    └── v1.0-mini/               # 메타데이터(JSON)
```

### Phase 2b — info(.pkl) 생성 (mini 버전)
```bash
cd $BEV
unset PYTHONPATH
# 시계열 info pkl 생성 (nuscenes_infos_temporal_{train,val}.pkl)
python tools/create_data.py nuscenes \
  --root-path ./data/nuscenes --out-dir ./data/nuscenes \
  --extra-tag nuscenes --version v1.0-mini --canbus ./data
```
> ⚠️ `--version v1.0-mini` 필수(기본은 full `v1.0-trainval`). config가 참조하는 파일명은 `nuscenes_infos_temporal_train.pkl` / `..._val.pkl`. → §T7

### 🚧 MANUAL GATE 2c — 사전학습 가중치
```bash
cd $BEV/ckpts
# BEVFormer-tiny (가장 가벼움 — 8GB 목표에 권장)
wget https://github.com/zhiqi-li/storage/releases/download/v1.0/bevformer_tiny_epoch_24.pth
export CKPT=$BEV/ckpts/bevformer_tiny_epoch_24.pth
# (base 쓰려면: bevformer_r101_dcn_24ep.pth + 백본 r101_dcn_fcos3d_pretrain.pth 도 필요, VRAM↑)
```
> wget이 막히면 브라우저로 releases 페이지에서 직접 받아 `$BEV/ckpts/`에 저장.
> 릴리스 저장소: https://github.com/zhiqi-li/storage/releases/tag/v1.0

✅ **Check**:
```bash
ls -lh $NUSCENES/nuscenes_infos_temporal_val.pkl && ls -lh $CKPT
```

---

## Phase 3 — 정량 평가 (NDS / mAP)  ← "돌아간다" 1차 확인
```bash
cd $BEV
unset PYTHONPATH
export CONFIG=projects/configs/bevformer/bevformer_tiny.py

# 단일 GPU 평가 (test.py 직접). --eval bbox 로 nuScenes 지표 산출
python tools/test.py $CONFIG $CKPT --eval bbox
```
✅ **Check**: 콘솔에 **NDS / mAP / mATE…** 지표 테이블 출력.
- ⚠️ 가중치는 **full train**으로 학습됐는데 여기선 **mini val**로 평가 → 수치가 논문(tiny ~NDS 0.35대)과 다를 수 있음. **목적은 파이프라인이 끝까지 도는지 확인.**
- OOM이면 §T8 (tiny인데도 부족하면 다른 프로세스 정리).
- 멀티 GPU면 `tools/dist_test.sh $CONFIG $CKPT <GPU수> --eval bbox`.

---

## Phase 4 — 시각화 (카메라 6뷰 + BEV 예측)
```bash
cd $BEV
unset PYTHONPATH
export MPLBACKEND=Agg     # 헤드리스 서버 대비(파일로 저장)

# 1) 결과를 파일로 덤프
python tools/test.py $CONFIG $CKPT --out test/results.pkl

# 2) 덤프 결과를 BEV+카메라로 렌더 (경로/스크립트명은 레포 버전에 따라 확인)
python tools/analysis_tools/visual.py test/results.pkl
```
✅ **Check**: 예측 BEV(3D 박스)와 카메라 뷰가 겹쳐진 이미지/영상이 생성.
- ⚠️ `visual.py`의 인자·출력 경로는 BEVFormer 버전마다 달라짐 → 스크립트 상단 주석/`--help`로 확인. 없으면 `tools/analysis_tools/` 목록 확인.
- 저장 위치는 스크립트 기준(cwd). 안 뜨면 파일로 저장되니 `ls`로 확인 후 사용자에게 전달.

---

## Phase 5 — 짧은 학습 (tiny, mini · 감 잡기용)
```bash
cd $BEV
unset PYTHONPATH
# 단일 GPU 학습 (tiny). 몇 iter만 돌려 루프·loss 확인 후 Ctrl+C
python tools/train.py $CONFIG --work-dir work_dirs/bevformer_tiny_mini
# 멀티 GPU: tools/dist_train.sh $CONFIG <GPU수>
```
✅ **Check**: `work_dirs/…`에 로그 생성 + **loss가 우하향**.
- 🎯 정확도가 아니라 "**학습 루프가 돈다 + loss 감소**" 확인이 목표. mini는 데이터가 적어 불안정/과적합 정상.
- OOM 시 §T8: config의 `data.samples_per_gpu`(배치)를 1로, `queue_length`(시계열 프레임 수)를 줄임.
- 원 config는 24 epoch 가정 — 여기선 수십~수백 iter만 돌리고 중단해도 충분.

---

## 🚑 트러블슈팅

| # | 증상 | 원인 | 해결 |
|---|------|------|------|
| **T1** | `mmcv-full` 설치가 소스 컴파일로 빠지거나 실패 | prebuilt wheel index-url 누락/torch 버전 불일치 | 반드시 `-f https://download.openmmlab.com/mmcv/dist/cu111/torch1.9.0/index.html` 로 **1.4.0** 설치. torch가 1.9.1인지 먼저 확인 |
| **T2** | mmdet3d `pip install -e .` 컴파일 에러 (nvcc/gcc) | Ubuntu 22.04 기본 gcc(11/12)가 CUDA11.1과 비호환 | conda로 구 컴파일러 설치: `conda install -c conda-forge gxx_linux-64=9`. `nvcc`는 conda cudatoolkit 것 사용(`which nvcc`) |
| **T3** | 컴파일 중 `ninja: build stopped` / `Killed` | 메모리 부족(병렬 빌드 과다) | `MAX_JOBS=2 python -m pip install -v -e .` 로 병렬도 축소 |
| **T4** | `AttributeError: np.long`/`np.float`/`np.int` 없음 | numpy≥1.20이 제거한 별칭을 구 mm 코드가 참조 | 핀 지킬 것: `numpy==1.19.5` (Phase 1-6 포함). 이미 최신이면 재설치 |
| **T5** | `setuptools` 관련 빌드/`easy_install` 오류 | 최신 setuptools가 구 빌드 스크립트와 충돌 | `python -m pip install setuptools==59.5.0` (Phase 1-6 포함) 후 재빌드 |
| **T6** | `can_bus` 관련 KeyError / pose 없음 | CAN bus 확장 미설치 또는 경로 오류 | MANUAL GATE 2a-2 `can_bus.zip`을 `$BEV/data/can_bus/`에. create_data의 `--canbus ./data` 확인 |
| **T7** | `Database version not found` / `..._temporal_val.pkl` 없음 | create_data를 full 버전으로 돌렸거나 미실행 | Phase 2b를 `--version v1.0-mini`로 재실행. config의 `data_root`가 `data/nuscenes/`인지 확인 |
| **T8** | CUDA out of memory (평가/학습) | tiny여도 VRAM 부족/타 프로세스 점유 | 다른 GPU 프로세스 종료. 학습은 config `samples_per_gpu=1`, `queue_length` 축소. 평가는 다른 앱 정리 |
| **T9** | `visual.py` 실행 실패/경로 오류 | 버전마다 스크립트·인자 상이 | `tools/analysis_tools/` 실제 파일명 확인, 스크립트 상단 주석의 사용법대로. Phase 3(수치)만으로도 "동작 확인" 목적 충족 |
| **T10** | `conda create` `CondaToSNonInteractiveError` | 기본 채널 ToS 미동의 | `-c conda-forge --override-channels` (Phase 1-1 포함) 또는 `conda tos accept` |
| **T11** | conda-forge env `No module named pip` | 최소 env에 pip 미포함 | `python -m ensurepip --upgrade` |
| **T12** | 설치 패키지가 `ModuleNotFoundError` / `which pip`이 `~/.local` | ROS 등 `PYTHONPATH` 오염이 conda env를 덮음 | 매 세션 `unset PYTHONPATH` 후 `conda activate bevformer`. 항상 `python -m pip` |

> **막히면 Docker 대안**: mm 계열 컴파일이 계속 깨지면, 핀 버전이 이미 맞춰진 커뮤니티 Docker 이미지 사용을 검토(호스트 NVIDIA 드라이버 + `nvidia-container-toolkit` 필요). 단, nuScenes 데이터는 볼륨 마운트로 공유.

---

## 📝 실행 로그
- [ ] Phase 0 환경: GPU=____ / VRAM=____ / driver=____
- [ ] Phase 1 스택: torch 1.9.1 / mmcv 1.4.0 / mmdet 2.14.0 / mmdet3d 0.17.1 import OK?
- [ ] Phase 3 평가: NDS=____ / mAP=____ (mini val, tiny)
- [ ] Phase 4 시각화: 결과 경로=____
- [ ] Phase 5 학습: loss 시작=____ → ____ (우하향?)
- 관찰/이슈:
  - (설치에서 걸린 지점·해결책을 여기 기록 → 다음 실행 시간 단축)

## 출처 (References)
- BEVFormer 공식 저장소 — https://github.com/fundamentalvision/BEVFormer
- 설치 가이드 — `docs/install.md` (mmcv/mmdet/mmdet3d 핀 버전)
- 데이터 준비 — `docs/prepare_dataset.md` (create_data, CAN bus)
- 논문 — Li et al., "BEVFormer," ECCV 2022 — https://arxiv.org/abs/2203.17270
- 사전학습 가중치 릴리스 — https://github.com/zhiqi-li/storage/releases/tag/v1.0
- nuScenes 데이터셋 — https://www.nuscenes.org

## 관련 노트
- [[BEVFormer]] — 개념(backward projection · [[deformable-attention]])
- [[BEV_LSS_vs_BEVFormer]] — LSS와의 이론·수치 비교
- [[LSS_실행_Runbook]] — 🛠️ 자매 런북(LSS, 검증 완료). 설치 난이도 대비용
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
