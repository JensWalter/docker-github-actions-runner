# actions/runner: https://github.com/actions/runner/releases/
FROM alpine:3.19

ARG TARGETOS
ARG TARGETARCH
ARG RUNNER_VERSION=2.315.0
ARG RUNNER_CONTAINER_HOOKS_VERSION=0.5.0
ARG DOTNET_VERSION=7

RUN addgroup -S runner && adduser -S runner -G runner && mkdir -p /etc/sudoers.d && echo "runner ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/runner && chmod 0440 /etc/sudoers.d/runner

RUN apk add -q --no-cache curl git unzip sudo jq bash openssl gcompat nodejs tar
# basic dev tools
RUN apk add -q --no-cache build-base musl-dev linux-headers

# Install dotnet
# https://github.com/actions/runner/blob/main/docs/start/envlinux.md#install-net-core-3x-linux-dependencies
# https://learn.microsoft.com/en-us/dotnet/core/install/linux-alpine#install-net-7
# RUN apk add -q --no-cache aspnetcore${DOTNET_VERSION}-runtime

# RUN export PATH=$HOME/.local/bin:$PATH

WORKDIR /actions-runner
RUN export RUNNER_ARCH=${TARGETARCH} \
    && if [ "$RUNNER_ARCH" = "amd64" ]; then export RUNNER_ARCH=x64 ; fi \
    && curl -f -L -o runner.tar.gz "https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-${TARGETOS}-${RUNNER_ARCH}-${RUNNER_VERSION}.tar.gz" \
    && tar xzf ./runner.tar.gz \
    && rm runner.tar.gz

RUN rm /actions-runner/externals/node16/bin/node && ln -s /usr/bin/node /actions-runner/externals/node16/bin/node
RUN rm /actions-runner/externals/node20/bin/node && ln -s /usr/bin/node /actions-runner/externals/node20/bin/node

RUN curl -f -L -o runner-container-hooks.zip https://github.com/actions/runner-container-hooks/releases/download/v${RUNNER_CONTAINER_HOOKS_VERSION}/actions-runner-hooks-k8s-${RUNNER_CONTAINER_HOOKS_VERSION}.zip \
    && unzip ./runner-container-hooks.zip -d ./k8s \
    && rm runner-container-hooks.zip

ENV RUNNER_MANUALLY_TRAP_SIG=1
ENV ACTIONS_RUNNER_PRINT_LOG_TO_STDOUT=1
# rust install
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | bash -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"

ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache
RUN mkdir -p /opt/hostedtoolcache

# configure directory permissions; ref https://github.com/actions/runner-images/blob/main/images/ubuntu/scripts/build/configure-system.sh
RUN chmod -R 777 /opt /usr/share

WORKDIR /home/runner
USER runner

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
