FROM ubuntu:25.04 AS builder

ENV GIT_LFS_VERSION="3.7.0"
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV PKG_CONFIG_PATH=/opt/local/lib/pkgconfig
ENV DEBIAN_FRONTEND=noninteractive
ENV DPKG_ARCH=amd64
# hadolint ignore=SC2086,DL3015,DL3008,DL3013,SC2015
RUN echo en_US.UTF-8 UTF-8 >> /etc/locale.gen \
  && apt-get update \
  && apt-get install -y --no-install-recommends gnupg \
    curl \
    tar \
    unzip \
    zip \
    ca-certificates \
    build-essential \
    pkg-config \
    sudo \
    openssh-client \
    locales \
    libpq-dev \
    libssl-dev \
    gpg-agent \
    zstd \
    jq \
    wget git git-lfs
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    gnupg \
    lsb-release \ 
    apt-transport-https \
    software-properties-common \
    gettext \
    inetutils-ping \
    dirmngr \
    dumb-init \
    handbrake-cli \
    gosu 

# gh-user
RUN groupadd -g 121 runner \
  && useradd -mr -d /home/runner -u 1001 -g 121 runner \
  && usermod -aG sudo runner \
  && echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && ( [[ -f /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list ]] && rm /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list || : )

ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache
RUN mkdir -p /opt/hostedtoolcache

ARG GH_RUNNER_VERSION="2.327.1"

ARG TARGETPLATFORM

WORKDIR /actions-runner
COPY install_actions.sh /actions-runner

RUN chmod +x /actions-runner/install_actions.sh \
  && /actions-runner/install_actions.sh ${GH_RUNNER_VERSION} ${TARGETPLATFORM} \
  && rm /actions-runner/install_actions.sh \
  && chown runner /_work /actions-runner /opt/hostedtoolcache

COPY token.sh entrypoint.sh app_token.sh /

RUN chmod +x /token.sh /entrypoint.sh /app_token.sh
# https://github.com/containers/buildah/issues/3666#issuecomment-1351992335
VOLUME /var/lib/containers

ENTRYPOINT ["/entrypoint.sh"]
CMD ["./bin/Runner.Listener", "run", "--startuptype", "service"]
