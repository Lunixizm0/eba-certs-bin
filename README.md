# eba-certs-bin

AUR package that installs the MEB Fatih root CA certificates on Arch Linux.
The certificates are repackaged from the official Pardus `.deb`:

```
https://depo.pardus.org.tr/pardus/pool/contrib/e/eba-certs/eba-certs_1.0.2_amd64.deb
```

## What it installs

| Path | Purpose |
| ---- | ------- |
| `/usr/share/ca-certificates/trust-source/anchors/MEB1.crt` | `fatihca` root CA, system trust store |
| `/usr/share/ca-certificates/trust-source/anchors/MEB2.crt` | `meb-ROOTCA-CA` root CA, system trust store |
| `/usr/local/share/ca-certificates/MEB1.crt`, `MEB2.crt` | Original Debian paths (kept, `policies.json` points here) |
| `/usr/lib/firefox/distribution/policies.json` | Symlink `/etc/firefox/policies/policies.json` |
| `/usr/lib/firefox-esr/distribution/policies.json` | Symlink `/etc/firefox/policies/policies.json` |
| `/etc/firefox/policies/policies.json` | Canonical Firefox enterprise policy (backupd edit this one) |
| `/etc/skel/.pki/nssdb/cert9.db` | Pre-seeded NSS database for new users (see note below) |
| `/usr/share/licenses/eba-certs-bin/LICENSE` | Package license |

## How it works

1. `makepkg` downloads the `.deb` and checks its sha256.
2. `prepare()` extracts `data.tar.xz` from the `.deb` (an `ar` archive) with `bsdtar`.
3. `package()` extracts the payload and copies the certs into the Arch trust
   anchor directory. Debian-only docs under `/usr/share/doc` are dropped,
   the Debian-only `/usr/share/firefox-esr` path is removed, and the license
   is installed to `/usr/share/licenses/eba-certs-bin/`.

## Arch adaptations

- Debian reads `/usr/local/share/ca-certificates`. Arch does not, so the
  certs are also installed to `trust-source/anchors/`.
- No install scriptlet is shipped. Pacman's `40-update-ca-trust` hook
  (from `ca-certificates-utils`) rebuilds the certificate stores on
  install, upgrade, and remove automatically.
- `firefox-esr` lives under `/usr/lib` on Arch, not `/usr/share` as on
  Debian, so the Debian path is deleted and the policy is installed to
  `/usr/lib/firefox-esr/`. The single canonical policy file is
  `/etc/firefox/policies/policies.json` both `/usr/lib/...` paths are
  symlinks to it so they can never diverge.
- Single runtime dependency: `ca-certificates` (pulls in the whole trust
  chain). Browsers are optional and listed under `optdepends`.
- Files under `/etc` are listed in `backup`. Upstream `conffiles` only
  tracks the `cert9.db` the `/etc/firefox/...` policy is an Arch-specific
  addition and is also backup'd as the editable canonical copy.

## Note on `cert9.db`: new users vs existing users

`/etc/skel/.pki/nssdb/cert9.db` is only copied for users created *after*
the package is installed. Existing Firefox users are covered through the Certificates.
Install enterprise policy, which imports the bundled CA certificates into Firefox. 
ImportEnterpriseRoots is not relied upon on Linux. And
Chromium/Chrome coverage via the system trust store - no manual import
needed. The db ships intentionally without `key4.db`/`pkcs11.txt`;
Firefox/NSS recreates the missing sidecar files on first run.

## Build and install

This repository is the package. There is no separate AUR repo. Clone it
and build with `makepkg`:

```bash
git clone https://github.com/Lunixizm0/eba-certs-bin eba-certs-bin
cd eba-certs-bin
makepkg -sif
```

(`-s` installs missing dependencies, `-i` installs the built package,
`-f` overwrites a previous build.)

Verify the anchors after install:

```bash
trust list | grep -i -E "fatihca|meb-ROOTCA"
```

## License

GPL-3.0-or-later