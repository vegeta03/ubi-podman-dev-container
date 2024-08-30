FROM registry.access.redhat.com/ubi8/ubi:8.7

# Enable EPEL repository
RUN dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# Install build dependencies
RUN dnf update -y && \
    dnf install -y \
    make \
    gcc \
    gcc-c++ \
    git \
    curl \
    wget \
    bzip2 \
    openssl-devel \
    libseccomp-devel \
    libassuan-devel \
    gpgme-devel \
    device-mapper-devel \
    glib2-devel \
    systemd-devel \
    iptables \
    go-md2man \
    python3 \
    && dnf clean all

# Install Golang 1.23.0
WORKDIR /tmp
RUN curl -O https://dl.google.com/go/go1.23.0.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz && \
    rm go1.23.0.linux-amd64.tar.gz

ENV PATH=$PATH:/usr/local/go/bin
ENV GOPATH=/go
ENV PATH=$PATH:$GOPATH/bin

# Build and install Podman 5.2.2
WORKDIR /tmp
RUN git clone https://github.com/containers/podman.git && \
    cd podman && \
    git checkout v5.2.2 && \
    make BUILDTAGS="seccomp exclude_graphdriver_btrfs" && \
    make install PREFIX=/usr && \
    cd .. && \
    rm -rf podman

# Build and install conmon
RUN git clone https://github.com/containers/conmon.git && \
    cd conmon && \
    make && \
    make install && \
    cd .. && \
    rm -rf conmon

# Install CNI plugins
RUN git clone https://github.com/containernetworking/plugins.git && \
    cd plugins && \
    ./build_linux.sh && \
    mkdir -p /opt/cni/bin && \
    cp bin/* /opt/cni/bin/ && \
    cd .. && \
    rm -rf plugins

# Install runc
RUN git clone https://github.com/opencontainers/runc.git && \
    cd runc && \
    make && \
    make install && \
    cd .. && \
    rm -rf runc

# Set up Podman configuration
RUN useradd podman && \
    echo "podman:100000:65536" >> /etc/subuid && \
    echo "podman:100000:65536" >> /etc/subgid

USER podman
RUN mkdir -p /home/podman/.config/containers && \
    echo "[[registry]]" > /home/podman/.config/containers/registries.conf && \
    echo 'location = "docker.io"' >> /home/podman/.config/containers/registries.conf

ENV _CONTAINERS_USERNS_CONFIGURED=""

WORKDIR /home/podman