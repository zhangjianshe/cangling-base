FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]
RUN apt update && apt install git curl openjdk-11-jdk ant maven -y \
    && apt install build-essential -y 

RUN bash -c "curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash"
RUN bash -c ". ~/.nvm/nvm.sh && nvm install --lts && nvm use --lts"
RUN echo "Hello, build environment is finished" \
    && echo "tzdata tzdata/areas select Asia" | debconf-set-selections \
    && apt-get install -y tzdata \
    && apt clean
