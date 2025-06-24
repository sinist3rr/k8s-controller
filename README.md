# k8s-controller

[![Overall CI/CD](https://github.com/sinist3rr/k8s-controller/actions/workflows/ci.yaml/badge.svg?branch=main)](https://github.com/sinist3rr/k8s-controller/actions/workflows/ci.yaml)
[![Go Version](https://img.shields.io/badge/Go-1.24-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A simple Kubernetes controller and informer that watches **Deployments** in the `default` namespace, logs events, and exposes a FastHTTP server with an endpoint to list current Deployments. Built using **controller-runtime** and **client-go** informers.

## Features

* **Deployment Informer**: Watches Deployment resources and logs add, update, and delete events.
* **Controller Runtime**: Runs a skeleton reconciler for Deployments (logs reconcile events).
* **HTTP Server**: FastHTTP-based server exposing:

  * `GET /deployments`: returns a JSON array of Deployment names from the informer's cache.
  * Other paths: returns a friendly greeting.
* **CLI Commands**:

  * `list`: Lists Deployments in the `default` namespace.
  * `server`: Starts the informer, controller, and HTTP server.
* **Leader Election & Metrics**: Supports leader election and exposes Prometheus metrics via controller-runtime at `/metrics`.
* **Configurable Logging**: Uses `zerolog` with configurable log levels.

## Prerequisites

* Go 1.20+ installed
* A Kubernetes cluster (v1.20+)
* `kubectl` configured with access to your cluster (e.g. valid `~/.kube/config`)

## Installation

```bash
git clone https://github.com/sinist3rr/k8s-controller.git
cd k8s-controller
# Build all CLI commands
go build -o bin/k8s-controller ./cmd
# Or install via `go install`
go install github.com/sinist3rr/k8s-controller/cmd/k8s-controller@latest
```

## Usage

All commands share the global flag:

* `--log-level` (default: `info`): Set log verbosity (`trace`, `debug`, `info`, `warn`, `error`).

### List Deployments

```bash
# List deployments in the default namespace
bin/k8s-controller list --kubeconfig ~/.kube/config
```

### Run Server

```bash
# Start the controller, informer, and HTTP server
bin/k8s-controller server --kubeconfig ~/.kube/config --enable-leader-election=false --metrics-port=9090
```

This command will:

1. Start a `controller-runtime` manager with leader election and metrics bound to `:8081`.
2. Register the Deployment controller (logs reconcile events).
3. Start a client-go informer for Deployments in the `default` namespace.
4. Launch a FastHTTP server on port `:8080`:

   * `/deployments`: returns `[
      "deployment1",
      "deployment2",
      ...
     ]`.
   * Other paths: returns `Hello from FastHTTP!`.

**Flags**:

* `--port` (default `8080`): HTTP server port.
* `--metrics-port` (default `8081`): Metrics server port.
* `--kubeconfig` (default empty): Path to the kubeconfig file.
* `--in-cluster` (default `false`): Use in-cluster Kubernetes config.
* `--enable-leader-election` (default `true`): Enable leader election.
* `--leader-election-namespace` (default `default`): Namespace for leader election.

---
## Project Structure

- `cmd/` — Contains CLI commands.
- `main.go` — Entry point for application.
- `server.go` - fasthttp server
- `Makefile` — Build automation tasks.
- `Dockerfile` — Distroless Dockerfile for secure containerization.
- `.github/workflows/` — GitHub Actions workflows for CI/CD.
- `list.go` - list cli command
- `charts/app` - helm chart
- `pkg/informer` - informer implementation
- `pkg/testutil` - envtest kit
- `pkg/ctrl` - controller implementation

## License

MIT License. See [LICENSE](LICENSE) for details.