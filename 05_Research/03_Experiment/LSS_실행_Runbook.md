---
title: "LSS (Lift-Splat-Shoot) 실행 런북 — Ubuntu 22.04"
type: runbook
tags: [runbook, experiment, LSS, BEV, 자율주행]
status: ready
target_env: "Ubuntu 22.04 + NVIDIA GPU"
created: 2026-07-06
---

← [[00_Home]] | [[MOC_AVP-ROS]] | [[BEV_LSS_vs_BEVFormer]]

# 🛠️ LSS 실행 런북 — Ubuntu 22.04

> **목적** [[BEV_LSS_vs_BEVFormer]]에서 정리한 **LSS(Lift-Splat-Shoot)**를 실제로 돌려본다.
> **순서** ① 사전학습 가중치로 카메라→BEV **시각화** → ② nuScenes **mini로 짧은 학습**.
> **대상 머신** Ubuntu 22.04 + NVIDIA GPU (이 볼트를 만든 노트북이 아니라 별도 우분투 PC).

---

## 🤖 이 문서를 Claude Code로 실행하는 법 (에이전트 지침)

> 다른 우분투 머신에서 Claude Code를 열고 이렇게 시키세요:
> **"`LSS_실행_Runbook.md`를 열어서 Phase 0부터 순서대로 실행해줘. 🚧 수동 게이트가 나오면 멈추고 나한테 알려줘."**

**에이전트 실행 규칙:**
1. **Phase 0 → 5 순서대로** 진행. 각 Phase 끝의 ✅ **검증(Check)**이 통과해야 다음으로.
2. 🚧 **수동 게이트(MANUAL GATE)** = 사람이 브라우저 로그인/다운로드해야 하는 단계. 여기서 **반드시 멈추고** 사용자에게 요청.
3. 명령이 실패하면 **§트러블슈팅**을 먼저 확인 후 해당 패치 적용. 2회 이상 같은 오류면 멈추고 보고.
4. 경로 변수(`$NUSCENES`, `$LSS`, `$MODEL`)를 실제 절대경로로 채워 사용.

---

## 📋 사전 준비물 (사람이 미리 확인)
- **NVIDIA 드라이버 설치됨** (`nvidia-smi` 동작). CUDA 런타임은 PyTorch가 들고 오므로 별도 설치 불필요.
- **nuScenes 계정** (무료) — https://www.nuscenes.org 가입. 데이터 다운로드에 로그인 필요.
- 디스크 여유 **~15GB** (mini 4GB + 맵 확장 + 코드/모델).
- 인터넷 연결.

---

## Phase 0 — 환경 점검
```bash
# GPU / 드라이버 확인
nvidia-smi
# GPU 이름과 VRAM 기록해 둘 것 (예: RTX 3060 12GB → Phase 1의 CUDA 선택에 사용)

# conda 존재 확인 (없으면 miniconda 설치 필요)
conda --version || echo "conda 없음 → miniconda 설치 필요"

# git 확인
git --version
```
✅ **Check**: `nvidia-smi`가 GPU를 보여주고, `conda`/`git` 버전이 출력됨.

> conda가 없으면:
> ```bash
> wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
> bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
> source $HOME/miniconda3/bin/activate && conda init bash && exec bash
> ```

---

## Phase 1 — 코드 클론 + 가상환경 + 라이브러리
```bash
# 작업 폴더 (원하는 위치로 조정 가능)
export WORK=$HOME/lss_practice
mkdir -p $WORK && cd $WORK

# 1) LSS 저장소 클론
git clone https://github.com/nv-tlabs/lift-splat-shoot.git
export LSS=$WORK/lift-splat-shoot

# 2) conda 환경 (Python 3.9 권장 — LSS는 2020년 코드라 최신 3.13 비호환)
conda create -y -n lss python=3.9
conda activate lss

# 3) PyTorch 설치 — ⚠️ GPU 세대에 맞춰 선택 (Phase 0의 GPU 이름 참고)
#    최신 안정 stable 채널 (대부분의 30/40 시리즈, CUDA 12.x):
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
#    ↑ RTX 50 시리즈(Blackwell, sm_120)면 위 대신 아래(최신 CUDA 12.8) 사용:
#    pip install torch torchvision --index-url https://download.pytorch.org/whl/cu128

# 4) LSS 의존성 (README 명시)
pip install nuscenes-devkit tensorboardX efficientnet_pytorch==0.7.0

# 5) numpy 호환 (구 devkit이 np.float 사용 시 대비 — §트러블슈팅 T1)
pip install "numpy<1.24"
```
✅ **Check**:
```bash
python -c "import torch; print('torch', torch.__version__, 'cuda', torch.cuda.is_available())"
# → cuda True 여야 함. False면 §트러블슈팅 T4.
python -c "import nuscenes, efficientnet_pytorch; print('deps ok')"
```

