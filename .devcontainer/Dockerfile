# Dockerfile for installing Podman on UBI 8
FROM registry.access.redhat.com/ubi8/ubi:8.7 AS builder

ARG PODMAN_VERSION=5.2.2
ARG UID_MIN=100000
ARG UID_MAX=165535

# Install dependencies and Podman
RUN dnf -y update && \
    dnf -y install 'dnf-command(copr)' && \
    dnf -y copr enable rhcontainerbot/podman-next && \
    dnf -y install \
    podman-${PODMAN_VERSION} \
    fuse-overlayfs \
    container-selinux \
    iptables \
    slirp4netns \
    && dnf clean all && \
    rm -rf /var/cache/dnf

# Configure system for Podman
RUN echo 'kernel.unprivileged_userns_clone=1' >> /etc/sysctl.conf && \
    echo 'user.max_user_namespaces=28633' >> /etc/sysctl.conf && \
    useradd -m podman && \
    echo "podman:${UID_MIN}:$((UID_MAX-UID_MIN+1))" >> /etc/subuid && \
    echo "podman:${UID_MIN}:$((UID_MAX-UID_MIN+1))" >> /etc/subgid && \
    mkdir -p /home/podman/.config/containers && \
    cat <<EOF > /home/podman/.config/containers/storage.conf
[storage]
driver = "overlay"
runroot = "/tmp/containers-user-podman"
graphroot = "/home/podman/.local/share/containers/storage"
EOF
RUN chown -R podman:podman /home/podman/.config

FROM registry.access.redhat.com/ubi8/ubi:8.7

LABEL maintainer="Your Name <your.email@example.com>"
LABEL description="UBI 8 with Podman for GitHub Codespaces"
LABEL version="1.0"

COPY --from=builder /usr/bin/podman /usr/bin/podman
COPY --from=builder /usr/lib64/libpod* /usr/lib64/
COPY --from=builder /etc/containers /etc/containers
COPY --from=builder /etc/subuid /etc/subuid
COPY --from=builder /etc/subgid /etc/subgid
COPY --from=builder /home/podman /home/podman

# Install runtime dependencies
RUN dnf -y install fuse-overlayfs slirp4netns container-selinux iptables && \
    dnf clean all && \
    rm -rf /var/cache/dnf

USER podman
ENV HOME /home/podman
WORKDIR /home/podman

# Verify Podman installation
RUN podman --version

HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD podman info || exit 1

CMD ["podman", "system", "info"]