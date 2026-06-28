# CCTV 기반 화물 적재 박스 크기 추정

## 1. 프로젝트 개요

본 프로젝트는 CJ대한통운 미래기술챌린지 2026의 'CCTV 영상 기반 화물 객체 분석' 과제를 수행하기 위한 솔루션으로, 단일 시점(Single View) Synthetic CCTV 영상을 이용하여 컨베이어 레일을 따라 이동하는 화물 박스의 가로(Width), 세로(Depth), 높이(Height)를 cm 단위로 추정하는 것을 목표로 한다.

객체 검출에는 YOLO11 Segmentation 모델을 사용하며, 검출된 객체를 Tracking한 뒤 카메라 기하학(Geometry) 기반의 후처리를 통해 실제 크기를 계산한다. 객체 검출과 크기 추정을 분리한 Hybrid 구조로, 객체 검출 성능과 기하학적 계산을 각각 독립적으로 개선할 수 있도록 설계하였다.

프로젝트명 'bO-NE(보네)'는 CJ대한통운의 물류 브랜드 'O-NE(오네)'에 box의 b를 결합한 이름으로, 'Box Object Neural Engine'의 약자이다. AI 기반 객체 분석을 통해 물류 자동화와 운영 효율 향상을 지향한다.

---

# 2. 기술 스택

## Development Environment

| 항목 | 버전 |
|------|------|
| OS | Ubuntu 22.04 |
| Python | 3.11 |
| CUDA | 12.1 |
| cuDNN | 9 |

## Deep Learning

| 항목 | 버전 |
|------|------|
| PyTorch | 2.5.1 + cu121 |
| torchvision | 0.20.1 |
| ONNX Runtime (GPU) | 1.20.1 |
| YOLO | YOLO11 Segmentation |

## Computer Vision

| 항목 | 버전 |
|------|------|
| OpenCV | 4.10.0 (contrib, headless) |
| NumPy | 1.26.4 |

---

# 3. 프로젝트 구조
```
bO-NE
 ├── main.py
 ├── src/
 │   ├── config.py
 │   ├── infer_yolo.py
 │   ├── video_reader.py
 │   ├── tracker.py
 │   ├── geometry.py
 │   ├── estimator.py
 │   ├── postprocess.py
 │   └── io_utils.py
 ├── train_src/
 │   ├── train_yolo11.ipynb
 │   ├── export_onnx.py
 │   ├── README.md
 │   └── requirements.txt
 ├── checkpoints/
 │   └── model.onnx
 └── README.md 
```
---

# 4. 전체 처리 흐름
```
   text 입력 영상
         │
         ▼
     프레임 추출
         │
         ▼
YOLO11 Segmentation 추론
         │
         ▼
  객체(박스) Mask 검출
         │
         ▼
      Tracking
         │
         ▼
Geometry 기반 좌표 변환
         │
         ▼
Width / Depth / Height 추정
         │
         ▼
Track 단위 후처리(Median)
         │
         ▼
   CSV / JSON 저장 
```
---

# 5. 모듈별 기능

## main.py

프로그램의 실행 진입점이다.

### 주요 기능

- 입력 영상 탐색
- 전체 파이프라인 실행
- 결과 저장

---

## video_reader.py

영상 입출력을 담당한다.

### 주요 기능

- VideoCapture 생성
- 프레임 추출
- FPS 및 해상도 확인
- 영상 종료 처리

입력

- Video

출력

- Frame

---

## infer_yolo.py

YOLO11 Segmentation 추론 모듈이다.

### 주요 기능

- ONNX 모델 로드
- 이미지 전처리
- YOLO 추론
- NMS 수행
- Segmentation Mask 반환

YOLO의 역할은 객체를 검출하는 것까지이며, 크기 추정은 수행하지 않는다.

입력

- Frame

출력

- Segmentation Mask
- Confidence
- Class

---

## tracker.py

동일 객체를 여러 프레임에서 추적한다.

### 주요 기능

- Track ID 생성
- 객체 ID 유지
- 동일 박스 연결

입력

- Detection

출력

- Track

---

## geometry.py

영상 좌표계를 실제 좌표계로 변환한다.

### 주요 기능

