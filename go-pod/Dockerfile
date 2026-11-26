# syntax=docker/dockerfile:1.6
ARG GO_VERSION

FROM golang:${GO_VERSION}-trixie AS tools
ARG GOLANGCI_LINT_VERSION
ENV GOBIN=/out GOFLAGS=-buildvcs=false CGO_ENABLED=0
RUN go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@${GOLANGCI_LINT_VERSION}

FROM golang:${GO_VERSION}-trixie AS final
ARG GOVULNCHECK_VERSION
LABEL maintainer="denis.itpro@gmail.com"

COPY --from=tools /out/golangci-lint /usr/local/bin/golangci-lint
# Installed in this stage without overrides, so the module and build caches stay in the image and
# `go run golang.org/x/vuln/cmd/govulncheck@<same version>` (the repos' `make vuln`) is served from the cache (only a module lookup goes to the network).
RUN go install golang.org/x/vuln/cmd/govulncheck@${GOVULNCHECK_VERSION}
