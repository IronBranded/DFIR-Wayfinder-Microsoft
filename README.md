<h1 align="center">Microsoft DFIR Wayfinder</h1>
<p align="center"><em>An Enterprise DFIR Field Guide — Windows Endpoints, Memory, Active Directory, and Hybrid/Cloud Identity</em></p>

<h3 align="center">
  <a href="https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/" target="_blank" rel="noopener noreferrer">
    🟢 NAVIGATE THE ATLAS 🟢
  </a>
</h3>

## What this is

Most DFIR references teach recognition: here is an artifact, here is what normal looks like, here is the red flag. That's necessary, but it isn't the job. The actual work is synthesis — taking five artifacts with five different timestamp formats and turning them into one defensible narrative of what happened, then explaining that narrative to two audiences who need completely different things from the same facts.

This guide is built around that whole loop, not just the recognition half of it. Every artifact page follows the same shape (what it is, what's normal, what's a red flag, how to collect it, how it maps to MITRE ATT&CK) so it's fast to look things up under pressure. But layered on top of that reference material is the part most guides skip: how to correctly order events across hosts with different clocks, how much corroboration actually justifies acting, and how to write the same confirmed finding two ways — one technical, one for a decision-maker who has never heard of a VAD tree — without either version being wrong.

It's written for enterprise incident response: SOC analysts, incident responders, and threat hunters working investigations that end in a report and a remediation plan, not a courtroom. Windows-first — endpoints, memory, Active Directory, and the hybrid identity layer connecting on-prem AD to Entra ID — because that's where most enterprise incidents actually live.

## What's inside

| Section | What it covers |
|---|---|
| **Foundations** | The shared vocabulary everything else assumes — IR lifecycle, order of volatility, MITRE ATT&CK, Pyramid of Pain, Diamond Model, evidence handling, timeline construction across mismatched clocks, and how to report a finding to both a technical reviewer and an executive from the same facts |
| **Windows Endpoint Forensics** | Prefetch, MFT/USN Journal, Amcache/Shimcache, registry artifacts, event log IDs, and the baseline process tree — what normal parent/child relationships look like, and the specific deviations that aren't |
| **Active Directory & Domain Controllers** | NTDS.dit, SYSVOL/GPO abuse, replication metadata, `krbtgt` and why it gets reset twice, DCSync, Golden/Silver Ticket, AdminSDHolder, Kerberoasting, AD CS abuse, and ACL/delegation attack paths |
| **Windows Memory Forensics** | Acquisition, Volatility 3 workflow, EPROCESS internals, injected-code and thread-level detection, LSASS analysis, and a capstone malware-triage methodology tying it all together |
| **PowerShell Forensics** | The four logging mechanisms, a verified hands-on obfuscation/decoding walkthrough, AMSI and evasion detection, and the cmdlet patterns worth alerting on |
| **Persistence Catalog** | Every mechanism, endpoint and cloud, tagged to its ATT&CK sub-technique — one canonical entry per mechanism, linked from wherever it's relevant instead of repeated |
| **Cloud Identity (Entra ID / Hybrid)** | Sign-in vs. audit logs and the retention trap between them, Conditional Access, Identity Protection, hybrid sync mechanics, the full hybrid account-compromise runbook, and why the Entra Connect server itself is a DCSync-equivalent target |
| **Microsoft Defender Suite** | The whole family mapped correctly (AV, Endpoint, Identity, Cloud Apps, Office 365, Cloud, XDR), an Advanced Hunting KQL library, and detection engineering from hunt query to standing rule |
| **Network & Perimeter Log Analysis** | DNS-based C2 and proxy/firewall triage — deliberately not full packet forensics, which is its own discipline |
| **Anti-Forensics & Data Recovery** | What attackers do to destroy or hide evidence — file carving, alternate data streams, log tampering and recovery, Volume Shadow Copy recovery, memory-based file recovery — and what survives anyway |
| **Case Studies** | Full narrated investigations showing the reasoning *between* the reference pages: why one finding sends you to check something specific next, and the moment a naive read of the data would've been wrong |
| **Investigation Playbooks** | Seven scenarios (BEC, ransomware, insider threat, domain compromise, data exfiltration, phishing, web shell) as short decision paths linking into everything above |
| **Practice Drills** | Six hands-on exercises with simulated data and a real collapsible answer — including one built specifically to force cross-source timestamp reconciliation |
| **Quick Reference & Glossary** | A dense, poster-style page for keeping open during an active incident, plus a full term lookup |

~54,000 words across 107 pages, one consistent design system throughout.

## How to use this

- **New to DFIR?** Start at Foundations, in order — everything downstream assumes the vocabulary and frameworks introduced there.
- **Mid-incident, need an answer fast?** The [Quick Reference](https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/quick-reference/windows-ir-poster/) has the highest-frequency lookups on one page. For a specific scenario, go straight to its [Playbook](https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/08-playbooks/) — each is a short path that links out to the detail rather than repeating it inline.
- **Want to see the reasoning, not just the facts?** Read a [Case Study](https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/case-studies/) end to end before touching the reference pages it draws from.
- **Testing yourself?** The [Practice Drills](https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/practice-drills/) give you real (simulated) data and hide the answer behind a click, not a scroll.
- **Looking something up?** Full-text search is in the top bar. The [Tags Index](https://ironbranded.github.io/Microsoft-DFIR-Wayfinder/tags/) filters by module or MITRE ATT&CK ID directly.

---

## Development & Deployment

<details>
<summary>Tech stack, running locally, contributing, and deployment notes</summary>

### Stack

Built with [Zensical](https://zensical.org) (the Material for MkDocs team's successor to MkDocs Material — this project migrated deliberately, after test-building the actual site against Zensical to confirm compatibility before switching, rather than staying on a toolchain with a confirmed end-of-life). Content is plain Markdown; `scripts/tag_pages.py` and `scripts/generate_backlinks.py` regenerate the tags front-matter and cross-page "Referenced From" sections and run automatically in CI before every deploy.

### Running it locally

```
pip install -r requirements.txt
zensical serve
```

Open `http://127.0.0.1:8000`. (`zensical build -s` runs a one-off strict build without serving — what CI uses.)

### Deployment

Pages source must be **Settings → Pages → Source → "GitHub Actions"** (not "Deploy from a branch" — the latter serves GitHub's own Jekyll rendering of this README instead of the actual built site, which is exactly what happened before this was fixed). The workflow (`.github/workflows/deploy.yml`) builds with Zensical and deploys via `actions/deploy-pages`, scoped to Pages-specific permissions rather than repo-wide write access. A weekly dead-link check (`.github/workflows/link-check.yml`) runs via `lychee` against every Sources section.

If you're standing this project up somewhere else: update `site_url`/`repo_url`/`repo_name` in `mkdocs.yml` to match your repo before following the deployment step above.

### Contributing / content standards

See [CONTRIBUTING.md](CONTRIBUTING.md) — every reference page follows one of four fixed templates (Artifact, Persistence Entry, Playbook, or Practice Drill) so the guide stays consistent as it grows.

### Sourcing

Every page is original writing built from the facts, structures, and taxonomies published by SANS (course material and posters), Microsoft Learn/MSTIC, and the wider DFIR community. Nothing is reproduced verbatim from any source; each page links out to primaries for readers who want the original material.

</details>
