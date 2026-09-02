# GitHub Pages for this fork

This repo is **`TimothyYao/factorio-pack-data`**, a fork of
[`trisiak/factorio-pack-data`](https://github.com/trisiak/factorio-pack-data).
It publishes its own data plane. Regeneration clones the sibling editor
[`TimothyYao/factorio-blueprint-editor-universal`](https://github.com/TimothyYao/factorio-blueprint-editor-universal)
(`FBE_REPO` in [`.github/workflows/deploy.yml`](../.github/workflows/deploy.yml)).

| What                         | URL                                                      |
| ---------------------------- | -------------------------------------------------------- |
| This data plane              | https://timothyyao.github.io/factorio-pack-data/         |
| Parent data plane            | https://trisiak.github.io/factorio-pack-data/            |
| Sibling editor (this line)   | https://timothyyao.github.io/factorio-blueprint-editor-universal/ |
| Parent editor                | https://trisiak.github.io/factorio-blueprint-editor/     |

A pack is served at `https://timothyyao.github.io/factorio-pack-data/<pack-id>/`
(`data.json`, optional `browser/`, `.basis` atlas). GitHub Pages sends
`Access-Control-Allow-Origin: *`, so the editor can fetch this origin
cross-site with no proxy.

## What the workflow does (in-repo)

[`.github/workflows/deploy.yml`](../.github/workflows/deploy.yml) — on push to
`main` (or `workflow_dispatch`), restores or rebuilds each pack's `.basis`
textures, assembles `_site/` from the committed JSON tiers plus those
textures, and publishes with `actions/deploy-pages`. That action only works
when the repo's Pages **source** is **GitHub Actions** (not "Deploy from a
branch").

`workflow_dispatch` input `regenerate` (`none` / `all` / a pack id) runs the
exporter from fbe-universal. That path needs the Factorio secrets below.

## What you must configure in GitHub (cannot be done from code)

Pages source, Actions enablement, the About homepage, and secrets are
**repository settings**. A workflow can *publish* a Pages deployment; it
cannot flip these on a fresh fork. Do them as the repo owner.

### 1. Allow GitHub Actions

Forks often ship with Actions off, so the deploy workflow never registers.

1. Open https://github.com/TimothyYao/factorio-pack-data/settings/actions
2. Under **Actions permissions**, choose **Allow all actions and reusable workflows**
   (or at least allow the actions this workflow pins: `actions/checkout`,
   `actions/cache`, `actions/upload-artifact`, `actions/download-artifact`,
   `actions/configure-pages`, `actions/upload-pages-artifact`,
   `actions/deploy-pages`).
3. Save.

### 2. Point Pages at GitHub Actions

The parent uses **GitHub Actions** as the Pages source. A fork sometimes
inherits **Deploy from a branch** (`main` / `/`), which would serve the raw
repo (JSON only, no atlases) instead of the assembled `_site/`.

1. Open https://github.com/TimothyYao/factorio-pack-data/settings/pages
2. **Build and deployment → Source:** **GitHub Actions**
   (not "Deploy from a branch").
3. Save.

### 3. Run a production deploy

1. Open
   https://github.com/TimothyYao/factorio-pack-data/actions/workflows/deploy.yml
   and run **Deploy data plane** via **Run workflow** (`regenerate: none` is
   enough if you only need bootstrap/cache).
2. Wait until both the `textures` matrix and `deploy` job are green. The
   site is then https://timothyyao.github.io/factorio-pack-data/

A first run with empty cache bootstraps full packs from fbe-universal git
history (no Factorio secrets). Slim packs (`vanilla-2.0-slim`,
`space-exploration-slim`) have no bootstrap; dispatch `regenerate` for those
once the secrets are set.

### 4. Website button on the repo

The About "Website" link is also a setting (this fork still has none, or may
have inherited nothing from the parent).

**UI:** repo home → gear next to **About** → **Website** →
`https://timothyyao.github.io/factorio-pack-data/`

**CLI** (owner machine, not something agents can do with a read-only `gh`):

```bash
gh repo edit TimothyYao/factorio-pack-data \
  --homepage "https://timothyyao.github.io/factorio-pack-data/"
```

### 5. Factorio secrets (regeneration only)

Bootstrap/cache deploys do not need these. `regenerate` does.

1. Open https://github.com/TimothyYao/factorio-pack-data/settings/secrets/actions
2. Add `FACTORIO_USERNAME` and `FACTORIO_TOKEN` (token is on your
   factorio.com profile). Both values are treated as secrets in the workflow
   — never echo them.

## What stays on the parent / original

Do **not** rewrite these as this fork:

- Historical issue numbers cited in fbe-universal docs (`#28`, `#87`, …) —
  those tickets live on `trisiak/factorio-blueprint-editor`
- FIB's design record
  ([`docs/data-plane.md`](https://github.com/trisiak/factorio-item-browser/blob/master/docs/data-plane.md))
- Factorio / mod-asset copyright notices

The sibling editor still has to *opt in* to this origin. Until
`VITE_DATA_URL` in fbe-universal points here, production builds of
https://timothyyao.github.io/factorio-blueprint-editor-universal/ keep
fetching the parent data plane.
