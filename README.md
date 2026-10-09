# Open Source Extensions

Automated Chromium extension builds from upstream source, published through GitHub Actions. Selected patches change privacy defaults, disable advertising, or modify locally enforced feature flags.

Public source code does not mean Chrome Web Store package was built from that source. This repository makes its source selection, patches, build commands, and release process transparent.

## Workflows

Each extension has a workflow in `.github/workflows/` containing its schedule, upstream-selection commands, and build commands.

The reusable `build-extension.yml` workflow handles:

- Alternating-week schedule gating
- Release existence checks
- Source checkout or ZIP download
- Node.js setup
- Packaging and basic manifest validation
- Artifact transfer and release publication

## Schedule

Scheduled checks run on Mondays around 3 AM UTC.

## Releases

Release tags point to the commit containing this repository's build configuration, not the upstream extension commit.

Existing releases are not rebuilt automatically when patches or workflows change. Rebuilding the same upstream version requires removing its existing release and tag before running the workflow manually.

## Adding an Extension

Add one workflow that calls `.github/workflows/build-extension.yml`.

Required inputs:

- `extension`: lowercase identifier used in release tags and artifact filenames
- `title`: extension name used in release titles
- `resolve`: Bash commands that write source metadata to `$GITHUB_OUTPUT`
- `build`: Bash commands that write an absolute build-output path to `$GITHUB_OUTPUT`

Optional inputs:

- `repository`: upstream GitHub repository
- `biweekly`: skip alternate scheduled weeks
- `node-version`: Node.js version; an empty string disables Node.js setup
- `submodules`: initialize upstream submodules
- `archive-root`: remove the single enclosing directory from a downloaded ZIP
- `tag-prefix`: prefix between the extension identifier and version; defaults to `v`

The `resolve` commands run with `GH_TOKEN` and `UPSTREAM_REPOSITORY` available.

Required resolution outputs:

- `version`: release version, optionally including a commit suffix
- `revision`: full upstream commit SHA for repository checkouts
- `download_url`: HTTPS ZIP URL instead of a repository checkout

A download URL takes precedence over repository checkout. Both `revision` and `download_url` may be supplied to record the commit associated with an archive. `upstream_tag` is an optional output for tagged releases.

The `build` commands run inside the upstream source directory with `VERSION`, `UPSTREAM_TAG`, and `UPSTREAM_REVISION` available.

Build commands must write `path` to `$GITHUB_OUTPUT`. It must be an absolute path to either:

- A directory with `manifest.json` at its root
- A ZIP with `manifest.json` at its root

Patch files belong under `extensions/` and are applied explicitly by the extension's build commands.

Calling workflows must grant `contents: write` so the publishing job can create releases. Resolution and build jobs use read-only repository permissions. Checkout credentials are not persisted.
