FROM ubuntu:25.04 AS builder

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV PKG_CONFIG_PATH=/opt/local/lib/pkgconfig
ENV DEBIAN_FRONTEND=noninteractive
ENV DPKG_ARCH=amd64

RUN echo en_US.UTF-8 UTF-8 >> /etc/locale.gen \
  && apt update \
  && apt install -y --no-install-recommends \
    curl \
    tar \
    unzip \
    zip \
    ca-certificates \
    sudo \
    zstd \
    jq \
    locales \
    python3-pip \
    python3-setuptools \
    python3 \
    libicu76 \
    git \
    wget \
    gosu \
    dumb-init \
    && rm -rf /var/lib/apt/lists/*
RUN apt-get update \
  && apt-get install -y --no-install-recommends handbrake-cli \
    && rm -rf /var/lib/apt/lists/*
# install nodejs 24
RUN wget -q -O- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash \
   && /bin/bash -c "source /root/.nvm/nvm.sh && nvm install 24" \
   && rm -rf /root/.nvm/.cache
# install awscli
RUN ( ( curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" && unzip awscliv2.zip -d /tmp/ && /tmp/aws/install && rm awscliv2.zip) || pip3 install --no-cache-dir awscli ) \
  && rm -rf /tmp/*

# gh-user
RUN groupadd -g 121 runner \
  && useradd -mr -d /home/runner -u 1001 -g 121 runner \
  && usermod -aG sudo runner \
  && echo '%sudo ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && ( [[ -f /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list ]] && rm /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list || : )

ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache
RUN mkdir -p /opt/hostedtoolcache

WORKDIR /actions-runner
COPY install_actions.sh /actions-runner

ARG TARGETPLATFORM

RUN chmod +x /actions-runner/install_actions.sh \
  && /actions-runner/install_actions.sh "2.327.1" ${TARGETPLATFORM} \
  && rm /actions-runner/install_actions.sh \
  && chown runner /_work /actions-runner /opt/hostedtoolcache \
  && rm -rf /actions-runner/externals/node20_alpine \
  && rm -rf /actions-runner/externals/node20 \
  && rm -rf /actions-runner/externals/node24_alpine

COPY token.sh entrypoint.sh app_token.sh /

RUN chmod +x /token.sh /entrypoint.sh /app_token.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["./bin/Runner.Listener", "run", "--startuptype", "service"]