- 원근 보정
- 픽셀 → 실제 좌표 변환
- Camera Geometry 계산

본 모듈은 크기를 계산하지 않고 좌표 변환만 수행한다.

---

## estimator.py

실제 박스 크기를 계산한다.

### 주요 기능

- Width 계산
- Depth 계산
- Height 계산

입력

- Track
- Geometry 정보

출력

- Width(cm)
- Depth(cm)
- Height(cm)

---

## postprocess.py

여러 프레임에서 계산된 결과를 하나의 최종 결과로 통합한다.

### 주요 기능

- 이상치 제거
- Median 계산
- 최종 크기 산출

---

## io_utils.py

입출력 관련 기능을 담당한다.

### 주요 기능

- CSV 저장
- JSON 저장
- 디렉터리 생성
- 파일 관리

---

## config.py

환경설정 및 상수를 관리한다.

### 예시

- Model Path
- Confidence Threshold
- IoU Threshold
- Frame Interval
- Geometry Parameter

---

# 6. 모델 설명

## 모델 구조

객체 검출은 YOLO11 Segmentation을 사용한다.

YOLO는 객체의 위치와 Segmentation Mask만 예측하며, 실제 크기 추정은 Geometry 기반 알고리즘을 통해 수행한다.
```
 text Frame
     │
     ▼
 YOLO11 Segmentation
     │
     ▼
   Mask
     │
     ▼
  Geometry
     │
     ▼
 Width / Depth / Height 
```
---

# 7. 학습 데이터 전처리

YOLO 학습에서는 이미지 자체를 과도하게 변형하지 않는다.

모델이 실제 추론 환경과 동일한 데이터를 학습할 수 있도록 원본 프레임을 그대로 사용한다.

## 전처리 과정

### 1. 프레임 추출

- 원본 영상에서 일정 간격(Frame Interval)으로 이미지 추출
- 중복 프레임 최소화

### 2. 프레임 선별

다음과 같은 다양한 상황을 포함하도록 데이터를 구성한다.

- 가까운 객체
- 먼 객체
- 작은 박스
- 큰 박스
- 부분 가림(Occlusion)
- 다양한 조명 환경
- 다양한 적재 상태

### 3. Segmentation 라벨링

YOLO11 Segmentation 학습을 위해 Polygon 기반 라벨을 생성한다.

라벨링 원칙

- 보이는 영역만 Polygon으로 라벨링
- 가려진 부분은 추정하여 그리지 않음
- 화면 밖 영역은 포함하지 않음
- 클래스는 box 1개만 사용

### 4. Train / Validation 분리

프레임 단위가 아닌 영상(Video) 단위로 데이터를 분리하여 데이터 누수를 방지한다.

### 5. Data Augmentation

학습 시 다음과 같은 증강을 적용한다.

- Brightness
- Contrast
- HSV
- Scale
- Translation
- Perspective
- Blur
- Noise

---

# 8. 모델 학습 과정
```
    text 원본 영상
         │
         ▼
     프레임 추출
         │
         ▼
     프레임 선별
         │
         ▼
 Segmentation 라벨링
         │
         ▼
Train / Validation 분리
         │
         ▼
YOLO11 Segmentation 학습
         │
         ▼
     Validation
         │
         ▼
    ONNX Export
         │
         ▼
   model.onnx 생성 
```
학습 완료 후 생성된 model.onnx는 제출 환경에서 ONNX Runtime을 이용하여 추론에 사용된다.

---

# 9. 설계 철학

객체 검출과 크기 추정을 분리한 모듈형(Hybrid) 구조로 설계하였다.

- YOLO11 Segmentation은 객체의 위치와 형태를 검출한다.
- Tracking은 동일 객체를 여러 프레임에서 연결한다.
- Geometry는 영상 좌표를 실제 좌표로 변환한다.
- Estimator는 실제 박스 크기를 계산한다.
- Postprocess는 여러 프레임의 결과를 통합하여 안정적인 최종 결과를 생성한다.

각 모듈은 독립적으로 개선할 수 있도록 설계되어 있으며, 객체 검출 모델 변경이나 Geometry 알고리즘 개선 시에도 전체 시스템의 구조를 유지할 수 있다.