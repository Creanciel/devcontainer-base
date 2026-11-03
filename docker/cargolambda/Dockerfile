# syntax=docker/dockerfile:1

# Tool versions.
# Declared in the global scope and re-declared right before use in the stage,
# so bumping a version keeps every layer before its first use cached
# (layers from that point on are rebuilt).
# Rust 1.98+ combined with cargo-zigbuild 0.20.1 (bundled with cargo-lambda 1.9.1)
# rejects --fix-cortex-a53-843419 when linking for aarch64, so stay on 1.97.
ARG RUST_VERSION=1.97.1
ARG CARGO_LAMBDA_VERSION=1.9.1
ARG MISE_VERSION=v2026.10.25
ARG ZIG_VERSION=0.16.0
ARG VP_VERSION=1.4.0
ARG NODE_VERSION=24.21.0
ARG PNPM_VERSION=12.16.1
ARG AWS_CDK_VERSION=2.1143.0

FROM debian:trixie-slim

# Fail the build when any command in a pipe fails (e.g. `curl ... | sh`)
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV LANG=en_US.UTF-8

RUN apt-get update && apt-get install -y --no-install-recommends \
    # Basic tools
    ca-certificates \
    curl \
    git \
    less \
    locales \
    openssh-client \
    procps \
    sudo \
    tar \
    unzip \
    wget \
    xz-utils \
    # Build tools
    g++ \
    gcc \
    make \
    perl \
    pkg-config \
    uuid-dev && \
    rm -rf /var/lib/apt/lists/* && \
    sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8

# Declared after apt so that changing them does not invalidate the apt layer
ARG USER_UID=1000
ARG USER_GID=1000
ARG USERNAME=development

RUN groupadd --gid "$USER_GID" "$USERNAME" && \
    useradd --uid "$USER_UID" --gid "$USER_GID" -m -s /bin/bash "$USERNAME" && \
    echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > "/etc/sudoers.d/$USERNAME" && \
    chmod 0440 "/etc/sudoers.d/$USERNAME"

# HOME is not set via ENV: Docker derives it from /etc/passwd for the current USER,
# so processes running as root (e.g. devcontainer features) keep HOME=/root.
USER "$USERNAME"
WORKDIR "/home/$USERNAME"

ENV CARGO_HOME="/home/$USERNAME/.cargo" \
    RUSTUP_HOME="/home/$USERNAME/.rustup" \
    PATH="/home/$USERNAME/.local/share/vite-plus/bin:/home/$USERNAME/.local/share/mise/shims:/home/$USERNAME/.cargo/bin:/home/$USERNAME/.local/bin:$PATH"

# RUSTUP_TOOLCHAIN is intentionally not set via ENV so that a project's rust-toolchain.toml takes precedence.
ARG RUST_VERSION
RUN curl -fsSL https://sh.rustup.rs | sh -s -- -y \
      --default-toolchain "$RUST_VERSION" \
      --profile minimal \
      --component rustfmt,clippy,rust-src \
      --target aarch64-unknown-linux-musl

ARG CARGO_LAMBDA_VERSION
RUN cargo install --locked cargo-lambda --version "$CARGO_LAMBDA_VERSION" && \
    rm -rf "$CARGO_HOME/registry" "$CARGO_HOME/git"

ARG MISE_VERSION
RUN curl -fsSL https://mise.run | MISE_VERSION="$MISE_VERSION" sh && \
    printf '# For mise (https://mise.jdx.dev/)\neval "$(~/.local/bin/mise activate bash)"\n' >> ~/.bash_aliases && \
    # /etc/profile resets PATH in login shells, so restore the shim directories there
    printf '\n# Tool shims (mise / vite-plus)\nexport PATH="$HOME/.local/share/vite-plus/bin:$HOME/.local/share/mise/shims:$PATH"\n' >> ~/.profile

# Linker backend used by cargo-lambda (cargo-zigbuild) for cross-compilation
ARG ZIG_VERSION
RUN mise use -g "zig@$ZIG_VERSION"

ARG VP_VERSION
RUN curl -fsSL https://vite.plus | CI=1 VP_NODE_MANAGER=yes VP_VERSION="$VP_VERSION" bash

ARG NODE_VERSION
ARG PNPM_VERSION
RUN vp env on && \
    vp env install "$NODE_VERSION" && \
    vp env default "node@$NODE_VERSION" && \
    vp env install "pnpm@$PNPM_VERSION" && \
    vp env default "pnpm@$PNPM_VERSION"

# Installed via mise's npm backend, which uses the Node.js provided by vite-plus
ARG AWS_CDK_VERSION
RUN mise use -g "npm:aws-cdk@$AWS_CDK_VERSION"
