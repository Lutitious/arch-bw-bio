# Maintainer: Alexander Epaneshnikov <alex19ep@archlinux.org>
# Contributor: libertylocked <libertylocked@disroot.org>

pkgname=bitwarden
pkgver=2023.7.2
pkgrel=1
_electronversion=24
pkgdesc='A secure and free password manager for all of your devices'
arch=('x86_64')
url='https://github.com/quexten/clients/tree/feature/unix-biometrics'
license=('GPL3')
depends=("electron$_electronversion" 'libnotify' 'libsecret' 'libxtst' 'libxss' 'libnss_nis')
makedepends=('git' 'npm' 'python' 'node-gyp' 'nodejs-lts-hydrogen' 'jq' 'rust')
source=(bitwarden::git+https://github.com/quexten/clients.git#branch=feature/unix-biometrics
        messaging.main.ts.patch
        nativelib.patch
        ${pkgname}.sh
        ${pkgname}.desktop)
sha512sums=('SKIP')

prepare() {
	cd bitwarden/apps/desktop

	export npm_config_build_from_source=true
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1

	# This patch is required to make "Start automatically on login" work
	patch --strip=1 src/main/messaging.main.ts "$srcdir/messaging.main.ts.patch"

	# Patch build to make it work with system electron
	export SYSTEM_ELECTRON_VERSION=$(electron$_electronversion -v | sed 's/v//g')
	export ELECTRONVERSION=$_electronversion
	sed -i "s|@electronversion@|${ELECTRONVERSION}|" "$srcdir/bitwarden.sh"
	# jq < package.json \
	#    '.build["electronVersion"]=$ENV.SYSTEM_ELECTRON_VERSION | .build["electronDist"]="/usr/lib/electron\(env.ELECTRONVERSION)"' \
	#    > package.json.patched
	# mv package.json.patched package.json
	cd ../../
	patch --strip=1 apps/desktop/desktop_native/index.js "$srcdir/nativelib.patch"
	npm ci
}

build() {
	cd bitwarden/apps/desktop
	electronDist=/usr/lib/electron$_electronversion
	electronVer=$(electron$_electronversion --version | tail -c +2)
	export npm_config_build_from_source=true
	export npm_config_cache="$srcdir/npm_cache"
	export ELECTRON_SKIP_BINARY_DOWNLOAD=1
	pushd desktop_native/
	npm run build
	popd
	npm run build
	npm run clean:dist 
	npm exec -c "electron-builder --linux --x64 --dir -c.electronDist=$electronDist \
	             -c.electronVersion=$electronVer"
}

package(){
	cd bitwarden/apps/desktop
	install -vDm644 dist/linux-unpacked/resources/app.asar -t "${pkgdir}/usr/lib/${pkgname}"
	install -vDm644 build/package.json -t "${pkgdir}/usr/lib/${pkgname}"
	cp -vr dist/linux-unpacked/resources/app.asar.unpacked -t "${pkgdir}/usr/lib/${pkgname}"

	for i in 16 32 64 128 256 512 1024; do
		install -vDm644 resources/icons/${i}x${i}.png "${pkgdir}/usr/share/icons/hicolor/${i}x${i}/apps/${pkgname}.png"
	done
	install -vDm644 resources/icon.png "${pkgdir}/usr/share/icons/hicolor/1024x1024/apps/${pkgname}.png"

	install -vDm755 "${srcdir}/${pkgname}.sh" "${pkgdir}/usr/bin/bitwarden-desktop"
	install -vDm644 "${srcdir}"/${pkgname}.desktop -t "${pkgdir}"/usr/share/applications
}
