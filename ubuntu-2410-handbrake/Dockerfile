FROM ubuntu:24.10 AS builder

ARG DUMB_INIT_VERSION="1.2.2"
ARG GIT_CORE_PPA_KEY="A1715D88E1DF1F24"

ENV GIT_LFS_VERSION="3.5.1"
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
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ${GIT_CORE_PPA_KEY} \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    gnupg \
    rsync \
    lsb-release \ 
    apt-transport-https \
    software-properties-common \
    zlib1g-dev \
    gettext \
    libcurl4-openssl-dev \
    inetutils-ping \
    dirmngr \
    python3-pip \
    nodejs \
    python3-setuptools \
    python3 \
    dumb-init \
    handbrake-cli \
    gosu 

RUN export LSB_RELEASE_CODENAME="$(lsb_release --codename | cut -f2)" 
RUN sed -e 's/Defaults.*env_reset/Defaults env_keep = "HTTP_PROXY HTTPS_PROXY NO_PROXY FTP_PROXY http_proxy https_proxy no_proxy ftp_proxy"/' -i /etc/sudoers 
RUN apt-get update 
RUN ( ( curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" && unzip awscliv2.zip -d /tmp/ && /tmp/aws/install && rm awscliv2.zip) || pip3 install --no-cache-dir awscli ) 
RUN mkdir -p /etc/apt/keyrings \
  && ( curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg ) \
  && version=$(lsb_release -cs | sed 's/trixie\|n\/a/oracular/g') \
  && ( echo "deb [arch=${DPKG_ARCH} signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu ${version} stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null ) \
  && apt-get update \
  && apt-get install -y docker-ce docker-ce-cli docker-buildx-plugin containerd.io docker-compose-plugin --no-install-recommends --allow-unauthenticated \
  && echo -e '#!/bin/sh\ndocker compose --compatibility "$@"' > /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "focal" ]] && ( echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_24.10/ /" | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list && curl -L "https://build.opensuse.org/projects/devel:kubic/public_key" | apt-key add -; echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_24.10/ /" | tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list && curl -L "https://build.opensuse.org/projects/devel:kubic/public_key" | apt-key add - && apt-get update) || : ) \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "focal" || "${LSB_RELEASE_CODENAME}" == "jammy" || "${LSB_RELEASE_CODENAME}" == "sid" || "${LSB_RELEASE_CODENAME}" == "bullseye" ]] && apt-get install -y --no-install-recommends podman buildah skopeo || : ) \
  && ( [[ "${LSB_RELEASE_CODENAME}" == "jammy" ]] && echo "Ubuntu Jammy is marked as beta. Please see https://github.com/actions/virtual-environments/issues/5490" || : )
# gh cli
RUN  GH_CLI_VERSION=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq -r '.tag_name' | sed 's/^v//g') \
  && GH_CLI_DOWNLOAD_URL=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq ".assets[] | select(.name == \"gh_${GH_CLI_VERSION}_linux_${DPKG_ARCH}.deb\")" | jq -r '.browser_download_url') \
  && curl -sSLo /tmp/ghcli.deb ${GH_CLI_DOWNLOAD_URL} && apt-get -y install /tmp/ghcli.deb && rm /tmp/ghcli.deb \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /tmp/*
# gh-user
RUN groupadd -g 121 runner \
  && useradd -mr -d /home/runner -u 1001 -g 121 runner \
  && usermod -aG sudo runner \
  && usermod -aG docker runner \
  && echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && ( [[ -f /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list ]] && rm /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list || : )

ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache
RUN mkdir -p /opt/hostedtoolcache

ARG GH_RUNNER_VERSION="2.322.0"

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
