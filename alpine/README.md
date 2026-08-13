# Alpine Linux packaging

dotkeeper ships a native Alpine package built with `abuild`. It is separate
from the generic `.apk` produced by nFPM: the native package carries Alpine's
own package metadata and is the authoritative artifact for Alpine users.

This directory contains the reusable packaging source:

| File | Purpose |
|------|---------|
| `APKBUILD` | Builds dotkeeper from a tagged source archive with the Alpine toolchain. |
| `README.md` | Explains the release and upstream-maintenance paths. |

## Release artifacts

Publishing a GitHub release starts `.github/workflows/alpine.yml`. The workflow
builds native x86_64 and aarch64 packages in `alpine:edge`, then attaches them
to the existing release. It can also be replayed manually for an existing tag.

The APKBUILD declares `arch="all"` because dotkeeper is a pure-Go binary, and
creates a conventional `dotkeeper-doc` subpackage. GitHub releases contain the
main package only, named `dotkeeper_vX.Y.Z_alpine_<arch>.apk`, plus a matching
SHA-256 sidecar.

Verify the package before installing it:

```sh
sha256sum -c dotkeeper_vX.Y.Z_alpine_x86_64.apk.sha256
apk add --allow-untrusted dotkeeper_vX.Y.Z_alpine_x86_64.apk
```

`--allow-untrusted` is required because each CI run uses a disposable signing
key that is not installed in Alpine's trusted keyring. Packages from Alpine's
official repositories use Alpine's signing infrastructure and do not need it.

## Maintaining the in-repo APKBUILD

`alpine/APKBUILD` deliberately carries `pkgver=0.0.0` and an empty checksum.
The release workflow replaces the version and runs `abuild checksum` for the
selected tag. Do not commit a release version here; doing so would create an
unnecessary second version-bump surface.

Change the template when build dependencies, build flags, installed files, or
subpackage boundaries change. The next release uses the updated template.

## Submitting to Alpine aports

New packages normally enter Alpine through `testing/`. Follow Alpine's current
[package-creation guide](https://wiki.alpinelinux.org/wiki/Creating_an_Alpine_package)
for account, fork, build-environment, and merge-request setup; Alpine's guide
and aports policy are authoritative if this summary ever differs.

For the first submission:

```sh
cd <aports-fork>
git switch -c dotkeeper-X.Y.Z origin/master
mkdir -p testing/dotkeeper
cp <dotkeeper-repo>/alpine/APKBUILD testing/dotkeeper/APKBUILD
sed -i 's/^pkgver=0\.0\.0/pkgver=X.Y.Z/' testing/dotkeeper/APKBUILD
cd testing/dotkeeper
abuild checksum
apkbuild-lint APKBUILD
abuild -r
git add APKBUILD
git commit -m 'testing/dotkeeper: new aport'
git push origin dotkeeper-X.Y.Z
```

Open a merge request to `alpine/aports` after the lint and build complete
without warnings. Once accepted, the copy in aports is the release-specific
package definition; future version bumps happen there. Keep this in-repo
template aligned when dotkeeper's build or installed payload changes.
