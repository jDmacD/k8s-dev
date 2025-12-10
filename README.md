# dev-example

A Python Flask web service for Kubernetes development with k3d.

## Requirements
- docker
- [devbox](https://www.jetify.com/docs/devbox/installing-devbox)

## Quick Start

```bash
# Enter devbox shell
devbox shell

# Create k3d cluster with registry
devbox run cluster-create

# Build and push images
devbox run docker-build

# Deploy to cluster
devbox run kustomize-apply
```

## Access the Service

Access at: `https://dev.127.0.0.1.sslip.io`

## Development

```bash
# Run locally
uv run python main.py

# Start devspace development mode
devbox run devspace-dev
```

## Cleanup

```bash
devbox run cluster-rm
```

## API Endpoints

- `GET /` - Welcome message
- `GET /health` - Health check
