# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>

pkgbase=linux-16IAP7
pkgver=5.19.3.arch1
pkgrel=1
pkgdesc='Linux'
_srctag=v${pkgver%.*}-${pkgver##*.}
url="https://github.com/archlinux/linux/commits/$_srctag"
arch=(x86_64)
license=(GPL2)
makedepends=(
  bc libelf pahole cpio perl tar xz
  xmlto python-sphinx python-sphinx_rtd_theme graphviz imagemagick texlive-latexextra
  git
)
options=('!strip')
_srcname=archlinux-linux
source=(
  "$_srcname::git+https://github.com/archlinux/linux?signed#tag=$_srctag"
   config         # the main kernel config file
  '0001-PCI-DPC-Quirk-poot-port-PIO-log-size-for-certain-Int.patch'
  '0002-ACPICA-Make-address-space-handler-install-and-_REG-e.patch'
  '0003-ACPI-EC-fix-ECDT-probe-ordering-issues.patch'
  '0004-Add-IdeaPad-WMI-Fn-Keys-driver.patch'
  '0005-Add-IdeaPad-Usage-Mode-driver.patch'
  '0006-Add-IdeaPad-quick_charge-attribute-to-sysfs.patch'
  '0007-ALSA-hda-realtek-Add-quirk-for-Yoga-devices.patch'
  '0008-HID-hid-sensor-custom-More-custom-iio-sensors.patch'
  '0009-IIO-hid-sensor-als-Use-generic-usage.patch'
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
  'A2FF3A36AAA56654109064AB19802F8B0D70FC30'  # Jan Alexander Steffens (heftig)
  'C7E7849466FE2358343588377258734B41C31549'  # David Runge <dvzrv@archlinux.org>
)
sha256sums=('SKIP'
            '2a1456028c22ab33b02d1a501b39e15c6ea00bcef0c4ebc972c4ea98eb5251a7'
            'b19a23d37f3c74aa928c5d577f4fb41f115dbe1acdc3f6383ac9a53c15dbcf71'
            '06cad2a429f2a694f55300a5153483f9883ae5cfb8f8223ed2821a944e6ea4a4'
            '43e0a20d037015742373f19def6f31710dd35a8ee0e121a97c29b2a57080b801'
            'c6f778d786fbdd3483c66d834321c788b2818828003862d5a2a12f4cbc1694e6'
            'c9420129ecdbdfaf3b2006923763d1291f9031f26911219910593b33b621e18d'
            'c5ade2a167b1337e5564e49f9bec135d40b30b2442174598c354d80580a0af4e'
            '4ccf87491541cd991fbb2cf05f87fd08ddb885144f7c3bc04fe16e406327b136'
            'd1b2c9c17b0c193d3df1184d0f7fc850daf9e3d84d1d34385c8a9ee10a6ae17c'
            '7ff6d9c2da686f3331c117d2f06f6aa9f37be5ac95eb781b54491e0ece517a8a')
 
export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  msg2 "Disable NUMA..."
  scripts/config --disable CONFIG_NUMA
  
  msg2 "Disable old dynticks..."
  scripts/config --disable CONFIG_NO_HZ

msg2 "Set tick rate to 1000HZ..."
  scripts/config --disable CONFIG_HZ_300
  scripts/config --enable CONFIG_HZ_1000
  scripts/config --set-val CONFIG_HZ 1000


msg2 "Set up a GCC -O3 optimized kernel..."
  scripts/config --disable CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE
  scripts/config --enable CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE_O3

msg2 "Set default CPU governor to performance..."
  scripts/config --disable CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL
  scripts/config --disable CONFIG_CPU_FREQ_DEFAULT_GOV_POWERSAVE
  scripts/config --disable CONFIG_CPU_FREQ_DEFAULT_GOV_USERSPACE
  scripts/config --enable CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE


msg2 "Disable kernel debugging for smaller builds..."
  scripts/config --disable CONFIG_CONTEXT_TRACKING
  scripts/config --disable CONFIG_CONTEXT_TRACKING_FORCE
  scripts/config --disable CONFIG_DEBUG_KERNEL
  scripts/config --disable CONFIG_DEBUG_INFO
  scripts/config --disable CONFIG_ENABLE_MUST_CHECK
  scripts/config --disable CONFIG_UNUSED_SYMBOLS
  scripts/config --disable CONFIG_DEBUG_FS
  scripts/config --disable CONFIG_DEBUG_SECTION_MISMATCH
  scripts/config --disable CONFIG_DEBUG_FORCE_WEAK_PER_CPU
  scripts/config --disable CONFIG_DEBUG_MEMORY_INIT
  scripts/config --disable CONFIG_KGDB
  scripts/config --disable CONFIG_FUNCTION_TRACER
  scripts/config --disable CONFIG_STACK_TRACER
  
  echo "Setting config..."
  cp ../config .config
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make all
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(coreutils kmod initramfs)
  optdepends=('wireless-regdb: to set the correct wireless channels of your country'
              'linux-firmware: firmware images needed for some devices')
  provides=(VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE)
  replaces=(virtualbox-guest-modules-arch wireguard-arch)

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
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install  

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(pahole)

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  if [ -f "$builddir/tools/bpf/resolve_btfids" ]; then install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids ; fi

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

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
    [[ $arch = */x86/ ]] && continue
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

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
