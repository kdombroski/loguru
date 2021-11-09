# Contributor: MBI <series-b-team@marsbioimaging.com>
# Maintainer: Emil Ernerfeldt <emil.ernerfeldt@gmail.com>
pkgname=mbi-loguru-dev
pkgver=2.1.0
pkgrel=23
pkgdesc="A lightweight C++ logging library"
url="https://github.com/emilk/loguru"
arch="all"
license="PD-loguru"
depends="fmt~7"
dev_depends="fmt-dev~7"
makedepends="cmake linux-headers fmt-dev~7"
options="!check"

build() {
	cmake -B ${builddir} \
		-DCMAKE_INSTALL_PREFIX=/usr \
		-DCMAKE_INSTALL_LIBDIR=lib \
		-DBUILD_SHARED_LIBS=False \
		-DCMAKE_BUILD_TYPE=None \
		.
	cmake --build ${builddir}
}

package() {
	DESTDIR="$pkgdir" cmake --install ${builddir}
}
