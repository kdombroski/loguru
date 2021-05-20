# Contributor: MBI <series-b-team@marsbioimaging.com>
# Maintainer: Emil Ernerfeldt <emil.ernerfeldt@gmail.com>
pkgname=mbi-loguru-dev
pkgver=2.1.0
pkgrel=18
pkgdesc="A lightweight C++ logging library"
url="https://github.com/emilk/loguru"
arch="all"
license="PD-loguru"
depends="fmt-dev~7 libexecinfo-dev"
makedepends="cmake linux-headers"
options="!check"

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
