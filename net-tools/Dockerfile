# Set the Ubuntu image version as a build argument
ARG UBUNTU_IMAGE_VERSION=24.04

# Use the official Ubuntu base image
FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS base

ENV VAULT_VERSION=1.19.3 \
    CONSUL_VERSION=1.21.0 \
    HELM_VERSION=v3.17.3 \
    HELM_DISTRO=linux-amd64 \
    KUBECTL_VERSION=v1.33.0
    YQ_VERSION=v4.45.2

# Re-declare all other build args AFTER FROM
ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG HELM_DISTRO
ARG KUBECTL_VERSION
ARG YQ_VERSION

# Set the maintainer label
LABEL maintainer="denis.itpro@gmail.com"

RUN apt-get update && \
    apt-get install -y \
    curl \
    wget \
    unzip \
    tar

RUN echo "Using: VAULT_VERSION=$VAULT_VERSION, CONSUL_VERSION=$CONSUL_VERSION, HELM_VERSION=$HELM_VERSION, KUBECTL_VERSION=$KUBECTL_VERSION, YQ_VERSION=$YQ_VERSION"
RUN wget https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64
RUN chmod +x yq_linux_amd64
RUN mv yq_linux_amd64 /usr/local/bin/yq

RUN wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip
RUN unzip -o vault_${VAULT_VERSION}_linux_amd64.zip
RUN mv vault /usr/local/bin/
RUN chmod +x /usr/local/bin/vault

RUN wget https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip
RUN unzip -o consul_${CONSUL_VERSION}_linux_amd64.zip
RUN mv consul /usr/local/bin/
RUN chmod +x /usr/local/bin/consul

RUN wget "https://get.helm.sh/helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz"
RUN tar -xvf helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz
RUN mv ${HELM_DISTRO}/helm /usr/local/bin/
RUN chmod +x /usr/local/bin/helm

RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl.sha256
RUN echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
RUN chmod +x kubectl
RUN mv kubectl /usr/local/bin/


FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS final

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