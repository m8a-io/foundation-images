# m8aio/node-dev
# Base image for Node.js development environments
# Wolfi Base
FROM cgr.dev/chainguard/wolfi-base

# Run everything as root initially for setup
USER root

# Install core dependencies (The 80%) via APK (Wolfi)
ARG NODE_VERSION=24
RUN apk update && apk add --no-cache \
    nodejs-${NODE_VERSION} \
    npm \
    git \
    build-base \
    python3 \
    make \
    g++ \
    openssh-client \
    curl \
    bash \
    zsh \
    vim \
    jq \
    sudo \
    procps \
    wget \
    net-tools \
    shadow \
    unzip \
    gzip

# Create 'dev' user
RUN useradd dev \
    --create-home \
    --shell=/bin/zsh \
    --groups=root \
    --uid=1001 \
    --user-group && \
    mkdir -p /etc/sudoers.d && \
    echo "dev ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/nopasswd

# Install global Node packages setup
USER dev
RUN mkdir /home/dev/.npm-global
ENV npm_config_prefix=/home/dev/.npm-global
ENV PATH="/home/dev/.npm-global/bin:$PATH"

WORKDIR /home/dev
