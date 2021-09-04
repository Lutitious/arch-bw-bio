# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=1.28.2
pkgrel=2
_jslibcommit='5ab045499f8a6a26ee10115f458136546a54bc22'
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/bitwarden/desktop'
license=('GPL3')
depends=('electron11' 'libnotify' 'libsecret' 'libxtst' 'libxss' 'libnss_nis')
makedepends=('git' 'npm' 'python' 'node-gyp' 'nodejs-lts-fermium' 'jq')
source=(${pkgname}-${pkgver}.tar.gz::https://github.com/bitwarden/desktop/archive/v${pkgver}.tar.gz
        jslib-${_jslibcommit}.tar.gz::https://github.com/bitwarden/jslib/archive/${_jslibcommit}.tar.gz
        package.json.patch
        messaging.main.ts.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('e950423968043480dcf9c93dafe0acad042baa93d04261292ce2e7d223c78f444d59d8f382e1a390ac942431c5b67f6950de4c6bba7e7deed685b3d9b11748c5'
            'c3705b257c080b72f15c62462bf8713cfc24a044395415d8884b31e7bdf5f4d7d04f56cdb1ac4ec3a7b92dc4eb2373f6135203925882357c886a4359908f210f'
            'd884221c615db95d6fd0da2d3470fb7514b6a5d2a2b3b20c8353ebb4a938dc39f93783fe7ef2b9f69f034db8f26abfa479616f9fd1c1b241af605da837fba20e'
            '822d97be407c2ac2a6926f5c925b0fd188c541014a623dd3815fdbf5ef67c0542f43aaf8d11535571a83a265f620e330f5326244f42c3902fddab442128fda95'
            '32c29a7baed80351acf5753d35df404a818d5c88cc85f3bbed2daa5351aaf0dba20fd03cbedbcb407324f305d4556adb476ecc9ccd07bac0511ca4a943020ea4'
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
	SYSTEM_ELECTRON_VERSION=$(electron11 -v | sed 's/v//g')
	jq < package.json --arg ver $SYSTEM_ELECTRON_VERSION \
	   '.build["electronVersion"]=$ver | .build["electronDist"]="/usr/lib/electron11"' \
	   > package.json.patched
	mv package.json.patched package.json
}

build() {
	cd desktop-${pkgver}
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
	electronDist=$(dirname $(realpath $(which electron11)))
	electronVer=$(electron11 --version | tail -c +2)
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
	install -vDm755 dist/linux-unpacked/resources/app.asar -t "${pkgdir}/usr/lib/${pkgname%-git}/resources"

	for i in 16 32 48 64 128 256 512; do
		install -vDm644 resources/icons/${i}x${i}.png "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname%-git}.png"
	done
	install -vDm644 resources/icon.png "${pkgdir}/usr/share/icons/hicolor/1024x1024/apps/${pkgname%-git}.png"

	install -vDm755 "${srcdir}/${pkgname%-git}.sh" "${pkgdir}/usr/bin/bitwarden-desktop"
	install -vDm644 "${srcdir}"/${pkgname%-git}.desktop -t "${pkgdir}"/usr/share/applications
}
