FROM debian:buster-slim AS qemu-base

ARG QEMU_VERSION=4.2.0
ENV QEMU_NAME="qemu-${QEMU_VERSION}"
ENV QEMU_TARBALL="${QEMU_NAME}.tar.xz"

WORKDIR /qemu

FROM qemu-base AS qemu-download

# Download QEMU, Verify signatures and remove unused stuff
RUN apt-get update && \
    apt-get install --yes wget gpg pkg-config && \
    wget "https://download.qemu.org/${QEMU_TARBALL}" && \
    wget "https://download.qemu.org/${QEMU_TARBALL}.sig" && \
    gpg --keyserver keyserver.ubuntu.com --recv-keys CEACC9E15534EBABB82D3FA03353C9CEF108B584 && \
    gpg --verify "${QEMU_TARBALL}.sig" "${QEMU_TARBALL}"  && \
    tar xvf "${QEMU_TARBALL}" && \
    rm -f "${QEMU_TARBALL}" "${QEMU_TARBALL}.sig"


FROM qemu-base AS qemu-builder

COPY --from=qemu-download /qemu/${QEMU_NAME} /qemu/${QEMU_NAME}

# These seem to be the only deps actually required for a successful build
RUN apt-get update && \
    apt-get install --yes --no-install-recommends python \
    build-essential \
    libglib2.0-dev \
    libpixman-1-dev

# These don't seem to be required but are specified here: https://wiki.qemu.org/Hosts/Linux
RUN apt-get install --yes --no-install-recommends libfdt-dev zlib1g-dev

# Not required or specified anywhere but supress build warnings
RUN apt-get install --yes --no-install-recommends flex bison

RUN ls 

RUN /qemu/${QEMU_NAME}/configure --static --target-list=arm-softmmu,aarch64-softmmu 

RUN echo $(nproc)

RUN make -j$(nproc)

# Strip the binary, this gives a substantial size reduction!
RUN strip "arm-softmmu/qemu-system-arm" "aarch64-softmmu/qemu-system-aarch64"

RUN ls -a
