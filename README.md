# rating.cards documentation

Shared documentation for all rating.cards projects (app, landing page, and future repos).

Start here: **[`_index.md`](./_index.md)** — the doc map, domain trigger matrix, and maintenance checklist for AI and humans.

**Layers:** `product/` (what it does) → `architecture/` + `integrations/` (how it works) → `sales/` + `brand/` (audience views) → `runbooks/` (ops and testing).

The monolithic [project-brief.md](./project-brief.md) has been split into `product/`; the stub redirects here.

## Submodule usage

This repo is consumed as a **git submodule** at `docs/` in consumer projects.

| Project | Role |
|---------|------|
| [rating-cards-app](https://github.com/ZarqiCode/rating-cards-app) | **Source of truth** — edits docs after code/product changes |
| Landing page & other repos | **Read-only consumers** — pull updates, do not edit docs here |

### Clone a consumer repo with docs

```bash
git clone --recurse-submodules https://github.com/ZarqiCode/rating-cards-app.git
# or, after a normal clone:
git submodule update --init --recursive
```

### Pull latest docs (consumer repos)

```bash
git submodule update --remote docs
git add docs
git commit -m "chore(docs): bump docs submodule"
```

### Edit docs (app repo only)

1. Edit files inside `docs/` (the submodule checkout).
2. Commit and push **inside the submodule** (`cd docs`).
3. In the app repo root, commit the submodule pointer bump (`git add docs`).

See [rating-cards-app `.cursor/rules/docs-maintenance.mdc`](https://github.com/ZarqiCode/rating-cards-app/blob/main/.cursor/rules/docs-maintenance.mdc) for the full maintenance workflow.
