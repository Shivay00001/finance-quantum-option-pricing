# finance-quantum-option-pricing

A lightweight Go HTTP microservice skeleton, containerized for portable deployment. Currently exposes a single health/status endpoint and is intended as a foundation for a quantitative finance / option-pricing service.

## 🚀 Overview

This repository contains a minimal, high-performance Go 1.20 service that:

- Starts an HTTP server on port **8080**
- Responds on `/` with a live system status message and current timestamp
- Is packaged as a minimal Alpine-based Docker image for deployment on any laptop or server

> **Note:** Despite the repository name, no option-pricing, Black-Scholes, Monte Carlo, or other quantitative finance logic is implemented yet. The current codebase is a service bootstrap only.

## ✨ Features

- **Zero external dependencies** — uses only the Go standard library (`net/http`, `fmt`, `time`, `log`)
- **Single-binary deployment** — compiles to one static executable
- **Docker-ready** — reproducible builds via the included `Dockerfile`
- **Fast startup** — Alpine-based image with minimal footprint

## 🏗️ Architecture / How It Works

The entire application lives in `main.go` (~20 lines):

1. **Entry point (`main`)** — registers a single HTTP handler and starts the server.
2. **Handler (`/`)** — on every request to the root path, writes a plain-text response:
   ```
   System Operational: 2026-01-01 12:00:00.000000000 +0000 UTC
   ```
   The timestamp is generated at request time via `time.Now()`, so each response confirms the server is alive and serving.
3. **Server bootstrap** — `http.ListenAndServe(":8080", nil)` binds to all interfaces on port 8080 using Go's default `ServeMux`. Startup and fatal errors are emitted through the standard `log` package; a listener failure terminates the process via `log.Fatal`.

### Request Flow

```
Client ──GET /──▶ Go net/http (port 8080) ──▶ handler ──▶ "System Operational: <timestamp>"
```

### File Layout

| File | Purpose |
|---|---|
| `main.go` | HTTP server + status endpoint |
| `go.mod` | Module definition (`go 1.20`), no dependencies |
| `Dockerfile` | Multi-stage-equivalent build: `golang:1.20-alpine` → compile → run |
| `.gitignore` | Excludes secrets, build artifacts, and OS noise |
| `LICENSE` | VisionQuantech Custom Commercial License (see below) |

## 🐳 Docker Deployment

The repository includes a `Dockerfile` and runs entirely with the standard Docker flow.

### Build and run

```bash
# Build the image
docker build -t finance-quantum-option-pricing .

# Run the container, mapping host port 8080 to container port 8080
docker run -d -p 8080:8080 --name option-pricing finance-quantum-option-pricing

# Verify
curl http://localhost:8080/
# → System Operational: <current timestamp>
```

### With Docker Compose (optional)

No `docker-compose.yml` is included, but this minimal one works:

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
```

Then:

```bash
docker-compose up -d --build
```

## 🛠️ Local Execution (without Docker)

Requires Go 1.20+:

```bash
go build -o app .
./app
# Server listens on http://localhost:8080
```

## ⚖️ Workability Assessment

An honest evaluation of the current state:

**What works:**
- ✅ The code compiles cleanly with Go 1.20 and has zero dependencies, so builds are reliable.
- ✅ The Dockerfile is valid and will produce a working container that serves the status endpoint on port 8080.
- ✅ The service is genuinely deployable on any laptop or server with Docker.

**What is missing / not production-ready:**
- ❌ **No actual option-pricing functionality exists.** There is no Black-Scholes model, no Monte Carlo engine, no Greeks calculation, no market data integration — nothing quantitative is implemented. The repository name currently overstates the contents.
- ❌ **No tests** (`*_test.go` files), no CI configuration, and no linting setup.
- ❌ **No graceful shutdown** — the server does not handle `SIGTERM`/`SIGINT` for clean container stops; in-flight requests are dropped on shutdown.
- ❌ **No timeouts** on the HTTP server (`ReadTimeout`, `WriteTimeout`, `IdleTimeout`), leaving it vulnerable to slow-client (Slowloris-style) resource exhaustion.
- ❌ **No structured logging, metrics, health/readiness split, or configuration management** (port is hardcoded to 8080).
- ❌ **Dockerfile inefficiencies** — the image copies the entire build context (no `.dockerignore`), builds without `-trimpath`/static flags, and runs as root. A multi-stage build with a `scratch` or `distroless` final stage would be significantly smaller and safer.

**Verdict:** This is a working, deployable *skeleton* — a fine starting point — but it is **not yet a functional option-pricing service and not production-ready**. Substantial feature implementation (pricing engines, input validation, API design) and hardening (timeouts, graceful shutdown, tests, non-root container) are required before real-world use.

## 📄 License

This project is distributed under the **VisionQuantech Custom Commercial License** — see [LICENSE](./LICENSE) for full terms. Summary:

- **Non-financial / educational use:** free.
- **Personal revenue-generating use:** 15–30% gross revenue share required.
- **Business / enterprise use:** requires a separate commercial license — contact **visionquantech@proton.me**.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND.