# hauntd-releases

Signed builds of the hauntd helper, the small program the hauntd app installs on hosts you own (Mac or Linux). The source is not public, and this repo holds only release binaries. Issues are closed. Questions go to support@rosstech.dev.

## Install

You don't need to do anything here. The hauntd app installs and updates the helper over your SSH session and checks everything below on the host before it keeps the binary. It installs to `~/.local/bin/hauntd` and needs no sudo.

Host requirements: `curl`, OpenSSH 8.1 or newer (`ssh-keygen -Y`), `shasum` or `sha256sum`, `mktemp`.

## Platforms

- `macos-arm64`
- `macos-x86_64`
- `linux-arm64`
- `linux-x86_64`

Each release `v<version>` has one `<os>-<arch>.release` file per platform, the binaries (`hauntd-<version>-<os>-<arch>`), and `SHA256SUMS`. Re-signed releases (`v<version>-r<n>`) hold only new `.release` files; their binaries stay in the original release.

## How verification works

A `<os>-<arch>.release` file is a manifest followed by its SSH signature (`-----BEGIN SSH SIGNATURE-----` through `-----END SSH SIGNATURE-----`). The installer:

1. Downloads `<os>-<arch>.release` from the latest release.
2. Splits it into manifest and signature, and rejects anything malformed.
3. Verifies the signature with `ssh-keygen -Y verify` against the release key below, namespace `hauntd-release`, before it reads a single manifest field.
4. Checks the manifest: product, platform, expiry (the manifest is only good until its `expires` time), minimum build, and no downgrade.
5. Downloads the binary at the manifest's `url` and checks its size and SHA-256 against the manifest.
6. On macOS, checks the code signature against the Developer ID requirement below.
7. Runs `--version`, then moves the binary into place.

The key is built into the app, not fetched from here. HTTPS and GitHub are transport only; a swapped file fails the signature or the hash.

Release key (ed25519), principal and namespace `hauntd-release`:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHuynZtDVlSkrkyZsvsgx8f4sFK4HsyB5KwGlnxNLVT4
```

## Verify by hand

Example for `macos-arm64`; swap the platform name as needed.

```sh
P=macos-arm64
curl -fLO https://github.com/wickdninja/hauntd-releases/releases/latest/download/$P.release

# split the bundle into manifest and signature
sed '/^-----BEGIN SSH SIGNATURE-----$/,$d' $P.release > $P.manifest
sed -n '/^-----BEGIN SSH SIGNATURE-----$/,$p' $P.release > $P.sig

# 1. signature, with the key above
echo 'hauntd-release ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHuynZtDVlSkrkyZsvsgx8f4sFK4HsyB5KwGlnxNLVT4' > allowed_signers
ssh-keygen -Y verify -f allowed_signers -I hauntd-release -n hauntd-release -s $P.sig < $P.manifest

# 2. read the manifest, then fetch the binary it names
cat $P.manifest
curl -fL -o hauntd "$(sed -n 's/^url=//p' $P.manifest)"

# 3. size and SHA-256 must match the manifest
wc -c < hauntd
shasum -a 256 hauntd        # sha256sum on Linux
sed -n -e 's/^size=//p' -e 's/^sha256=//p' $P.manifest
```

`ssh-keygen` should print `Good "hauntd-release" signature ...`. Also check `expires` (unix seconds) is in the future.

On macOS, check the Developer ID signature too:

```sh
chmod 755 hauntd
codesign --verify --strict -R='identifier "dev.rosstech.hauntd" and anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] and certificate leaf[field.1.2.840.113635.100.6.1.13] and certificate leaf[subject.OU] = "6N8ZPD4W4W"' hauntd
```

No output and exit 0 means it passed.

## Extra checks from GitHub

These are not a substitute for the signature, only more evidence.

- Each release asset has a SHA-256 `digest` in the REST API: `gh api repos/wickdninja/hauntd-releases/releases/latest --jq '.assets[] | {name, digest}'`
- Releases here are immutable: once published, assets can't be changed or deleted and the tag is locked. Publishing creates an attestation for the release.

## License

All rights reserved. These binaries are for use with the hauntd app.
