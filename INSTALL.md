# Install Pushkin

Use a prebuilt binary from a package registry. The intended platform matrix is
macOS on Apple Silicon or Intel, and Linux on ARM64 or x86-64 with glibc or musl.
Windows is not currently in that matrix. Availability depends on the selected
package version; an unsupported platform should fail instead of building from source.

## Python package

With uv installed:

```sh
uv tool install --no-build pushkin-cli
pushkin --version
```

The `--no-build` flag requires a compatible wheel. An alternative in a suitable
Python environment is `pip install --only-binary=:all: pushkin-cli`.

## npm package

For a project-local installation with Node.js 18 or newer:

```sh
npm i -D @mindmonkey/pushkin
npx --no-install pushkin --version
```

The npm launcher selects a platform-specific optional dependency. Keep optional
dependencies enabled. It does not compile Pushkin from source.

## Check the installation

For the uv installation, run:

```sh
pushkin --help
pushkin doctor
```

For the project-local npm installation, prefix these commands with
`npx --no-install`. Use the native executable for latency-sensitive hooks where
possible. Consult `pushkin --help` before configuring repository enforcement;
installation alone does not establish your project's contracts.

If installation fails, collect the version, platform, and error details described
in [support](SUPPORT.md). Do not substitute an untrusted binary or a source-build
fallback. See [release status](CHANGELOG.md) and [verification status](THIRD_PARTY_NOTICES.md)
before treating any package as an audited release.

## Verify a release

Each released version `X.Y.Z` has a GitHub Release `vX.Y.Z` on this repository with
exactly three assets: `SHA256SUMS` (the manifest of the thirteen archives the owner
signed), `SHA256SUMS.sig` (the 64 raw Ed25519 signature bytes over that manifest) and
`pushkin-release-signing.pub.pem` (the public key the release was verified against).
With OpenSSL 3, or with Node as shown below, you can prove that an archive you hold is
one the owner signed, using nothing from any source repository.

The system `/usr/bin/openssl` on macOS is LibreSSL and cannot verify a raw Ed25519
signature (it fails with `unable to load Public Key`); use OpenSSL 3 (for example
`brew install openssl`, then `/opt/homebrew/bin/openssl`) or the Node form.

```sh
VERSION=X.Y.Z
curl -fsSLO https://github.com/mindmonkey-ai/pushkin-public/releases/download/v$VERSION/SHA256SUMS
curl -fsSLO https://github.com/mindmonkey-ai/pushkin-public/releases/download/v$VERSION/SHA256SUMS.sig
curl -fsSLO https://github.com/mindmonkey-ai/pushkin-public/releases/download/v$VERSION/pushkin-release-signing.pub.pem

# 1. The key is the pinned one: the digest must be a fingerprint in the table below.
openssl pkey -pubin -in pushkin-release-signing.pub.pem -outform DER | shasum -a 256

# 2. The manifest is the one the owner signed.
openssl pkeyutl -verify -pubin -inkey pushkin-release-signing.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
# prints: Signature Verified Successfully

# 3. The archive you hold is in the signed manifest. One manifest line is printed per
#    archive; an archive with no line is not one the owner signed.
pip download --no-deps --only-binary=:all: --dest archives "pushkin-cli==$VERSION"
for f in archives/*; do grep -F "$(shasum -a 256 "$f" | cut -d' ' -f1)  $(basename "$f")" SHA256SUMS; done
```

An npm tarball is checked the same way: `npm pack @mindmonkey/pushkin-<platform>@$VERSION`
writes it under the name the manifest lists (for example
`mindmonkey-pushkin-darwin-arm64-X.Y.Z.tgz`), and the `grep -F` line above applies to it.

Without OpenSSL 3, Node (18 or newer) computes the same fingerprint and verdict:

```sh
node -e '
const { createHash, createPublicKey, verify } = require("node:crypto");
const { readFileSync } = require("node:fs");
const key = createPublicKey(readFileSync("pushkin-release-signing.pub.pem"));
console.log(createHash("sha256").update(key.export({ type: "spki", format: "der" })).digest("hex"));
console.log(verify(null, readFileSync("SHA256SUMS"), key, readFileSync("SHA256SUMS.sig")) ? "Signature Verified Successfully" : "signature mismatch");
'
```

### Release signing keys

This page is the pin. A release's key must digest to a fingerprint in this table, and the
table changes only with this page: a rotation adds a row beside the old one with the dates
each key was in force, and releases already published stay verifiable against their retired
key; a revocation marks the row retired with the reason and is announced in
[SECURITY.md](SECURITY.md) and [CHANGELOG.md](CHANGELOG.md).

| Fingerprint (SHA-256 of the SPKI DER) | In force from | Retired |
|---|---|---|
| `e1f4a72845e0efd4c6d8aa6847a54513b02087292ff9f85e2984d0bc8c236713` | 2026-10-19 | current |

Public key `e1f4a728…`:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAmugk90Dt8/TEwm7MdWv/sAGjbK6R/j040DVns7qdncg=
-----END PUBLIC KEY-----
```
