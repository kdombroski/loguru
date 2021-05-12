# Contributor: MBI <series-b-team@marsbioimaging.com>
# Maintainer: Emil Ernerfeldt <emil.ernerfeldt@gmail.com>
pkgname=mbi-loguru
pkgver=2.1.0
pkgrel=13
pkgdesc="A lightweight C++ logging library"
url="https://github.com/emilk/loguru"
arch="all"
license="PD-loguru"
depends="fmt~7 libexecinfo"
makedepends="cmake fmt-dev~7 linux-headers libexecinfo-dev"
install=""
source=""
builddir=""
srcdir="tempsrc"
options="!check"
filter_xattrs_selinux=1

build() {
	cmake -B build \
		-DCMAKE_INSTALL_PREFIX=/usr \
		-DCMAKE_INSTALL_LIBDIR=lib \
		-DBUILD_SHARED_LIBS=False \
		-DCMAKE_BUILD_TYPE=None \
		.
	cmake --build build
}

package() {
	DESTDIR="$pkgdir" cmake --install build
}
