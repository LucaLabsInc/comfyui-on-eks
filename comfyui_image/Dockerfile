# Stage 1: Base image with CUDA and system dependencies
FROM nvidia/cuda:12.9.0-cudnn-runtime-ubuntu24.04 AS base

RUN apt-get update && DEBIAN_FRONTEND="noninteractive" apt-get install -y \
    git \
    python3.12 \
    python3-pip \
    python3-opencv \
    libcairo2-dev \
    pkg-config \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Stage 2: Python dependencies (changes less frequently)
FROM base AS python-deps

# # Uninstall numpy 
# RUN sudo apt remove python-numpy

# Install PyTorch and core Python packages
RUN pip3 install torch torchvision torchaudio opencv-python --extra-index-url https://download.pytorch.org/whl/cu128 --break-system-packages --ignore-installed

# Stage 3: ComfyUI core (may change occasionally)
FROM python-deps AS comfyui-core

# Clone and checkout ComfyUI
RUN git clone https://github.com/LucaLabsInc/ComfyUI.git && \
    cd ComfyUI && \
    git checkout feat/workflow-file-mountpoint-s3-support-2

# Install ComfyUI requirements
RUN pip3 install -r /app/ComfyUI/requirements.txt --break-system-packages

# Fix BrushNet nodes installation
RUN pip3 install -U accelerate==0.32.0 --break-system-packages

# Stage 4a: Custom nodes without additional dependencies
FROM comfyui-core AS custom-nodes-simple

# Clone all simple custom nodes (no additional pip requirements)
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/rgthree/rgthree-comfy.git rgthree-comfy && \
    git clone https://github.com/ltdrdata/ComfyUI-Custom-Scripts.git comfyui-custom-scripts && \
    git clone https://github.com/omar92/ComfyUI-QualityOfLifeSuit_Omar92.git comfyui-quality-of-life-suit && \
    git clone https://github.com/Suzie1/ComfyUI_Comfyroll_CustomNodes.git comfyroll-studio && \
    git clone https://github.com/Derfuu/Derfuu_ComfyUI_ModdedNodes.git derfuu-comfyui-modded-nodes

# Stage 4b: Custom nodes with lightweight dependencies
FROM custom-nodes-simple AS custom-nodes-light

# ComfyUI-Manager (frequently used, put first)
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Manager.git comfyui-manager && \
    cd comfyui-manager && \
    pip3 install -r requirements.txt --break-system-packages

# ComfyUI_essentials
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/cubiq/ComfyUI_essentials.git comfyui-essentials && \
    cd comfyui-essentials && \
    pip3 install -r requirements.txt --break-system-packages

# Stage 4c: Custom nodes with heavy dependencies
FROM custom-nodes-light AS custom-nodes-heavy

# Group by similar dependencies to maximize layer reuse
# Computer vision related nodes
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/Fannovel16/comfyui_controlnet_aux.git comfyui-controlnet-aux && \
    cd comfyui-controlnet-aux && \
    pip3 install -r requirements.txt --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-Florence2.git comfyui-florence2 && \
    cd comfyui-florence2 && \
    pip3 install -r requirements.txt --break-system-packages

# Face-related nodes
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/nosiu/comfyui-instantId-faceswap.git comfyui-instantid-faceswap && \
    cd comfyui-instantid-faceswap && \
    pip3 install -r requirements.txt --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/Gourieff/ComfyUI-ReActor.git comfyui-reactor && \
    cd comfyui-reactor && \
    python3 install.py --break-system-packages

# Segmentation nodes
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/LucaLabsInc/ComfyUI-segment-anything-2.git --branch chore/set-setuptools-py-modules --single-branch comfyui-segment-anything-2 && \
    cd comfyui-segment-anything-2 && \
    pip3 install . --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/storyicon/comfyui_segment_anything.git comfyui-segment-anything && \
    cd comfyui-segment-anything && \
    pip3 install -r requirements.txt --break-system-packages

# Other heavy nodes
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git comfyui-impact-pack && \
    cd comfyui-impact-pack && \
    pip3 install -r requirements.txt --break-system-packages --ignore-installed && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-KJNodes.git comfyui-kjnodes && \
    cd comfyui-kjnodes && \
    pip3 install -r requirements.txt --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/WASasquatch/was-node-suite-comfyui.git was-node-suite-comfyui && \
    cd was-node-suite-comfyui && \
    pip3 install -r requirements.txt --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-SUPIR.git comfyui-supir && \
    cd comfyui-supir && \
    pip3 install -r requirements.txt --break-system-packages && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/quasiblob/ComfyUI-EsesImageResize.git eses-image-resize

# Stage 5: Private repository and configuration (changes most frequently)
FROM custom-nodes-heavy AS final

# Install ComfyUI-AWS-Cognito-Login (requires GitHub token)
RUN --mount=type=secret,id=GH_TOKEN \
    GITHUB_TOKEN=$(cat /run/secrets/GH_TOKEN) && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://${GITHUB_TOKEN}:@github.com/LucaLabsInc/ComfyUI-AWS-Cognito-Login.git --branch v1.0.7 --single-branch comfyui-aws-cognito-login && \
    cd comfyui-aws-cognito-login && \
    pip3 install -r requirements.txt --break-system-packages

# Copy configuration files (most likely to change)
COPY .env /app/ComfyUI/custom_nodes/comfyui-aws-cognito-login/.env

EXPOSE 8848
CMD ["python3", "-u", "/app/ComfyUI/main.py", "--listen", "0.0.0.0", "--port", "8848"]
