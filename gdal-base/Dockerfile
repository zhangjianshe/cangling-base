FROM ghcr.io/osgeo/gdal:ubuntu-full-3.8.0

# Install locales package, generate en_US.UTF-8, and install openjdk-17-jdk
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:openjdk-r/ppa && \
    apt-get update && \
    apt-get install -y locales openjdk-17-jdk && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
ENV JAVA_OPTS="--add-opens java.base/java.lang=ALL-UNNAMED"
# Set environment variables for UTF-8
ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# (Optional) Copy custom config files if needed
COPY environment /etc/environment
COPY locale /etc/default/locale