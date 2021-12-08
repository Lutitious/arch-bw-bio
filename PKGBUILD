# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=1.30.0
pkgrel=1
_jslibcommit='f4c66b2c8c243935bf25f689b16afaa5d6345f1b'
_electronversion=14
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/desktop'
license=('GPL3')
depends=("electron$_electronversion" 'libnotify' 'libsecret' 'libxtst' 'libxss' 'libnss_nis')
makedepends=('npm' 'python' 'node-gyp' 'nodejs-lts-fermium' 'jq')
source=(${pkgname}-${pkgver}.tar.gz::https://github.com/bitwarden/desktop/archive/v${pkgver}.tar.gz
        jslib-${_jslibcommit}.tar.gz::https://github.com/bitwarden/jslib/archive/${_jslibcommit}.tar.gz
        package.json.patch
        messaging.main.ts.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('b15c4b90b2f541231b090e711b078204de2d9115301b126018b9ab3d828f4d0376a7771ffb30c9eb33e005b684bbc4deb11a83841b9c570eaabd5ebadb38d5ff'
            'b0fa3cd70031dcb87c515beecda35fadb8980ec696a90bbe051a750fd6fdd202902c3c717285a7ccb99d8629b106fbd365b25b39f629001c64123dab2e5521f0'
            '87cdb8287cbc0c4eb49b0fd456a66e200551b5da5c14991505f6301cf1b11132d938dfdf795c4df2a4b3e1ae2badf5dfe33c1207923ec8abc6f9b3e064af6015'
            '822d97be407c2ac2a6926f5c925b0fd188c541014a623dd3815fdbf5ef67c0542f43aaf8d11535571a83a265f620e330f5326244f42c3902fddab442128fda95'
            '44ee70d71abf9cf399736d00df0aa6815d452792c9589f5517fed4454bdfff6ad2a39ffee401eab0db180718b19e9565d9ecff8d1bd96a93d13e4f63eaf4d5fc'
            '05b771e72f1925f61b710fb67e5709dbfd63855425d2ef146ca3770b050e78cb3933cffc7afb1ad43a1d87867b2c2486660c79fdfc95b3891befdff26c8520fd')

prepare() {
	cd desktop-${pkgver}
	# Link jslib
	rmdir -v jslib
	ln -vs ../jslib-${_jslibcommit} jslib

	# Remove pre and postinstall routines from package.json.
	patch --strip=1 package.json ../package.json.patch
	# This patch is required to make "Start automatically on login" work
	patch --strip=1 src/main/messaging.main.ts ../messaging.main.ts.patch
	# Patch build to make it work with system electron
	export SYSTEM_ELECTRON_VERSION=$(electron$_electronversion -v | sed 's/v//g')
	export ELECTRONVERSION=$_electronversion
	jq < package.json \
	   '.build["electronVersion"]=$ENV.SYSTEM_ELECTRON_VERSION | .build["electronDist"]="/usr/lib/electron\(env.ELECTRONVERSION)"' \
	   > package.json.patched
	mv package.json.patched package.json
}

build() {
	cd desktop-${pkgver}
	electronDist=/usr/lib/electron$_electronversion
	electronVer=$(electron$_electronversion --version | tail -c +2)
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
	pushd jslib
	npm install
	popd
	npm install
	npm run build
	npm run clean:dist
	npm exec -c "electron-builder --linux --x64 --dir -c.electronDist=$electronDist \
	             -c.electronVersion=$electronVer"
}

package(){
	cd desktop-${pkgver}
	install -vDm644 dist/linux-unpacked/resources/app.asar -t "${pkgdir}/usr/lib/${pkgname}"
	install -vDm644 build/package.json -t "${pkgdir}/usr/lib/${pkgname}"

	for i in 16 32 48 64 128 256 512; do
		install -vDm644 resources/icons/${i}x${i}.png "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname}.png"
	done
	install -vDm644 resources/icon.png "${pkgdir}/usr/share/icons/hicolor/1024x1024/apps/${pkgname}.png"

	install -vDm755 "${srcdir}/${pkgname}.sh" "${pkgdir}/usr/bin/bitwarden-desktop"
	install -vDm644 "${srcdir}"/${pkgname}.desktop -t "${pkgdir}"/usr/share/applications
}
