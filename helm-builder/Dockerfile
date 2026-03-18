# Build arguments for version control
ARG ALPINE_VERSION=3.22

# Stage 1: Version Configuration Reader
# Reads version specifications from version.txt file
# Expected format: alpine=3.18, helm=3.12.0

FROM alpine:${ALPINE_VERSION} AS version-reader
WORKDIR /tmp
COPY version.txt .
RUN echo "export $(cat version.txt | xargs)" > versions.env

# Stage 2: Main Application Image
FROM alpine:${ALPINE_VERSION}

# Build arguments for version control and architecture targeting
# These can be overridden during build time if needed
ARG ALPINE_VERSION
ARG HELM_VERSION
ARG YQ_VERSION
ARG TARGETARCH
ENV TARGETARCH=${TARGETARCH}

# Install core dependencies required for package management and JSON processing
RUN apk add --no-cache curl jq

# Install yq YAML processor with multi-architecture support
# Automatically detects and installs the appropriate binary for the target platform
RUN ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/') && \
    curl -fsSL "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${ARCH}" -o /usr/bin/yq && \
    chmod +x /usr/bin/yq

# Install Helm package manager with version pinning and multi-architecture support
# Version is controlled via HELM_VERSION build argument (expects format: v3.17.3)
RUN ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/') && \
    curl -fsSL "https://get.helm.sh/helm-${HELM_VERSION}-linux-${ARCH}.tar.gz" \
    | tar -xzO linux-${ARCH}/helm > /usr/bin/helm && \
    chmod +x /usr/bin/helm

WORKDIR /app
CMD ["/bin/sh"]