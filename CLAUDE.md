# CLAUDE.md

This project builds base images for DevContainers.

Each image lives in `docker/<image>/` with a `Dockerfile` and a `VERSION` file:

- `docker/development` — the main image (Rust, Ruby, Python, Node.js)
- `docker/rust`
- `docker/web`
- `docker/cargolambda`

Each image has its own release workflow at `.github/workflows/release-<image>.yaml`, tagged from its `VERSION` file.

## Build

```sh
cd docker/<image>
docker build -t devcontainer-<image>:test .
```

Use the `devcontainer-` prefix for local tags (e.g. `devcontainer-rust:test`) to avoid colliding with official images such as `rust`.

## Testing

- Do not build images or run tests unless the user explicitly asks for it.
- When the user asks for a build or test, delegate it to a subagent running a lightweight model (e.g. Haiku) instead of running it in the main session.
