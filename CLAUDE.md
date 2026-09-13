# CLAUDE.md — xiangao.github.io (personal Quarto site)

Personal website. Quarto website project, rendered to `docs/`.

## Deployment

GitHub Pages serves the **committed `docs/` directory** from the default branch.
There is no `.github/workflows/` and no CI render. Nothing reaches the live site
until `docs/` is rendered locally and committed. Always render before committing.

## Build

```bash
quarto render                      # whole site -> docs/
quarto render posts/<slug>/index.qmd   # one post, forces re-execution
```

Never pass several `.qmd` files to a single `quarto render` — Quarto merges them
into the first file.

## `freeze: true` — read this before editing any post

`posts/_metadata.yml` sets `freeze: true`. The freeze cache at
`_freeze/posts/<slug>/index/execute-results/html.json` stores the post's
**entire rendered markdown — prose, front matter and headings, not just code
output**.

Consequence: editing a post's `.qmd` and running a project-level `quarto render`
changes **nothing**. The stored markdown is restored wholesale and your edit is
silently discarded. It looks like a successful render.

To actually publish a change to a post, render that file explicitly:

```bash
quarto render posts/<slug>/index.qmd
```

That forces re-execution — see the drift warnings below before doing it.

## Re-executing posts causes output drift

Re-execution runs today's package versions, and output has already degraded once:

- **`lmtp`** — the current `lmtp` print method emits its header lines
  (`LMTP Estimator`, `Trt. Policy`, `95% Conf. int.`) through `cli`, which writes
  to stderr and is dropped by the chunks' `message=FALSE`. The remaining stdout
  block also lands indented by three spaces, so Quarto's
  `::: {.cell-output ...}` wrapper stops parsing and the literal `:::` shows on
  the page. Point estimates are unchanged.
- **`numpyro`** — see below.
- **`uplift`** — needs `causalml`; the pinned version string printed on the page
  comes from whatever is installed (`0.15.5` originally, `0.17.0` now).

Where re-execution would regress the page, the fix used was: restore the
committed freeze (`git checkout HEAD -- _freeze/posts/<slug>`) and patch the
prose directly in the stored `result.markdown`. Prose-only edits are exactly
what freeze is meant to preserve, so this is consistent, not a hack.

## `numpyro` source is out of sync with the published page

`posts/numpyro/index.qmd` contains only the two R chunks and stops after the
`lme4` model. The **published page additionally contains a whole
`## hierarchical model in numpyro` section with three Python chunks** (`MCMC`,
`NUTS`, ~34,000 characters). That content exists only in the freeze cache and in
`docs/`, never in the source.

Re-rendering `posts/numpyro/index.qmd` therefore deletes that section from the
live site. Do not render it explicitly until the source is repaired. Repair
needs the section restored into the `.qmd` plus `numpyro` and `jax` installed.

## Python

Only `posts/uplift/index.qmd` uses Python. It needs `causalml` and `duecredit`
(without `duecredit`, an import warning prints onto the page).

```bash
source .venv/bin/activate            # .venv is gitignored
QUARTO_PYTHON=$PWD/.venv/bin/python quarto render posts/uplift/index.qmd
```

## knitr caches

`posts/*/index_cache/` are knitr chunk caches — **distinct from `_freeze/`**,
which stores rendered output and IS tracked. The caches are gitignored but must
stay on disk: they keep re-executed chunk results stable and fast. Do not delete
them.

## CSS conventions (`styles.css`)

- All colours resolve through five custom properties on `:root`. Dark mode
  redefines them on `body.quarto-dark`; because custom properties inherit, no
  individual rule needs a dark variant. Anything hardcoded outside those tokens
  will not theme — promote it to a token first.
- Package-table column widths are scoped to `.pkg-table`, not `.table`, so they
  do not hit model-output tables inside posts. `software.qmd` wraps each table in
  `::: {.pkg-table}`. Below 768px those rows become stacked cards.

## Adding a package to the Software page

Add a row inside the relevant `::: {.pkg-table}` block in `software.qmd`, then
`quarto render` and commit `docs/`.
