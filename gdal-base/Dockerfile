FROM ghcr.io/osgeo/gdal:ubuntu-full-3.8.0

COPY environment /etc/environment
COPY locale /etc/default/locale

RUN apt-get update && \
    apt-get install -y openjdk-11-jdk && \
    apt-get clean