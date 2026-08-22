<!-- Release notes template for dotkeeper.
     Copy the relevant section from CHANGELOG.md and fill in the blanks.
     Delete sections that have no entries for this release. -->

## What's changed

### Added

-

### Changed

-

### Removed

-

### Fixed

-

### Security

-

## Install

**Arch Linux (AUR):**
```bash
paru -S dotkeeper-bin
```

**macOS / Linux (Homebrew):**
```bash
brew tap corbet-labs/dotkeeper
brew install dotkeeper
```

**From source / binary:** download the archive for your platform below, or:
```bash
go install -tags noassets github.com/corbet-labs/dotkeeper/cmd/dotkeeper@v0.0.0
```
(replace `v0.0.0` with this release tag)

## Checksums

`checksums_sha256.txt` is attached to this release. Verify with:
```bash
sha256sum -c checksums_sha256.txt
```

## Provenance

The release also includes a `dotkeeper_vX.Y.Z.sigstore.json` bundle covering every artifact in
the checksum manifest. Verify a downloaded artifact against GitHub's signed record with:

```bash
gh attestation verify dotkeeper_X.Y.Z_linux_amd64.tar.gz \
  --repo corbet-labs/dotkeeper \
  --signer-workflow corbet-labs/dotkeeper/.github/workflows/release.yml
```
