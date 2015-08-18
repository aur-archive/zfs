# Maintainer: Jesus Alvarez <jeezusjr at gmail dot com>
# Contributor: Kyle Fuller <inbox at kylefuller dot co dot uk>

pkgname="zfs"
#
# This script doesn't use bash variables for the version number because AUR
# doesn't properly parse bash variables. We use a custom build script to
# automatically change the appropriate variables before building in a clean
# chroot environment (using systemd-ndspawn).
#
# The build script can be found at
# https://github.com/demizer/archzfs/blob/master/build.sh
#
pkgver=0.6.2_3.14
pkgrel=3

# Used incase the i686 and x86_64 linux packages get out of sync with the
# PKGREL. This occurred on January 31, 2014 where i686 was versioned at
# 3.12.9-1 and x86_64 was versioned at 3.12.9-2.
LINUX_VERSION_X32="3.14-5"
LINUX_VERSION_X64="3.14-5"
[[ $CARCH == "i686" ]] && LINUX_VERSION=$LINUX_VERSION_X32 || LINUX_VERSION=$LINUX_VERSION_X64

# If LINUX_VERSION does not have a minor version, we need to add "0" as the
# minor version for the kernel module path. Kernel modules for linux versions
# with no minor version (i.e. 3.14) are kept in /usr/lib/modules/3.14.0-4-ARCH
[[ $LINUX_VERSION =~ ^[:digit:]+\.[:digit:]+\.([:digit:]+)\-[:digit:]+ ]]
if [[ ${BASH_REMATCH[1]} == "" ]]; then
    [[ $LINUX_VERSION =~ ^([[:digit:]\.]+)\-([[:digit:]]+)  ]] &&
        MOD_LINUX_VERSION=${BASH_REMATCH[1]};
        MOD_LINUX_PKGREL=${BASH_REMATCH[2]}
    MOD_BASENAME=${MOD_LINUX_VERSION}.0-$MOD_LINUX_PKGREL-ARCH
else
    MOD_BASENAME=$LINUX_VERSION-ARCH
fi

pkgdesc="Kernel modules for the Zettabyte File System."
depends=("spl=0.6.2_3.14-3" "zfs-utils" "linux=$LINUX_VERSION")
makedepends=("linux-headers=$LINUX_VERSION")
arch=("i686" "x86_64")
url="http://zfsonlinux.org/"
source=(http://archive.zfsonlinux.org/downloads/zfsonlinux/zfs/zfs-0.6.2.tar.gz
        http://demizerone.com/patches/20140411-zfs-git-master.patch)
groups=("archzfs")
md5sums=('0b183b0abdd5be287046ad9ce4f899fd'
         'b7632a5a80d474272d1de046b6da5f81')
license=("CDDL")
install=zfs.install

prepare() {
    cd "$srcdir/zfs-0.6.2"
    patch -Np1 < ../20140411-zfs-git-master.patch
}

build() {
  cd "$srcdir/zfs-0.6.2"
  ./autogen.sh
  if [[ $CARCH == "i686" ]]; then
    ./configure --prefix=/usr \
                --sysconfdir=/etc \
                --sbindir=/usr/bin \
                --libdir=/usr/lib \
                --datadir=/usr/share \
                --includedir=/usr/include \
                --with-udevdir=/lib/udev \
                --libexecdir=/usr/lib/zfs-0.6.2 \
                --with-config=kernel \
                --with-linux=/usr/lib/modules/$MOD_BASENAME/build
  else
    ./configure --prefix=/usr \
                --sysconfdir=/etc \
                --sbindir=/usr/bin \
                --libdir=/usr/lib \
                --datadir=/usr/share \
                --includedir=/usr/include \
                --with-udevdir=/lib/udev \
                --libexecdir=/usr/lib/zfs-0.6.2 \
                --with-config=kernel \
                --with-linux=/usr/lib/modules/$MOD_BASENAME/build
  fi
  make
}

package() {
  cd "$srcdir/zfs-0.6.2"
  make DESTDIR="$pkgdir" install

  # move module tree /lib -> /usr/lib
  cp -r "$pkgdir"/{lib,usr}
  rm -r "$pkgdir"/lib
}
