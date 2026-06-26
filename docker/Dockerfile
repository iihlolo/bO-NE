FROM docker.io/pytorch/pytorch:2.5.1-cuda12.1-cudnn9-devel@sha256:e8e63dd7baca894ba11fe1ba48a52a550793c8974f89b533d697784dd20a4dc0
# Base image 포함 사항:
#   Python 3.11, CUDA 12.1, cuDNN 9
#   torch==2.5.1+cu121, torchvision==0.20.1+cu121, torchaudio==2.5.1+cu121

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN apt-get update && apt-get install -y --no-install-recommends \
        git wget curl \
        ffmpeg \
        libgl1 libglib2.0-0 libsm6 libxext6 libxrender1 \
        libgomp1 libusb-1.0-0 \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt \
    && rm /tmp/requirements.txt

WORKDIR /workspace
CMD ["/bin/bash"]
