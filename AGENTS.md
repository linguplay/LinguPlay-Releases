# Instructions for agents and contributors

Read this before doing anything in this repository. It exists to stop a
plausible-looking wrong turn.

## What this repository is

`linguplay/LinguPlay-Releases` is **public** and holds three things:

1. `.github/workflows/release.yml` — the only pipeline that builds LinguPlay
2. `.github/actions/setup-source/` — the shared "check out the private source" step
3. Published release assets, plus this documentation

The **application source is not here and must never be committed here.** It
lives in the private repository `linguplay/Linguplay`.

## The rule that explains everything else

Builds run in this public repository because **GitHub bills Actions minutes to
the repository that runs the job**, and the private repository's Actions are
billing-blocked. Public repositories run standard runners for free.

So: **all building and releasing happens here; all source lives there.**

## Before you act

**To cut a release** → Actions → **Release** → Run workflow. Nothing builds on
push; every build is manual and takes an explicit version.

**To change how something is built** → edit `.github/workflows/release.yml`
*in this repository*.

**To change the app itself** → that is the private repository. Not here.

**To deploy the web app** → you do not. Cloudflare Pages builds it directly from
the private repository. Do not add a web-deploy workflow anywhere.

**Read [PIPELINE.md](PIPELINE.md) first** for anything beyond the above.

## Known wrong turns

These all look reasonable and all waste time:

- **"CI is broken, let me fix the failing workflows in the private repo."**
  They fail in 3–6 seconds with no logs because the jobs are rejected before
  they start — that is the billing block, not a defect in the YAML. Do not
  debug them. Do not re-add their triggers.
- **"I'll add a build workflow to the private repo, that's where the code is."**
  It will never run. The code is fetched from there at build time instead.
- **"I'll make the source public so Actions is free."** No. The split exists
  precisely so the source can stay private.
- **"There are two macOS release workflows, I'll use the older one."** There is
  one. `publish-macos.yml` was folded into `release.yml`.
- **"I'll rename the release assets to something tidier."** The landing page
  matches assets by extension and hardcodes `LinguPlay_native_arm64.dmg`.
  Renaming breaks every download button. See PIPELINE.md, "Asset names are
  load-bearing".
- **"iOS just needs the workflow finished."** It needs a paid Apple Developer
  account and signing certificates. No workflow edit substitutes for that.
- **"I'll re-run the release to fix an asset."** Publishing over an existing tag
  changes what a public, already-shared download URL serves. Bump the version.
  The workflow blocks this on purpose.

## Secrets

`SOURCE_DEPLOY_KEY` is read-only and scoped to the source repository.
`GITHUB_TOKEN` is scoped to this repository. The `ANDROID_KEYSTORE_*` secrets
sign the Android build.

Never add a Supabase service-role key, a Cloudflare API token, or a payment
provider secret here. No release job needs one, and this repository is public —
only the *values* of secrets are hidden, everything else is visible.
