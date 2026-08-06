---
tags:
  - Windows Endpoint
  - T1070
  - T1070.006
---
# Artifact: $MFT & Timestomping
## What it is / where it lives
The Master File Table (`$MFT`) is the index NTFS uses to track every file and directory on a volume — it isn't a normal file you can just open, but it's extractable with any forensic imaging or triage tool. Every object on the volume gets at least one MFT record (typically 1024 bytes), and each record carries several "attributes." Two matter enormously for DFIR:
- **`$STANDARD_INFORMATION` (SI)** — the MACB timestamps (Modified, Accessed, Changed, Born/Created) that most tools, including Windows Explorer's own Properties dialog, show by default. These are easy for a user-mode tool to modify.
- **`$FILE_NAME` (FN)** — a *second*, independent set of MACB timestamps, updated far more restrictively by the OS (normally only at creation and on rename/move). Ordinary timestomping utilities that only touch the file via standard Win32 APIs modify SI but can't easily touch FN.
## Normal baseline
For a legitimately created file, SI and FN timestamps sit close together — FN's creation time is essentially never *later* than SI's, and both roughly track when the file actually arrived on the volume.

The two attributes have fixed type codes — `$STANDARD_INFORMATION` is `0x10`, `$FILE_NAME` is `0x30` — which is why a parsed MFT shows paired columns like `Created0x10`/`Created0x30`: the suffix tells you which attribute a value came from without having to remember which is which.

SI and FN diverge under everyday, non-adversarial operation because they're maintained by different logic: SI tracks the file's *content and metadata*, FN tracks its *name and location*. What each common operation actually touches:

**SI — `$STANDARD_INFORMATION` (`0x10`)**

| Operation | M | A | C | B |
|---|---|---|---|---|
| Rename (same directory) | — | — | **Update** | — |
| Local move (same volume) | — | — | **Update** | — |
| Cross-volume move | — | **Update**¹ | **Update** | — |
| Copy | — | **Update**¹ | **Update** | **Update** |
| Open / read | — | **Update**¹ | — | — |
| Write / modify content | **Update** | — | **Update** | — |
| Create | **Update** | **Update**¹ | **Update** | **Update** |
| Delete | — | — | — | — |

**FN — `$FILE_NAME` (`0x30`)**

| Operation | M | A | C | B |
|---|---|---|---|---|
| Rename (same directory) | — | — | — | — |
| Local move (same volume) | **Update** | — | **Update** | — |
| Cross-volume move | **Update** | **Update** | **Update** | **Update** |
| Copy | **Update** | **Update** | **Update** | **Update** |
| Open / read | — | — | — | — |
| Write / modify content | — | — | — | — |
| Create | **Update** | **Update** | **Update** | **Update** |
| Delete | — | — | — | — |

**Update** = timestamp is rewritten · **—** = existing value survives unchanged
¹ Depends on the host's Last Access policy — see below. On most enterprise endpoints this column is inert.

Two shapes in that table get mistaken for tampering more often than anything else in this guide:

- **Copy inherits Modified but not Created.** A copy engine typically preserves the source's SI Modified time on the new file while stamping SI Created fresh, so Modified earlier than Created is the *ordinary* signature of a copy — software deployment and users pulling files off a share produce it constantly. MFTECmd flags this automatically as a `Copied` column; on its own it isn't evidence of anything.
- **Cross-volume moves invert the "FN never later than SI" rule stated above.** A move across volumes is a copy-then-delete under the hood: the destination gets an entirely fresh FN record, while Windows deliberately preserves the *original* SI Created value for continuity. FN Created ends up later than SI Created — the identical shape to the red flag below, produced by an ordinary drive-to-drive move. Check the [USN Journal](usn-journal.md) for a `FILE_CREATE` matching the FN timestamp before calling this timestomping.

**Access is the least trustworthy column here.** Whether it updates at all is controlled by `NtfsDisableLastAccessUpdate` (`HKLM\SYSTEM\CurrentControlSet\Control\FileSystem`), a 4-state flag since Windows 10 1803 rather than a simple on/off switch:

| Value | Mode | Updates |
|---|---|---|
| `0x80000000` | User Managed | Enabled |
| `0x80000001` | User Managed | Disabled |
| `0x80000002` | System Managed | Enabled |
| `0x80000003` | System Managed | Disabled |

Default is System Managed, which enables updates only on system volumes at or under ~128 GB and disables them above that threshold — meaning Access is switched off by default on most enterprise endpoints. Confirm with `fsutil behavior query disablelastaccess` before citing an Access timestamp as fact in a finding.

