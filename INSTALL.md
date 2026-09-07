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
