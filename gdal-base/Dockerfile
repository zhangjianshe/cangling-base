# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 安装编译依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential cmake wget git swig ant \
    python3-dev python3-numpy python3-setuptools python3-wheel \
    openjdk-11-jdk \
    libproj-dev libgeos-dev libsqlite3-dev \
    libtiff-dev libgeotiff-dev libjpeg-dev libpng-dev \
    libwebp-dev libzstd-dev liblzma-dev libcurl4-openssl-dev \
    libhdf5-dev libnetcdf-dev pkg-config bison flex \
    libxml2-dev libopenjp2-7-dev \
    && rm -rf /var/lib/apt/lists/*

# 2. 配置 JAVA_HOME
RUN export J_ARCH=$( [ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64" ) && \
    ln -s /usr/lib/jvm/java-11-openjdk-$J_ARCH /usr/lib/jvm/default-java
ENV JAVA_HOME=/usr/lib/jvm/default-java
ENV PATH=$JAVA_HOME/bin:$PATH

WORKDIR /src
RUN wget https://download.osgeo.org/gdal/3.13.0/gdal-3.13.0.tar.gz \
    && tar -xzf gdal-3.13.0.tar.gz

# 3. 编译 GDAL
WORKDIR /src/gdal-3.13.0/build
RUN cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DGDAL_BUILD_OPTIONAL_DRIVERS=ON \
    -DBUILD_JAVA_BINDINGS=ON \
    -DBUILD_PYTHON_BINDINGS=ON \
    -DGDAL_USE_LIBDEFLATE=ON \
    -DGDAL_USE_ZSTD=ON \
    -DBUILD_SHARED_LIBS=ON \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DGDAL_USE_INTERNAL_LIBS=WHEN_NO_EXTERNAL \
    -DJAVA_INCLUDE_PATH=$JAVA_HOME/include \
    -DJAVA_AWT_LIBRARY=$JAVA_HOME/lib/libjawt.so \
    -DJAVA_JVM_LIBRARY=$JAVA_HOME/lib/server/libjvm.so \
    && cmake --build . -j$(nproc) || cmake --build . -j1 \
    && cmake --install .

# =========================================================
# Stage 2: Runtime Stage
# =========================================================
FROM ubuntu:22.04

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 基础运行时依赖 + Docker 官方源依赖 + bsdutils (包含 script 工具)
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    tzdata \
    bsdutils \
    libgomp1 \
    libproj22 libgeos-c1v5 libsqlite3-0 libtiff5 libgeotiff5 \
    libjpeg8 libpng16-16 libwebp7 libzstd1 liblzma5 \
    libcurl4 libhdf5-103-1 libnetcdf19 \
    libxml2 libopenjp2-7 \
    openjdk-11-jre-headless python3-minimal locales \
    fontconfig \
    fonts-wqy-zenhei \
    fonts-wqy-microhei \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# 2. 核心新增：添加 Docker 官方 Apt 源并仅安装 Docker CLI & Docker Compose Plugin
RUN install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        docker-ce-cli \
        docker-compose-plugin && \
    rm -rf /var/lib/apt/lists/*

# 3. 设置时区 (Asia/Shanghai)
RUN ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure --frontend noninteractive tzdata

ENV TZ=Asia/Shanghai

# 4. 拷贝 GDAL 产物
COPY --from=builder /usr/local /usr/local
COPY --from=builder /usr/share/proj /usr/share/proj

# 5. 动态库 & JNI 映射
RUN ln -s /usr/local/lib/jni/libgdalalljni.so /usr/local/lib/libgdalalljni.so && \
    ldconfig

# 6. 配置环境变量
RUN export J_ARCH=$( [ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64" ) && \
    echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$J_ARCH" >> /etc/profile

ENV LD_LIBRARY_PATH=/usr/local/lib/jni:/usr/local/lib:/usr/lib/x86_64-linux-gnu
ENV PROJ_LIB=/usr/share/proj
ENV GDAL_DATA=/usr/local/share/gdal
ENV CLASSPATH=/usr/local/share/java/gdal.jar:$CLASSPATH

ENV JAVA_OPTS="-Xms4g -Xmx12g -XX:MaxDirectMemorySize=8g \
               -Djava.library.path=/usr/local/lib/jni \
               -Xbootclasspath/a:/usr/local/share/java/gdal.jar \
               --add-opens java.base/java.lang=ALL-UNNAMED \
               --add-opens java.base/java.nio=ALL-UNNAMED"

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

WORKDIR /app

# 校验 Docker CLI, Docker Compose 及 GDAL 是否完全可用
CMD ["sh", "-c", "docker --version && docker compose version && gdalinfo --version"]