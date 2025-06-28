FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

WORKDIR /app

RUN apt-get update && apt-get install -y \
    git \
    python3.10 \
    python3-pip

# Python Evn
RUN pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu121
RUN git clone https://github.com/comfyanonymous/ComfyUI.git && \
    cd ComfyUI && \
    git checkout v0.3.39
RUN pip3 install -r /app/ComfyUI/requirements.txt

# Install ComfyUI-Manager
RUN cd /app/ComfyUI/custom_nodes && \
    git clone https://github.com/ltdrdata/ComfyUI-Manager.git comfyui-manager && \
    cd comfyui-manager && \
    pip3 install -r requirements.txt
    
# Install ComfyUI-AWS-Cognito-Login
RUN --mount=type=secret,id=GH_TOKEN \
    GITHUB_TOKEN=$(cat /run/secrets/GH_TOKEN) && \
    cd /app/ComfyUI/custom_nodes && \
    git clone https://${GITHUB_TOKEN}:@github.com/LucaLabsInc/ComfyUI-AWS-Cognito-Login.git --branch v1.0.5 --single-branch comfyui-aws-cognito-login && \
    cd comfyui-aws-cognito-login && \
    pip3 install -r requirements.txt

COPY .env /app/ComfyUI/custom_nodes/comfyui-aws-cognito-login/.env

EXPOSE 8848
CMD ["python3", "-u", "/app/ComfyUI/main.py", "--listen", "0.0.0.0", "--port", "8848"]
