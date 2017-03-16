# $Id: PKGBUILD 202763 2016-12-26 15:38:29Z anatolik $
# Mantainer: Felipe Magno de Almeida <felipe.m.almeida@gmail.com>
# Contributor: schuay <jakob.gruber@gmail.com>
# Contributor: Brad Fanella <bradfanella@archlinux.us>
# Contributor: Corrado Primier <bardo@aur.archlinux.org>
# Contributor: danst0 <danst0@west.de>

# Build order: avr-binutils -> avr-gcc -> avr-libc -> avr-gcc-libstdcxx

pkgname=avr-gcc-libstdcxx
pkgver=6.3.0
pkgrel=1
#_snapshot=6-20161222
_snapshot=7-20170115
#_snapshot=7-20170312
_islver=0.18
pkgdesc='The GNU AVR Compiler Collection'
arch=(i686 x86_64)
license=(GPL LGPL FDL custom)
url='http://gcc.gnu.org/'
depends=(avr-binutils gcc-libs libmpc)
optdepends=('avr-libc: Standard C library for Atmel AVR development')
options=(!emptydirs !strip)
source=(#ftp://gcc.gnu.org/pub/gcc/releases/gcc-${pkgver}/gcc-${pkgver}.tar.bz2
        ftp://gcc.gnu.org/pub/gcc/snapshots/${_snapshot}/gcc-${_snapshot}.tar.bz2
        http://isl.gforge.inria.fr/isl-${_islver}.tar.bz2
       )
sha1sums=('SKIP'
          'bbffc5a2b05e4f0c97e882f96c448504491dc4ed')
provides=(avr-gcc)

if [ -n "${_snapshot}" ]; then
  _basedir=gcc-${_snapshot}
else
  _basedir=gcc-${pkgver}
fi

build() {
    cd ${srcdir}/${_basedir} 

    # link isl for in-tree build
    ln -s ../isl-${_islver} isl

    # https://bugs.archlinux.org/task/34629
    # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
    sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

    echo ${pkgver} > gcc/BASE-VER

    cd ${srcdir}
    mkdir gcc-build && cd gcc-build

    export CFLAGS_FOR_TARGET='-O2 -pipe'
    export CXXFLAGS_FOR_TARGET='-O2 -pipe'

    # --disable-linker-build-id   https://bugs.archlinux.org/task/34902
    # --disable-__cxa_atexit   https://bugs.archlinux.org/task/50848
    ${srcdir}/${_basedir}/configure \
                --disable-install-libiberty \
                --disable-libssp \
                --disable-libstdcxx-pch \
                --disable-libunwind-exceptions \
                --disable-linker-build-id \
                --disable-nls \
                --disable-werror \
                --disable-__cxa_atexit \
                --enable-checking=release \
                --enable-clocale=gnu \
                --enable-gnu-unique-object \
                --enable-gold \
                --enable-languages=c,c++ \
                --enable-ld=default \
                --enable-lto \
                --enable-plugin \
                --enable-shared \
                --infodir=/usr/share/info \
                --libdir=/usr/lib \
                --libexecdir=/usr/lib \
                --mandir=/usr/share/man \
                --prefix=/usr \
                --target=avr \
                --with-as=/usr/bin/avr-as \
                --with-gnu-as \
                --with-gnu-ld \
                --with-ld=/usr/bin/avr-ld \
                --with-plugin-ld=ld.gold \
                --with-system-zlib \
                --with-isl \
                --enable-gnu-indirect-function \
                --enable-libstdcxx \
                --enable-cxx-flags='-fno-exceptions -fno-rtti' \
                --disable-libstdcxx-verbose \
                --disable-vtable-verify \
                --disable-libstdcxx-time \
                --disable-libstdcxx-threads \
                --disable-threads \
                --enable-libstdcxx-allocator=malloc \
                --disable-libstdcxx-filesystem-ts
                

    make
}

package() {
    cd ${srcdir}/gcc-build

    make -j1 DESTDIR=${pkgdir} install

    # Strip debug symbols from libraries; without this, the package size balloons to ~500MB.
    find ${pkgdir}/usr/lib -type f -name "*.a" \
        -exec /usr/bin/avr-strip --strip-debug '{}' \;

    # Install Runtime Library Exception
    install -Dm644 ${srcdir}/${_basedir}/COPYING.RUNTIME \
        ${pkgdir}/usr/share/licenses/avr-gcc/RUNTIME.LIBRARY.EXCEPTION

    rm -r ${pkgdir}/usr/share/man/man7
    rm -r ${pkgdir}/usr/share/info
    rm ${pkgdir}/usr/lib/libcc1.*
}
