# Stage 1: Build Stage
FROM ubuntu:22.04 AS builder

# Prevent interactive prompts during build
ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies
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
    libcurl4-openssl-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Set Java Home for JNI bindings
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-$(dpkg --print-architecture)

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