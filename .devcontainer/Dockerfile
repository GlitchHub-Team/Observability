FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

ARG YQ_VERSION="v4.44.3"

RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        ca-certificates \
        curl \
        git \
        jq \
        make \
        unzip \
    && ARCH="$(dpkg --print-architecture)" \
    && case "${ARCH}" in \
        amd64) YQ_ARCH="amd64" ;; \
        arm64) YQ_ARCH="arm64" ;; \
        *) echo "Unsupported architecture: ${ARCH}" && exit 1 ;; \
    esac \
    && curl -fsSL "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${YQ_ARCH}" -o /usr/local/bin/yq \
    && chmod +x /usr/local/bin/yq \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
