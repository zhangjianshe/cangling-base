# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 完善依赖
RUN apt-get update && \
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
    libxml2-dev libopenjp2-7-dev \
    && rm -rf /var/lib/apt/lists/*

# 2. 动态修复 JAVA_HOME
RUN export J_ARCH=$( [ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64" ) && \
    ln -s /usr/lib/jvm/java-11-openjdk-$J_ARCH /usr/lib/jvm/default-java
ENV JAVA_HOME=/usr/lib/jvm/default-java
ENV PATH=$JAVA_HOME/bin:$PATH

WORKDIR /src
RUN wget https://download.osgeo.org/gdal/3.12.3/gdal-3.12.3.tar.gz \
    && tar -xzf gdal-3.12.3.tar.gz

# 3. 编译 GDAL
WORKDIR /src/gdal-3.12.3/build
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

# Stage 2: Runtime Stage
FROM ubuntu:22.04

ARG TARGETARCH
ENV DEBIAN_FRONTEND=noninteractive

# 1. 安装运行时必备库 (注意：去掉了末尾多余的反斜杠)
RUN apt-get update && apt-get install -y \
    tzdata \
    libproj22 libgeos-c1v5 libsqlite3-0 libtiff5 libgeotiff5 \
    libjpeg8 libpng16-16 libwebp7 libzstd1 liblzma5 \
    libcurl4 libhdf5-103-1 libnetcdf19 \
    libxml2 libopenjp2-7 \
    openjdk-11-jre-headless python3-minimal locales \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# 2. 设置时区 (Asia/Shanghai)
RUN ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    dpkg-reconfigure --frontend noninteractive tzdata

ENV TZ=Asia/Shanghai

# 3. 拷贝编译成果
COPY --from=builder /usr/local /usr/local
COPY --from=builder /usr/share/proj /usr/share/proj

# 4. 刷新动态库 & 建立 JNI 软链接
RUN ln -s /usr/local/lib/jni/libgdalalljni.so /usr/local/lib/libgdalalljni.so && \
    ldconfig

# 5. 设置环境变量 (修正 JAVA_HOME 变量在运行时的准确性)
RUN export J_ARCH=$( [ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64" ) && \
    echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$J_ARCH" >> /etc/profile
ENV LD_LIBRARY_PATH=/usr/local/lib/jni:/usr/local/lib:/usr/lib/x86_64-linux-gnu
ENV PROJ_LIB=/usr/share/proj
ENV GDAL_DATA=/usr/local/share/gdal
ENV CLASSPATH=/usr/local/share/java/gdal.jar:$CLASSPATH

# 6. JVM 内存与 JNI 权限 (增加 MaxDirectMemorySize 适配 98TiB 数据处理)
ENV JAVA_OPTS="-Xms4g -Xmx12g -XX:MaxDirectMemorySize=8g \
               -Djava.library.path=/usr/local/lib/jni \
               -Xbootclasspath/a:/usr/local/share/java/gdal.jar \
               --add-opens java.base/java.lang=ALL-UNNAMED \
               --add-opens java.base/java.nio=ALL-UNNAMED"

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

WORKDIR /app
# 验证：检查版本与时区
CMD ["sh", "-c", "date && gdalinfo --version && gdalinfo --formats"]