# Contributing to dotkeeper

Contributions are welcome. Bug reports, feature requests, and pull requests all help.

## Individual Contributor License Agreement

Contributions remain available under AGPL-3.0. Before submitting a pull request, read and agree to
version 1.0 of the organization-wide [Individual Contributor License Agreement][icla]. Do not
contribute material unless you have the right to grant its terms.

> I have read and agree to version 1.0 of the Individual Contributor License Agreement at https://github.com/corbet-labs/.github/blob/cla-v1.0/CLA.md.

[icla]: https://github.com/corbet-labs/.github/blob/cla-v1.0/CLA.md

## Development

### Building

```bash
make build
```

Requires Go ≥ 1.26 and `git`. The `-tags noassets` flag is handled by the Makefile.

### Testing

```bash
make test
```

Go's built-in test runner. The suite covers unit, integration, and end-to-end flows plus fuzz targets under `internal/`.

### Linting

```bash
go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.11.4
golangci-lint run --build-tags noassets ./...
```

The CI workflow gates PRs on this; locally it's optional but recommended before pushing.

### Running the binary locally

```bash
make install    # installs to ~/.local/bin/dotkeeper
dotkeeper status
```

## Pull request workflow

**All non-trivial changes go through a pull request.** `main` is protected, and project policy
requires the pull-request path for maintainers as well as external contributors.

1. **Branch.** `git checkout -b <topic>/<short-name>` (e.g. `fix/timer-race`, `docs/setup-guide`).
2. **Commit.** Small, focused commits with clear messages. No tool-generated attribution lines.
3. **Open a PR.** `gh pr create` or the GitHub UI.
4. **Wait for CI.** Required checks: `build`, `lint`, `multipeer-e2e`, `fuzz-smoke`, and
   `Analyze (go)`. All must pass.
5. **Merge.** Squash-merge is the convention (keeps history linear). Delete the branch after merge.

### PR size

Prefer small, reviewable PRs over sweeping changes. If a feature naturally splits into independent pieces, split the PRs. Aim for under ~300 lines of diff where reasonable.

### When tests are required

- **New feature** → must include tests exercising the new behaviour. If the feature is inherently hard to test in-process (e.g. integration with systemd), call that out in the PR description.
- **Bug fix** → must include a regression test that fails on the parent commit and passes on the fix.
- **Refactor** → existing tests should continue to pass; add tests only if the refactor uncovers behaviour that wasn't previously covered.
- **Docs / CI / packaging** → no tests required.

### Commit message conventions