---

## Phase 2 — 데이터셋 & 사전학습 가중치 준비

### 🚧 MANUAL GATE 2a — nuScenes mini 다운로드 (사람이 직접)
> 로그인이 필요해 자동화 불가. 사용자가 수행:
1. https://www.nuscenes.org → 로그인 → **Downloads** → **Full dataset (v1.0)** → **Mini** 선택.
2. `v1.0-mini.tgz` (~4GB) 다운로드.
3. **(시각화에 필요)** 같은 페이지의 **Map expansion (v1.3)** = `nuScenes-map-expansion-v1.3.zip`도 다운로드. → §트러블슈팅 T3 이유.
4. 두 파일을 우분투 머신의 `$HOME/Downloads/`에 둔다.

에이전트는 파일이 준비되면 압축 해제:
```bash
export NUSCENES=$HOME/nuscenes
mkdir -p $NUSCENES && cd $NUSCENES
tar -xf $HOME/Downloads/v1.0-mini.tgz -C $NUSCENES
# 맵 확장(시각화용): maps/ 아래로 병합
unzip -o $HOME/Downloads/nuScenes-map-expansion-v1.3.zip -d $NUSCENES/maps/
```
디렉토리 구조(확인용):
```
$NUSCENES/
├── maps/           # basemap PNG + expansion JSON(도로 레이어)
├── samples/        # 키프레임 센서 데이터
├── sweeps/         # 중간 프레임
└── v1.0-mini/      # 메타데이터(JSON)
```

### 🚧 MANUAL GATE 2b — 사전학습 가중치 (Google Drive)
> LSS README 제공 가중치(차량 BEV 분할 모델).
```bash
pip install gdown
cd $LSS
# README의 Google Drive 파일 (id: 1bsUYveW_eOqa4lglryyGQNeC4fyQWvQQ)
gdown 1bsUYveW_eOqa4lglryyGQNeC4fyQWvQQ -O model_pretrained.pt
export MODEL=$LSS/model_pretrained.pt
```
> gdown이 권한/쿼터로 실패하면, 사용자가 브라우저로 직접 받아 `$LSS/model_pretrained.pt`로 저장:
> https://drive.google.com/file/d/1bsUYveW_eOqa4lglryyGQNeC4fyQWvQQ/view

✅ **Check**:
```bash
ls -lh $NUSCENES/v1.0-mini/ && ls -lh $MODEL
```

---

## Phase 3 — 시각화 (사전학습 가중치로 카메라→BEV 예측)
```bash
cd $LSS
export MPLBACKEND=Agg   # 헤드리스 서버 대비(디스플레이 없이 파일로 저장)

# (선택) 데이터 로딩·라이다 투영 확인 — 파이프라인 sanity check
python main.py lidar_check mini/trainval --dataroot=$NUSCENES --viz_train=False

# 핵심: 예측 시각화 (map_folder는 맵 확장이 병합된 nuScenes 루트)
python main.py viz_model_preds mini/trainval \
  --modelf=$MODEL --dataroot=$NUSCENES --map_folder=$NUSCENES
```
✅ **Check**: 실행 폴더에 예측 시각화 이미지(`*.jpg`)가 생성됨. 카메라 6뷰 + 예측된 BEV 차량 분할이 겹쳐 보이면 성공.
- 이미지가 안 뜨는 환경이면 파일로 저장되니 `ls *.jpg`로 확인 후 사용자에게 전달.

---

