# devcontainer-base

Base images for DevContainers, published to GitHub Container Registry.

| Image | Contents |
|---|---|
| `ghcr.io/creanciel/devcontainer-development` | Rust, Ruby, Python, Node.js (pnpm) |
| `ghcr.io/creanciel/devcontainer-rust` | Rust |
| `ghcr.io/creanciel/devcontainer-web` | Node.js (pnpm) |
| `ghcr.io/creanciel/devcontainer-cargolambda` | Rust, cargo-lambda, zig, Node.js (pnpm), AWS CDK |

## Pull

```sh
docker pull ghcr.io/creanciel/devcontainer-development:latest
```

Each image is tagged with `latest` and the version in `docker/<image>/VERSION`.

## Use in a DevContainer

```dockerfile
FROM ghcr.io/creanciel/devcontainer-development:latest
```

## Build locally

```sh
cd docker/development
docker build -t devcontainer-development:test .
docker run -it devcontainer-development:test /bin/bash
```