dotkeeper follows [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
for the subject line and the [Tim Pope rules](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
for the body.

#### Subject

```
<type>(<scope>): <imperative summary>
```

- **Types:** `feat`, `fix`, `docs`, `refactor`, `test`, `ci`, `build`, `chore`, `deps`, `perf`, `style`.
- **Scope:** optional, lowercase, single word matching a package or area (`gitsync`, `reconcile`, `doctor`, `cmd`).
- **Summary:** imperative mood (`add`, not `added`/`adds`); no trailing period; ≤ 72 characters total including the prefix.
- **Breaking changes:** add `!` after the type/scope (`feat(cmd)!: ...`) and explain in a `BREAKING CHANGE:` footer.

#### Body

- Blank line between subject and body. Wrap body lines at 72 characters.
- Explain **why** the change is being made and the user-visible or operational impact. Do not narrate the diff — `git show` already shows it.
- Use third-person, neutral voice. No first-person (`I manually`, `we should`, `let's`), no anecdotes, no speculation about future work.
- Do not narrate alternatives that were considered and rejected. That belongs in an ADR or in inline comments next to the surprising choice, not in version history.
- Do not reference the PR review process (`addresses three nits from review`, `per @reviewer`). The commit stands alone after merge.
- For squash-merges: rewrite the body as a single coherent message. Do not ship a list of `* feat: ...` / `* fix: ...` bullets that mirror the original PR commit history.

#### Footer

- Reference issues and PRs at the end: `Closes #42`, `Refs #17`.
- No `Co-Authored-By:` lines from coding assistants. Tool-generated attribution is removed before merge.

#### Examples

Good:

```
fix(stengine): disable QUIC listener by default

The QUIC listener pulls in two reachable advisories
(GO-2025-4017, GO-2025-4233) and a v0.52.0 startup panic that triggers
a systemd restart loop on some peer-state combinations. With QUIC off,
both CVE paths become unreachable on a default install and Syncthing
falls back to TCP, which is unaffected.

The default can be re-enabled per machine in machine.toml.
```

Bad:

```
fix(stengine): disable QUIC listener by default

I manually disabled QUIC after observing 49 restarts in a couple of
minutes. We can revisit when Syncthing bumps past quic-go v0.54.1.
No need for the operator workaround anymore.
```

The "bad" version leaks first-person narrative, speculates about a future bump, and references operator history rather than the resulting behaviour.

## Code style

- Standard Go conventions (`gofmt`, `go vet`). CI enforces both.
- `golangci-lint` baseline is **zero findings**. If you add code that flags a new finding, either fix it or justify in the PR.
- Prefer clarity over cleverness. dotkeeper is maintained by few people — code should read easily on a fresh mind six months later.
- Comments explain *why*, not *what*. Don't describe what the code does if the code is clear; do describe non-obvious invariants, platform caveats, or upstream bugs you're working around.
- No copyright headers in new files beyond the standard `// Copyright (C) 2026 Julian Corbet\n// SPDX-License-Identifier: AGPL-3.0-only` preamble.

## Reporting issues

### Bug reports

Open a GitHub issue at <https://github.com/corbet-labs/dotkeeper/issues>. Include:

- What you expected to happen
- What actually happened
- Your OS and Go version (`go version`)
- dotkeeper version (`dotkeeper version`)
- Relevant output from `dotkeeper status`
- Output from `dotkeeper doctor` (or `dotkeeper doctor --json`) — a single command that captures version, config, service, Syncthing API, peers, folders, git remotes, timer, and conflict state in one paste-friendly block
- A minimal reproduction if possible

### Security issues

**Do not open public issues for security vulnerabilities.** See [SECURITY.md](SECURITY.md) for the private disclosure channel and response SLA.

### Feature requests

Open a GitHub issue describing the use case, the proposed behaviour, and any alternatives you considered. Feature requests that match the project's scope get triaged; ones outside scope get politely declined with rationale.

## Review expectations

- **Reviewer turnaround:** best-effort within a few business days for most PRs. Security fixes get priority.
- **CI must be green** before merge. No exceptions.
- **Breaking changes** to the CLI, config file format, or on-disk layout require a major version bump and a migration path documented in the release notes.
- **Large PRs** (>500 lines) are likely to be sent back for splitting. Splitting is healthy — it gives each change its own review pass.

## Release process

dotkeeper follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Releases are cut by
pushing an annotated git tag matching `v*` to `main`; the workflows under [`.github/workflows`](.github/workflows)
then build and publish the release and distribution updates.

### Pre-release checklist

1. **CI green on `main`.** Every workflow in the required-checks set (`build`, `lint`, `multipeer-e2e`, `fuzz-smoke`, `Analyze (go)`) must be passing for the commit you're about to tag.
2. **CHANGELOG entry written.** Add a `## [X.Y.Z] - YYYY-MM-DD` section under `## [Unreleased]` summarising what changed. Keep [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) section ordering: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.
3. **SECURITY.md supported-versions table** updated if a major or minor version is dropping out of support.
4. **`gofmt` and `go vet`** clean (the CI gate enforces this, but local check catches it sooner).

### Cutting the tag

```bash
git checkout main
git pull
git tag -a vX.Y.Z -m "vX.Y.Z — short summary"
git push origin vX.Y.Z
```

The tag must be annotated (`-a`), not lightweight: the release workflow consumes the tag's commit metadata. Push the tag itself, not a `--all` push that bundles other refs.

### What the release automation does

A tag push to `v*` starts the release, AUR, and Homebrew workflows. Publishing the GitHub release
then starts the Docker and native Alpine workflows:

- **`Release`** builds archives for Linux, macOS, Windows, FreeBSD, and OpenBSD; builds deb, rpm,
  apk, and Arch Linux packages for the supported Linux architectures; generates SHA-256 checksums
  and signed Sigstore provenance for every artifact; and creates the GitHub release with generated
  notes. The versioned changelog remains in the tagged source rather than being copied into the
  generated release notes.
- **`Publish to AUR`** refreshes both `dotkeeper-bin` and `dotkeeper-git`.
- **`Publish to Homebrew tap`** updates the formula in `corbet-labs/homebrew-dotkeeper`.
- **`Docker`** builds and pushes the multi-architecture
  `ghcr.io/corbet-labs/dotkeeper:vX.Y.Z` and `:latest` images, then attaches signed provenance to
  the pushed manifest.
- **`Alpine`** builds native x86_64 and aarch64 packages with `abuild` and attaches them to the
  GitHub release.

If any of these fail post-tag, fix forward (cut a follow-up patch release) rather than retag — published artifacts on AUR / Homebrew / GHCR persist independent of the GitHub release.

### After release

Verify:

- `gh release list --limit 1` shows the new tag as `Latest`.
- The release contains the expected archives, native packages, aggregate `checksums_sha256.txt`,
  `dotkeeper_vX.Y.Z.sigstore.json` provenance bundle, and a `.sha256` sidecar for each native
  Alpine APK (those APKs are attached after the aggregate manifest is created).
- `gh attestation verify <downloaded-artifact> --repo corbet-labs/dotkeeper` verifies at least
  one archive and one nFPM package against the release workflow's signed record.
- AUR (`https://aur.archlinux.org/packages/dotkeeper-bin`) reflects the new `pkgver`.
- The Homebrew formula in `corbet-labs/homebrew-dotkeeper` reflects the new version.
- The Docker and native Alpine workflows completed successfully; the GHCR manifest has a verifiable
  OCI provenance bundle; and the image can be pulled anonymously (package visibility is configured
  separately from repository visibility).
- The `Latest` badge in the README now points at the new tag.

## Governance

dotkeeper is owned by the [Corbet Labs Maintainers team](https://github.com/orgs/corbet-labs/teams/maintainers).
The project lead has final responsibility for scope, direction, and releases. Changes to this
decision model will be announced in the repository.
