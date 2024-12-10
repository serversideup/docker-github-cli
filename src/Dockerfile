ARG BUILD_BASE_OS_VERSION=alpine:3.20

FROM ${BUILD_BASE_OS_VERSION}
ARG BUILD_GITHUB_CLI_VERSION=2.60.0
ARG ALPINE_DEPENDENCIES="curl,git,ca-certificates,su-exec,openssh-client,shadow,sshpass"
ARG DEBIAN_DEPENDENCIES="curl,git,ca-certificates,gosu,openssh-client,sshpass"
ARG PUID='1000'
ARG PGID='1000'
ARG GITHUB_WORK_DIR='/app'
ARG REPOSITORY_BUILD_VERSION='dev'

COPY --chown=root:root --chmod=755 src/rootfs /

RUN serversideup-dep-install-alpine ${ALPINE_DEPENDENCIES} && \
    serversideup-dep-install-debian ${DEBIAN_DEPENDENCIES} && \
    \
    # Install GitHub CLI
    serversideup-install-github-cli ${BUILD_GITHUB_CLI_VERSION} && \
    \
    # Create default unprivileged user and group
    serversideup-create-unprivileged-user github "${PUID}" "${PGID}" && \
    \
    # Create default GitHub working directory and set permissions
    mkdir -p "${GITHUB_WORK_DIR}" && \
    chown -R "${PUID}:${PGID}" "${GITHUB_WORK_DIR}" && \
    git config --global --add safe.directory "${GITHUB_WORK_DIR}" && \
    \
    # Create SSH directory for unprivileged user symbolic link target
    mkdir -p /ssh /config/gh && \
    chmod 700 /ssh /config/gh && \
    chown "${PUID}:${PGID}" /ssh /config/gh && \
    ln -s /ssh /home/github/.ssh && \
    ln -s /config /home/github/.config && \
    \
    # Verify installation
    gh --version

LABEL org.opencontainers.image.title="serversideup/github-cli" \
  org.opencontainers.image.description="Run GitHub CLI anywhere with a lightwieght and powerful Docker image" \
  org.opencontainers.image.url="https://github.com/serversideup/docker-github-cli" \
  org.opencontainers.image.source="https://github.com/serversideup/docker-github-cli" \
  org.opencontainers.image.documentation="https://github.com/serversideup/docker-github-cli" \
  org.opencontainers.image.vendor="ServerSideUp" \
  org.opencontainers.image.authors="Jay Rogers (@jaydrogers)" \
  org.opencontainers.image.version="${REPOSITORY_BUILD_VERSION}" \
  org.opencontainers.image.licenses="GPL-3.0-or-later"

ENTRYPOINT ["/entrypoint.sh"]

WORKDIR /app

CMD ["gh"]
