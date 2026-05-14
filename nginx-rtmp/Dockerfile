# --- Stage 1: Builder ---
FROM ubuntu:22.04 AS builder
ARG NGINX_VERSION=1.31.0

RUN apt-get update && apt-get install -y \
    build-essential libpcre3-dev libssl-dev zlib1g-dev wget git \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -xzvf nginx-${NGINX_VERSION}.tar.gz && \
    git clone https://github.com/arut/nginx-rtmp-module.git && \
    git clone https://github.com/aperezdc/ngx-fancyindex.git

RUN cd nginx-${NGINX_VERSION} && \
    ./configure --add-module=../nginx-rtmp-module \
        --add-module=../ngx-fancyindex \
        --prefix=/usr/local/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-threads \
        --pid-path=/var/run/nginx.pid  && \
    make && make install

# --- Stage 2: Final Image ---
FROM ubuntu:22.04
LABEL authors="satway"

# Only install RUNTIME dependencies (no compilers)
RUN apt-get update && apt-get install -y \
    libpcre3 libssl3 zlib1g \
    && rm -rf /var/lib/apt/lists/*

# Copy only the compiled output from the builder
COPY --from=builder /usr/local/nginx /usr/local/nginx
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/local/nginx/html/stat.xsl /usr/local/nginx/html/stat.xsl

# Set up directories
RUN mkdir -p /tmp/hls /var/log/nginx /logs && \
    chown -R nobody:nogroup /tmp/hls /var/log/nginx /logs

EXPOSE 1935 8080
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]