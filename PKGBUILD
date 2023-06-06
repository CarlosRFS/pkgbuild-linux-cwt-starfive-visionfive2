# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgbase=linux-cwt-515-starfive-visionfive2
_variant=cwt #5.15-VF2-xxx-x
pkgver=3.0.4
epoch=13 #Based on cwt image version
pkgrel=1
_tag=VF2_v${pkgver}
_desc='Linux 5.15.x (-cwt) for StarFive RISC-V VisionFive 2 Board'
_srcname=linux-$_tag
url="https://github.com/starfive-tech/linux/"
arch=(riscv64)
license=(GPL2)
makedepends=(bc libelf pahole cpio perl tar xz clang lld)
options=('!strip')
source=("https://github.com/starfive-tech/linux/archive/refs/tags/${_tag}.tar.gz"
  'linux-0-5.15.0-5.15.2.patch'
  'linux-1-Revert-fbcon-Disable_accelerated_scrolling.patch'
  'linux-2-fbcon-Add_option_to_enable_legacy_hardware_acceleration.patch'
  'linux-3-riscv-zba_zbb.patch'
  'linux-4-eswin_6600u-llvm.patch'
  'linux-5-fix_CVE-2022-0847_DirtyPipe.patch'
  'linux-6-fix_wrong_offset_for_fwcfg_and_make_whole_QSPI_be_available.patch'
  'linux-7-constify_struct_dh_pointer_members.patch'
  'linux-8-fix_broken_gpu-drm-i2c-tda998x.patch'
  'config'
  'linux.preset'
  '90-linux.hook')

sha256sums=('1f5445fe0e176e4731a2ef3ec92ec9dfdf7f87e838621084676c316818c5825e'
            '3bd9dc1b0843b77b51b269ad2ca30895121d94a6993f149496a7c9a83e08b369'
            '1582369c7a9365d98a03e08d0dbe8e0affc9417672f00aa57d6957ba559da878'
            'e16e2f8eafe310a561a553d8e2af16af7a50d2c499221d0b9348a94aea571dfa'
            'ef2196f0626265198454972dce9e873b620382465b4e66380de6506ccfc564d0'
            'bcd1f14392af6adce2760c36a7d1b631c60a4f590bf1241934c401187ba1b40e'
            '725875c1d8c7bf93cadfbceedcdfaa4062661b2deeb70a75852b87cff1d50831'
            '95d40b97e61245095fa4dcfd67669d71d9a72c346a8ae22003352090e39ae1e1'
            '01cf756c307a4aeda0b8c940340b75759f00ec712b9ccc217889c6ea8f94f59e'
            'a5955ef6043e89080be902f9133f56fbeb78919fa7b45d4decb9191875217897'
            '7293c522abb6ab757365eaa12c6400de812e2208b9f9fc6ca94bbafcb4f8e3c5'
            '57acae869144508c5600d6c8f41664f073f731c40cad2c58d2a1d55240495ddb'
            '5308a6dcabff290c627cab5c9db23c739eddbf7aa8a4984468ed59e6a5250702')

prepare() {
  cd $_srcname

  local src
  for src in $(ls ../); do
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 <"../$src"
  done

  echo "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-${_variant}" >localversion.10-variant
  echo "-${pkgver}" >localversion.20-pkgver
  echo "-$pkgrel" >localversion.30-pkgrel

  echo "Setting config..."
  cp ../config .config
  make LLVM=1 olddefconfig
  #make starfive_visionfive2_defconfig
  cp .config ../../config.new

  make LLVM=1 -s kernelrelease >version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make LLVM=1 all
}

_package() {
  pkgdesc="The $_desc kernel and modules"
  depends=(coreutils kmod mkinitcpio)
  optdepends=('wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices')
  provides=("linux=${pkgver}" "WIREGUARD-MODULE")
  conflicts=('linux')

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  install -Dm644 "arch/riscv/boot/Image.gz" "$modulesdir/vmlinuz"
  install -Dm644 "arch/riscv/boot/Image.gz" "$pkgdir/boot/vmlinuz"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  echo "Installing dtbs..."
  make INSTALL_DTBS_PATH="$pkgdir/usr/share/dtbs/$kernver" dtbs_install
  make INSTALL_DTBS_PATH="$pkgdir/boot/dtbs/" dtbs_install

  # remove build links
  rm "$modulesdir"/build

  install -Dm644 ../linux.preset "${pkgdir}/etc/mkinitcpio.d/linux.preset"
  install -Dm644 ../90-linux.hook "${pkgdir}/usr/share/libalpm/hooks/90-linux.hook"

}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $_desc kernel"
  depends=(pahole)
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/riscv" -m644 arch/riscv/Makefile
  cp -t "$builddir" -a scripts

  # required when DEBUG_INFO_BTF_MODULES is enabled
  cp --parents -r -t "$builddir/" tools/bpf/resolve_btfids

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/riscv" -a arch/riscv/include
  install -Dt "$builddir/arch/riscv/kernel" -m644 arch/riscv/kernel/asm-offsets.s

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
    [[ $arch = */riscv/ ]] && continue
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
    case "$(file -bi "$file")" in
    application/x-sharedlib\;*) # Libraries (.so)
      strip -v $STRIP_SHARED "$file" ;;
    application/x-archive\;*) # Libraries (.a)
      strip -v $STRIP_STATIC "$file" ;;
    application/x-executable\;*) # Binaries
      strip -v $STRIP_BINARIES "$file" ;;
    application/x-pie-executable\;*) # Relocatable binaries
      strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
