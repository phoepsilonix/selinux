# Maintainer: Nicolas Iooss (nicolas <dot> iooss <at> m4x <dot> org)
# Contributor: Timothée Ravier <tim@siosm.fr>
# Contributor: Nicky726 (Nicky726 <at> gmail <dot> com)
# Contributor: Sergej Pupykin (pupykin <dot> s+arch <at> gmail <dot> com)
# Contributor: Zezadas
#
# This PKGBUILD is maintained on https://github.com/archlinuxhardened/selinux.
# If you want to help keep it up to date, please open a Pull Request there.

pkgname=libselinux
pkgver=3.7
pkgrel=2
pkgdesc="SELinux library and simple utilities"
arch=('i686' 'x86_64' 'armv6h' 'aarch64')
url='https://github.com/SELinuxProject/selinux'
license=('custom')
groups=('selinux')
makedepends=('pkgconf' 'python' 'python-pip' 'python-setuptools' 'ruby' 'xz' 'swig')
depends=('libsepol>=3.7' 'pcre2')
optdepends=('python: python bindings'
            'ruby: ruby bindings')
conflicts=("selinux-usr-${pkgname}")
provides=("selinux-usr-${pkgname}=${pkgver}-${pkgrel}")
validpgpkeys=(
  '63191CE94183098689CAB8DB7EF137EC935B0EAF'  # Jason Zaman <perfinion@gentoo.org>
  'B8682847764DF60DF52D992CBC3905F235179CF1'  # Petr Lautrbach <plautrba@redhat.com>
)
source=("https://github.com/SELinuxProject/selinux/releases/download/${pkgver}/${pkgname}-${pkgver}.tar.gz"{,.asc}
        "0001-libselinux-fix-swig-bindings-for-4.3.0.patch"
        "libselinux.tmpfiles.d")
sha256sums=('ea03f42d13a4f95757997dba8cf0b26321fac5d2f164418b4cc856a92d2b17bd'
            'SKIP'
            'd176a9ec5f48decd8443b48403d03d24546f426cd1858f25270b489e06d29942'
            'afe23890fb2e12e6756e5d81bad3c3da33f38a95d072731c0422fbeb0b1fa1fc')

prepare() {
  cd "${pkgname}-${pkgver}"

  # Apply https://github.com/SELinuxProject/selinux/commit/8e0e718bae53fff30831b92cd784151d475a20da
  # libselinux: fix swig bindings for 4.3.0
  patch -Np2 -i "../0001-libselinux-fix-swig-bindings-for-4.3.0.patch"
}

build() {
  cd "${pkgname}-${pkgver}"

  # Do not build deprecated rpm_execcon() interface. It is useless on Arch Linux anyway.
  export DISABLE_RPM=y

  # Use pcre2 explicitely even though it is the default since
  # https://github.com/SELinuxProject/selinux/commit/e0da140d82c0ebebf1060ce87d0f11276c7fc59a
  export USE_PCRE2=y

  export CFLAGS="${CFLAGS} -fno-semantic-interposition"
  make swigify
  make all
  make PYTHON=/usr/bin/python3 pywrap
  make RUBY=/usr/bin/ruby rubywrap
}

package() {
  provides+=(
    libselinux.so
  )

  cd "${pkgname}-${pkgver}"

  export DISABLE_RPM=y

  make DESTDIR="${pkgdir}" SBINDIR=/usr/bin SHLIBDIR=/usr/lib install
  make DESTDIR="${pkgdir}" PYTHON=/usr/bin/python3 SBINDIR=/usr/bin SHLIBDIR=/usr/lib install-pywrap
  make DESTDIR="${pkgdir}" RUBY=/usr/bin/ruby SBINDIR=/usr/bin SHLIBDIR=/usr/lib install-rubywrap
  /usr/bin/python3 -m compileall "${pkgdir}/$(/usr/bin/python3 -c 'from distutils.sysconfig import *; print(get_python_lib(plat_specific=1))')"

  install -Dm 0644 "${srcdir}"/libselinux.tmpfiles.d "${pkgdir}"/usr/lib/tmpfiles.d/libselinux.conf

  install -Dm 0644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
