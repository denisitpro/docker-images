# syntax=docker/dockerfile:1.6
# Set the Ubuntu image version as a build argument
ARG UBUNTU_IMAGE_VERSION=24.04
ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG KUBECTL_VERSION
ARG YQ_VERSION

# Use the official Ubuntu base image
FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS base

ENV VAULT_VERSION=${VAULT_VERSION} \
    CONSUL_VERSION=${CONSUL_VERSION} \
    HELM_VERSION=${HELM_VERSION} \
    KUBECTL_VERSION=${KUBECTL_VERSION} \
    YQ_VERSION=${YQ_VERSION}


# multi arch support
ARG TARGETARCH
ARG TARGETOS

# Re-declare all other build args AFTER FROM
ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG KUBECTL_VERSION
ARG YQ_VERSION

# Set the maintainer label
LABEL maintainer="denis.itpro@gmail.com"



# Теперь эти переменные доступны в командах RUN
RUN echo "build for OS: ${TARGETOS}"
RUN echo "build for arch: ${TARGETARCH}"

RUN apt-get update && \
    apt-get install -y \
    curl \
    wget \
    unzip \
    tar

RUN echo "Using: VAULT_VERSION=$VAULT_VERSION, CONSUL_VERSION=$CONSUL_VERSION, HELM_VERSION=$HELM_VERSION, KUBECTL_VERSION=$KUBECTL_VERSION, YQ_VERSION=$YQ_VERSION, TARGETARCH=$TARGETARCH"
RUN curl -L -o /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${TARGETARCH} && chmod +x /usr/local/bin/yq

RUN curl -L -o vault.zip https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_${TARGETARCH}.zip
RUN ls -la
RUN unzip -o vault.zip
RUN mv vault /usr/local/bin/
RUN chmod +x /usr/local/bin/vault

RUN ls -la /usr/local/bin/

RUN curl -L -o consul.zip https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_${TARGETARCH}.zip
RUN ls -la
RUN unzip -o consul.zip
RUN mv consul /usr/local/bin/
RUN chmod +x /usr/local/bin/consul

RUN wget "https://get.helm.sh/helm-${HELM_VERSION}-linux-${TARGETARCH}.tar.gz"
RUN ls -la
RUN tar -xvf helm-${HELM_VERSION}-linux-${TARGETARCH}.tar.gz
RUN mv linux-${TARGETARCH}/helm /usr/local/bin/
RUN chmod +x /usr/local/bin/helm

RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl
RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256
RUN echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
RUN chmod +x kubectl
RUN mv kubectl /usr/local/bin/


FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS final

# multi arch support
ARG TARGETARCH
ARG TARGETOS

WORKDIR /app
# Copy the necessary files from the base image
COPY --from=base /usr/local/bin/* /usr/local/bin/
RUN ls -la /usr/local/bin/

Update the package list and install the necessary utilities
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