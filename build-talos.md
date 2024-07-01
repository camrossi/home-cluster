# How to build a custom image

Start by cloning the talos and pkgs repos (our update them):

```bash
git clone https://github.com/siderolabs/talos.git
git clone https://github.com/siderolabs/pkgs.gi
```

I need newer Kernel for my RPI so I need to modify the kernel version used in the `pkgs` and add the correct `linux_sha256` and `linux_sha512` I didn't find a smart way to calculate the `linux_sha512` other then downloading the kernel locally and calculate it

Once you have the shas edit the Pkgfile in the pkgs repo and update to the Kernel Version you need to use. 

Build the Kernel (I assume I am in the pkgs repod already):

```bash
KERNEL=v6.8.12
TALOS=v1.7.4
PLATFORM=linux/arm64,linux/amd64
REGISTRY=quay.io
USER=camillo
REGISTRY_AND_USERNAME=$REGISTRY/$USER
export REGISTRY_AND_USERNAME
make kernel REGISTRY=$REGISTRY_AND_USERNAME PUSH=true TAG=$KERNEL
cd ../talos
git checkout $TALOS

make kernel initramfs PKG_KERNEL=$REGISTRY_AND_USERNAME/siderolabs/kernel:$KERNEL PLATFORM=$PLATFORM REGISTRY=$REGISTRY_AND_USERNAME PUSH=true
make installer PKG_KERNEL=$REGISTRY_AND_USERNAME/siderolabs/kernel:$KERNEL PLATFORM=$PLATFORM REGISTRY=$REGISTRY_AND_USERNAME PUSH=true
make imager PKG_KERNEL=$REGISTRY_AND_USERNAME/siderolabs/kernel:$KERNEL PLATFORM=$PLATFORM REGISTRY=$REGISTRY_AND_USERNAME PUSH=true

```

Next we need to build the artifacts to install or upgrade the RPIs

To generate a disk image for RPis (the version of the extension and the overlay should be updated to whatever is the latest)

```bash
make image-metal REGISTRY_AND_USERNAME=$REGISTRY_AND_USERNAME/siderolabs IMAGER_ARGS="--base-installer-image $REGISTRY_AND_USERNAME/siderolabs/installer:$TALOS --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.1.4@sha256:4370d0740f27a7ae7aee56a8da6cd4f00ed8019bd4024fa73b44cb388ec86194  --system-extension-image ghcr.io/siderolabs/util-linux-tools:2.39.3@sha256:6a0d86f1cfbb296dfe2c29e033d9cb3d9f78ed98413865522238f4e1505365c4 --overlay-image ghcr.io/siderolabs/sbc-raspberrypi:v0.1.0-beta.0@sha256:47c6b7dc1cf697fc1ced0928eb4e8a37e83e99898289f59aaa49f8ed97249352 --overlay-name=rpi_generic" PLATFORM=linux/arm64
```

To generate a image for upgrades for RPis (the version of the extension and the overlay should be updated to whatever is the latest)

```bash
make image-installer REGISTRY_AND_USERNAME=$REGISTRY_AND_USERNAME/siderolabs IMAGER_ARGS="--base-installer-image $REGISTRY_AND_USERNAME/siderolabs/installer:$TALOS --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.1.4@sha256:4370d0740f27a7ae7aee56a8da6cd4f00ed8019bd4024fa73b44cb388ec86194  --system-extension-image ghcr.io/siderolabs/util-linux-tools:2.39.3@sha256:6a0d86f1cfbb296dfe2c29e033d9cb3d9f78ed98413865522238f4e1505365c4 --overlay-image ghcr.io/siderolabs/sbc-raspberrypi:v0.1.0-beta.0@sha256:47c6b7dc1cf697fc1ced0928eb4e8a37e83e99898289f59aaa49f8ed97249352 --overlay-name=rpi_generic" PLATFORM=linux/arm64

crane push _out/installer-arm64.tar $REGISTRY_AND_USERNAME/siderolabs/installer-rpi:$TALOS 
```

To generate a disk image for amd64 without overlays

```bash
make image-metal REGISTRY_AND_USERNAME=$REGISTRY_AND_USERNAME/siderolabs IMAGER_ARGS="--base-installer-image $REGISTRY_AND_USERNAME/siderolabs/installer:$TALOS --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.1.4@sha256:4370d0740f27a7ae7aee56a8da6cd4f00ed8019bd4024fa73b44cb388ec86194  --system-extension-image ghcr.io/siderolabs/util-linux-tools:2.39.3@sha256:6a0d86f1cfbb296dfe2c29e033d9cb3d9f78ed98413865522238f4e1505365c4" PLATFORM=linux/amd64

To generate a image for upgrades for RPis (the version of the extension and the overlay should be updated to whatever is the latest)

```bash
make image-installer REGISTRY_AND_USERNAME=$REGISTRY_AND_USERNAME/siderolabs IMAGER_ARGS="--base-installer-image $REGISTRY_AND_USERNAME/siderolabs/installer:$TALOS --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.1.4@sha256:4370d0740f27a7ae7aee56a8da6cd4f00ed8019bd4024fa73b44cb388ec86194  --system-extension-image ghcr.io/siderolabs/util-linux-tools:2.39.3@sha256:6a0d86f1cfbb296dfe2c29e033d9cb3d9f78ed98413865522238f4e1505365c4" PLATFORM=linux/amd64 

crane push _out/installer-amd64.tar $REGISTRY_AND_USERNAME/siderolabs/installer-amd64:$TALOS
```
