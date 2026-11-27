# syntax=docker/dockerfile:1.6
ARG GO_VERSION
ARG BUN_VERSION

FROM oven/bun:${BUN_VERSION} AS bun

# govulncheck publishes no release binaries (golang/vuln has no release assets), so it is the only tool that is built.
# Cross-compiled on the build host for TARGETARCH (pure Go, CGO off), so a linux/arm64 build does not compile under emulation.
FROM --platform=$BUILDPLATFORM golang:${GO_VERSION}-trixie AS govulncheck
ARG TARGETARCH
ARG GOVULNCHECK_VERSION
RUN : "${TARGETARCH:?}" "${GOVULNCHECK_VERSION:?}"
ENV GOPATH=/gp GOFLAGS=-buildvcs=false CGO_ENABLED=0 GOOS=linux
# go install refuses a cross-compile with GOBIN set and writes to $GOPATH/bin/linux_<arch>/ when GOARCH differs from the host's, so find the binary.
RUN set -eu; \
    GOARCH="${TARGETARCH}" go install golang.org/x/vuln/cmd/govulncheck@${GOVULNCHECK_VERSION}; \
    mkdir /out; \
    mv "$(find /gp/bin -type f -name govulncheck)" /out/govulncheck

FROM golang:${GO_VERSION}-trixie AS final
ARG TARGETARCH
ARG GO_VERSION
ARG CUE_VERSION
ARG GOLANGCI_LINT_VERSION
ARG OASDIFF_VERSION
ARG RTK_VERSION
RUN : "${TARGETARCH:?}" "${GO_VERSION:?}" "${CUE_VERSION:?}" "${GOLANGCI_LINT_VERSION:?}" "${OASDIFF_VERSION:?}" "${RTK_VERSION:?}"

# Everything the pod needs as root is installed here: nothing runs apt-get at run time.
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
COPY --from=govulncheck /out/govulncheck /usr/local/bin/govulncheck

# cue: release archive (amd64, arm64). The release has no checksums file, so the sha256 of both archives is pinned in
# cue-checksums.txt (sha256sum format, names carry the version); a CUE_VERSION without a matching line fails the build.
COPY cue-checksums.txt /tmp/cue-checksums.txt
RUN set -eu; \
    asset="cue_${CUE_VERSION}_linux_${TARGETARCH}.tar.gz"; \
    cd /tmp; \
    curl -fsSLO "https://github.com/cue-lang/cue/releases/download/${CUE_VERSION}/${asset}"; \
    grep -E "  ${asset}\$" cue-checksums.txt > checksum.txt; \
    sha256sum -c checksum.txt; \
    tar -xzf "${asset}" cue; \
    install -m 0755 cue /usr/local/bin/cue; \
    rm -f "${asset}" cue checksum.txt cue-checksums.txt; \
    cue version

# golangci-lint: same step as go-pod/Dockerfile. Release archive, sha256 from the release's checksums file.
# It refuses a module whose go.mod `go` directive is newer than the Go it was built with, so the build fails when the
# binary's built-with Go (major.minor) is older than GO_VERSION's (a patch difference is not enforced by golangci-lint).
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
      echo "ERROR: golangci-lint ${GOLANGCI_LINT_VERSION} is built with go${built}, older than GO_VERSION ${GO_VERSION}; it would refuse go.mod files with go ${GO_VERSION%.*}. Use a golangci-lint release built with Go >= ${GO_VERSION%.*} (app-version.txt)." >&2; \
      exit 1; \
    fi

# oasdiff: release archive, sha256 from the release's checksums.txt (names carry the version without the leading v).
RUN set -eu; \
    ver="${OASDIFF_VERSION#v}"; \
    base="https://github.com/oasdiff/oasdiff/releases/download/${OASDIFF_VERSION}"; \
    asset="oasdiff_${ver}_linux_${TARGETARCH}.tar.gz"; \
    cd /tmp; \
    curl -fsSLO "${base}/${asset}"; \
    curl -fsSL "${base}/checksums.txt" | grep -E "  ${asset}\$" > checksum.txt; \
    sha256sum -c checksum.txt; \
    tar -xzf "${asset}" oasdiff; \
    install -m 0755 oasdiff /usr/local/bin/oasdiff; \
    rm -f "${asset}" oasdiff checksum.txt; \
    oasdiff --version

# rtk: release archive, sha256 from the release's checksums.txt. v1.2.1 ships no x86_64 glibc
# tarball; both arches have a static musl build (x86_64 static-pie, aarch64 statically linked),
# which runs on the glibc trixie base. The archive name is the Rust triple, not TARGETARCH.
RUN set -eu; \
    case "${TARGETARCH}" in \
      amd64) rust_arch=x86_64 ;; \
      arm64) rust_arch=aarch64 ;; \
      *) echo "ERROR: unsupported TARGETARCH ${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    base="https://github.com/rtk-ai/rtk/releases/download/${RTK_VERSION}"; \
    asset="rtk-${rust_arch}-unknown-linux-musl.tar.gz"; \
    cd /tmp; \
    curl -fsSLO "${base}/${asset}"; \
    curl -fsSL "${base}/checksums.txt" | grep -E "  ${asset}\$" > checksum.txt; \
    sha256sum -c checksum.txt; \
    tar -xzf "${asset}" rtk; \
    install -m 0755 rtk /usr/local/bin/rtk; \
    rm -f "${asset}" rtk checksum.txt; \
    rtk --version

# Non-root, numeric uid:gid above 1000 (the kubelet verifies runAsNonRoot only for a numeric user). The pod sets no
# runAsUser, so this is the UID of every process; it owns HOME (AI CLIs install into ~/.local and ~/.grok at run time),
# and /work and /cache so that a plain `docker run` volume inherits the ownership. emptyDir mounts shadow both
# directories at run time (kubelet creates an emptyDir 0777).
RUN groupadd --gid 10001 runner && \
    useradd --uid 10001 --gid 10001 --create-home --home-dir /home/runner --shell /bin/bash runner && \
    install -d -o runner -g runner /work /cache
ENV HOME=/home/runner
WORKDIR /work
USER 10001:10001
