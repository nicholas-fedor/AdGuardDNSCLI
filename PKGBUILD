# shellcheck disable=SC2148,SC2034,SC2154,SC2164
# Maintainer: Nicholas Fedor <nick@nickfedor.com>

pkgname=adguard-dns-cli
_pkgname=AdGuardDNSCLI
_basever=0.0.1
pkgver=0.1.1.r55.g6bce74a
pkgrel=1
pkgdesc='A cross-platform lightweight DNS client for AdGuard DNS'
arch=('x86_64' 'aarch64')
backup=('opt/adguard-dns-cli/config.yaml')
url='https://github.com/nicholas-fedor/AdGuardDNSCLI'
license=('Apache-2.0')
depends=()
makedepends=('go>=1.26.2' 'git')
checkdepends=()
source=()

# Locate the extracted source directory (named AdGuardDNSCLI-<branch>).
_srcdir() {
    echo "${startdir}"
}

# Update the package version to use the conventional git version string
pkgver() {
    cd "${startdir}"
    git describe --long --tags 2>/dev/null | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g' || echo "${_basever}+master"
}

# Download dependencies
prepare() {
    cd "$(_srcdir)" || exit

    # Create directory for Go module cache
    mkdir -p "$srcdir/go"

    # Download dependencies
    export GOPATH="$srcdir/go"
    export GO111MODULE=on
    export GOPROXY=https://proxy.golang.org,direct
    go mod download
}

# Run tests
check() {
    cd "$(_srcdir)" || exit

    export GOPATH="$srcdir/go"
    export GO111MODULE=on
    export CGO_ENABLED=0

    # Run tests
    go test -v ./...
}

# Build the release package
build() {
    cd "$(_srcdir)" || exit

    # Set Go environment
    export GOPATH="$srcdir/go"
    export GO111MODULE=on
    export CGO_ENABLED=0
    export GOPROXY=https://proxy.golang.org,direct

    # Set branch
    local _branch
    _branch="$(git rev-parse --abbrev-ref HEAD)"

    # Version package path
    local version_pkg='github.com/AdguardTeam/AdGuardDNSCLI/internal/version'

    # Build ldflags
    local ldflags="-s -w"
    ldflags="${ldflags} -X ${version_pkg}.branch=${_branch}"
    ldflags="${ldflags} -X ${version_pkg}.committime=${SOURCE_DATE_EPOCH:-$(date +%s)}"
    ldflags="${ldflags} -X ${version_pkg}.revision=v${_basever}+${_branch}"
    ldflags="${ldflags} -X ${version_pkg}.version=$(git describe --tags --abbrev=0 2>/dev/null || echo v${_basever})"

    # Build the binary
    go build \
        -ldflags="$ldflags" \
        -trimpath \
        -o "adguarddns-cli" \
        .
}

# Install the release binary and config to /opt and symlink to /usr/bin
package() {
    cd "$(_srcdir)" || exit

    # Install the binary to /opt (the app looks for config.yaml in the same directory)
    install -Dm755 "adguarddns-cli" "$pkgdir/opt/$pkgname/adguarddns-cli"

    # Install the configuration file to /opt alongside the binary
    # The backup array above ensures pacman creates .pacnew on upgrades
    # when the user has modified their config
    install -Dm644 "config.dist.yaml" "$pkgdir/opt/$pkgname/config.yaml"

    # Create a symlink in /usr/bin for easy access
    install -d "$pkgdir/usr/bin"
    ln -s "/opt/$pkgname/adguarddns-cli" "$pkgdir/usr/bin/adguarddns-cli"

    # Install documentation
    install -Dm644 "README.md" "$pkgdir/usr/share/doc/$pkgname/README.md"
    install -Dm644 "CHANGELOG.md" "$pkgdir/usr/share/doc/$pkgname/CHANGELOG.md"
    install -Dm644 "CONTRIBUTING.md" "$pkgdir/usr/share/doc/$pkgname/CONTRIBUTING.md"

    # Install license
    install -Dm644 "LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
