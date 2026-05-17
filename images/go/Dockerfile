# Foundational image for Go services in the m8a platform.
#
# Targets (built via --target):
#   prod       Minimal runtime. Final FROM target for shipped service images.
#              Contains ca-certificates, tzdata, glibc. No Mise, no compilers.
#   dev-build  Extends prod with Mise + CGO build toolchain. Used by CI and
#              by Dockerfile builder stages in app repos.
#
# The Go runtime is NOT installed here. It is resolved at build time by Mise
# from the consuming repo's mise.toml. See ADR-INF-007.

# ---------------------------------------------------------------------------
# prod: minimal runtime layer
# ---------------------------------------------------------------------------
FROM cgr.dev/chainguard/wolfi-base AS prod

RUN apk add --no-cache \
    ca-certificates \
    tzdata

WORKDIR /app

# ---------------------------------------------------------------------------
# dev-build: extends prod with Mise + CGO toolchain
# ---------------------------------------------------------------------------
FROM prod AS dev-build

# Renovate-managed pin. Bump when Mise releases a new stable version.
ARG MISE_VERSION=2026.5.10

USER root

RUN apk add --no-cache \
    bash \
    build-base \
    curl \
    git \
    make

# MISE_DATA_DIR is decoupled from $HOME so the tool install cache is mountable
# as a Dagger cache volume at a stable, user-agnostic path.
ENV MISE_DATA_DIR=/opt/mise \
    MISE_INSTALL_PATH=/usr/local/bin/mise \
    PATH=/opt/mise/shims:/usr/local/bin:/usr/bin:/bin

RUN curl -fsSL https://mise.run | MISE_VERSION="v${MISE_VERSION}" sh \
    && mkdir -p "${MISE_DATA_DIR}" \
    && mise --version

WORKDIR /app
