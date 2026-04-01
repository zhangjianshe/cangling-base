# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

# 必须声明 TARGETARCH 才能捕捉 buildx 传入的架构信息 (amd64/arm64)
ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 完善编译依赖：加入 libgeotiff, bison, flex, 以及遥感常用的 HDF5/NetCDF
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    wget \
    git \
    python3-dev \
    python3-numpy \
    swig \
    ant \
    openjdk-11-jdk \
    libproj-dev \
    libgeos-dev \
    libsqlite3-dev \
    libtiff-dev \
    libgeotiff-dev \
    libjpeg-dev \
    libpng-dev \
    libwebp-dev \
    libzstd-dev \
    liblzma-dev \
    libcurl4-openssl-dev \
    libhdf5-dev \
    libnetcdf-dev \
    pkg-config \
    bison \
    flex \
    && rm -rf /var/lib/apt/lists/*

# 2. 修复 JAVA_HOME：利用 TARGETARCH 动态设置路径
# Ubuntu 的路径规律是 java-11-openjdk-amd64 或 java-11-openjdk-arm64
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$TARGETARCH
ENV PATH=$JAVA_HOME/bin:$PATH

# 下载并解压 GDAL
WORKDIR /src
RUN wget https://download.osgeo.org/gdal/3.12.0/gdal-3.12.0.tar.gz \
    && tar -xvf gdal-3.12.0.tar.gz

# 3. 编译 GDAL
# 显式指定 JAVA 相关的 JNI 路径确保编译成功
WORKDIR /src/gdal-3.12.0/build
RUN cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DGDAL_BUILD_OPTIONAL_DRIVERS=ON \
    -DBUILD_JAVA_BINDINGS=ON \
    -DBUILD_PYTHON_BINDINGS=ON \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DGDAL_USE_INTERNAL_LIBS=WHEN_NO_EXTERNAL \
    && cmake --build . -j$(nproc) \
    && cmake --install .

# Stage 2: Runtime Stage
FROM ubuntu:22.04

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 4. 安装运行时必备库 (保持与 Builder 阶段版本一致)
RUN apt-get update && apt-get install -y \
    libproj-dev \
        libgeos-dev \
        libsqlite3-dev \
        libtiff-dev \
        libgeotiff-dev \
        libjpeg-dev \
        libpng-dev \
        libwebp-dev \
        libzstd-dev \
        liblzma-dev \
        libcurl4-openssl-dev \
        libhdf5-dev \
        libnetcdf-dev \
        bison \
        flex \
    openjdk-11-jre-headless \
    python3-minimal \
    locales \
    && locale-gen en_US.UTF-8 \
    && update-locale LANG=en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# 拷贝编译成果
COPY --from=builder /usr/local /usr/local
RUN ldconfig

# 5. 设置环境变量
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$TARGETARCH
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
ENV GDAL_DATA=/usr/local/share/gdal
# Java 11+ 访问 JNI 原生内存必须开启此选项
ENV JAVA_OPTS="--add-opens java.base/java.lang=ALL-UNNAMED"
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

WORKDIR /app
CMD ["gdalinfo", "--version"]