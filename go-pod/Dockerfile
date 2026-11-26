# syntax=docker/dockerfile:1.6
ARG GO_VERSION

FROM golang:${GO_VERSION}-trixie AS final
ARG TARGETARCH
ARG GO_VERSION
ARG GOLANGCI_LINT_VERSION
ARG GOVULNCHECK_VERSION

# Prebuilt release binary, sha256 taken from the release's checksums file (asset picked by TARGETARCH: amd64, arm64).
# golangci-lint refuses a module whose go.mod `go` directive is newer than the Go it was built with, so the build
# fails when the binary's built-with Go (major.minor) is older than GO_VERSION's: a patch difference is not enforced by golangci-lint
# (verified: a binary built with go1.27.0 lints a module with `go 1.27.4` in golang:1.27.4).
RUN set -eu; \
    ver="${GOLANGCI_LINT_VERSION#v}"; \
    base="https://github.com/golangci/golangci-lint/releases/download/${GOLANGCI_LINT_VERSION}"; \
    asset="golangci-lint-${ver}-linux-${TARGETARCH}.tar.gz"; \
    cd /tmp; \
    curl -fsSLO "${base}/${asset}"; \
    curl -fsSL "${base}/golangci-lint-${ver}-checksums.txt" | grep -E "  ${asset}\$" > checksum.txt; \
    sha256sum -c checksum.txt; \
    tar -xzf "${asset}" "golangci-lint-${ver}-linux-${TARGETARCH}/golangci-lint"; \
    install -m 0755 "golangci-lint-${ver}-linux-${TARGETARCH}/golangci-lint" /usr/local/bin/golangci-lint; \
    rm -rf /tmp/golangci-lint-* /tmp/checksum.txt; \
    built="$(golangci-lint --version | sed -n 's/.* built with go\([0-9][0-9.]*\).*/\1/p')"; \
    [ -n "$built" ] || { echo "ERROR: cannot read the built-with Go from: $(golangci-lint --version)" >&2; exit 1; }; \
    minor() { echo "$1" | cut -d. -f1,2; }; \
    oldest="$(printf '%s\n%s\n' "$(minor "$built")" "$(minor "${GO_VERSION}")" | sort -V | head -n1)"; \
    if [ "$(minor "$built")" != "$(minor "${GO_VERSION}")" ] && [ "$oldest" = "$(minor "$built")" ]; then \
      echo "ERROR: golangci-lint ${GOLANGCI_LINT_VERSION} is built with go${built}, older than GO_VERSION ${GO_VERSION}; it would refuse go.mod files with go ${GO_VERSION%.*}. Use a golangci-lint release built with Go >= ${GO_VERSION%.*} (app-version.txt) or drop this Go version (go-versions.txt)." >&2; \
      exit 1; \
    fi

# Installed in this stage without overrides, so the module and build caches stay in the image and
# `go run golang.org/x/vuln/cmd/govulncheck@<same version>` (the repos' `make vuln`) is served from the cache (only a module lookup goes to the network).
# govulncheck publishes no release binaries (golang/vuln has no release assets), so it stays a go install.
RUN go install golang.org/x/vuln/cmd/govulncheck@${GOVULNCHECK_VERSION}
