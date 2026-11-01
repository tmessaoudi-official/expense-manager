# expense-manager Plan

## Decisions Log
- [2026-11-01 22:25] AGREED: Scope: Large research-only deliverable, no app code in this pass.
- [2026-11-01 22:25] AGREED: Audience: personal daily driver first, architected to become a commercial product later.
- [2026-11-01 22:25] AGREED: Bank sync: research both manual/smart capture and PSD2 aggregators; decide later.
- [2026-11-01 22:25] AGREED: Deliverable: rich HTML artifact (docs/report.html, published privately) plus a standalone repo with GPG-signed commits (personal key 0C537690706EB7F5).
- [2026-11-01 22:25] AGREED: Remote: PUBLIC GitHub repo tmessaoudi-official/expense-manager so GitHub Actions CI stays free.
- [2026-11-01 22:25] AGREED: Certification tier: advisor() only for this task's gates.
- [2026-11-01 22:51] ASSUMED (review): Sync and household features placed in v2 (roadmap) rather than v1 (product research) because they are the largest cost and depend on the sync decision D3. Alternatives: v1 as in 04-product.md.

## Formal Plan
1. Standalone repo: `git init -b master`, local identity, signing with personal key `0C537690706EB7F5`, `.gitignore`, README.
2. Research workflow: 5 parallel researchers → adversarial fact-check → roadmap and effort (`docs/research/*.md`).
3. This plan file and its Decisions Log.
4. HTML report `docs/report.html`, also published as a private claude.ai artifact.
5. Signed initial commit; public GitHub repo; push.
6. 6C certification (advisor), present the report, stop.

## Status
Report published (private): https://claude.ai/artifact/Fbjg72qHCG87LW7pExhGe2

<!-- progress-block v1 -->
| # | Step | Size | State | Evidence | Files |
|---|------|------|-------|----------|-------|
| 1 | Repo + signing setup | S | done | - | .gitignore, README.md |
| 2 | Research workflow | L | done | - | docs/research/** |
| 3 | Plan file | S | done | - | docs/plans/** |
| 4 | HTML report + artifact | M | done | - | docs/report.html |
| 5 | Signed commit + public remote | S | todo | - | - |
| 6 | 6C certification + present | S | todo | - | - |
<!-- /progress-block -->
### Blocked
### Needs input
### Needs research
- Commercial aggregator quotes; Play stance on notification-listener access; Drift+PowerSync+SQLite3MC triple (Phase 0 spike).
### Fragile
### Known issues
