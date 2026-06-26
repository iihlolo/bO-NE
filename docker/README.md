# CCTV 박스 크기 추정 챌린지 - 평가 환경 안내

## 1. 개요

최종 평가는 제공된 Docker 환경에서 참가자의 ONNX 모델과 추론 코드를 실행하여 수행됩니다.

참가자는 아래 파일로 Docker 이미지를 빌드하고, 해당 환경에서 제출 코드가 정상 실행되는지 확인해야 합니다.

## 2. 제공 파일

```
docker/
├── Dockerfile          # Docker 이미지 빌드 파일
├── requirements.txt    # 설치된 Python 패키지 전체 목록 (버전 고정)
└── README.md           # 이 문서
```

## 3. 환경 사양

| 항목 | 버전 |
|------|------|
| OS (base) | Ubuntu 22.04 |
| Python | 3.11 |
| CUDA | 12.1 |
| cuDNN | 9 |
| PyTorch | 2.5.1+cu121 |
| torchvision | 0.20.1+cu121 |
| torchaudio | 2.5.1+cu121 |
| ONNX Runtime (GPU) | 1.20.1 |
| OpenCV | 4.10.0 (contrib, headless) |
| numpy | 1.26.4 |

전체 패키지 목록은 `requirements.txt`를 참조하세요.

## 4. Docker 이미지 빌드

### 사전 요구사항

- Docker 또는 Podman 설치
- NVIDIA Container Toolkit 설치 (GPU 사용 시)
- 인터넷 연결 (base image 및 pip 패키지 다운로드)

### 빌드 명령

```bash
# Dockerfile과 requirements.txt가 같은 디렉토리에 있어야 합니다.
docker build -t challenge-inference:latest .
```

빌드 시간: 약 10~20분 (네트워크 속도에 따라 다름)

### 빌드 확인

```bash
docker images challenge-inference
# SIZE: 약 16GB
```

### 환경 검증

컨테이너 안에서 아래 명령으로 환경이 정상인지 확인할 수 있습니다:

```bash
python -c "
import torch, onnxruntime, cv2, numpy, open3d
print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}')
print(f'ORT: {onnxruntime.__version__}, Providers: {onnxruntime.get_available_providers()}')
print(f'OpenCV: {cv2.__version__}')
print(f'NumPy: {numpy.__version__}')
print(f'Open3D: {open3d.__version__}')
"
```


## 5. 제출물 형식

압축 파일(ZIP)과 result.json 파일을 제출합니다.

```
submission.zip
├── main.py                 # 실행 진입점 (필수)
├── src/                    # 추가 소스코드 (선택)
├── train_src/              # 학습 코드 (AI 모델 사용 시 필수)
│   ├── train 코드
│   ├── README.md
│   └── requirements.txt
├── checkpoints/            # 모델 가중치 (AI 모델 사용 시 필수)
│   └── model.onnx
└── README.md               # 설명 문서 (선택)
```

- `main.py`: Docker 환경에서 실행되는 진입점. 이 파일만 평가 환경에서 정상 동작하면 됩니다.
- `train_src/`: 학습 코드. Docker 환경과 무관하게 참가자가 자유롭게 환경을 구성하여 사용합니다. 평가 대상이 아닙니다.
- `checkpoints/`: ONNX 모델 파일. `main.py`에서 로드하여 사용합니다.

### main.py 인터페이스

```bash
python main.py \
    --input_dir /data/test_videos \
```

- `--input_dir`: 평가 영상이 위치한 디렉토리 (`.mp4` 파일들)

### 출력 형식 (result.json)
- box_id는 선택 사항이며, 점수에 영향을 미치지 않습니다.
```json
{
    "videos": [
        {
            "video_id": "ConveyorScene_0000",
            "objects": [
                {"box_id": 0, "size_cm": {"w": 88.1, "d": 22.0, "h": 20.0}},
                {"box_id": 1, "size_cm": {"w": 45.5, "d": 30.2, "h": 25.1}}
            ]
        }
    ]
}
```

```json
{
    "videos": [
        {
            "video_id": "ConveyorScene_0000",
            "objects": [
                {"size_cm": {"w": 88.1, "d": 22.0, "h": 20.0}},
                {"size_cm": {"w": 45.5, "d": 30.2, "h": 25.1}}
            ]
        }
    ]
}
```

## 6. 주의사항

### 반드시 지켜야 할 사항

- `main.py`는 **Docker 이미지에 설치된 패키지만** 사용해야 합니다.
- 평가 중 **추가 pip install은 허용되지 않습니다.**
- ONNX 모델은 **표준 ONNX operator**만 사용해야 합니다. Custom operator는 실행되지 않을 수 있습니다.
- 모델 가중치(.onnx)는 `checkpoints/` 디렉토리에 포함하여 제출합니다.
- `train_src/`의 학습 코드는 평가 Docker 환경과 무관합니다. 자유롭게 구성하세요.

### 사용 가능한 주요 패키지

| 용도 | 패키지 |
|------|--------|
| ONNX 추론 | `onnxruntime-gpu`, `onnx`, `onnxsim` |
| 영상 처리 | `opencv-contrib-python-headless`, `decord`, `av`, `imageio` |
| 수치 연산 | `numpy`, `scipy`, `pandas`, `scikit-learn`, `numba` |
| 이미지 전처리 | `scikit-image`, `albumentations`, `Pillow` |
| 3D/기하 | `open3d`, `trimesh`, `shapely`, `kornia`, `pyransac3d` |
| 텐서 연산 | `torch`, `torchvision` (NMS, transforms 등) |
| 후처리 | `filterpy` (Kalman filter), `scipy.optimize` |
| 유틸 | `tqdm`, `pyyaml`, `orjson`, `loguru`, `rich` |

전체 목록: `requirements.txt` 참조

### GPU 관련

- 평가 서버 GPU: NVIDIA A100 40GB
- CUDA Execution Provider가 기본 사용됩니다.
- `onnxruntime`에서 `CUDAExecutionProvider`를 명시적으로 지정하는 것을 권장합니다.

```python
import onnxruntime as ort

session = ort.InferenceSession(
    "model.onnx",
    providers=["CUDAExecutionProvider", "CPUExecutionProvider"]
)
```


## 7. FAQ

**Q: PyTorch로 학습했는데, 꼭 ONNX로 변환해야 하나요?**

A: 네. 최종 제출의 모델은 ONNX 형식입니다. 단, `main.py`의 전처리/후처리에서 PyTorch 연산(예: `torchvision.ops.nms`, `torch.nn.functional.interpolate` 등)은 사용 가능합니다. 단, Docker 이미지에 포함된 PyTorch 버전에서 동작해야합니다.

**Q: 여러 ONNX 모델을 제출해도 되나요?**

A: 네. 예를 들어 detection용 ONNX + depth estimation용 ONNX + size regression용 ONNX를 조합하는 것이 가능합니다.

**Q: 학습 환경은 어떻게 구성하나요?**

A: `train_src/`의 학습 코드는 Docker 환경과 무관합니다. 참가자가 원하는 프레임워크(PyTorch, TensorFlow 등)와 환경에서 자유롭게 학습하고, 최종 모델만 ONNX로 변환하여 `checkpoints/`에 넣으면 됩니다.
