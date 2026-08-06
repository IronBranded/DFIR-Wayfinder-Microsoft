# Enterprise DFIR Field Guide

An interactive, data-driven reference for enterprise digital forensics and incident response — Windows endpoints, memory, Active Directory, and hybrid/cloud identity. Built for SOC analysts, incident responders, and threat hunters. Not written for criminal-prosecution workflows.

**Live site:** `https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/`

## Status

Complete. All 9 original modules plus Anti-Forensics & Data Recovery, Network & Perimeter Log Analysis, Case Studies, Investigation Playbooks, and Practice Drills are fully built. ~50,000+ words across 107 pages, rebuilding clean under `zensical build -s` (strict mode) after every change.

## Functionality added beyond the original build

- **Tags** — every page tagged by module and, where one is named directly, MITRE ATT&CK ID. Browse via the Tags Index. Regenerate after adding new pages with `python3 scripts/tag_pages.py` (idempotent — skips files that already have front matter).
- **Backlinks** — every page with incoming links gets an auto-generated "Referenced From" section. Regenerate with `python3 scripts/generate_backlinks.py` (idempotent, safe to run repeatedly — this runs automatically in CI before every deploy).
- **Breadcrumbs**.
- **Scheduled dead-link checking** — `.github/workflows/link-check.yml` runs weekly (and on any PR touching `docs/`) via `lychee`, opening an issue automatically if something 404s.
- **Light-mode contrast fix** — the original teal/amber accents failed WCAG AA against a white background (verified by computing actual contrast ratios, not eyeballing); replaced with darker variants that pass in light mode while dark mode keeps the original, already-compliant colors.

### Two features currently disabled, not removed

Per-page "last updated" dates (`git-revision-date-localized`) and combined PDF export (`mkdocs-to-pdf`) both worked under the old MkDocs Material build. Both are third-party plugins outside Material's own ecosystem, and after migrating to Zensical (see below), Zensical silently no-ops any plugin it doesn't yet natively support rather than erroring — so the build stays clean, but neither feature currently does anything. Their config is still documented in `mkdocs.yml`'s plugins section as comments. Revisit both once Zensical's module/plugin system (on their public roadmap) lands; until then, per-file history is fully visible via normal `git log`, and PDF export would need to run through a separate MkDocs-Material build if wanted in the meantime.

## Why this runs on Zensical, not Material for MkDocs

This project was originally built on MkDocs Material. Partway through, it became clear that wasn't a safe long-term foundation: the underlying MkDocs project has been unmaintained since August 2024, Material for MkDocs itself entered maintenance mode with an end-of-life of **November 5, 2026**, and a from-scratch "MkDocs 2.0" in development elsewhere offered no migration path and would have broken every plugin this project depended on.

The Material for MkDocs team's own response was **Zensical** — a ground-up rewrite by the same team, MIT-licensed, explicitly built for near drop-in compatibility with existing Material for MkDocs projects. Rather than take that compatibility claim on faith, this project's actual `mkdocs.yml` and all 107 pages were test-built against Zensical directly before switching: it built clean on the first try, in strict mode, with zero configuration changes, and every feature specific to this project — admonition callouts, Mermaid diagrams, the tags plugin, custom CSS/fonts, breadcrumbs, the backlinks script's generated content — came through correctly. The only casualties were the two third-party (non-Material) plugins noted above. Real organizations (DDEV, USGS's public docs, Renovate) have already made the same move, which was part of the confidence behind doing this now rather than waiting.

If you're reading this after November 2026 and Zensical has since reached a stable 1.0, that's expected — it was Alpha at the time of this migration, under active, frequent development, with a committed 12-month window to reach feature parity with Material for MkDocs.

## Getting this live on your own GitHub Pages

This repo is `IronBranded/Microsoft-DFIR-Wayfinder`, already pushed. One setting is the difference between what's live now (GitHub's default Jekyll rendering of this README) and the actual built site — **Pages source has to be set to "GitHub Actions," not "Deploy from a branch."** This is a one-time manual step; nothing in the repo can set it automatically.

1. In this repo: **Settings → Pages → Build and deployment → Source**, change the dropdown from "Deploy from a branch" to **"GitHub Actions."**
2. **Settings → Actions → General → Workflow permissions** — confirm this is set to allow Actions to run (the deploy workflow's own `permissions:` block scopes exactly what it needs — `pages: write` and `id-token: write` — so this shouldn't need to be "Read and write," but if a run still fails with a permissions error, check here first).
3. Push any commit to `main` (or go to **Actions → Deploy Zensical site to GitHub Pages → Run workflow** to trigger it manually without waiting for a push) — the workflow regenerates tags/backlinks, builds with Zensical, and deploys via `actions/deploy-pages`, GitHub's own Pages deployment action, rather than pushing to a separate branch.
4. Give it a minute, then check `https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/` — you're looking for the actual dark/teal site with a sidebar and search, not this README rendered plain.

### Why the previous attempts didn't go live

Every earlier run of the deploy workflow failed at its final step — pushing the built site to a `gh-pages` branch via a third-party action that needs repo-wide write access, which the default token permissions didn't grant. Independently, and regardless of that, Pages was set to "Deploy from a branch," so even a successful run wouldn't have been served — GitHub's own automatic Jekyll build of `main` (specifically rendering `README.md`) was what's actually been live this whole time. The rewritten workflow above uses GitHub's own Pages-deployment action instead, scoped to Pages-specific permissions rather than full repo write access, which avoids this failure mode rather than working around it.

### If you push this project somewhere else instead

Steps 1–2 are specific to *this* repo already existing. Starting fresh elsewhere: create an empty GitHub repo, `git remote add origin <your-repo-url>`, `git push -u origin main`, then update `site_url`/`repo_url`/`repo_name` in `mkdocs.yml` to match before following steps 1–4 above.

## Running it locally

```
pip install -r requirements.txt
zensical serve
```

Then open `http://127.0.0.1:8000`. (`zensical build -s` runs a one-off strict build without serving, and is what CI uses.)

## Contributing / content standards

See [CONTRIBUTING.md](CONTRIBUTING.md) — every reference page follows one of four fixed templates (Artifact, Persistence Entry, Playbook, or Practice Drill) so the guide stays consistent as it grows.

## Sourcing

Every page is original writing built from the facts, structures, and taxonomies published by SANS (course material and posters), Microsoft Learn/MSTIC, and the wider DFIR community. Nothing is reproduced verbatim from any source; each page links out to primaries for readers who want the original material.
