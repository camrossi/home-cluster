
# How to build a custom image

Once you have setup git action to run locally [git action to run locally](gitaction.md)


## Build the kernel
Do the work and figure out which patches are integrated and which aren't you can look at  https://gitlab.collabora.com/hardware-enablement/rockchip-3588/notes-for-rockchip-3588/-/blob/main/mainline-status.md
then is a matter or figuring out which patch is integrated in which kernel release and:

- My current good branch is v1.8.0-6.12
- Update the relavant patch code in `talos-pkgs/kernel/build/patches`
- Ensure the Kernel config is enabling the right Configs `kernel/build/config-amd64`
  - if the patch modifies `Kconfig` you will need to add a new entrt in the `config-amd64`
- Tag your kernel `git tag <tag>
- Push the tag `git push --tags`
  - This will trigger the CI and buld the kernel and push it in your repo. If you set:
  ```
  env:
    REGISTRY: registry.camsab.me:443
    USER: "talos"
  ```
The resulting image will be pused as  `registry.camsab.me:443/talos/kernel:<TAG>`

## Build the SBC overlay

This has the u-Boot code with its own patches for now is a 1:1 copy from Nico's repo and only thing I changed is the `Makefile` and the gitaction to use my local git registry. 
- In the Makefile I have set
  ```
  REGISTRY ?= registry.camsab.me
  USERNAME ?= talos
  ```
   change as requried
- Set `image` in `installers/pkg.yaml` top point to the right kernel.


## Build the extensions

If some of the requried modules changed you need to update `sbcs/rk3588/files/modules.txt` and list the module path so it can be copied. 
- If there is a new version of talos:
  - Create a new branch from that version so you have the right images/updates
- Change the `PKG_KERNEL` in the Makefile to point to the Kernel generated in the previous step. i.e. `registry.camsab.me:443/talos/kernel:v1.8.3-6.12.0`
- In the Makefile I have set 
  ```
  REGISTRY ?= registry.camsab.me
  USERNAME ?= talos
  ```
  change as requried

Update the Makefile and ensure `PKGS` matches whatever version is used in the siderolabls repo. I.e. copy the calue from `https://github.com/siderolabs/extensions/blob/v1.8.3/Makefile`


## Build the talos images




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
