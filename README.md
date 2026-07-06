# Funput Scoop Bucket

[Scoop](https://scoop.sh) manifests for [Funput](https://funput.app) on **Windows**.
Ship this directory as its own public GitHub repo named **`Funput/scoop-funput`**.

This is the Windows analogue of [`Funput/homebrew-tap`](https://github.com/Funput/homebrew-tap):
same `funput` umbrella binary, same `funput term` subcommand — just delivered through
Scoop instead of Homebrew (Homebrew has no native Windows install).

## Install

Scoop is **not** bundled with Windows, so install it (and git, needed for third-party
buckets) first — no admin required:

```powershell
# 1) install Scoop if you don't have it
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned   # if scripts are blocked
irm get.scoop.sh | iex
scoop install git

# 2) install Funput Term
scoop bucket add funput https://github.com/Funput/scoop-funput
scoop install funput
```

Then `funput term -- <program>` types Vietnamese (Telex/VNI) inside terminal apps via a
transparent PTY wrapper. Toggle with `Ctrl-\`. No IME, no system permissions.

Update / uninstall:

```powershell
scoop update funput
scoop uninstall funput
```

> **Note:** Windows ConPTY support for `funput term` is still landing (phase TT6). The
> binary installs and runs, but interactive composition may be rough until that ships.

## How releases flow

1. A tag push on `Funput/Funput` runs its `release.yml`, which (for real,
   non-prerelease releases) fires a `repository_dispatch` of type `release` at this
   repo with the version in `client_payload.version`.
   - Requires a `SCOOP_BUCKET_TOKEN` secret on the main repo: a PAT with
     `contents: write` on `Funput/scoop-funput`.
2. [`bump.yml`](.github/workflows/bump.yml) here then points
   [`bucket/funput.json`](bucket/funput.json)'s `version`, download `url`, and `hash` at
   the new release zip (`funput-term-<version>-x86_64-pc-windows-msvc.zip`, built by the
   main repo's `build-windows.yml`) and commits — instant, no community review.

You can also run `bump.yml` manually via **workflow_dispatch**, passing the version, to
re-point the manifest at an existing release.

## Notes

- Scoop installs to a per-user directory (`~/scoop`), so **no admin / UAC** — matching
  funput-term's "no system permissions" design.
- The `checkver` + `autoupdate` block lets Scoop's own updater track new GitHub
  releases too, but the `bump.yml` dispatch above is the primary, instant path.
- Only `64bit` (x86_64) is published; that's the only Windows target the main repo's CI
  builds `funput-cli` for.
