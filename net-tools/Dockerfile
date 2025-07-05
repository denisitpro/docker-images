# syntax=docker/dockerfile:1.6
# Set the Ubuntu image version as a build argument
ARG UBUNTU_IMAGE_VERSION=24.04

# Multi-arch build args
ARG TARGETPLATFORM
ARG BUILDPLATFORM

# Use the official Ubuntu base image
FROM --platform=$BUILDPLATFORM ubuntu:${UBUNTU_IMAGE_VERSION} AS base

# Enable buildx support for multi-arch

ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG HELM_DISTRO
ARG KUBECTL_VERSION
ARG YQ_VERSION
ARG TARGETPLATFORM
ARG BUILDPLATFORM

ENV VAULT_VERSION=${VAULT_VERSION} \
    CONSUL_VERSION=${CONSUL_VERSION} \
    HELM_VERSION=${HELM_VERSION} \
    HELM_DISTRO=${HELM_DISTRO} \
    KUBECTL_VERSION=${KUBECTL_VERSION} \
    YQ_VERSION=${YQ_VERSION}

# Re-declare all other build args AFTER FROM
ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG HELM_DISTRO
ARG KUBECTL_VERSION
ARG YQ_VERSION
ARG TARGETPLATFORM
ARG BUILDPLATFORM

# Set the maintainer label
LABEL maintainer="denis.itpro@gmail.com"

RUN apt-get update && \
    apt-get install -y \
    curl \
    wget \
    unzip \
    tar

RUN echo "Using: VAULT_VERSION=$VAULT_VERSION, CONSUL_VERSION=$CONSUL_VERSION, HELM_VERSION=$HELM_VERSION, KUBECTL_VERSION=$KUBECTL_VERSION, YQ_VERSION=$YQ_VERSION"
RUN wget https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${TARGETARCH}
RUN chmod +x yq_linux_${TARGETARCH}
RUN mv yq_linux_${TARGETARCH} /usr/local/bin/yq

RUN wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_${TARGETARCH}.zip
RUN unzip -o vault_${VAULT_VERSION}_linux_${TARGETARCH}.zip
RUN mv vault /usr/local/bin/
RUN chmod +x /usr/local/bin/vault

RUN wget https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_${TARGETARCH}.zip
RUN unzip -o consul_${CONSUL_VERSION}_linux_${TARGETARCH}.zip
RUN mv consul /usr/local/bin/
RUN chmod +x /usr/local/bin/consul

RUN wget "https://get.helm.sh/helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz"
RUN tar -xvf helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz
RUN mv ${HELM_DISTRO}/helm /usr/local/bin/
RUN chmod +x /usr/local/bin/helm

RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl
RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256
RUN echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
RUN chmod +x kubectl
RUN mv kubectl /usr/local/bin/


FROM --platform=$BUILDPLATFORM ubuntu:${UBUNTU_IMAGE_VERSION} AS final

ARG TARGETPLATFORM
ARG BUILDPLATFORM

WORKDIR /app
# Copy the necessary files from the base image
COPY --from=base /usr/local/bin/* /usr/local/bin/

# Update the package list and install the necessary utilities
RUN apt-get update && \
    apt-get install -y \
    net-tools \
    nano \
    iputils-ping \
    tar \
    curl \
    wget \
    netcat-traditional \
    tcpdump \
    traceroute \
    telnet \
    nmap \
    jq \
    dnsutils && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# Set a default command for the container
CMD ["bash"]