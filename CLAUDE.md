# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python Flask web service named `dev-example` that uses:
- **Web Framework**: Flask for building REST APIs
- **Package Manager**: `uv` for fast Python package management
- **Development Environment**: Devbox with k3d and devspace for Kubernetes development

## Development Commands

### Dependency Management
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

```bash
# Build the Docker image
docker build -t dev-example:latest .

# Run the container locally
docker run -p 8080:8080 dev-example:latest

# Build and push to k3d registry
docker build -t k3d-registry.localhost:12345/dev-example:latest .
docker push k3d-registry.localhost:12345/dev-example:latest
```

### Kustomize Deployment

```bash
# Preview the generated manifests
kubectl kustomize kustomize/base

# Deploy to Kubernetes
kubectl apply -k kustomize/base

# Delete deployment
kubectl delete -k kustomize/base

# Access the service (after deployment)
# Add to /etc/hosts: 127.0.0.1 dev-example.local
curl http://dev-example.local
```

The base kustomize configuration includes:
- **Deployment**: 2 replicas with health checks and resource limits
- **Service**: ClusterIP service exposing port 80
- **Ingress**: Routes traffic from `dev-example.local` to the service

### Kubernetes & Container Registry

The README documents a workflow for creating a k3d cluster with a local registry:

```bash
# Create cluster with registry
k3d cluster create mycluster --registry-create mycluster-registry:12345

# Pull, tag, and push images to the local registry
docker pull nginx:latest
docker tag nginx:latest k3d-registry.localhost:12345/nginx:latest
docker push k3d-registry.localhost:12345/nginx:latest
```

When deploying to k3d, images should reference the registry as `mycluster-registry:5000/<image>:<tag>`.

## Project Structure

- `main.py` - Flask web service with REST API endpoints
- `Dockerfile` - Multi-stage Docker build using uv for dependencies
- `kustomize/base/` - Kustomize base configuration with deployment, service, and ingress manifests
- `pyproject.toml` - Project metadata and dependencies (requires Python >=3.9)
- `devbox.json` - Development environment configuration with k3d, devspace, and uv
- `uv.lock` - Lock file for reproducible dependencies
- `README.md` - Contains k3d cluster setup and local registry usage examples

## Architecture Notes

This project is set up for Kubernetes development with a local k3d cluster. The typical workflow involves:
1. Building and pushing container images to a local registry
2. Deploying applications to the k3d cluster
3. Using devspace for development workflows (hot-reloading, port-forwarding, etc.)
