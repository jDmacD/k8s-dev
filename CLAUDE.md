# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python Flask web service named `dev-example` that demonstrates a complete Kubernetes development workflow using:
- **Web Framework**: Flask for building REST APIs
- **Package Manager**: `uv` for fast Python package management
- **Container Runtime**: Docker with buildx for multi-architecture builds
- **Kubernetes**: k3d for local Kubernetes cluster with built-in registry
- **Deployment**: Kustomize for declarative Kubernetes configuration
- **Development Environment**: Devbox with automated scripts for the complete workflow

The service exposes two endpoints and deploys to a local k3d cluster with HTTPS ingress.

## Development Commands

### Devbox Workflow (Recommended)

```bash
# Enter devbox shell (provides k3d, devspace, uv)
devbox shell

# Create k3d cluster with local registry
devbox run cluster-create

# Build and push both image variants
devbox run docker-build

# Deploy to cluster
devbox run kustomize-apply

# Start devspace development mode (hot-reload)
devbox run devspace-dev

# Remove cluster
devbox run cluster-rm
```

### Local Development

```bash
# Install dependencies
uv sync

# Add a new dependency
uv add <package-name>

# Run the web service locally
uv run python main.py
```

The web service runs on `http://localhost:8080` with the following endpoints:
- `GET /` - Returns a welcome message
- `GET /health` - Returns health status

### Docker

The Dockerfile supports switching base images via build arguments:

```bash
# Build with default image (python:3.9-slim)
docker build -t dev-example:latest .

# Build with devspace image (python:3-alpine)
docker build --build-arg BASE_IMAGE=ghcr.io/loft-sh/devspace-containers/python:3-alpine -t dev-example:devspace .

# Run container locally
docker run -p 8080:8080 dev-example:latest

# Build and push to k3d registry
docker buildx build -t k3d-registry.localhost:12345/dev-example:latest --push .
```

### Kustomize Deployment

```bash
# Preview the generated manifests
kubectl kustomize kustomize/base

# Deploy to Kubernetes
kubectl apply -k kustomize/base

# Delete deployment
kubectl delete -k kustomize/base
```

Access the service at: `https://dev.127.0.0.1.sslip.io`

The base kustomize configuration includes:
- **Deployment**: 2 replicas with liveness/readiness probes on `/health` endpoint and resource limits
- **Service**: ClusterIP service exposing port 80, forwarding to container port 8080
- **Ingress**: HTTPS-enabled ingress routing traffic from `dev.127.0.0.1.sslip.io` (uses Traefik's default certificate)

## Project Structure

- `main.py` - Flask web service with REST API endpoints
- `Dockerfile` - Docker build with ARG support for switching base images (python:3.9-slim or devspace alpine)
- `kustomize/base/` - Kustomize base configuration with deployment, service, and ingress manifests
- `k3d.yaml` - k3d cluster configuration (referenced by devbox scripts)
- `devbox.json` - Development environment with automated scripts for cluster, docker, and deployment workflows
- `pyproject.toml` - Project metadata and dependencies (requires Python >=3.9, includes Flask and click)
- `uv.lock` - Lock file for reproducible dependencies
- `README.md` - Quick start guide using devbox scripts

## Architecture Notes

This project demonstrates a production-like local Kubernetes development workflow:

1. **Local Cluster**: k3d creates a lightweight Kubernetes cluster with a built-in container registry (`registry:5000`)
2. **Image Building**: Docker buildx builds and pushes images directly to the k3d registry
3. **Deployment**: Kustomize provides declarative configuration with overlays support
4. **HTTPS Access**: Traefik ingress controller (built into k3d) provides automatic HTTPS using default certificates
5. **Development Mode**: devspace enables hot-reloading for rapid iteration

The deployment references images as `registry:5000/dev-example:latest` which resolves to the k3d cluster's local registry.
