# go-pod

`denisitpro/go-pod:<go directive of go.mod>`: `golang:<ver>-trixie` (Go, git, make, gcc, upstream user root, upstream GOPATH/GOFLAGS, no entrypoint) plus `golangci-lint` (`/usr/local/bin`, prebuilt release binary) and `govulncheck` (`/go/bin`, module and build caches kept so `go run golang.org/x/vuln/cmd/govulncheck@<same version>` is a cache hit: 3 s instead of 12 s measured). Versions: `app-version.txt`.

Tags per Go version: `X.Y.Z` (exact) and `X.Y` (floating, for a go.mod with `go X.Y`). A go.mod directive `X.Y.Z` that is not listed has no image. `latest` is the highest version in `go-versions.txt` (`sort -V`, not line order); the `matrix` job outputs it and only the matrix leg building that version adds the tag, so a newly appended higher version takes `latest` on its own.

golangci-lint: the `golangci-lint-<ver>-linux-<TARGETARCH>.tar.gz` asset of the `GOLANGCI_LINT_VERSION` release (amd64, arm64), sha256-checked against the release's `-checksums.txt`; nothing is compiled. golangci-lint refuses a module whose go.mod `go` directive is newer than the Go it was built with, so the Dockerfile reads `golangci-lint --version` ("built with goX.Y.Z") and fails the build when its major.minor is lower than `GO_VERSION`'s (patch is not enforced by golangci-lint). govulncheck publishes no release binaries and stays a `go install`.

Add a Go version: append a line to `go-versions.txt` (full `X.Y.Z`), merge to `main`; the workflow builds every listed version and pushes both tags. Changing `app-version.txt` rebuilds all versions and overwrites their tags.
