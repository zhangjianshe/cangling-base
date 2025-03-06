FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install git openjdk-11-jdk ant maven -y \
    && apt install build-essential -y \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash \
    && nvm install --lts && nvm use --lts \
    && echo "Hello, build environment is finished" \
    && echo "tzdata tzdata/areas select Asia" | debconf-set-selections \
    && apt-get install -y tzdata

