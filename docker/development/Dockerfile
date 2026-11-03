# syntax=docker/dockerfile:1

# Tool versions.
# Declared in the global scope and re-declared right before use in the stage,
# so bumping a version keeps every layer before its first use cached
# (layers from that point on are rebuilt).
ARG RUST_VERSION=1.99.1
ARG MISE_VERSION=v2026.10.25
ARG RUBY_VERSION=4.0.9
ARG PYTHON_VERSION=3.15.2
ARG VP_VERSION=1.4.0
ARG NODE_VERSION=24.21.0
ARG PNPM_VERSION=12.16.1

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
    # Opt-in faster linker: `mold -run cargo build`
    mold \
    perl \
    pkg-config \
    # x86_64-unknown-linux-musl target
    musl-tools \
    # Build dependencies for Ruby / Python
    libbz2-dev \
    libffi-dev \
    liblzma-dev \
    libncurses-dev \
    libreadline-dev \
    libsqlite3-dev \
    libssl-dev \
    libyaml-dev \
    tk-dev \
    uuid-dev \
    zlib1g-dev && \
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

# Install Rust before Ruby so that YJIT / ZJIT are enabled when Ruby is built from source.
# RUSTUP_TOOLCHAIN is intentionally not set via ENV so that a project's rust-toolchain.toml takes precedence.
ARG RUST_VERSION
RUN curl -fsSL https://sh.rustup.rs | sh -s -- -y \
      --default-toolchain "$RUST_VERSION" \
      --profile minimal \
      --component rustfmt,clippy,rust-src \
      --target x86_64-unknown-linux-musl

ARG MISE_VERSION
RUN curl -fsSL https://mise.run | MISE_VERSION="$MISE_VERSION" sh && \
    printf '# For mise (https://mise.jdx.dev/)\neval "$(~/.local/bin/mise activate bash)"\n' >> ~/.bash_aliases && \
    # /etc/profile resets PATH in login shells, so restore the shim directories there
    printf '\n# Tool shims (mise / vite-plus)\nexport PATH="$HOME/.local/share/vite-plus/bin:$HOME/.local/share/mise/shims:$PATH"\n' >> ~/.profile

ARG RUBY_VERSION
RUN mise use -g "ruby@$RUBY_VERSION"

ARG PYTHON_VERSION
RUN mise use -g "python@$PYTHON_VERSION"

ARG VP_VERSION
RUN curl -fsSL https://vite.plus | CI=1 VP_NODE_MANAGER=yes VP_VERSION="$VP_VERSION" bash

ARG NODE_VERSION
ARG PNPM_VERSION
RUN vp env on && \
    vp env install "$NODE_VERSION" && \
    vp env default "node@$NODE_VERSION" && \
    vp env install "pnpm@$PNPM_VERSION" && \
    vp env default "pnpm@$PNPM_VERSION"