## Phase 4 — 정량 평가 (IoU)
```bash
cd $LSS
python main.py eval_model_iou mini/trainval --modelf=$MODEL --dataroot=$NUSCENES
```
✅ **Check**: 콘솔에 IoU 수치 출력.
- ⚠️ **주의**: 이 가중치는 full 데이터로 학습됐고 여기선 **mini로 평가**하므로 논문 수치와 다를 수 있음(연습용 확인이 목적).

---

## Phase 5 — 짧은 학습 (mini, 감 잡기용)
```bash
cd $LSS
python main.py train mini/trainval \
  --dataroot=$NUSCENES --logdir=./runs --gpuid=0

# 별도 터미널에서 학습 곡선 모니터링
tensorboard --logdir=$LSS/runs --bind_all
# 브라우저: http://<우분투머신IP>:6006
```
✅ **Check**: `./runs/`에 로그 생성 + TensorBoard에서 **loss가 우하향**하면 성공.
- 🎯 **목표는 정확도가 아니라** "학습 루프가 돈다 + loss가 준다"를 눈으로 확인하는 것. mini는 데이터가 적어 과적합/불안정 정상.
- VRAM 부족(OOM) 시 §트러블슈팅 T5 (batch size / grid 축소).
- 몇백~몇천 step만 돌려보고 `Ctrl+C`로 중단해도 연습 목적 충분.

---

## 🚑 트러블슈팅

| # | 증상 | 원인 | 해결 |
|---|------|------|------|
| **T1** | `module 'numpy' has no attribute 'float'` | numpy≥1.24가 `np.float` 제거, 구 devkit 참조 | `pip install "numpy<1.24"` (Phase 1에 이미 포함). 또는 `pip install -U nuscenes-devkit` |
| **T2** | `torch.load` 시 `weights_only` 관련 에러/경고 | torch≥2.6 기본값 변경 | 코드에서 `torch.load(path)` → `torch.load(path, weights_only=False)`로 수정 (LSS는 `src/tools.py`·`src/explore.py`의 `load`/`get_model` 부근) |
| **T3** | 시각화 시 맵 관련 파일 없음/`NuScenesMap` 에러 | 맵 확장 미설치 | MANUAL GATE 2a-3의 `nuScenes-map-expansion-v1.3.zip`을 `$NUSCENES/maps/`에 풀기 |
| **T4** | `torch.cuda.is_available()` False | torch-CUDA 빌드 ↔ 드라이버/GPU 불일치 | GPU 세대에 맞는 index-url 재설치. RTX 50=cu128, 30/40=cu121. 드라이버 최신화 |
| **T5** | 학습 중 CUDA out of memory | VRAM 부족 | `main.py`의 train 인자에서 배치/그리드 축소: `--bsz=1`, 필요 시 `--nworkers=2`. (인자명은 `python main.py train -- --help` 또는 `src/train.py` 시그니처 확인) |
| **T6** | `efficientnet_pytorch` 로딩 경고 | 구버전 API | 무시 가능(동작엔 영향 없음). 정 안되면 `pip install efficientnet_pytorch==0.7.0` 재확인 |
| **T7** | matplotlib `no display` | 헤드리스 서버 | `export MPLBACKEND=Agg` (Phase 3에 포함). 결과는 `.jpg`로 저장됨 |

---

## 📝 실행 로그 (실제 돌린 뒤 채우기)
- [ ] Phase 0 환경: GPU=____ / VRAM=____ / torch=____ / cuda=____
- [ ] Phase 3 시각화 결과 이미지 경로: ____
- [ ] Phase 4 IoU 수치: ____
- [ ] Phase 5 학습: step ____ 에서 loss ____ → ____
- 관찰/이슈: 

## 출처 (References)
- LSS 공식 저장소 (명령어·의존성·가중치 링크) — https://github.com/nv-tlabs/lift-splat-shoot
- 논문 — Philion & Fidler, "Lift, Splat, Shoot," ECCV 2020 — https://arxiv.org/abs/2008.05711
- 사전학습 가중치 (Google Drive) — https://drive.google.com/file/d/1bsUYveW_eOqa4lglryyGQNeC4fyQWvQQ/view
- nuScenes 데이터셋 — https://www.nuscenes.org

## 관련 노트
- [[BEV_LSS_vs_BEVFormer]] — 개념·이론 비교
- [[HL FMA 2026]] · [[MOC_AVP-ROS]]
