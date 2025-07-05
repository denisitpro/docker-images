# syntax=docker/dockerfile:1.6
# Set the Ubuntu image version as a build argument
ARG UBUNTU_IMAGE_VERSION=24.04

# Multi-arch build args
ARG TARGETPLATFORM
ARG VAULT_VERSION
ARG CONSUL_VERSION
ARG HELM_VERSION
ARG HELM_DISTRO
ARG KUBECTL_VERSION
ARG YQ_VERSION

# Use the official Ubuntu base image
FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS base

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

# Set the maintainer label
LABEL maintainer="denis.itpro@gmail.com"

RUN echo ${TARGETARCH}
#
# RUN apt-get update && \
#     apt-get install -y \
#     curl \
#     wget \
#     unzip \
#     tar
#
# RUN echo "Using: VAULT_VERSION=$VAULT_VERSION, CONSUL_VERSION=$CONSUL_VERSION, HELM_VERSION=$HELM_VERSION, KUBECTL_VERSION=$KUBECTL_VERSION, YQ_VERSION=$YQ_VERSION, TARGETPLATFORM=$TARGETPLATFORM"
# RUN arch=$(echo "$TARGETPLATFORM" | cut -d'/' -f2) && \
#     curl -L -o /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${arch} && chmod +x /usr/local/bin/yq

# RUN arch=$(echo "$TARGETPLATFORM" | cut -d'/' -f2) && \
#     curl -L -o vault.zip https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_${arch}.zip
# RUN ls -la
# RUN unzip -o vault.zip
# RUN mv vault /usr/local/bin/
# RUN chmod +x /usr/local/bin/vault
#
# RUN arch=$(echo "$TARGETPLATFORM" | cut -d'/' -f2) && \
#     curl -L -o consul.zip https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_${arch}.zip
# RUN ls -la
# RUN unzip -o consul.zip
# RUN mv consul /usr/local/bin/
# RUN chmod +x /usr/local/bin/consul
#
# RUN wget "https://get.helm.sh/helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz"
# RUN ls -la
# RUN tar -xvf helm-${HELM_VERSION}-${HELM_DISTRO}.tar.gz
# RUN mv ${HELM_DISTRO}/helm /usr/local/bin/
# RUN chmod +x /usr/local/bin/helm
#
# RUN arch=$(echo "$TARGETPLATFORM" | cut -d'/' -f2) && \
#     curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${arch}/kubectl
# RUN curl -LO https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${arch}/kubectl.sha256
# RUN echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# RUN chmod +x kubectl
# RUN mv kubectl /usr/local/bin/


FROM ubuntu:${UBUNTU_IMAGE_VERSION} AS final

ARG TARGETPLATFORM
ARG TARGETARCH

WORKDIR /app
# Copy the necessary files from the base image
COPY --from=base /usr/local/bin/* /usr/local/bin/

# Update the package list and install the necessary utilities
# RUN apt-get update && \
#     apt-get install -y \
#     net-tools \
#     nano \
#     iputils-ping \
#     tar \
#     curl \
#     wget \
#     netcat-traditional \
#     tcpdump \
#     traceroute \
#     telnet \
#     nmap \
#     jq \
#     dnsutils && \
#     apt-get clean && \
#     rm -rf /var/lib/apt/lists/*


# Set a default command for the container
CMD ["bash"]