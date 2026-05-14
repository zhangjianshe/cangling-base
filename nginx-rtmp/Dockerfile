# --- Stage 1: Builder ---
FROM ubuntu:22.04 AS builder
ARG NGINX_VERSION=1.31.0

RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /src
RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzvf nginx-${NGINX_VERSION}.tar.gz && \
    git clone https://github.com/arut/nginx-rtmp-module.git && \
    git clone https://github.com/aperezdc/ngx-fancyindex.git

# 3. Build with most common default modules enabled
RUN cd nginx-${NGINX_VERSION} && \
    ./configure \
    --prefix=/usr/local/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/usr/local/nginx/sbin/nginx \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --add-module=../nginx-rtmp-module \
    --add-module=../ngx-fancyindex \
    --with-threads \
    --with-file-aio \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    && make -j$(nproc) && make install

# Ensure stat.xsl is in the html directory
RUN cp /src/nginx-rtmp-module/stat.xsl /usr/local/nginx/html/stat.xsl

# --- Stage 2: Final Image ---
FROM ubuntu:22.04
LABEL authors="satway"

# Only install RUNTIME dependencies
RUN apt-get update && apt-get install -y \
    libpcre3 libssl3 zlib1g \
    && rm -rf /var/lib/apt/lists/*

# Copy compiled binary and config
COPY --from=builder /usr/local/nginx /usr/local/nginx
COPY --from=builder /etc/nginx /etc/nginx

# Ensure all log and runtime directories exist
RUN mkdir -p /var/log/nginx /logs /tmp/hls /var/run && \
    chown -R nobody:nogroup /var/log/nginx /logs /tmp/hls /var/run

EXPOSE 1935 8080
# Explicitly path the binary to ensure it uses your custom build
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]