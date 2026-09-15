# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- Reusable `claude.yml` (full Claude Code Action job — mention filter, permissions,
  git-policies fetch) and `ps-lint.yml` (PowerShell parse-check + PSScriptAnalyzer,
  Error-severity fail only, `paths` input) workflows; adopted fleet-wide across ~25 repos
  ([#4](https://github.com/J-MaFf/.github/pull/4))
- `STATUS.md` and `CHANGELOG.md` ([#6](https://github.com/J-MaFf/.github/pull/6))

### Fixed
- Discord notify: use `head_ref` instead of the PR merge ref so pull_request-triggered
  runs show the actual source branch ([#2](https://github.com/J-MaFf/.github/pull/2))
- `release.yml`: the "Create GitHub Release" step now checks whether a release for the
  tag already exists and updates it (`gh release edit` + `gh release upload --clobber`)
  instead of unconditionally calling `gh release create`, which failed with "a release
  with the same tag name already exists" whenever a `vX.Y.Z` tag was re-pushed to a new
  commit after its release was first published
  ([#14](https://github.com/J-MaFf/.github/pull/14))

## [1.0.0] — 2026-08-09
### Added
- Reusable `discord-notify.yml` workflow posting CI failures to Discord `#ci`
  ([`8120200`](https://github.com/J-MaFf/.github/commit/8120200))
