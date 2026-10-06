# Maintainer: Lunix <aur@lunixizm.website>
pkgname=eba-certs-bin
pkgver=1.0.2
pkgrel=1
pkgdesc='MEB Fatih root CA certificates for Fatih Network access (repackaged from Pardus .deb)'
arch=('any')
url='https://github.com/Lunixizm0/eba-certs-bin'
license=('GPL-3.0-or-later')
makedepends=('libarchive')
depends=('ca-certificates')
optdepends=(
  'firefox: to apply the bundled policies.json'
  'firefox-esr: to apply the bundled policies.json'
  'chromium: uses the system trust store'
  'google-chrome: uses the system trust store'
)
provides=('eba-certs')
conflicts=('eba-certs')
backup=(
  'etc/firefox/policies/policies.json'
  'etc/skel/.pki/nssdb/cert9.db'
)
source=("https://depo.pardus.org.tr/pardus/pool/contrib/e/eba-certs/eba-certs_${pkgver}_amd64.deb")
sha256sums=('83929e36c68fb5950423e1a76ad498be67785e4f30137fc4462959597736d6f7')
noextract=("eba-certs_${pkgver}_amd64.deb")
# nothing to strip or debug in package
options=('!strip' '!debug')

prepare() {
  bsdtar -xf "${srcdir}/eba-certs_${pkgver}_amd64.deb" -C "${srcdir}" data.tar.xz
}

package() {
  bsdtar -xf "${srcdir}/data.tar.xz" -C "${pkgdir}"

# Arch reads trust anchors from trust-source/ not from debians
# /usr/local/share/ca-certificates keep the originals (policies.json
# points at them) and additionally install into the Arch anchor
# directory pacmans 40-update-ca-trust hook rebuilds the stores on
# install/upgrade/remove automatically so no install scriptlet is needed
  install -Dm644 "${pkgdir}/usr/local/share/ca-certificates/MEB1.crt" \
    "${pkgdir}/usr/share/ca-certificates/trust-source/anchors/MEB1.crt"
  install -Dm644 "${pkgdir}/usr/local/share/ca-certificates/MEB2.crt" \
    "${pkgdir}/usr/share/ca-certificates/trust-source/anchors/MEB2.crt"

  # Canonical policy file lives in /etc (backupd) the /usr/lib copies are
  # symlinks to it
  install -Dm644 "${pkgdir}/usr/lib/firefox/distribution/policies.json" \
    "${pkgdir}/etc/firefox/policies/policies.json"

  # firefox-esr lives under /usr/lib on arch not /usr/share as on debian drop the Debian path entirely
  rm -rf "${pkgdir}/usr/share/firefox-esr"
  rm -f "${pkgdir}/usr/lib/firefox/distribution/policies.json"
  install -dm755 "${pkgdir}/usr/lib/firefox/distribution" \
    "${pkgdir}/usr/lib/firefox-esr/distribution"
  ln -s /etc/firefox/policies/policies.json \
    "${pkgdir}/usr/lib/firefox/distribution/policies.json"
  ln -s /etc/firefox/policies/policies.json \
    "${pkgdir}/usr/lib/firefox-esr/distribution/policies.json"

  install -Dm644 "${startdir}/LICENSE" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  #drop debian-specific packaging docs
  rm -rf "${pkgdir}/usr/share/doc"
}
