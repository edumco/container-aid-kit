FROM debian:buster-slim AS qemu-base

ARG QEMU_VERSION=6.0.0
ENV QEMU_NAME="qemu-${QEMU_VERSION}"
ENV QEMU_TARBALL="${QEMU_NAME}.tar.xz"

WORKDIR /qemu

FROM qemu-base AS qemu-download

# Download QEMU, verify signatures and remove unused stuff
RUN apt-get update && \
    apt-get install --yes wget gpg && \
    wget "https://download.qemu.org/${QEMU_TARBALL}" && \
    wget "https://download.qemu.org/${QEMU_TARBALL}.sig" && \
    gpg --keyserver keyserver.ubuntu.com --recv-keys CEACC9E15534EBABB82D3FA03353C9CEF108B584 && \
    gpg --verify "${QEMU_TARBALL}.sig" "${QEMU_TARBALL}"  && \
    rm -f "${QEMU_TARBALL}.sig" && \
    apt-get remove --yes wget gpg && \
    apt-get autoremove --yes && \
    rm -rf /var/lib/apt/lists/*

FROM qemu-base AS dependencies

# These seem to be the only deps actually required for a successful build
RUN apt-get update && \
    apt-get install --yes --no-install-recommends python \
    build-essential \
    libglib2.0-dev \
    libpixman-1-dev

# These don't seem to be required but are specified here: https://wiki.qemu.org/Hosts/Linux
#RUN apt-get install --yes --no-install-recommends libfdt-dev zlib1g-dev

# Not required or specified anywhere but supress build warnings
#RUN apt-get install --yes --no-install-recommends flex bison


FROM dependencies AS builder

COPY --from=qemu-download /qemu/${QEMU_TARBALL} /qemu/${QEMU_TARBALL}

RUN tar xf ${QEMU_TARBALL} && rm ${QEMU_TARBALL} && \
    /qemu/${QEMU_NAME}/configure --static --target-list=aarch64-softmmu

# Strip the binary, this gives a substantial size reduction!
RUN make -j$(nproc) && \
    strip "aarch64-softmmu/qemu-system-aarch64" && \
    cp ./aarch64-softmmu/qemu-system-aarch64 /qemu-system-aarch64 && \
    rm -rf /qemu

FROM busybox:1.32 as production

COPY --from=builder /qemu-system-aarch64 /usr/local/bin/qemu-system-aarch64
