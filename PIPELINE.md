# The LinguPlay build & release pipeline

**This file is the single source of truth for how LinguPlay artifacts are
produced.** If something you read elsewhere disagrees with this file, this file
wins — and the other document is a bug worth fixing.

---

## The one-paragraph version

LinguPlay's source is private (`linguplay/Linguplay`). Its installers are built
and published here, in the public repository (`linguplay/LinguPlay-Releases`),
by a single workflow: **Release** (`.github/workflows/release.yml`). The web app
is not built by GitHub Actions at all — Cloudflare Pages builds it directly from
the private repository. There is no other supported path.

## Why builds live in the public repository

GitHub bills Actions minutes to *the repository that runs the job*. The private
repository's Actions are billing-blocked: its jobs are rejected before they
start, which is why they fail in three to six seconds with no logs and no steps.
Public repositories run standard GitHub-hosted runners (`ubuntu-*`,
`windows-*`, `macos-*`) at no cost, so the builds run here instead.

The source never lands in this repository. Each job checks the private
repository out into `./source` at build time using a **read-only deploy key**
(`SOURCE_DEPLOY_KEY`), with `persist-credentials: false` so nothing downstream
can reuse that key. Only the resulting installers are ever published.

## Producing a release

1. Open **Actions → Release** in this repository.
2. **Run workflow**, and fill in:
   - `version` — semantic, **no leading `v`** (e.g. `0.3.0`)
   - `source_ref` — the branch, tag, or commit of the private source to build
   - `prerelease` — leave unchecked for a normal release
3. Wait. The run creates `v<version>` with every installer attached.

The workflow refuses to run if the release tag already exists. Bump the version
instead of re-publishing: download URLs like `releases/latest/download/...` are
stable and public, and silently changing what they serve is how you ship a
different binary to someone who already trusted the old one.

### What gets built

| Job | Runner | Ships |
|---|---|---|
| `quality` | ubuntu | nothing — typecheck, lint, tests |
| `macos` | macos | `LinguPlay_native_arm64.dmg` (+ `.sha256`) |
| `desktop` | ubuntu + windows | `.deb`, `.AppImage`, `.msi`, `-setup.exe` |
| `android` | ubuntu | `LinguPlay_<v>_arm64.apk`, `LinguPlay_<v>.aab` |
| `extension` | ubuntu | `LinguPlay_Extension_<v>.zip` |
| `ios` | macos | nothing — build validation only (see below) |
| `publish` | ubuntu | the GitHub Release itself |

`quality` runs once, up front, so a broken commit fails in about two minutes
rather than after six parallel platform builds each rediscover it.

`publish` waits on `macos`, `desktop`, `android` and `extension` — every job
that actually ships something. It deliberately does **not** wait on `ios`,
because a platform that ships nothing must not be able to block the platforms
that do.

### Asset names are load-bearing

The landing page finds downloads by file extension, and the macOS filename is
hardcoded, in the private repo's `src/lib/releaseDownloads.ts`:

| Platform | Matched by |
|---|---|
| macOS | the exact name `LinguPlay_native_arm64.dmg` |
| Windows | ends with `.msi` |
| Linux | ends with `.deb` |
| Android | ends with `.apk` |
| Extension | ends with `.zip` |

So: do not rename these assets, and do not attach a second file ending in one of
those extensions — the page takes the *first* match and would start handing
users the wrong file. Checksums are safe because `.dmg.sha256` does not end in
`.dmg`.

## The web app is not built here

The web app deploys through **Cloudflare Pages' own Git integration**, which
builds from the private repository on Cloudflare's infrastructure and uses zero
GitHub Actions minutes.

- Production: <https://linguplay.com>
- Build command: `npm run build`, output directory: `dist`
- Environment variables live in the Cloudflare Pages project settings, not here.

Do not add a web-deploy workflow to this repository or the private one. The
private repo's `deploy-web.yml` is superseded and its automatic triggers have
been removed.

## Turning on the parts that are switched off

### iOS

The `ios` job compiles the app for the simulator, which proves the
Rust/Swift/WebView integration still builds. It cannot produce an installable
`.ipa`, because that requires:

- a paid Apple Developer Program membership (99 USD/year),
- a distribution certificate and provisioning profile, and
- secrets for both, plus an App Store Connect API key for TestFlight upload.

Until those exist, **there is no iPhone installer to download**, and no amount
of workflow editing will create one. When the account exists, add the signing
secrets and extend the `ios` job to build, sign, export an `.ipa`, and add `ios`
to `publish`'s `needs`.

### macOS notarization

The DMG is ad-hoc signed but not notarized, so first launch needs a
Control-click → **Open**. Notarizing needs the same paid Apple account plus an
`altool`/`notarytool` credential.

### Tauri auto-update

Silent in-app updates need a Tauri updater signing key and a signed
`latest.json` attached to each release. Until then the app links users to the
public releases page.

## Required secrets and variables (this repository)

Secrets — Settings → Secrets and variables → Actions → Secrets:

| Secret | Purpose |
|---|---|
| `SOURCE_DEPLOY_KEY` | read-only SSH key for the private source repo |
| `ANDROID_KEYSTORE_BASE64` | base64 of the Play upload keystore |
| `ANDROID_KEYSTORE_PASSWORD` | keystore password |
| `ANDROID_KEY_ALIAS` | key alias inside the keystore |
| `ANDROID_KEY_PASSWORD` | key password |

`GITHUB_TOKEN` is provided automatically and is scoped to this repository, which
is exactly enough to publish a release here and nothing more.

Variables — same page, Variables tab:

| Variable | Value |
|---|---|
| `VITE_SUPABASE_URL` | the Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | the **publishable** anon key |
| `VITE_PUBLIC_SITE_URL` | `https://linguplay.com` |
| `VITE_PAYMENT_GATEWAY` | the active gateway, or empty to disable checkout |

Only the publishable anon key belongs in a variable. A Supabase **service-role**
key, a Cloudflare token, or a payment provider's secret must never be set here —
none of the release jobs need one.

### About the Android keystore

Losing this keystore means never being able to update the Play Store listing
again, because Android refuses an update signed with a different key. Keep the
original `.jks` backed up somewhere safe and offline; the copy in Actions
secrets is a build input, not a backup.

Secrets are encrypted, are never exposed to workflows triggered from forks, and
can only be used by runs that someone with write access started. That is the
same protection they had in the private repository.

## Things that will waste your time

- **Adding a build workflow to the private repo.** Its Actions are
  billing-blocked. The job will fail in seconds with no logs. That failure is
  not a bug in your workflow.
- **Expecting a push to build something.** Every build here is
  `workflow_dispatch` — manual, with an explicit version. Nothing builds on push.
- **Looking for `publish-macos.yml`.** It was folded into `release.yml`; having
  two ways to cut a macOS release is how two different `v0.2.6` builds happen.
- **Committing source here.** This repository holds workflows, docs and release
  assets. The application source is private and must stay that way.
