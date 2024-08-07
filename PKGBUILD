# Maintainer: Kuan-Yen Chou <kuanyenchou@gmail.com>

pkgname=mininet-git
pkgver=2.3.1b4.r13.g6eb8973
pkgrel=3
pkgdesc='Emulator for rapid prototyping of Software Defined Networks'
depends=('python' 'iproute2' 'net-tools' 'iputils' 'inetutils' 'iperf' 'ethtool'
         'libcgroup' 'openvswitch' 'psmisc')
optdepends=('xorg-xhost: for X11 forwarding'
            'socat: for X11 forwarding'
            'xterm: required for MiniEdit'
            'tk: required for MiniEdit')
makedepends=('git' 'help2man' 'python-setuptools' 'python-build'
             'python-installer' 'python-wheel')
arch=('x86_64')
url='https://github.com/mininet/mininet'
license=('custom')
provides=('mininet')
conflicts=('mininet')
install=mininet.install
source=("$pkgname"::'git+https://github.com/mininet/mininet'
        'git+https://github.com/mininet/openflow'   # for UserSwitch
        'fix-openflow-strlcpy.patch'
        'fix-openflow-w-gcc-14-clang-17.patch')
sha256sums=('SKIP'
            'SKIP'
            '0a85f8a5ce2dd900d4f874849b28301aa47d7b9d7b03ed405c973d917d98383a'
            '7258329a8df02c2b4bc87697daff12bc45ce8b2fc3145d2b6c520a8dd7ad8986')

pkgver() {
    cd "$srcdir/$pkgname"
    if git describe --long --tags >/dev/null 2>&1; then
        git describe --long --tags | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
    else
        printf 'r%s.%s' "$(git rev-list --count HEAD)" "$(git describe --always)"
    fi
}

prepare() {
    cd "$srcdir/openflow"
    sed '/^include debian\/automake.mk/d' -i Makefile.am
    # Patch controller to handle more than 16 switches
    patch -Np1 -i "$srcdir/$pkgname/util/openflow-patches/controller.patch"
    patch -Np1 -i "$srcdir/fix-openflow-strlcpy.patch"
    patch -Np1 -i "$srcdir/fix-openflow-w-gcc-14-clang-17.patch"

    cd "$srcdir/$pkgname"
    # shellcheck disable=SC2016
    sed -i Makefile \
        -e 's:PREFIX ?= /usr:PREFIX ?= "$(DESTDIR)"/usr:' \
        -e '/^[[:space:]]*$(PYTHON) /d'
}

build() {
    cd "$srcdir/openflow"
    autoreconf --install --force
    ./configure --prefix=/usr --sbindir=/usr/bin
    make

    cd "$srcdir/$pkgname"
    make mnexec man
    python -m build --wheel --no-isolation
}

package() {
    cd "$srcdir/openflow"
    make DESTDIR="${pkgdir}" install

    cd "$srcdir/$pkgname"
    make DESTDIR="${pkgdir}" install
    python -m installer \
        --prefix=/usr \
        --destdir="$pkgdir" \
        --compile-bytecode=1 \
        dist/*.whl
    install -Dm 644 LICENSE "${pkgdir}/usr/share/licenses/mininet/LICENSE"
}

# vim: set sw=4 ts=4 et:
