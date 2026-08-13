# Working on dotkeeper

This is the quick reference for coding agents and contributors. Keep it short;
[`CONTRIBUTING.md`](CONTRIBUTING.md) is the authoritative workflow and policy
document.

## Ground truth

- Read [`README.md`](README.md) for the supported user experience and command
  surface.
- Read [`docs/architecture.md`](docs/architecture.md) and the
  [ADR index](docs/adr/README.md) before changing state ownership, discovery,
  reconciliation, or transport behavior.
- Verify documentation against the implementation and tests. Do not preserve a
  stale claim merely because another document repeats it.
- Never add private machine names, repository topology, credentials, or local
  paths to this public repository.

## Build and validation

Use the Go version declared in [`go.mod`](go.mod). Build and test through the
Makefile:

```sh
make build
make test
```

dotkeeper embeds Syncthing without its web assets. Direct Go commands must use
the `noassets` build tag:

```sh
go build -tags noassets ./cmd/dotkeeper
go test -tags noassets ./...
go vet -tags noassets ./...
golangci-lint run --build-tags noassets ./...
```

Required pull-request checks are `build`, `lint`, `multipeer-e2e`,
`fuzz-smoke`, and `Analyze (go)`. Let GitHub Actions run the container-backed
multipeer suite and the repository's pinned tool versions.

## Repository map

- `cmd/dotkeeper/` — CLI entry point and command wiring.
- `internal/config/` — `machine.toml`, `state.toml`, and per-repository config.
- `internal/reconcile/` — pure diff, action model, and idempotent application.
- `internal/stengine/` and `internal/stclient/` — embedded Syncthing lifecycle
  and API access.
- `internal/gitsync/`, `internal/subscribe/`, and `internal/transport/` — git
  history, folder subscriptions, and transport selection.
- `tests/multipeer/` — container-backed peer-to-peer behavior tests.
- `docs/` — architecture, ADRs, and portable examples.
- `alpine/`, `nfpm.yml`, and `.github/workflows/` — packaging and release paths.
- `site/` — static project site served by Cloudflare.

## Change discipline

- Work through a pull request; keep changes focused and reviewable.
- Follow the commit and test rules in `CONTRIBUTING.md`. Do not add
  tool-generated attribution or coding-assistant credit lines.
- New behavior needs tests. A bug fix needs a regression test. Refactors must
  preserve the existing suite.
- CLI, config, or on-disk breaking changes require a migration path and the
  appropriate major-version decision.
- Update `CHANGELOG.md` for user-visible, security, packaging, or release
  changes. Do not retag a published release; fix forward.
- Treat documentation as maintained product code: prefer one canonical source,
  check commands and links against the current tree, and remove duplicated
  explanations that can drift.

Security reports use the private channel in [`SECURITY.md`](SECURITY.md).
Contributions are licensed under AGPL-3.0 and the CLA in `CONTRIBUTING.md`.
