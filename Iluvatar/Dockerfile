# Copyright (c) 2023, Shanghai Iluvatar CoreX Semiconductor Co., Ltd.
# All Rights Reserved.
#
#    Licensed under the Apache License, Version 2.0 (the "License"); you may
#    not use this file except in compliance with the License. You may obtain
#    a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
#    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
#    License for the specific language governing permissions and limitations
#    under the License.

FROM skddj/dl_image:torch2.0.1-cuda11.8.0-ubuntu22.04-666

LABEL image=deepspark/deepsparkhub

ARG IMAGE_VERSION
ARG PYTHON_VERSION
ARG CUDA_INSTALL
ARG COREX_INSTALL

USER root

ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/zsh", "-c"]

# WORKDIR /workspace/

RUN sed -i '/^deb.*secur/d; s@/archive.ubuntu.com/@/mirrors.tuna.tsinghua.edu.cn/@g' /etc/apt/sources.list

## Install dependancies
RUN apt clean && apt update && apt install dialog -y && \
    apt install -y -qq --no-install-recommends unzip build-essential cmake ca-certificates \
                    make git curl vim wget \
                    perl lsb-release \
                    iproute2 pciutils \
                    kmod openssh-server \
                    automake libglib2.0-0 \
                    autoconf m4 libncursesw5 \
                    libbz2-dev \
                    cmake libgl1-mesa-glx python3-distutils \
                    gnupg \
                    apt-transport-https && rm -rf /var/lib/apt/lists/*

# RUN curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py
RUN apt-get update && apt-get install -y build-essential zlib1g-dev libncurses5-dev liblzma-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libopenmpi-dev wget

## Install CUDA
COPY ${CUDA_INSTALL} .
RUN sh ${CUDA_INSTALL} -- --silent --toolkit --librarypath=/usr/local/cuda --override
RUN rm ${CUDA_INSTALL}

## Install CoreX
COPY ${COREX_INSTALL} .
RUN bash ${COREX_INSTALL} --silent --toolkit
RUN rm ${COREX_INSTALL} 

## Install python library
# COPY ${PYTHON_VERSION}/ ./${PYTHON_VERSION}

# 更新并安装 zsh
RUN apt-get update && apt-get install -y zsh

# 添加环境变量到 .zshrc
RUN echo 'export PATH=/usr/local/corex/bin:$PATH' >> ~/.zshrc && \
echo 'export LD_LIBRARY_PATH=/usr/local/corex/lib64:/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.zshrc && \
echo 'export TERM=xterm-256color' >> ~/.zshrc

## Activate anaconda environment
# Display the contents of .zshrc
# RUN cat ~/.zshrc
RUN . /root/.zshrc && . /root/.zshrc

# RUN rm -rf ./${PYTHON_VERSION}

## Clone deepspark
# RUN git clone --depth 1 --branch ${IMAGE_VERSION} https://gitee.com/deep-spark/deepsparkhub.git
RUN git clone --depth 1  https://gitee.com/deep-spark/deepsparkhub.git

# 添加 sagernet 的 GPG 密钥并添加 APT 源
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://sing-box.app/gpg.key -o /etc/apt/keyrings/sagernet.asc && \
    chmod a+r /etc/apt/keyrings/sagernet.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/sagernet.asc] https://deb.sagernet.org/ * *" | tee /etc/apt/sources.list.d/sagernet.list > /dev/null

# 更新包列表并安装 sing-box
RUN apt-get update && apt-get install -y sing-box

# 清理缓存以减少镜像大小
RUN rm -rf /var/lib/apt/lists/*

### CMD ["/usr/bin/bash"]
