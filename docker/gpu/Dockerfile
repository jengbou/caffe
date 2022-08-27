FROM nvidia/cuda:11.4.0-devel-ubuntu18.04
LABEL maintainer caffe-maint@googlegroups.com

RUN apt-get update && \
        DEBIAN_FRONTEND=noninteractive \
        TZ=US/Eastern \
        apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        git \
        wget \
        libatlas-base-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python-dev \
        python-numpy \
        python-pip \
        python-setuptools \
        python-scipy && \
    rm -rf /var/lib/apt/lists/*

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

# FIXME: use ARG instead of ENV once DockerHub supports this
# https://github.com/docker/hub-feedback/issues/460
ENV CLONE_TAG=1.3

RUN wget --quiet https://github.com/Kitware/CMake/releases/download/v3.12.2/cmake-3.12.2-Linux-x86_64.sh -O /opt/cmake-3.12.2-Linux-x86_64.sh && \
    cd /opt && chmod a+x cmake-3.12.2-Linux-x86_64.sh && bash ./cmake-3.12.2-Linux-x86_64.sh --prefix=/usr/local --skip-license && \
    cd -

RUN wget --quiet https://bootstrap.pypa.io/pip/2.7/get-pip.py && \
    python get-pip.py && \
    pip install --upgrade setuptools

RUN rm -rf /opt/caffe

RUN cmake --version && git clone -b ${CLONE_TAG} --depth 1 https://github.com/jengbou/caffe.git . && \
    python -m pip install --upgrade pip --user && \
    cd python && for req in $(cat requirements.txt) pydot; do python -m pip install $req --user; done && cd .. && \
#    git clone https://github.com/NVIDIA/nccl.git && cd nccl && make -j install && cd .. && rm -rf nccl && \
    mkdir build && cd build && \
#    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 .. && \
    cmake -DCPU_ONLY=0 -DUSE_CUDNN=0 -DUSE_NCCL=0 .. && \
    make -j"$(nproc)"

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

WORKDIR /workspace
