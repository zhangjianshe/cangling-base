FROM ghcr.io/osgeo/gdal:ubuntu-full-3.8.0

COPY environment /etc/environment
COPY locale /etc/default/locale

RUN apt-get update && \
    apt-get install -y locales && \
    locale-gen en_US.UTF-8 

RUN update-locale LANG=en_US.UTF-8 \
    &&  apt-get clean