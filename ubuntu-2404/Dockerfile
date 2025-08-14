FROM ubuntu:24.04 AS builder

ENV GIT_LFS_VERSION="3.7.0"
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV PKG_CONFIG_PATH=/opt/local/lib/pkgconfig
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
RUN echo en_US.UTF-8 UTF-8 >> /etc/locale.gen \
  && apt-get update \
  && apt-get install -y --no-install-recommends gnupg curl tar unzip zip zstd ca-certificates \
      lsb-release \
      apt-transport-https \
      sudo \
      libcurl4-openssl-dev \
      gpg-agent \
      software-properties-common \
      zlib1g-dev \
      gettext \
      && rm -rf /var/lib/apt/lists/*
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    build-essential \
  && rm -rf /var/lib/apt/lists/*
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    inetutils-ping \
    dirmngr \
    openssh-client \
    python3-pip \
    python3-setuptools \
    python3-venv \
    python3 \
    locales \
  && rm -rf /var/lib/apt/lists/*
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    jq \
    wget \
    dumb-init \
    rsync \
    libpq-dev \
    gosu \
    libssl-dev \
    pkg-config \
    git \
  && rm -rf /var/lib/apt/lists/*
# install nodejs 24
RUN wget -q -O- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash \
   && /bin/bash -c "source /root/.nvm/nvm.sh && nvm install 24" \
   && rm -rf /root/.nvm/.cache
RUN DPKG_ARCH="$(dpkg --print-architecture)" \
  && apt-get update \
  && ( ( curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" && unzip awscliv2.zip -d /tmp/ && /tmp/aws/install && rm awscliv2.zip) || pip3 install --no-cache-dir awscli ) \
  && ( curl -s "https://github.com/git-lfs/git-lfs/releases/download/v${GIT_LFS_VERSION}/git-lfs-linux-${DPKG_ARCH}-v${GIT_LFS_VERSION}.tar.gz" -L -o /tmp/lfs.tar.gz && tar -xzf /tmp/lfs.tar.gz -C /tmp && /tmp/git-lfs-${GIT_LFS_VERSION}/install.sh && rm -rf /tmp/lfs.tar.gz  /tmp/git-lfs-${GIT_LFS_VERSION}) \
  && distro=$(lsb_release -is | awk '{print tolower($0)}') \
  && mkdir -p /etc/apt/keyrings \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /tmp/*
RUN DPKG_ARCH="$(dpkg --print-architecture)" \
  && apt-get update \
  && GH_CLI_VERSION=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq -r '.tag_name' | sed 's/^v//g') \
  && GH_CLI_DOWNLOAD_URL=$(curl -sL -H "Accept: application/vnd.github+json"   https://api.github.com/repos/cli/cli/releases/latest | jq ".assets[] | select(.name == \"gh_${GH_CLI_VERSION}_linux_${DPKG_ARCH}.deb\")" | jq -r '.browser_download_url') \
  && curl -sSLo /tmp/ghcli.deb ${GH_CLI_DOWNLOAD_URL} && apt-get -y install /tmp/ghcli.deb && rm /tmp/ghcli.deb \
  && groupadd -g 121 runner \
  && useradd -mr -d /home/runner -u 1001 -g 121 runner \
  && usermod -aG sudo runner \
  && echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && rm -rf /var/lib/apt/lists/* \
  && rm -rf /tmp/*

ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache
RUN mkdir -p /opt/hostedtoolcache

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /actions-runner
COPY install_actions.sh /actions-runner

ARG TARGETPLATFORM

RUN chmod +x /actions-runner/install_actions.sh \
  && /actions-runner/install_actions.sh "2.327.1" ${TARGETPLATFORM} \
  && rm /actions-runner/install_actions.sh \
  && chown runner /_work /actions-runner /opt/hostedtoolcache \
  && rm -rf /actions-runner/externals/node20_alpine \
  && rm -rf /actions-runner/externals/node24_alpine

COPY token.sh entrypoint.sh app_token.sh /
RUN chmod +x /token.sh /entrypoint.sh /app_token.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["./bin/Runner.Listener", "run", "--startuptype", "service"]
