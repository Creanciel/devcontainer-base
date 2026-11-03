# syntax=docker/dockerfile:1

# Tool versions.
# Declared in the global scope and re-declared right before use in the stage,
# so bumping a version keeps every layer before its first use cached
# (layers from that point on are rebuilt).
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
    # Build tools for native Node.js addons (node-gyp)
    g++ \
    gcc \
    make \
    python3 \
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

ENV PATH="/home/$USERNAME/.local/share/vite-plus/bin:/home/$USERNAME/.local/bin:$PATH"

ARG VP_VERSION
RUN curl -fsSL https://vite.plus | CI=1 VP_NODE_MANAGER=yes VP_VERSION="$VP_VERSION" bash && \
    # /etc/profile resets PATH in login shells, so restore the shim directory there
    printf '\n# Tool shims (vite-plus)\nexport PATH="$HOME/.local/share/vite-plus/bin:$PATH"\n' >> ~/.profile

ARG NODE_VERSION
ARG PNPM_VERSION
RUN vp env on && \
    vp env install "$NODE_VERSION" && \
    vp env default "node@$NODE_VERSION" && \
    vp env install "pnpm@$PNPM_VERSION" && \
    vp env default "pnpm@$PNPM_VERSION"
