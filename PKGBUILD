# Maintainer: Béla (Greenman999) <git@greenman999.de>
# LearnHarness fork with integrated PRs #201 (Exa search) and #78
# (OpenAI-compatible provider). Built from source the same way upstream's
# official Linux packages are made (docs/desktop-release.md steps 1-5 +
# electron-builder --linux), installed as a native pacman package instead
# of a .deb/AppImage.

pkgname=learnharness-fork-git
pkgver=0.3.9.r814.ff08443
pkgrel=1
pkgdesc="Agent harness for adaptive teaching, practice, and a knowledge graph (Greenman999 fork: +Exa, +OpenAI-compatible provider)"
arch=('x86_64')
url="https://github.com/Greeenman999/learnharness"
license=('AGPL-3.0-only')
depends=('alsa-lib' 'gtk3' 'nss' 'libsecret' 'libnotify' 'libxss' 'xdg-utils')
makedepends=('git' 'nodejs>=20' 'npm' 'python' 'unzip')
provides=('learnharness')
conflicts=('learnharness' 'learnharness-bin')
source=("$pkgname::git+https://github.com/Greeenman999/learnharness.git#branch=integration/exa-openai-compatible")
sha256sums=('SKIP')

pkgver() {
  cd "$srcdir/$pkgname"
  printf "%s.r%s.%s" "$(grep -oP '(?<="version": ")[^"]*' package.json | head -1)" \
    "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

build() {
  cd "$srcdir/$pkgname"

  # CI parity: install exactly what the lockfile pins.
  npm ci

  # Next standalone build (docs/desktop-release.md step 3).
  npm run build

  # Curate the standalone tree: static assets in, old installers and the
  # dev electron runtime out, leaked data refused (step 4).
  node scripts/build-server.mjs

  # Electron's zip download needs the network on first build; makepkg's
  # env is fine for that. Native deps (@napi-rs/canvas, resvg) are
  # prebuilt binaries from npm, no compiler needed beyond 'python'.
  npx electron-builder --linux --x64 --dir --publish never
}

package() {
  local _app=/opt/LearnHarness
  local _res="$srcdir/$pkgname/dist/linux-unpacked"

  install -d "$pkgdir/$_app" \
             "$pkgdir/usr/bin" \
             "$pkgdir/usr/share/applications" \
             "$pkgdir/usr/share/icons/hicolor/512x512/apps" \
             "$pkgdir/usr/share/licenses/$pkgname"

  # The unpacked electron app (app.asar + resources/server standalone tree).
  cp -a "$_res/." "$pkgdir/$_app/"
  # Electron's SUID sandbox helper needs root ownership + setuid to work
  # in the installed location; pacman handles the 4755 mode fine.
  chmod 4755 "$pkgdir/$_app/chrome-sandbox"

  # Launcher
  cat > "$pkgdir/usr/bin/learnharness" <<EOF
#!/bin/sh
exec /opt/LearnHarness/learnharness "\$@"
EOF
  chmod 755 "$pkgdir/usr/bin/learnharness"

  # Desktop entry + icon (build/icon.png is 512x512 per the repo).
  cat > "$pkgdir/usr/share/applications/learnharness.desktop" <<EOF
[Desktop Entry]
Type=Application
Name=LearnHarness
GenericName=AI learning harness
Comment=Adaptive teaching, practice, and a knowledge graph
Exec=/opt/LearnHarness/learnharness %U
Icon=learnharness
Terminal=false
Categories=Education;Science;
StartupWMClass=LearnHarness
EOF
  install -Dm644 "$srcdir/$pkgname/build/icon.png" \
    "$pkgdir/usr/share/icons/hicolor/512x512/apps/learnharness.png"
  install -Dm644 "$srcdir/$pkgname/LICENSE" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
