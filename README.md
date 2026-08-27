# LinguPlay — downloads

Official installers for [LinguPlay](https://linguplay.com), a video player for
language learners: click any word in the subtitles and the film pauses, a
dictionary shows the meaning, and you can save the word to a deck.

### [⬇ Download the latest release](https://github.com/linguplay/LinguPlay-Releases/releases/latest)

| Platform | File | Notes |
|---|---|---|
| **macOS** (Apple Silicon) | `LinguPlay_native_arm64.dmg` | See the note below on first launch |
| **Windows** | `.msi` or `-setup.exe` | VLC is bundled — no separate install needed |
| **Linux** | `.deb` or `.AppImage` | |
| **Android** | `LinguPlay_<version>_arm64.apk` | Sideload; allow installs from unknown sources |
| **Chrome / Edge extension** | `LinguPlay_Extension_<version>.zip` | Unzip, then load unpacked at `chrome://extensions` |
| **Web app** | — | Runs in the browser at [linguplay.com](https://linguplay.com) |

There is no iPhone build yet — see [PIPELINE.md](PIPELINE.md#ios) for why.

## macOS first launch

These builds are ad-hoc signed but not yet notarized by Apple. The first time
you open LinguPlay, macOS will refuse with a warning. Control-click the app and
choose **Open**, then confirm. You only have to do this once.

## Verifying a download

Each macOS release ships a `.sha256` file beside the DMG:

```bash
shasum -a 256 -c LinguPlay_native_arm64.dmg.sha256
```

## About this repository

This repository contains **release binaries and the workflow that builds them**.
The application source is private.

If you are here to cut a release, or to work out how these files are produced,
read **[PIPELINE.md](PIPELINE.md)** — it is the single source of truth, and it
lists the paths that look plausible but do not work.

## Issues

Bug reports and feature requests are welcome in
[Issues](https://github.com/linguplay/LinguPlay-Releases/issues).
