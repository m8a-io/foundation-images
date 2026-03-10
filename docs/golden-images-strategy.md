# Foundation Images Strategy (Golden Base Images)

This document outlines the architectural and process strategy for managing "Golden" base images (e.g., `go-base`, `node-base`) in a dedicated repository.

## 1. Repository Structure

Create a new repository (e.g., `m8a-io/foundation-images`) with the following structure:

```text
foundation-images/
├── dagger/                 # Dagger Module for CI/CD
│   ├── main.go
│   ├── go.mod
│   └── ...
├── images/
│   ├── go/
│   │   ├── Dockerfile      # The Wolfi-based Go image
│   │   └── README.md
│   ├── node/
│   │   ├── Dockerfile      # The Wolfi-based Node image
│   │   └── README.md
│   └── ...
├── .github/                # GitHub Actions (calls Dagger)
└── README.md
```

## 2. Dockerfile Standards (Wolfi)

All images should be based on `cgr.dev/chainguard/wolfi-base`.

**Example (`images/go/Dockerfile`):**
```dockerfile
FROM cgr.dev/chainguard/wolfi-base

ARG GO_VERSION=1.25.5

# Install Core Tools
RUN apk add --no-cache \
    go~=${GO_VERSION} \
    git \
    build-base \
    ca-certificates \
    curl \
    bash

# Harden / Configure
WORKDIR /app
ENV GOPATH=/go
ENV PATH=$GOPATH/bin:/usr/local/go/bin:$PATH
```

## 3. Tagging Strategy

We strictly follow a **Semantic + Date** tagging scheme to ensure reproducibility while allowing rolling updates.

**Format:**
`[SoftwareVersion]-[BaseOS]-[Date]-[CommitShort]`

**Examples:**
*   `1.25.5-wolfi-20260109-a1b2c3d` (Immutable Release)
*   `1.25-wolfi` (Floating Major.Minor, updates nightly)
*   `latest` (Floating, not recommended for prod)

**Implementation in Dagger:**
```go
softwareVer := "1.25.5"
baseOS := "wolfi"
date := time.Now().Format("20060102")
commit := dag.Git().Commit().Short()

tag := fmt.Sprintf("%s-%s-%s-%s", softwareVer, baseOS, date, commit)
```

## 4. The Pipeline (Dagger)

The Dagger module in this repo should export a `PublishAll` function.

**Workflow:**
1.  **Build**: Build all Dockerfiles concurrently.
2.  **Test**: Run a smoke test (e.g., `go version` inside the container).
3.  **Scan**: Run `trivy image --exit-code 1 ...` to block high/critical CVEs.
4.  **Sign**: Use `cosign` to sign the image digest.
5.  **Publish**: Push to registry (`ttl.sh` for dev, `ghcr.io` or internal for prod).

## 5. Usage in Downstream Repos

In the Application Repos (like `m8a-go`), the `ci/main.go` module should reference these images:

```go
// ci/main.go

func (m *CI) Base(src *dagger.Directory) *dagger.Container {
    // Instead of building from scratch, use the golden image
    return dag.Container().
        From("m8a-io/foundation-images/go-base:1.25.5-wolfi-20260109").
        WithMountedDirectory("/src", src).
        WithWorkdir("/src")
}
```

## 6. Organizational Process

*   **Ownership**: Platform Engineering / DevOps.
*   **Cadence**: Nightly builds (to pick up upstream Wolfi security patches) + On-Commit builds (for Dockerfile changes).
*   **SLA**: Base images must be CVE-free (severity > medium) to ever reach specific tags.

---
**Next Steps for Setup:**
1. Initialize repo.
2. `dagger init --sdk go`.
3. Copy **Base Image Logic** (e.g. `images/go-dev`, `images/node-dev`) to the new repo.
   *   **Note**: Application images (e.g. `ci-engine`, `m8a-service`) **STAY** in the application repo. They will reference these new base images via `FROM`.
4. Implement `Publish` function with caching and tagging.
