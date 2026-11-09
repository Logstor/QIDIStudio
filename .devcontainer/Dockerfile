FROM docker.io/debian:bookworm-slim

ARG DEBIAN_FRONTEND=noninteractive
ARG USERNAME=qidi
ARG USER_UID=1000
ARG USER_GID=${USER_UID}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Keep this list aligned with linux.d/debian. The few additions are the CA
# bundle, sudo for the non-root account, clangd for Zed, and tools directly
# invoked while bootstrapping dependencies.
RUN apt-get update \
    && apt-get install --yes --no-install-recommends \
        autoconf \
        build-essential \
        ca-certificates \
        clangd \
        cmake \
        ccache \
        curl \
        xvfb \
        eglexternalplatform-dev \
        extra-cmake-modules \
        file \
        gettext \
        git \
        libbz2-dev \
        libcurl4-openssl-dev \
        libdbus-1-dev \
        libglew-dev \
        libgl1-mesa-dev \
        libgstreamerd-3-dev \
        libgtk-3-dev \
        libmspack-dev \
        libosmesa6-dev \
        libsecret-1-dev \
        libssl-dev \
        libtool \
        libudev-dev \
        libunwind-dev \
        libwebkit2gtk-4.1-dev \
        libxkbcommon-dev \
        libx264-dev \
        libfuse2 \
        nasm \
        ninja-build \
        pkg-config \
        sudo \
        texinfo \
        wget \
        yasm \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

RUN groupadd --gid "${USER_GID}" "${USERNAME}" \
    && useradd \
        --uid "${USER_UID}" \
        --gid "${USER_GID}" \
        --create-home \
        --shell /bin/bash \
        "${USERNAME}" \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" \
        > "/etc/sudoers.d/${USERNAME}" \
    && chmod 0440 "/etc/sudoers.d/${USERNAME}"

USER ${USERNAME}
