# learnharness-fork-git

Arch/Manjaro PKGBUILD for Béla's LearnHarness fork
([Greeenman999/learnharness](https://github.com/Greeenman999/learnharness),
branch `integration/exa-openai-compatible`): upstream + PR 201 (Exa search
provider) + PR 78 (OpenAI-compatible provider).

## Install (on the Manjaro PC)

```sh
sudo pacman -S --needed base-devel git
git clone <this folder's contents or a repo you push it to> learnharness-fork-git
cd learnharness-fork-git
makepkg -si
```

makepkg will: clone the fork, `npm ci`, `npm run build` (Next standalone),
curate the server tree, pack it with electron-builder, and install to
`/opt/LearnHarness` with a `/usr/bin/learnharness` launcher, desktop entry,
and icon. Updates: re-run `makepkg -si` after the fork's branch moves
(pacman-managed: `pacman -Q learnharness-fork-git`, upgrades replace it).

## Build requirements

- RAM: the Next production build needs ~4 GB free. On smaller machines add
  `NODE_OPTIONS="--max-old-space-size=1536" npm run build` in build().
- Disk: ~3 GB in src/ + ~600 MB package.
- First build downloads the Electron 44 zip (~100 MB) from GitHub.

## Notes

- `conflicts=('learnharness' 'learnharness-bin')`: uninstall the AUR
  learnharness-bin before installing this.
- The app's auto-updater is baked to upstream's GitHub releases
  (electron-builder.yml publish owner=amosblomqvist). This pacman package
  bypasses that: either ignore the in-app update banner (it would install
  upstream's AppImage over your fork's data dir is NOT the case - the data
  dir is shared `~/.config/LearnHarness`, the binary path differs), or
  point it at your own releases later.
- pkgver follows upstream scheme: `<pkg version>.r<commit count>.<sha>`.
