# Security

## Report a vulnerability

Email support@rosstech.dev with the subject `hauntd security`. Please don't open a public issue. Include what you found, the build, and what an attacker gains. Don't send real credentials or other people's data.

## What the signing covers

The hauntd app has the release key and the macOS code-signing requirement built in. A host installs a binary only if its manifest verifies against that key, the size and SHA-256 match, and (on macOS) the Developer ID requirement holds. A compromised GitHub account, DNS, or network can deny service or serve an older signed, unexpired build, but can't get a host to install an unsigned one.

A change to the "verify by hand" key or commands in this repo's README does not change what the app trusts. If the key in the README ever differs from the one in the app, report it.
