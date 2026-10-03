# Datavalv releases

Every version of the Datavalv web application, recorded here **before** it is
served. Nothing else is in this repository: no source, only what is needed to
check that the program your browser received is a published one.

Datavalv encrypts and decrypts in your browser, with code we deliver every
time you open the page. That makes the delivered code the one thing you have
to trust us about, and [datavalv.se/security](https://datavalv.se/en/security)
lists it as a limit. This repository narrows it: a version we served to
everybody without publishing it here first could be noticed by anybody.

## What is here

```
prod/
  LIVE                                   the release datavalv.se serves now
  2026-10-03-<commit>/
    manifest-prod-<commit>.sha256        SHA-256 of every file served
    manifest-prod-<commit>.txt           what built it: commit, Node, platform
    manifest-prod-<commit>.sha256.sigstore.json
                                         the signature, and its log entry
staging/
  ...the same, for staging.datavalv.se
```

A release folder is written by the deploy workflow before the site is
updated, and `LIVE` is moved only after the update succeeds. A folder `LIVE`
has never pointed at was published and then not served, because that deploy
failed.

## Checking a release

**1. The manifest is ours, and was logged when we say.** Each manifest is
signed with [Sigstore](https://www.sigstore.dev) by the deploy workflow
itself, and the signature is recorded in Sigstore's public transparency log,
Rekor, which nobody -- us included -- can edit or delete. With
[cosign](https://docs.sigstore.dev/cosign/system_config/installation/):

```bash
release=prod/$(cat prod/LIVE)
manifest=$(ls "$release"/*.sha256)
cosign verify-blob "$manifest" \
  --bundle "$manifest.sigstore.json" \
  --certificate-identity "https://github.com/Hypercode-SE/datavalv/.github/workflows/deploy.yml@refs/heads/production" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

For staging, use `staging/` and `refs/heads/staging`.

**2. The site serves exactly those files.** Fetch every file the manifest
names and compare. Nothing here needs any tool of ours:

```bash
mkdir live && cd live
while read -r _ path; do
  mkdir -p "$(dirname "$path")"
  curl -sf "https://datavalv.se/$path" -o "$path"
done < "../$manifest"
sha256sum --strict -c "../$manifest"
```

## What this proves, and what it does not

- It proves the files you checked are the ones published here, and that the
  record was signed by our deploy workflow and logged at the time it says.
- It shows any change we made **for everybody**: a version served without a
  folder here, or a folder rewritten after the fact, would not match.
- It does **not** show what source code the files were built from. The
  application's source is not public. We check internally that every commit
  builds byte for byte the same twice; you cannot repeat that check.
- It **cannot** reveal a version served to you alone. Only checking what
  your own browser received, at the time, could catch that.
