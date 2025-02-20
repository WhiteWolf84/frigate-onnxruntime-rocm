FROM crzynik/rocm-builder AS rocm

FROM ubuntu:jammy AS py311-wheels

COPY --from=rocm /opt/rocm-dist /
ARG DEBIAN_FRONTEND=noninteractive

# create softlink
RUN ln -s /opt/rocm* /opt/rocm

# install base deps
RUN apt-get -qq update \
    && apt-get -qq install -y \
    apt-transport-https \
    gnupg \
    wget \
    libssl-dev \
    && apt-get -qq update \
    && apt-get -qq install -y \
    build-essential git \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# install necessary cmake version
RUN wget https://github.com/Kitware/CMake/releases/download/v3.26.5/cmake-3.26.5.tar.gz && \
    tar -zxvf cmake-3.26.5.tar.gz && \
    cd cmake-3.26.5 && \
    ./bootstrap && make && make install

RUN add-apt-repository ppa:deadsnakes/ppa \
    && apt-get -qq update \
    && apt-get -qq install -y python3.11 python3.11-dev python3.11-distutils \
    && rm -rf /var/lib/apt/lists/*

# Ensure python3 defaults to python3.11
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1

RUN wget -q https://bootstrap.pypa.io/get-pip.py -O get-pip.py \
    && python3 get-pip.py "pip"

# clone source for build
RUN cd / && git clone --depth 1 --branch v1.20.1 https://github.com/microsoft/onnxruntime.git && \
    cd ~ && git clone https://github.com/ROCm/AMDMIGraphX.git && \
    cd AMDMIGraphX && git config --global --add safe.directory "*" && \
    python3 -m pip config set global.break-system-packages true && \
    tools/build_and_test_onnxrt.sh && \
    ln -s /onnxruntime/build/Linux/Release/dist/ /wheels

FROM scratch AS package
COPY --from=py311-wheels /wheels /
