FROM registry.access.redhat.com/ubi9/ubi:9.4-1181.1724035907

# Enable EPEL repository
RUN dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

# Install build dependencies
RUN dnf update -y && \
    dnf install -y --allowerasing \
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
    zlib-devel \
    libffi-devel \
    sqlite-devel \
    glibc-static \
    diffutils \
    && dnf clean all

# Install Python 3.12
WORKDIR /tmp
RUN curl -O https://www.python.org/ftp/python/3.12.5/Python-3.12.5.tgz && \
    tar -xzf Python-3.12.5.tgz && \
    cd Python-3.12.5 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    cd .. && \
    rm -rf Python-3.12.5 Python-3.12.5.tgz

# Set Python 3.12 as the default python3
RUN ln -sf /usr/local/bin/python3.12 /usr/bin/python3 && \
    ln -sf /usr/local/bin/pip3.12 /usr/bin/pip3

# Install Golang 1.23.0
WORKDIR /tmp
RUN curl -O https://dl.google.com/go/go1.23.0.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz && \
    rm go1.23.0.linux-amd64.tar.gz

ENV PATH=$PATH:/usr/local/go/bin
ENV GOPATH=/go
ENV PATH=$PATH:$GOPATH/bin

# Install go-md2man
WORKDIR /tmp
RUN curl -L https://github.com/cpuguy83/go-md2man/archive/refs/tags/v2.0.4.tar.gz -o go-md2man.tar.gz && \
    tar -xzf go-md2man.tar.gz && \
    cd go-md2man-2.0.4 && \
    go build && \
    mv go-md2man /usr/local/bin/ && \
    cd .. && \
    rm -rf go-md2man-2.0.4 go-md2man.tar.gz

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
    git checkout v1.1.13 && \
    make BUILDTAGS="seccomp" && \
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