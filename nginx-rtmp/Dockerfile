FROM ubuntu:22.04
LABEL authors="satway"
RUN apt-get update && apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    zlib1g-dev \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/* 
RUN wget https://nginx.org/download/nginx-1.26.0.tar.gz &&  \
    tar -xzvf nginx-1.26.0.tar.gz &&  \
    git clone https://github.com/arut/nginx-rtmp-module.git
RUN cd nginx-1.26.0 && \
    ./configure --add-module=../nginx-rtmp-module \
    --with-http_ssl_module \
    --prefix=/usr/local/nginx \
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
RUN rm -rf nginx-1.26.0 nginx-1.26.0.tar.gz nginx-rtmp-module

RUN mkdir -p /tmp/hls /var/log/nginx && \
chown -R nobody:nogroup /tmp/hls /var/log/nginx

EXPOSE 1935 8080

CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
