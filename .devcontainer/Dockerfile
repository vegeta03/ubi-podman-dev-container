# Use multi-stage builds to reduce final image size
FROM registry.access.redhat.com/ubi9/ubi:9.4-1181.1724035907 AS builder

# Combine RUN commands to reduce layers
RUN dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm && \
    dnf update -y && \
    dnf install -y --allowerasing \
    make gcc gcc-c++ git curl wget bzip2 openssl-devel libseccomp-devel \
    libassuan-devel gpgme-devel device-mapper-devel glib2-devel systemd-devel \
    iptables zlib-devel libffi-devel sqlite-devel glibc-static diffutils && \
    dnf clean all && \
    rm -rf /var/cache/dnf

# Install Python 3.12
WORKDIR /tmp
RUN curl -O https://www.python.org/ftp/python/3.12.5/Python-3.12.5.tgz && \
    tar -xzf Python-3.12.5.tgz && \
    cd Python-3.12.5 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    cd .. && \
    rm -rf Python-3.12.5 Python-3.12.5.tgz && \
    ln -sf /usr/local/bin/python3.12 /usr/bin/python3 && \
    ln -sf /usr/local/bin/pip3.12 /usr/bin/pip3

# Install Golang 1.23.0
RUN curl -O https://dl.google.com/go/go1.23.0.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz && \
    rm go1.23.0.linux-amd64.tar.gz

ENV PATH=$PATH:/usr/local/go/bin
ENV GOPATH=/go
ENV PATH=$PATH:$GOPATH/bin

# Install go-md2man
RUN curl -L https://github.com/cpuguy83/go-md2man/archive/refs/tags/v2.0.4.tar.gz -o go-md2man.tar.gz && \
    tar -xzf go-md2man.tar.gz && \
    cd go-md2man-2.0.4 && \
    go build && \
    mv go-md2man /usr/local/bin/ && \
    cd .. && \
    rm -rf go-md2man-2.0.4 go-md2man.tar.gz

# Build Podman 5.2.2
RUN git clone --depth 1 --branch v5.2.2 https://github.com/containers/podman.git && \
    cd podman && \
    make BUILDTAGS="seccomp exclude_graphdriver_btrfs" && \
    make install PREFIX=/usr && \
    cd .. && \
    rm -rf podman

# Build conmon
RUN git clone --depth 1 https://github.com/containers/conmon.git && \
    cd conmon && \
    make && \
    make install && \
    cd .. && \
    rm -rf conmon

# Install CNI plugins
RUN git clone --depth 1 https://github.com/containernetworking/plugins.git && \
    cd plugins && \
    ./build_linux.sh && \
    mkdir -p /opt/cni/bin && \
    cp bin/* /opt/cni/bin/ && \
    cd .. && \
    rm -rf plugins

# Install runc
RUN git clone --depth 1 --branch v1.1.13 https://github.com/opencontainers/runc.git && \
    cd runc && \
    make BUILDTAGS="seccomp" && \
    make install && \
    cd .. && \
    rm -rf runc

# Final stage
FROM registry.access.redhat.com/ubi9/ubi:9.4-1181.1724035907

# Copy built binaries and libraries from builder stage
COPY --from=builder /usr/local /usr/local
COPY --from=builder /usr/bin/podman /usr/bin/
COPY --from=builder /usr/lib/libpodman.so* /usr/lib/
COPY --from=builder /opt/cni /opt/cni

# Set up Podman configuration
RUN useradd podman && \
    echo "podman:100000:65536" >> /etc/subuid && \
    echo "podman:100000:65536" >> /etc/subgid

USER podman
RUN mkdir -p /home/podman/.config/containers && \
    echo "[[registry]]" > /home/podman/.config/containers/registries.conf && \
    echo 'location = "docker.io"' >> /home/podman/.config/containers/registries.conf

ENV _CONTAINERS_USERNS_CONFIGURED=""
ENV PATH=$PATH:/usr/local/go/bin
ENV GOPATH=/go
ENV PATH=$PATH:$GOPATH/bin

WORKDIR /home/podman