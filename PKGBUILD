# uConsole Linux Kernel (RPi CM4)
# Maintainer: Konstancja Łukawska <connie@backlight.ca>
#
# Based off of the work by PotatoMania in https://github.com/PotatoMania/uconsole-cm3

pkgbase=linux-uconsole-rpi64-arch
pkgver=6.13.2+g8ca012c
_desc="uConsole (CM4) kernel package using RPi's fork"

_srcname="linux"
_srcbranch="rpi-6.13.y"
_srcdir="$_srcname-$_srcbranch"
_archive="$_srcbranch.zip"
_archiveurl="https://github.com/raspberrypi/linux/archive/refs/heads/$_archive"

pkgrel=1
arch=('aarch64')
url="https://github.com/raspberrypi/linux"
license=('GPL2')
makedepends=(
  bc
  cpio
  gettext
  git
  libelf
  pahole
  perl
  python
  tar
  xz
)
options=(
  !debug
  !strip
)
source=("${_archive}::${_archiveurl}"
        'config'
        '90-linux-dtbs.hook'
        0001-video-backlight-Add-OCP8178-backlight-driver.patch
        0002-drm-panel-add-clockwork-cwu50.patch
        0003-driver-staging-add-uconsole-simple-amplifier-switch.patch
        0004-arm-dts-overlays-add-uconsole.patch
        0005-drivers-power-axp20x-customize-PMU-and-battery-implement-calibration.patch
        0006-drm-panel-cwu50-expose-dsi-error-status-to-userspace.patch
)

b2sums=('SKIP'
        '476f7d94e6f77d6e9e501482daa6fad470e13c9b154c63bceb5090768bf34f1413ea82b280b26510f83d91431c480c894f2c910cfe0c2105cc1d75d1466343ff'
        'bf23fa8846d66d358d5bc4f25719dc5adea4cd43837e1c9eb0ff292c03c83951bf02363e8bbf96bbee3fa618d9b8b92a6ff27c65319186ff08677f1d4d74c128'
        '90f3773e08643d7e674a505e4450960116f460271ac01b0d5863c197acd8408f0c10ad06c49fe809d075d5b5700a8cfd490ffc656938bf7381166d0246620b12'
        '6042deb1e349203cb60695761d4b90a619e02993d73831c597d5d213ca4d0fc3aa13a2be1e2ac81d50a4ceca3305f26305fc94ec2bc57f0c1ec17a65b600770f'
        '21aa8d171b15704cab29e4df597b9720f031557694733a6678f7aad6674f9a10d286ed08254c8790271a9952dc7388e4925813714d049f9b7d49a5ff5b902af8'
        '5f700d4cc484f6c0577c103c9af278bd9ea53601797bc3f268db8a346fcae9be1e20133dcdf2d3a9f5061cb9304b6b424505ba8c76ae8e6e23e2212206b30dc4'
        'df00855defeb04af2adba7b2895020385b9893f3a28525acf6b98fe41d8762a841e6fb6972ef86b93656d28b68cea56f8eae99b3e0fca7b615d428664adc849d'
        'e08320b0f13dae9139d474f6558ef96fede42dc4d94ae84e35a3cdfba94c29387211c5decf61e5118de3bae028651e49ed34571527f36bf7c9209ddd768479ae')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  mv "$_srcdir" "$_srcname"
  cd ${_srcname}

  
  # add upstream patch, this is for upstream kernels
  #patch -Np1 < ${srcdir}/patch-${pkgver}

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname
  make -s defconfig
  make -s kernelrelease > version
  make -s mrproper

  # add custom patches
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ${srcdir}/config .config
  make KERNELRELEASE="$(<version)" olddefconfig
  diff -u ${srcdir}/config .config || :

  echo "Prepared $pkgbase version $(<version)"
}

pkgver() {
	cd "${srcdir}/${_srcname}"	
	eval $(grep -o "^\(VERSION\|PATCHLEVEL\|SUBLEVEL\) = [0-9a-zA-Z_-]\+" Makefile | tr -d \ )
	printf "%s.%s.%s+g%s" $VERSION $PATCHLEVEL $SUBLEVEL "$(git rev-parse --short HEAD)"
}

build() {
  cd ${_srcname}
  make KERNELRELEASE="$(<version)" all
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=(
    coreutils
    initramfs
    kmod
  )
  optdepends=(
    'wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices'
    'firmware-raspberrypi: firmware images needed for on-board wireless module'
    'brcmfmac43456-firmware: firmware for on board WiFi'
    'ap6256-firmware: WiFi&BT firmware, replaces brcmfmac43456-firmware and firmware-raspberrypi on uConsole'
    'raspberrypi-bootloader: bootloader for RPis'
  )
  provides=(
    "WIREGUARD-MODULE"
  )
  install=${pkgname}.install

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  echo "Installing dtbs..."
  make INSTALL_DTBS_PATH="$modulesdir/dtb" dtbs_install

  # remove build link
  rm "$modulesdir"/build

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${kernver}|g
  "

  # install extra pacman hooks
  sed "${_subst}" ../90-linux-dtbs.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}-dtbs.hook"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  depends=(pahole)

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  # required when STACK_VALIDATION is enabled
  # install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  # install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}


pkgname=("${pkgbase}")
if uname -m | grep "aarch64"
then
  pkgname+=("${pkgbase}-headers")
fi


for _p in "${pkgname[@]}"; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#${pkgbase}}
  }"
done
