FROM debian:bullseye

ARG DEBIAN_FRONTEND=noninteractive
ARG GO_VERSION=1.18.10
ARG TARGETARCH
# Snapshot date for bullseye-security (not yet on archive.debian.org).
ARG SECURITY_SNAPSHOT=20260901T000000Z

# Bullseye has been removed from the regular mirrors: use archive.debian.org
# for main/updates and snapshot.debian.org for security.
RUN printf '%s\n' \
      'deb http://archive.debian.org/debian bullseye main' \
      'deb http://archive.debian.org/debian bullseye-updates main' \
      "deb http://snapshot.debian.org/archive/debian-security/${SECURITY_SNAPSHOT} bullseye-security main" \
      > /etc/apt/sources.list \
 && rm -f /etc/apt/sources.list.d/*.list /etc/apt/sources.list.d/*.sources \
 && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until

RUN apt-get update \
 && apt-get dist-upgrade -y \
 && apt-get install -y locales sudo

RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
 && locale-gen en_US.UTF-8 \
 && update-locale LANG=en_US.UTF-8 LANGUAGE=en_US:en
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# Match these to your host user (id -u / id -g) so the build can write to the
# mounted source tree: docker build --build-arg UID=$(id -u) --build-arg GID=$(id -g) ...
ARG UID=1001
ARG GID=1001
RUN groupadd -g "${GID}" docker \
 && useradd docker -u "${UID}" -g "${GID}" -G sudo -m -s /bin/bash \
 && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

RUN apt-get update

# gcc-10 / g++-10 are bullseye's default compilers; installed explicitly and
# pinned as the system gcc/g++/cc/c++ via update-alternatives.
RUN apt-get install -y \
    curl ca-certificates bash bc gcc-10 g++-10 gcc g++ sed patch patchutils tar bzip2 gzip \
      xz-utils zstd perl gawk gperf zip unzip diffutils lzop make file xfonts-utils xsltproc \
      default-jre-headless python3 libc6-dev libncurses5-dev libjson-perl libxml-parser-perl \
      libparse-yapp-perl rdfind git openssh-client rsync nodejs npm \
      pigz cpio xxd libgirepository-1.0-1 xmlstarlet wget \
    --no-install-recommends \
 && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100 \
 && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 100 \
 && update-alternatives --install /usr/bin/cc  cc  /usr/bin/gcc-10 100 \
 && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-10 100

# Go 1.21 isn't packaged for bullseye, so use the official upstream tarball
# (same layout as the bookworm image: /usr/lib/go, /usr/bin/go, /usr/bin/gofmt).
RUN ARCH="${TARGETARCH:-$(dpkg --print-architecture)}" \
 && curl -fsSL "https://go.dev/dl/go${GO_VERSION}.linux-${ARCH}.tar.gz" -o /tmp/go.tgz \
 && tar -C /usr/lib -xzf /tmp/go.tgz \
 && rm /tmp/go.tgz \
 && ln -s /usr/lib/go/bin/go /usr/bin/go \
 && ln -s /usr/lib/go/bin/gofmt /usr/bin/gofmt

RUN rm -rf /var/lib/apt/lists/*

USER docker
