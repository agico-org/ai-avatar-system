FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /app

# Install Python 3.10 + system dependencies
RUN apt-get update && apt-get install -y \
    python3.10 \
    python3.10-dev \
    python3-pip \
    ffmpeg \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libgomp1 \
    git \
    wget \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && ln -sf python3.10 /usr/bin/python3 \
    && ln -sf python3 /usr/bin/python \
    && ln -sf pip3 /usr/bin/pip

# Upgrade pip and build tools
RUN pip install --upgrade pip setuptools wheel

# Install PyTorch with CUDA 11.8 FIRST (matches base image)
# Doing this before requirements.txt ensures we get CUDA builds, not PyPI CPU-only
RUN pip install --no-cache-dir \
    torch==2.6.0+cu118 \
    torchvision==0.21.0+cu118 \
    torchaudio==2.6.0+cu118 \
    --index-url https://download.pytorch.org/whl/cu118

# Copy requirements
COPY requirements.txt .

# Pre-install numpy and cython to handle legacy packages like pkuseg
# that import numpy at setup.py runtime (fails with pip's build isolation)
# Using 1.25.2 because chatterbox-tts requires numpy<1.26.0
RUN pip install --no-cache-dir numpy==1.25.2 cython

# Install Python dependencies with --no-build-isolation so legacy setup.py
# scripts can find numpy in the main environment. --no-deps prevents re-installing torch.
RUN pip install --no-cache-dir --no-build-isolation -r requirements.txt

# Copy application code
COPY . .

# Create necessary directories
RUN mkdir -p /tmp/avatars /tmp/videos /tmp/audio voice_profiles

# Entrypoint: run migrations then start server
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

# Expose port
EXPOSE 8000

# Health check — longer start-period for model loading on first boot
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=5 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

# Run application via entrypoint (handles migrations automatically)
CMD ["./entrypoint.sh"]
