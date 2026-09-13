# xiangao.github.io

Source for [xiangao.github.io](https://xiangao.github.io), the personal site of
Xiang Ao — applied economist and statistician working on causal inference,
econometrics, and policy evaluation.

Built with [Quarto](https://quarto.org). Pages are served from the committed
`docs/` directory; there is no CI render step.

## Layout

| Path | Contents |
|---|---|
| `index.qmd` | About page (Quarto `about` template) |
| `software.qmd` | R, Python, and Julia packages, grouped by topic |
| `books.qmd` | Online books |
| `blogs.qmd` | Post listing, RSS feed, and a link to the earlier Hugo blog |
| `posts/<slug>/index.qmd` | Individual posts |
| `styles.css` | Site styles, light and dark |
| `_freeze/` | Cached post render output (tracked) |
| `docs/` | Rendered site — this is what GitHub Pages serves |

## Build

```bash
quarto render
```

Then commit both the sources and `docs/`.

To change a post, render that file on its own — a project-level render restores
posts from `_freeze/` and will not pick up the edit:

```bash
quarto render posts/<slug>/index.qmd
```

The `uplift` post needs Python; see `CLAUDE.md` for the venv invocation and for
the freeze, output-drift, and `numpyro` caveats.
