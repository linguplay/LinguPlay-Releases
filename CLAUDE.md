# LinguPlay-Releases

The instructions for this repository live in **[AGENTS.md](AGENTS.md)**, and the
full build and release reference is **[PIPELINE.md](PIPELINE.md)**.

Read AGENTS.md before acting. The short version:

- This public repository builds and publishes LinguPlay's installers.
- The application source is private (`linguplay/Linguplay`) and is checked out
  at build time. Never commit source here.
- Builds run here because the private repository's Actions are billing-blocked;
  public repositories run standard runners for free.
- To release: Actions → **Release** → Run workflow. Nothing builds on push.
