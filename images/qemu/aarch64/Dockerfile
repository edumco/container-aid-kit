FROM debian:buster-slim AS qemu-builder

ARG QEMU_VERSION=4.2.0
ENV QEMU_TARBALL="qemu-${QEMU_VERSION}.tar.xz"

WORKDIR /qemu

# Download QEMU
RUN apt-get update && \
    apt-get install --yes --no-install-recommends wget gpg && \
    wget "https://download.qemu.org/${QEMU_TARBALL}" && \
    wget "https://download.qemu.org/${QEMU_TARBALL}.sig" && \
    apt-get remove --purge --yes wget

# Verify signatures
RUN apt-get install --yes --no-install-recommends gpg && \
    gpg --keyserver keyserver.ubuntu.com --recv-keys CEACC9E15534EBABB82D3FA03353C9CEF108B584 && \
    gpg --verify "${QEMU_TARBALL}.sig" "${QEMU_TARBALL}" && \
    apt-get remove --purge --yes gpg

# Extract source tarball
RUN apt-get install --yes --no-install-recommends pkg-config
RUN tar xvf "${QEMU_TARBALL}"

# Build source
# These seem to be the only deps actually required for a successful  build
RUN apt-get install --yes --no-install-recommends python \
    build-essential \
    libglib2.0-dev \
    libpixman-1-dev

# These don't seem to be required but are specified here: https://wiki.qemu.org/Hosts/Linux
RUN apt-get install --yes --no-install-recommends libfdt-dev zlib1g-dev

# Not required or specified anywhere but supress build warnings
RUN apt-get install --yes --no-install-recommends flex bison

RUN "qemu-${QEMU_VERSION}/configure" --static \
    --target-list=arm-softmmu,aarch64-softmmu && \
    make -j$(nproc)

# Strip the binary, this gives a substantial size reduction!
RUN strip "arm-softmmu/qemu-system-arm" "aarch64-softmmu/qemu-system-aarch64"