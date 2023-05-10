FROM ubuntu:22.04 AS deps
### DEPENDENCIES
# Generic: build-essential gcc gcc-aarch64-linux-gnu curl ca-certificates
# ARM Firmware: bison flex
# UBOOT: python3 python3-dev python3-setuptools python3-pyelftools
#        libssl-dev bc swig uuid-dev
# IMAGE ASSEMBLY: fdisk jq
WORKDIR /build
RUN apt update \
 && apt install -y --no-install-recommends \
                   build-essential gcc gcc-aarch64-linux-gnu \
                   curl git bison flex device-tree-compiler \
                   python3 swig uuid-dev python3-dev \
                   libssl-dev python3-setuptools bc ca-certificates \
                   python3-pyelftools fdisk jq
RUN mkdir -p /build/artifacts

FROM deps AS download
ARG ARM_FIRMWARE_TAG=lts-v2.8.6
ARG ARM_FIRMWARE_SHASUM=489f36323dbeb426d3c129a49ced9d76330027fbc23fd8f533489561e1fc84ef
RUN curl -L -O https://github.com/ARM-software/arm-trusted-firmware/archive/refs/tags/${ARM_FIRMWARE_TAG}.tar.gz \
 && echo "${ARM_FIRMWARE_SHASUM}  ${ARM_FIRMWARE_TAG}.tar.gz" | shasum -c \
 && tar xf ${ARM_FIRMWARE_TAG}.tar.gz \
 && ln -s arm-trusted-firmware-${ARM_FIRMWARE_TAG} arm-trusted-firmware \
 && git clone -b "u-boot/v2022.10/roc-rk3328-cc" \
      https://github.com/libre-computer-project/libretech-u-boot.git


FROM download AS build
RUN cd arm-trusted-firmware \
 && make CROSS_COMPILE=aarch64-linux-gnu- PLAT=rk3328 \
 && cp build/rk3328/release/bl31/bl31.elf /build/artifacts/ \
 && cd ..
RUN cd libretech-u-boot \
 && export BL31=/build/artifacts/bl31.elf \
 && make roc-cc-rk3328_defconfig \
 && make CROSS_COMPILE=aarch64-linux-gnu- \
 && cp u-boot-rockchip.bin /build/artifacts/

FROM deps as assemble
ARG SECTORS_TO_ADD=22528
ARG IMG_FILE=20230430-jammy-preinstalled-server-arm64.img
ARG IMG_URL=https://archive.very.engineer
ARG OUT_IMG=rk3328-jammy-20230430.img
COPY --from=build /build/artifacts/* /build/artifacts/
RUN curl -L -O ${IMG_URL}/${IMG_FILE}.xz \
 && unxz ${IMG_FILE}.xz \
 && mv ${IMG_FILE} ${OUT_IMG}
RUN export IMAGE_SIZE=$(sfdisk --json ${OUT_IMG} | jq ".partitiontable.lastlba + 1 + 64") \
 && dd if=/dev/zero of=${OUT_IMG} bs=512 seek=${IMAGE_SIZE} \
       count=${SECTORS_TO_ADD} \
 && for p in 1 14 15; do \
      echo "+${SECTORS_TO_ADD}," | sfdisk --move-data ${OUT_IMG} -N ${p}; \
    done \
 && dd if=artifacts/u-boot-rockchip.bin of=${OUT_IMG} \
       bs=512 seek=64 conv=notrunc \
 && mv ${OUT_IMG} artifacts/ \
 && xz -T 0 artifacts/${OUT_IMG}
