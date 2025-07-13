FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

WORKDIR /app

RUN apt-get update && DEBIAN_FRONTEND="noninteractive" apt-get install -y \
    git \
    python3.10 \
    python3-pip \
    python3-opencv

# Python Evn
RUN pip3 install torch torchvision torchaudio opencv-python --extra-index-url https://download.pytorch.org/whl/cu121
RUN git clone https://github.com/comfyanonymous/ComfyUI.git && \
    cd ComfyUI && \
    git checkout v0.3.39
RUN pip3 install -r /app/ComfyUI/requirements.txt

# Install ComfyUI-Manager
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Manager.git comfyui-manager && \
    cd comfyui-manager && \
    pip3 install -r requirements.txt

# Install ComfyUI-Impact-Pack
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack.git comfyui-impact-pack && \
    cd comfyui-impact-pack && \
    pip3 install -r requirements.txt

# Install ComfyUI-KJNodes
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-KJNodes.git comfyui-kjnodes && \
    cd comfyui-kjnodes && \
    pip3 install -r requirements.txt

# Install rgthree-comfy
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/rgthree/rgthree-comfy.git rgthree-comfy && \
    cd rgthree-comfy

# Install ComfyUI-Custom-Scripts
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Custom-Scripts.git comfyui-custom-scripts && \
    cd comfyui-custom-scripts

# Install ComfyUI_essentials
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/cubiq/ComfyUI_essentials.git comfyui-essentials && \
    cd comfyui-essentials && \
    pip3 install -r requirements.txt

# Install ComfyUI-QualityOfLifeSuit_Omar92
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/omar92/ComfyUI-QualityOfLifeSuit_Omar92.git comfyui-quality-of-life-suit && \
    cd comfyui-quality-of-life-suit

# Install comfyui_controlnet_aux
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/Fannovel16/comfyui_controlnet_aux.git comfyui-controlnet-aux && \
    cd comfyui-controlnet-aux && \
    pip3 install -r requirements.txt

# Install ComfyUI-Florence2
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/kijai/ComfyUI-Florence2.git comfyui-florence2 && \
    cd comfyui-florence2 && \
    pip3 install -r requirements.txt

# Install comfyui-instantId-faceswap
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/nosiu/comfyui-instantId-faceswap.git comfyui-instantid-faceswap && \
    cd comfyui-instantid-faceswap && \
    pip3 install -r requirements.txt

# Install was-node-suite-comfyui
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/WASasquatch/was-node-suite-comfyui.git was-node-suite-comfyui && \
    cd was-node-suite-comfyui && \
    pip3 install -r requirements.txt

# Install ComfyUI-ReActor
# RUN cd /app/ComfyUI/custom_nodes && \
#     git clone https://github.com/Gourieff/ComfyUI-ReActor.git comfyui-reactor && \
#     cd comfyui-reactor && \

# Install ComfyUI-AWS-Cognito-Login
RUN --mount=type=secret,id=GH_TOKEN \
    GITHUB_TOKEN=$(cat /run/secrets/GH_TOKEN) && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://${GITHUB_TOKEN}:@github.com/LucaLabsInc/ComfyUI-AWS-Cognito-Login.git --branch v1.0.6 --single-branch comfyui-aws-cognito-login && \
    cd comfyui-aws-cognito-login && \
    pip3 install -r requirements.txt
    
COPY .env /app/ComfyUI/custom_nodes/comfyui-aws-cognito-login/.env

EXPOSE 8848
CMD ["python3", "-u", "/app/ComfyUI/main.py", "--listen", "0.0.0.0", "--port", "8848"]
