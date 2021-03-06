ARG   BASE_IMAGE="ubuntu:16.04"
FROM $BASE_IMAGE

LABEL maintainer="Thomas Johansen <thomasjo@gmail.com>"

ADD build /tmp/build
WORKDIR /tmp

SHELL ["bash", "-c"]

RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    software-properties-common \
&&  rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y python3 \
&&  rm -rf /var/lib/apt/lists/* \
&&  curl https://bootstrap.pypa.io/get-pip.py | python3

# Build dependencies for protobuf.
RUN apt-get update && apt-get install -y \
    autoconf \
    automake \
    build-essential \
    libtool \
    python3-dev \
    unzip \
&&  rm -rf /var/lib/apt/lists/*

RUN curl -L https://github.com/google/protobuf/archive/v3.4.0.tar.gz | tar -xz \
&&  cd protobuf* \
&&  ./autogen.sh \
&&  ./configure \
&&  make install -j$(nproc) \
&&  cd python \
&&  python3 setup.py install --cpp_implementation

# Build dependencies for TensorFlow.
RUN apt-get update && apt-get install -y \
    bash-completion \
    openjdk-8-jdk \
    python \
    zlib1g-dev \
&&  rm -rf /var/lib/apt/lists/* \
&&  curl -LO https://github.com/bazelbuild/bazel/releases/download/0.7.0/bazel_0.7.0-linux-x86_64.deb \
&&  dpkg -i bazel*.deb

# With recent versions of TensorFlow, we unfortunately have to provide the
# container access to NVIDIA drivers and libraries at build time.
# Sadly Docker does not follow symlinks, so the hacky solution is therefore:
# $ mount --bind <nvidia-docker driver mointpoint> mnt/nvidia_driver
ADD mnt/nvidia_driver /usr/local/nvidia

RUN curl -L https://github.com/tensorflow/tensorflow/archive/v1.4.0.tar.gz | tar -xz \
&&  cd tensorflow* \
&&  pip3 install --no-cache-dir numpy \
&&  source /tmp/build/tensorflow-env && ./configure \
&&  bazel build \
    --config=opt \
    # Build with CUDA support when using a CUDA base image.
    ${CUDA_VERSION:+--config=cuda} \
    tensorflow/tools/pip_package/build_pip_package \
&&  bazel-bin/tensorflow/tools/pip_package/build_pip_package $PWD \
&&  pip3 install "$(ls -1 tensorflow*.whl | head -n 1)"

# Remove temporary "nvidia-docker" volume clone.
RUN rm -rf /usr/local/nvidia

# Ensure compatible combination of bleach and html5lib.
RUN pip3 install --no-cache-dir \
    bleach>=2.0 \
    html5lib>=0.99999999

RUN apt-get purge --autoremove -y \
    autoconf \
    automake \
    bash-completion \
    bazel \
    ca-certificates \
    curl \
    libtool \
    openjdk-8-jdk \
    python \
    python3-dev \
    software-properties-common \
    unzip \
    zlib1g-dev \
&&  rm -rf /etc/apt/sources.list.d/*

RUN rm -rf /tmp/*
WORKDIR /
