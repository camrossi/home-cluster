1) Build the Kernel with my git action

2) Create Boot Assets "talos" repo
sudo make kernel initramfs installer-base imager \
   PKG_KERNEL=registry.camsab.me/talos/kernel:v1.13.0-alpha.0-33-g8f40221 \
   IMAGE_REGISTRY=registry.camsab.me USERNAME=talos \
   PLATFORM=linux/arm64 INSTALLER_ARCH=arm64 PUSH=true



3) Create Packages that requires Kernel Modules "pkgs" repo

sudo make linux-firmware \
   PKG_KERNEL=registry.camsab.me/talos/kernel:v1.13.0-alpha.0-33-g8f40221 \
   REGISTRY=registry.camsab.me USERNAME=talos \
   PLATFORM=linux/arm64 INSTALLER_ARCH=arm64 PUSH=true

4) Build and Push Extensions that requires kernel modules "extension" repo

sudo make panfrost rknn \
 REGISTRY=registry.camsab.me USERNAME=talos \
 PKGS_PREFIX=registry.camsab.me/talos \
 PKGS=v1.13.0-alpha.0-33-g8f40221\
 PUSH=true PLATFORM=linux/arm64

5) Make Overlay
sudo make \
 REGISTRY=registry.camsab.me USERNAME=talos \
 PKGS_PREFIX=registry.camsab.me/talos \
 PKGS=v1.13.0-alpha.0-33-g8f40221 \
 PLATFORM=linux/arm64

5) Make the image:
sudo docker run --rm --privileged multiarch/qemu-user-static --reset --persistent yes
sudo docker run --rm --platform linux/arm64 -t \
   -v "$PWD/_out:/out" \
   -v /dev:/dev \
   --privileged \
   imager:v1.12.1@sha256:13476fe6687cd0e043b7cfb1e00384d3bdebc778d93b4fd871c721e97a86004f \
   installer \
   --arch arm64 \
   --base-installer-image=registry.camsab.me/talos/installer-base:v1.12.1 \
   --system-extension-image ghcr.io/siderolabs/iscsi-tools:v0.2.0@sha256:ace4f05eb2073aedfe08d5dadaf7f3c02e4a669ffe28b64cd4863ecf44e91bb9 \
   --system-extension-image ghcr.io/siderolabs/nut-client:2.8.4@sha256:92010a4958dd5cbed3fa33e155c1209d0d64c68cf837af8d4155c8da7a278b6e \
   --system-extension-image ghcr.io/siderolabs/util-linux-tools:2.41.2@sha256:8433604b21ec56871b00f363008ab455b2ece264246be915a2d6f742ec90ffcb \
   --system-extension-image registry.camsab.me/talos/panfrost:20251125-v1.12.1-27-ga019424 \
   --system-extension-image registry.camsab.me/talos/rknn:v1.12.1-27-ga019424 \
   --overlay-image registry.camsab.me/talos/sbc-rockchip:v0.1.7-1-ge97bafc-dirty \
   --overlay-name=turingrk1

imager:v1.12.1@sha256:13476fe6687cd0e043b7cfb1e00384d3bdebc778d93b4fd871c721e97a86004f