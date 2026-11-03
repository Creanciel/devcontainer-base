# syntax=docker/dockerfile:1

# Tool versions.
# Declared in the global scope and re-declared right before use in the stage,
# so bumping a version keeps every layer before its first use cached
# (layers from that point on are rebuilt).
ARG RUST_VERSION=1.99.1

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
    clang \
    g++ \
    gcc \
    make \
    # Opt-in faster linker: `mold -run cargo build`
    mold \
    perl \
    pkg-config \
    # x86_64-unknown-linux-musl target
    musl-tools \
    # Common native dependencies for crates
    libssl-dev \
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
    PATH="/home/$USERNAME/.cargo/bin:/home/$USERNAME/.local/bin:$PATH"

# RUSTUP_TOOLCHAIN is intentionally not set via ENV so that a project's rust-toolchain.toml takes precedence.
ARG RUST_VERSION
RUN curl -fsSL https://sh.rustup.rs | sh -s -- -y \
      --default-toolchain "$RUST_VERSION" \
      --profile minimal \
      --component rustfmt,clippy,rust-src \
      --target x86_64-unknown-linux-musl
