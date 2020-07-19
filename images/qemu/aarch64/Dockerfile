# Build stage for qemu-system-arm
FROM debian:stable-slim AS qemu-builder
ARG QEMU_VERSION=4.2.0
ENV QEMU_TARBALL="qemu-${QEMU_VERSION}.tar.xz"
WORKDIR /qemu

RUN # Update package lists
RUN apt-get update

RUN # Pull source
RUN apt-get -y install wget
RUN wget "https://download.qemu.org/${QEMU_TARBALL}"

RUN # Verify signatures
RUN apt-get -y install gpg
RUN wget "https://download.qemu.org/${QEMU_TARBALL}.sig"
RUN gpg --keyserver keyserver.ubuntu.com --recv-keys CEACC9E15534EBABB82D3FA03353C9CEF108B584
RUN gpg --verify "${QEMU_TARBALL}.sig" "${QEMU_TARBALL}"

RUN # Extract source tarball
RUN apt-get -y install pkg-config
RUN tar xvf "${QEMU_TARBALL}"

RUN # Build source
# These seem to be the only deps actually required for a successful  build
RUN apt-get -y install python build-essential libglib2.0-dev libpixman-1-dev
# These don't seem to be required but are specified here: https://wiki.qemu.org/Hosts/Linux
RUN apt-get -y install libfdt-dev zlib1g-dev
# Not required or specified anywhere but supress build warnings
RUN apt-get -y install flex bison
RUN "qemu-${QEMU_VERSION}/configure" --static --target-list=arm-softmmu,aarch64-softmmu
RUN make -j$(nproc)

RUN # Strip the binary, this gives a substantial size reduction!
RUN strip "arm-softmmu/qemu-system-arm" "aarch64-softmmu/qemu-system-aarch64"