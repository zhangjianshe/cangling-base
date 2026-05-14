# --- Stage 1: Builder ---
FROM ubuntu:22.04 AS builder
ARG NGINX_VERSION=1.31.0

# 1. Install EVERYTHING needed to build
RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

# 2. Set WORKDIR and then download
WORKDIR /src
RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzvf nginx-${NGINX_VERSION}.tar.gz && \
    git clone https://github.com/arut/nginx-rtmp-module.git && \
    git clone https://github.com/aperezdc/ngx-fancyindex.git

# 3. Build
RUN cd nginx-${NGINX_VERSION} && \
    ./configure \
    --prefix=/usr/local/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --add-module=../nginx-rtmp-module \
    --add-module=../ngx-fancyindex \
    --with-http_ssl_module \
    && make -j$(nproc) && make install

# 4. Move stat.xsl to a predictable place BEFORE finishing Stage 1
RUN cp /src/nginx-rtmp-module/stat.xsl /usr/local/nginx/html/stat.xsl

# --- Stage 2: Final Image ---
FROM ubuntu:22.04
LABEL authors="satway"

RUN apt-get update && apt-get install -y \
    libpcre3 libssl3 zlib1g \
    && rm -rf /var/lib/apt/lists/*

# Copy everything from the builder's install path
COPY --from=builder /usr/local/nginx /usr/local/nginx
COPY --from=builder /etc/nginx /etc/nginx

RUN mkdir -p /var/log/nginx /logs /tmp/hls && \
    chown -R nobody:nogroup /var/log/nginx /logs /tmp/hls

EXPOSE 1935 8080
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]