# Stage 1: Build Stage
# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

# 必须声明这个 ARG 才能在 ENV 中使用内置变量
ARG TARGETARCH

ENV DEBIAN_FRONTEND=noninteractive

# ... 安装依赖的步骤保持不变 ...

# 修改这里：根据架构映射路径
# amd64 对应 x86_64, arm64 对应 aarch64
RUN if [ "$TARGETARCH" = "amd64" ]; then \
        echo "JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> /etc/environment; \
    else \
        echo "JAVA_HOME=/usr/lib/jvm/java-11-openjdk-arm64" >> /etc/environment; \
    fi

# 或者更直接地在 ENV 中使用转换逻辑（如果路径遵循特定规则）
# 对于 Ubuntu 11-jdk，路径通常就是这样：
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$TARGETARCH
# Download GDAL 3.12.0
WORKDIR /src
RUN wget https://download.osgeo.org/gdal/3.12.0/gdal-3.12.0.tar.gz \
    && tar -xvf gdal-3.12.0.tar.gz

# Build GDAL with CMake
WORKDIR /src/gdal-3.12.0/build
RUN cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DGDAL_BUILD_OPTIONAL_DRIVERS=ON \
    -DBUILD_JAVA_BINDINGS=ON \
    -DBUILD_PYTHON_BINDINGS=ON \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    && cmake --build . -j$(nproc) \
    && cmake --install .

# Stage 2: Runtime Stage
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install only runtime libraries
RUN apt-get update && apt-get install -y \
    libproj22 \
    libgeos-c1v5 \
    libsqlite3-0 \
    libtiff5 \
    libcurl4 \
    openjdk-11-jre-headless \
    python3-minimal \
    locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# Copy compiled binaries and libraries from builder
COPY --from=builder /usr/local /usr/local
RUN ldconfig

# Set Environment Variables for Java/JNI
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
# This is where your gdal.jar will be located
ENV GDAL_DATA=/usr/local/share/gdal
ENV JAVA_OPTS="--add-opens java.base/java.lang=ALL-UNNAMED"

# Set environment variables for UTF-8

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8
# (Optional) Copy custom config files if needed
COPY environment /etc/environment
COPY locale /etc/default/locale

WORKDIR /app
CMD ["gdalinfo", "--version"]