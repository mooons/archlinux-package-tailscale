# Maintainer: Morten Linderud <foxboron@archlinux.org>
# Maintainer: Christian Heusel <gromit@archlinux.org>
# Contributor: David Anderson <dave@natulte.net>

pkgname=tailscale
pkgver=1.48.2
pkgrel=1
pkgdesc="A mesh VPN that makes it easy to connect your devices, wherever they are."
arch=("x86_64")
url="https://tailscale.com"
license=("MIT")
makedepends=("git" "go")
depends=("glibc")
backup=("etc/default/tailscaled")
# Important: Check if the version has been published before updating
# curl -s "https://pkgs.tailscale.com/stable/?mode=json"
_commit=a6bcfd69149c491d7542cc758b762bae8882db04 #  git rev-parse tags/v1.48.2
source=("git+https://github.com/tailscale/tailscale.git#commit=${_commit}"
        "tailscale_1.48.2_no-self-update.patch")
sha256sums=('SKIP'
            '33105b1986b3b77924fbc3f34f219a67a6cc3846d7411a2165b48af1c616175e')

pkgver() {
  cd "${pkgname}"
  git describe --tags | sed 's/^[vV]//;s/-/+/g'
}

prepare() {
    cd "${pkgname}"
    # disable the autoupdate feature, see https://github.com/tailscale/tailscale/pull/8655#discussion_r1300682857
    # can be removed with the next major release
    patch --forward --strip=1 --input="${srcdir}/tailscale_1.48.2_no-self-update.patch"
    go mod vendor
}

build() {
    cd "${pkgname}"
    export CGO_CPPFLAGS="${CPPFLAGS}"
    export CGO_CFLAGS="${CFLAGS}"
    export CGO_CXXFLAGS="${CXXFLAGS}"
    export CGO_LDFLAGS="${LDFLAGS}"
    # pacman bug
    # export GOPATH="${srcdir}"
    export GOFLAGS="-buildmode=pie -mod=readonly -modcacherw"
    GO_LDFLAGS="\
        -compressdwarf=false \
        -linkmode=external \
        -X tailscale.com/version.longStamp=${pkgver} \
        -X tailscale.com/version.shortStamp=$(cut -d+ -f1 <<< "${pkgver}") \
        -X tailscale.com/version.gitCommitStamp=${_commit}"
    for cmd in ./cmd/tailscale ./cmd/tailscaled; do
        go build -v -tags xversion -ldflags "$GO_LDFLAGS" "$cmd"
    done
}

#TODO: Figure out why tests are failing
# check() {
#     cd "${pkgname}"
#     go test $(go list ./... | grep -v tsdns_test)
# }

package() {
    cd "${pkgname}"
    install -Dm755 tailscale tailscaled -t "$pkgdir/usr/bin"
    install -Dm644 cmd/tailscaled/tailscaled.defaults "$pkgdir/etc/default/tailscaled"
    install -Dm644 cmd/tailscaled/tailscaled.service -t "$pkgdir/usr/lib/systemd/system"
    install -Dm644 LICENSE -t "$pkgdir/usr/share/licenses/$pkgname"
}