Older material — including the widely-used *Incident Response & Computer Forensics* text — labels this same four-field set **MACE** (Modified, Accessed, Created, Entry Modified) rather than MACB. Same four values; only the letter and position of the fourth field differ. This guide uses MACB throughout.

!!! success "Baseline"
    SI and FN timestamps consistent with each other and with the surrounding MFT record sequence — accounting for the copy and cross-volume-move exceptions above.
## Red flags
- **SI creation time earlier than FN creation time.** This is the textbook timestomping signature: a tool backdated the visible (SI) timestamp but couldn't touch the harder-to-reach FN timestamp underneath it. Rule out a cross-volume move first (see Normal Baseline) — it produces the identical shape with zero tampering involved.
- **A file's SI timestamp that doesn't fit its neighbors.** MFT records created close together in time (adjacent record numbers) should have close creation timestamps. A record with a wildly out-of-sequence SI creation time, sitting among records from a completely different period, is worth a second look.
- **A deleted file's MFT record still readable with intact metadata.** NTFS marks a record "available for reuse" on deletion but doesn't necessarily zero it immediately — a freshly deleted malicious binary's original name, size, and timestamps can often still be recovered.
!!! danger "Red flag"
    SI creation time predates FN creation time, or a timestamp badly out of sequence with neighboring records.
## How to collect it
Extract `$MFT` as part of standard triage (it's accessible via most imaging tools, or directly via a raw volume read). Parse with Eric Zimmerman's `MFTECmd`, which outputs both SI and FN timestamp sets side by side specifically so they can be diffed:
```
MFTECmd.exe -f C:\triage\$MFT --csv C:\triage\output
```
Three of the output columns do that diffing for you instead of requiring a manual compare: `SI<FN` flags SI Created or Modified earlier than the corresponding FN value (the timestomping shape), `Copied` flags SI Modified earlier than SI Created (the benign-copy shape), and `uSecZeros` flags an exactly-zero sub-second component, consistent with a tool writing a whole-second value rather than a genuine filesystem write.
## ATT&CK mapping
Directly maps to [T1070.006 (Indicator Removal: Timestomp)](https://attack.mitre.org/techniques/T1070/006/). More broadly, `$MFT` is a foundational **data source** for filesystem timeline reconstruction across nearly every other technique in this guide.
<!-- BACKLINKS:START -->
## Referenced From
- [ATT&CK Coverage Map](../00-foundations/attack-coverage-map.md)
- [Timeline Construction & Correlation](../00-foundations/timeline-construction.md)
- [Module 1: Windows Endpoint Forensics](index.md)
- [Artifact: Shimcache (AppCompatCache)](shimcache.md)
- [Artifact: USN Journal](usn-journal.md)
- [Alternate Data Streams (ADS)](../anti-forensics/alternate-data-streams.md)
- [File Carving](../anti-forensics/file-carving.md)
- [Anti-Forensics & Data Recovery](../anti-forensics/index.md)
- [Memory-Based File Recovery](../anti-forensics/memory-based-file-recovery.md)
- [Glossary](../glossary.md)
<!-- BACKLINKS:END -->
## Sources
- [SANS Digital Forensics Certifications overview](https://www.sans.org/cyber-security-certifications/digital-forensics-certifications) (FOR500: Windows Forensic Analysis — NTFS/MFT internals is a core module)
- [MITRE ATT&CK T1070.006](https://attack.mitre.org/techniques/T1070/006/) — SI/FN split and double-timestomping evasion
- Maxim Suhanov (dfir.ru) — [`NtfsDisableLastAccessUpdate` semantics](https://dfir.ru/2018/12/08/the-last-access-updates-are-almost-back/) and [SI vs. FN timestamp behavior](https://dfir.ru/2021/01/10/standard_information-vs-file_name/)
- David Cowen (Hacking Exposed Computer Forensics Blog) — [empirical testing of the Windows 10 1803 Last Access change and the 128 GB System Managed threshold](https://www.hecfblog.com/2018/12/daily-blog-559-forensic-lunch-test.html)
- Eric Zimmerman (binaryforay) — [`MFTECmd` column reference](https://binaryforay.blogspot.com/2018/06/introducing-mftecmd.html), including the `SI<FN`/`Copied`/`uSecZeros` flags
- Microsoft — [`ATTRIBUTE_LIST_ENTRY` reference](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-list-entry) confirming the `0x10`/`0x30` attribute type codes
