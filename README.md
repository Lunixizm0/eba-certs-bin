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
| `/usr/lib/firefox/distribution/policies.json` | Firefox enterprise policy, imports the certs |
| `/usr/lib/firefox-esr/distribution/policies.json` | Same policy for Firefox ESR |
| `/etc/firefox/policies/policies.json` | Distribution-independent policy path |
| `/etc/skel/.pki/nssdb/cert9.db` | Pre-seeded NSS database for new users |

## How it works

1. `makepkg` downloads the `.deb` and checks its sha256.
2. `prepare()` extracts `data.tar.xz` from the `.deb` (an `ar` archive) with `bsdtar`.
3. `package()` extracts the payload and copies the certs into the Arch trust
   anchor directory. Debian-only docs under `/usr/share/doc` are dropped.

## Arch adaptations

- Debian reads `/usr/local/share/ca-certificates`. Arch does not, so the
  certs are also installed to `trust-source/anchors/`.
- No install scriptlet is shipped. Pacman's `40-update-ca-trust` hook
  (from `ca-certificates-utils`) rebuilds the certificate stores on
  install, upgrade, and remove automatically.
- `firefox-esr` lives under `/usr/lib` on Arch, not `/usr/share` as on
  Debian, so the policy file is installed to both.
- Single runtime dependency: `ca-certificates` (pulls in the whole trust
  chain). Browsers are optional and listed under `optdepends`.
- Files under `/etc` are listed in `backup`, mirroring the `conffiles`
  of the original `.deb`.

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

GPL-3.0