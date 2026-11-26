# go-pod

`denisitpro/go-pod:<go directive of go.mod>`: `golang:<ver>-trixie` (Go, git, make, gcc, upstream user root, upstream GOPATH/GOFLAGS, no entrypoint) plus `golangci-lint` (`/usr/local/bin`) and `govulncheck` (`/go/bin`, module and build caches kept so `go run golang.org/x/vuln/cmd/govulncheck@<same version>` runs offline). Versions: `app-version.txt`.

Tags per Go version: `X.Y.Z` (exact) and `X.Y` (floating, for a go.mod with `go X.Y`). A go.mod directive `X.Y.Z` that is not listed has no image.

Add a Go version: append a line to `go-versions.txt` (full `X.Y.Z`), merge to `main`; the workflow builds every listed version and pushes both tags. Changing `app-version.txt` rebuilds all versions and overwrites their tags.
