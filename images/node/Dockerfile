# m8aio/node-dev:0.2.7
# Base image for Node.js development environments
# Wolfi Base
FROM cgr.dev/chainguard/wolfi-base

LABEL org.opencontainers.image.version="0.2.7" \
      org.opencontainers.image.title="m8aio/node-dev" \
      org.opencontainers.image.description="Base image for Node.js development environments"

# Run everything as root initially for setup
USER root

# Install core dependencies (The 80%) via APK (Wolfi)
ARG NODE_VERSION=24
ARG GH_VERSION=2.74.0
ARG PNPM_VERSION=10.34.1
RUN apk update && apk add --no-cache \
    nodejs-${NODE_VERSION} \
    npm \
    git \
    build-base \
    python3 \
    make \
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

# Install gh CLI from official release (not in Wolfi apk registry)
RUN ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/') && \
    curl -fsSL "https://github.com/cli/cli/releases/download/v${GH_VERSION}/gh_${GH_VERSION}_linux_${ARCH}.tar.gz" \
      -o /tmp/gh.tar.gz && \
    tar -xzf /tmp/gh.tar.gz -C /tmp && \
    mv /tmp/gh_${GH_VERSION}_linux_${ARCH}/bin/gh /usr/local/bin/gh && \
    rm -rf /tmp/gh.tar.gz /tmp/gh_${GH_VERSION}_linux_${ARCH}

# Create 'dev' user
RUN useradd dev \
    --create-home \
    --shell=/bin/zsh \
    --groups=root \
    --uid=1001 \
    --user-group && \
    mkdir -p /etc/sudoers.d && \
    echo "dev ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/nopasswd

# Install global Node packages as dev user
# pnpm is installed globally so it's available outside of Rush contexts;
# Rush manages its own pinned version via rush.json for project workflows.
USER dev
ENV PATH="/home/dev/.npm-global/bin:$PATH"
RUN npm config set prefix /home/dev/.npm-global && \
    npm install -g pnpm@${PNPM_VERSION}

WORKDIR /home/dev
