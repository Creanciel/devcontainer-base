# devcontainer-base

Base images for DevContainers, published to GitHub Container Registry.

| Image | Contents |
|---|---|
| `ghcr.io/creanciel/development` | Rust, Ruby, Python, Node.js (pnpm) |
| `ghcr.io/creanciel/rust` | Rust |
| `ghcr.io/creanciel/web` | Node.js (pnpm) |
| `ghcr.io/creanciel/cargolambda` | Rust, cargo-lambda, zig, Node.js (pnpm), AWS CDK |

## Pull

```sh
docker pull ghcr.io/creanciel/development:latest
```

Each image is tagged with `latest` and the version in `docker/<image>/VERSION`.

## Use in a DevContainer

```dockerfile
FROM ghcr.io/creanciel/development:latest
```

## Build locally

```sh
cd docker/development
docker build -t development:test .
docker run -it development:test /bin/bash
```
