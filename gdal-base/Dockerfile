# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

# 捕捉架构信息
ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 完善依赖
RUN apt-get update && \
    # 核心修复：防止 libc-bin 在 arm64 模拟器下崩溃
    mkdir -p /util-linux && \
    apt-get install -y --no-install-recommends eatmydata && \
    eatmydata apt-get install -y \
    build-essential cmake wget git swig ant \
    python3-dev python3-numpy python3-setuptools python3-wheel \
    openjdk-11-jdk \
    libproj-dev libgeos-dev libsqlite3-dev \
    libtiff-dev libgeotiff-dev libjpeg-dev libpng-dev \
    libwebp-dev libzstd-dev liblzma-dev libcurl4-openssl-dev \
    libhdf5-dev libnetcdf-dev pkg-config bison flex \
    && rm -rf /var/lib/apt/lists/*

# 2. 动态修复 JAVA_HOME 路径 (更健壮的写法)
RUN export J_ARCH=$( [ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64" ) && \
    ln -s /usr/lib/jvm/java-11-openjdk-$J_ARCH /usr/lib/jvm/default-java
ENV JAVA_HOME=/usr/lib/jvm/default-java
ENV PATH=$JAVA_HOME/bin:$PATH

WORKDIR /src
RUN wget https://download.osgeo.org/gdal/3.12.0/gdal-3.12.0.tar.gz \
    && tar -xzf gdal-3.12.0.tar.gz

# 3. 编译 GDAL
WORKDIR /src/gdal-3.12.0/build
# 关键修改：增加内存安全判断和 Java 路径硬连接
RUN cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DGDAL_BUILD_OPTIONAL_DRIVERS=ON \
    -DBUILD_JAVA_BINDINGS=ON \
    -DBUILD_PYTHON_BINDINGS=ON \
    # 强制 Python 绑定安装到系统目录，避免权限问题
    -DGDAL_PYTHON_INSTALL_PREFIX=/usr/local \
    -DBUILD_SHARED_LIBS=ON \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DGDAL_USE_INTERNAL_LIBS=WHEN_NO_EXTERNAL \
    -DJAVA_INCLUDE_PATH=$JAVA_HOME/include \
    -DJAVA_AWT_LIBRARY=$JAVA_HOME/lib/libjawt.so \
    -DJAVA_JVM_LIBRARY=$JAVA_HOME/lib/server/libjvm.so \
    -DJAVA_INCLUDE_PATH=/usr/lib/jvm/default-java/include \
    -DJAVA_AWT_LIBRARY=/usr/lib/jvm/default-java/lib/libjawt.so \
    -DJAVA_JVM_LIBRARY=/usr/lib/jvm/default-java/lib/server/libjvm.so \
    && cmake --build . -j$(nproc) || cmake --build . -j1 \
    && cmake --install .

# Stage 2: Runtime Stage
FROM ubuntu:22.04

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 4. 运行时环境
RUN apt-get update && apt-get install -y \
    libproj22 libgeos-c1v5 libsqlite3-0 libtiff5 libgeotiff5 \
    libjpeg8 libpng16-16 libwebp7 libzstd1 liblzma5 \
    libcurl4 libhdf5-103 libnetcdf19 \
    openjdk-11-jre-headless python3-minimal locales \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# 拷贝成果
COPY --from=builder /usr/local /usr/local
RUN ldconfig

# 5. 环境变量修复
# 修复报错的 LD_LIBRARY_PATH 定义方式
ENV LD_LIBRARY_PATH=/usr/local/lib
ENV GDAL_DATA=/usr/local/share/gdal
# 高光谱处理必备：增加内省权限
ENV JAVA_OPTS="--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED"
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

WORKDIR /app
CMD ["gdalinfo", "--version"]