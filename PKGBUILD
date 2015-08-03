# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Sébastien "Seblu" Luttringer <seblu@seblu.net>

pkgbase=qemu
pkgname=('qemu'
         'qemu-block-iscsi'
         'qemu-block-rbd'
         'qemu-block-gluster'
         'libcacard')
pkgver=2.3.0
pkgrel=6
arch=('i686' 'x86_64')
license=('GPL2' 'LGPL2.1')
url='http://wiki.qemu.org/'
makedepends=('pixman' 'libjpeg' 'libpng' 'sdl' 'alsa-lib' 'nss' 'glib2'
             'gnutls' 'bluez-libs' 'vde2' 'util-linux' 'curl' 'libsasl'
             'libgl' 'libpulse' 'seabios' 'libcap-ng' 'libaio' 'libseccomp'
             'libiscsi' 'libcacard' 'spice' 'spice-protocol' 'python2'
             'usbredir' 'ceph' 'glusterfs' 'libssh2' 'lzo' 'snappy')
source=(http://wiki.qemu.org/download/${pkgname}-${pkgver}.tar.bz2
        CVE-2015-3456.patch
        CVE-2015-5154.patch
        CVE-2015-3214.patch
        CVE-2015-5158.patch
        65-kvm.rules)
md5sums=('2fab3ea4460de9b57192e5b8b311f221'
         '5e8a68940c4e0267e795a6ddd144e00e'
         '311d3845dda4795bf63107c3dcbf2bea'
         '29840d5f2fa93ff447bf9dd120d12e5a'
         'cd87c265dfec4d8aa3767d5d047cd397'
         '33ab286a20242dda7743a900f369d68a')

prepare() {
  for _p in *.patch; do
    [[ -e "$_p" ]] || continue
    msg2 "Patching $_p"
    patch -p1 -d ${pkgname}-${pkgver} < "$_p"
  done
}

build ()
{
  cd ${pkgname}-${pkgver}
  # qemu vs. make 4 == bad
  export ARFLAGS="rv"
  # http://permalink.gmane.org/gmane.comp.emulators.qemu/238740
  export CFLAGS+=' -fPIC'
  # gtk gui breaks keymappings at the moment
  ./configure --prefix=/usr --sysconfdir=/etc --audio-drv-list='pa alsa sdl' \
              --python=/usr/bin/python2 --smbd=/usr/bin/smbd \
              --enable-docs --libexecdir=/usr/lib/qemu \
              --disable-gtk --enable-linux-aio --enable-seccomp \
              --enable-spice --localstatedir=/var \
              --enable-tpm \
              --enable-modules --enable-{rbd,glusterfs,libiscsi,curl}
  make V=99
}

package_qemu() {
  pkgdesc='A generic and open source processor emulator which achieves a good emulation speed by using dynamic translation'
  depends=('glibc' 'pixman' 'libjpeg' 'libpng' 'sdl' 'alsa-lib' 'nss' 'glib2'
           'gnutls' 'bluez-libs' 'vde2' 'util-linux' 'libsasl' 'mesa-libgl'
           'seabios' 'libcap' 'libcap-ng' 'libaio' 'libseccomp' 'libcacard'
           'spice' 'usbredir' 'lzo' 'snappy' 'gcc-libs' 'zlib' 'bzip2' 'nspr'
           'ncurses' 'libx11' 'libusb' 'libpulse')
  backup=('etc/qemu/target-x86_64.conf')
  replaces=('qemu-kvm')
  optdepends=('samba: SMB/CIFS server support'
              'qemu-block-iscsi: iSCSI block support'
              'qemu-block-rbd: RDB block support'
              'qemu-block-gluster: glusterfs block support')
  options=(!strip)
  install=qemu.install

  make -C ${pkgname}-${pkgver} DESTDIR="${pkgdir}" libexecdir="/usr/lib/qemu" install

  cd "${pkgdir}"

  # provided by seabios package
  rm usr/share/qemu/bios.bin
  rm usr/share/qemu/acpi-dsdt.aml
  rm usr/share/qemu/q35-acpi-dsdt.aml
  rm usr/share/qemu/bios-256k.bin
  rm usr/share/qemu/vgabios-cirrus.bin
  rm usr/share/qemu/vgabios-qxl.bin
  rm usr/share/qemu/vgabios-stdvga.bin
  rm usr/share/qemu/vgabios-vmware.bin

  # remove conflicting /var/run directory
  rm -r var

  # systemd stuff
  install -D -m644 "${srcdir}/65-kvm.rules" usr/lib/udev/rules.d/65-kvm.rules
  install -D -m644 "${srcdir}/qemu.sysusers" usr/lib/sysusers.d/qemu.conf

  # bridge_helper needs suid
  # https://bugs.archlinux.org/task/32565
  chmod u+s usr/lib/qemu/qemu-bridge-helper

  # add sample config
  echo 'allow br0' > etc/qemu/bridge.conf.sample

  # manual striping in scripts directory
  find usr/src/linux-${_kernver}/scripts -type f -perm -u+w 2>/dev/null|while read binary ; do
      case "$(file -bi "$binary")" in
        *application/x-executable*) # Binaries
        /usr/bin/strip $STRIP_BINARIES "$binary";;
      esac
    done

  # remove libcacard files
  rm -r usr/include/cacard
  rm usr/lib/libcacard*
  rm usr/lib/pkgconfig/libcacard.pc
  rm usr/bin/vscclient

  # remove splited block modules
  rm usr/lib/qemu/block-{iscsi,rbd,gluster}.so
}

package_libcacard() {
 pkgdesc='Common Access Card (CAC) Emulation'
 depends=('glibc' 'nss' 'nspr' 'glib2')

  cd "${pkgdir}"
  install -d usr/{bin,lib/pkgconfig,include/cacard}
  install "${srcdir}"/qemu-${pkgver}/libcacard/*.h usr/include/cacard/
  install "${srcdir}"/qemu-${pkgver}/.libs/libcacard.so* usr/lib/
  install "${srcdir}"/qemu-${pkgver}/libcacard.pc usr/lib/pkgconfig/
  install "${srcdir}"/qemu-${pkgver}/.libs/vscclient usr/bin/
}

package_qemu-block-iscsi() {
  pkgdesc='Qemu iSCSI block module'
  depends=('glibc' 'glib2' 'libiscsi')

  install -D qemu-${pkgver}/block-iscsi.so "${pkgdir}"/usr/lib/qemu/block-iscsi.so
}

package_qemu-block-rbd() {
  pkgdesc='Qemu RBD block module'
  depends=('glibc' 'glib2' 'ceph')

  install -D qemu-${pkgver}/block-rbd.so "${pkgdir}"/usr/lib/qemu/block-rbd.so
}

package_qemu-block-gluster() {
  pkgdesc='Qemu GlusterFS block module'
  depends=('glibc' 'glib2' 'glusterfs')

  install -D qemu-${pkgver}/block-gluster.so "${pkgdir}"/usr/lib/qemu/block-gluster.so
}

# vim:set ts=2 sw=2 et:
