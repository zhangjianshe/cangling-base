FROM ubuntu:22.04
LABEL authors="satway"

# --- 定义版本参数，默认值为 1.31.0 ---
ARG NGINX_VERSION=1.31.0

RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

# 使用 ${NGINX_VERSION} 替代具体版本号
RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz &&  \
    tar -xzvf nginx-${NGINX_VERSION}.tar.gz &&  \
    git clone https://github.com/arut/nginx-rtmp-module.git

RUN cd nginx-${NGINX_VERSION} && \
    ./configure --add-module=../nginx-rtmp-module \
    --with-http_ssl_module \
    --prefix=/usr/local/nginx \
    --with-http_sub_module \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --with-threads \
    --with-file-aio \
    --with-http_mp4_module \
    --with-http_flv_module && \
    make && make install

RUN cp ./nginx-rtmp-module/stat.xsl  /usr/local/nginx/html/stat.xsl

# 清理时同样使用变量，确保删除正确的目录和压缩包
RUN rm -rf nginx-${NGINX_VERSION} nginx-${NGINX_VERSION}.tar.gz nginx-rtmp-module

RUN mkdir -p /tmp/hls /var/log/nginx && \
    chown -R nobody:nogroup /tmp/hls /var/log/nginx

EXPOSE 1935 8080

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]