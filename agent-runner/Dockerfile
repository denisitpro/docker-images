# syntax=docker/dockerfile:1.6
ARG GO_VERSION
ARG BUN_VERSION

FROM oven/bun:${BUN_VERSION} AS bun

FROM golang:${GO_VERSION}-trixie AS tools
ARG CUE_VERSION
ARG GOLANGCI_LINT_VERSION
ARG GOVULNCHECK_VERSION
ARG OASDIFF_VERSION
ENV GOBIN=/out GOFLAGS=-buildvcs=false CGO_ENABLED=0
RUN go install cuelang.org/go/cmd/cue@${CUE_VERSION} && \
    go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@${GOLANGCI_LINT_VERSION} && \
    go install golang.org/x/vuln/cmd/govulncheck@${GOVULNCHECK_VERSION} && \
    go install github.com/oasdiff/oasdiff@${OASDIFF_VERSION}

FROM golang:${GO_VERSION}-trixie AS final

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    make \
    git \
    bash \
    curl \
    jq \
    ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY --from=bun /usr/local/bin/bun /usr/local/bin/bun
COPY --from=tools /out/ /usr/local/bin/
