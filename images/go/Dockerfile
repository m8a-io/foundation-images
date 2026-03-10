# m8aio/go-dev
# Base image for Go development environments
# Wolfi Base
FROM cgr.dev/chainguard/wolfi-base

# Run everything as root initially for setup
USER root

# Install core dependencies (The 80%) via APK (Wolfi)
ARG GO_VERSION=1.26
RUN apk update && apk add --no-cache \
    go-${GO_VERSION} \
    git \
    build-base \
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
# Wolfi uses standard useradd if 'shadow' package is installed or adduser
# We use UID 1001 to match previous
RUN useradd dev \
    --create-home \
    --shell=/bin/zsh \
    --groups=root \
    --uid=1001 \
    --user-group && \
    mkdir -p /etc/sudoers.d && \
    echo "dev ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/nopasswd

# Setup GOPATH
ENV GOPATH=/go
ENV PATH=$GOPATH/bin:/usr/local/go/bin:$PATH
RUN mkdir -p /go && chown -R dev:dev /go

# Set back to dev user
USER dev

WORKDIR /home/dev
