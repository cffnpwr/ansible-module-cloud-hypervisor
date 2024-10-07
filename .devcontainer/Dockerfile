FROM debian:bookworm-slim AS uv

ENV UV_INSTALL_DIR="/"

RUN apt-get -qq update
RUN apt-get -qq install -y --no-install-recommends ca-certificates curl

RUN curl -LsSf https://astral.sh/uv/install.sh | sh

FROM debian:bookworm-slim AS mold

ARG MOLD_VERSION=2.34.1

WORKDIR /mold

RUN apt-get update -qq; \
    apt-get install -qq --no-install-recommends curl ca-certificates
RUN curl -L https://github.com/rui314/mold/releases/download/v${MOLD_VERSION}/mold-${MOLD_VERSION}-$(uname -m)-linux.tar.gz --output mold.tar.gz
RUN tar -xzf mold.tar.gz --strip-components 1

FROM mcr.microsoft.com/devcontainers/base:debian

ARG USER="developer"
ARG UID=1000
ARG GID=1000

ENV UV_LINK_MODE="copy"
ENV RUSTUP_HOME=/usr/local/rustup
ENV CARGO_HOME=/usr/local/cargo
ENV PATH=/usr/local/cargo/bin:$PATH

COPY --from=uv /bin/uv /usr/local/bin/uv
COPY --from=mold /mold/ /usr/local/

RUN apt-get -qq update; \
  apt-get -qq install -y --no-install-recommends libvirt-daemon-system; \
  if id -ng ${GID}; then groupmod -n ${USER} $(id -ng ${GID}); else groupadd -g ${GID} ${USER}; fi; \
  if id -nu ${UID}; then usermod -l ${USER} -d /home/${USER} -m $(id -nu ${UID}); else useradd -u ${UID} -g ${GID} -m -s /bin/bash ${USER}; fi; \
  echo "${USER}   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers; \
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain nightly -y; \
  chmod -R 777 ${RUSTUP_HOME} ${CARGO_HOME}; \
  usermod -aG libvirt,kvm ${USER}
