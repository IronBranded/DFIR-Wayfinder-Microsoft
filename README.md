# Enterprise DFIR Field Guide

An interactive, data-driven reference for enterprise digital forensics and incident response — Windows endpoints, memory, Active Directory, and hybrid/cloud identity. Built for SOC analysts, incident responders, and threat hunters. Not written for criminal-prosecution workflows.

**Live site:** update this once deployed — `https://YOUR-USERNAME.github.io/dfir-field-guide/`

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

This project already has one git commit in it (see the note on revision dates above) — you're pushing an existing repo, not starting fresh.

1. Create a new, empty repository on GitHub (don't initialize it with a README, license, or .gitignore — this project already has all three).
2. Point it at your new repo and push:
   ```
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git push -u origin main
   ```
3. In `mkdocs.yml`, replace `YOUR-USERNAME`/`dfir-field-guide` in `site_url`, `repo_url`, and `edit_uri` with your actual GitHub username and repo name.
4. The included GitHub Action (`.github/workflows/deploy.yml`) runs automatically on every push to `main` — it regenerates tags/backlinks, builds the site with Zensical, and deploys straight to the `gh-pages` branch via `peaceiris/actions-gh-pages`.
5. After the first successful run, go to **Settings → Pages** in your GitHub repo and confirm the source is set to the `gh-pages` branch (GitHub sometimes needs this pointed at manually the first time).
6. Optional: enable the weekly dead-link check by doing nothing — `.github/workflows/link-check.yml` is scheduled and runs on its own once the repo is on GitHub.

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
