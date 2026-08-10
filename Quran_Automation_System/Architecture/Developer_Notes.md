# Developer Notes — Quran Automation System

Running log of problems, solutions, decisions, and ideas that don't belong in the frozen architecture but are useful to remember.

---

## Format
```
### [Date] — [Short Title]
**Context:** what was happening
**Problem/Idea:** what came up
**Resolution/Decision:** what was decided
**Affects:** which file/module this touches, if any
```

---

### 2026-08-06 — Project Reorganized Into Professional Folder Structure
**Context:** Project had grown across multiple standalone documents (Architecture v3.2, Content Selection Lists, Hadith References) during planning.
**Decision:** Organized into `Quran_Automation_System/` with `Architecture/`, `Source/`, `Tests/`, `Docs/`, `Config/`, `Logs/` subfolders, following standard software project conventions.
**Affects:** All existing docs moved into `Architecture/` and `Docs/`; this is the baseline structure going forward.

### 2026-08-06 — Personal AI Assistant Suite Idea Deferred
**Context:** A much larger "Personal AI Assistant Suite" concept was proposed (browser automation, account creation help, email management, etc.) alongside the Quran project.
**Decision:** Deferred to a future, separate project (Phase 2/future). Reasoning: splitting focus across two large builds simultaneously risks slowing both down, and several proposed features overlap with existing Anthropic products (Claude in Chrome for browser automation, Claude Code/Cowork for background cloud processing) that should be evaluated first rather than rebuilt from scratch.
**Affects:** Not part of `System_Architecture.md` scope. Revisit only after Quran Automation System reaches stable production.

### 2026-08-06 — YouTube Community Post API: Correction
**Context:** Earlier architecture draft assumed Community Posts could be created via the YouTube Data API.
**Problem:** On verification, no official YouTube API (Data API v3, Analytics, Reporting, Live Streaming) exposes a community-post-creation endpoint.
**Resolution:** Marked as a manual task in System_Architecture.md Section 11 — Gemini can draft the post text, but the operator posts it manually via YouTube Studio unless a future API update changes this.
**Affects:** System_Architecture.md Section 11.

### 2026-08-06 — Chunked Rendering Must Never Compromise Quality
**Context:** Discussion about VM memory limits (free-tier ~1GB) potentially requiring long videos to be rendered in parts.
**Decision:** Quality (1080p, 8-10 Mbps) is fixed; number of chunks is the flexible variable. If chunking strains memory, increase chunk count (even to 10-12+) rather than lowering quality. Chunks must use identical encoding parameters and be joined via FFmpeg stream-copy concat (no re-encode) at ayah-boundary cut points only, so no seam is audible/visible.
**Affects:** System_Architecture.md Section 14.7.

### 2026-08-06 — Google Cloud Blocked; GitHub Actions + Cloudflare Workers Proposed (Pending Formal v3.3)
**Context:** Google Cloud free trial sign-up blocked — available debit cards not accepted for identity verification.
**Finding:** Oracle Cloud Always Free has the same card requirement (explicitly rejects debit cards with a PIN, prepaid, or virtual cards) — would likely hit the same wall.
**Proposed resolution (researched, not yet formally merged into System_Architecture.md):**
- GitHub Actions as main execution + scheduling platform (no card required, 2,000 free min/month private repos, 2-core/7GB RAM runners)
- Cloudflare Workers hosting the Telegram bot in webhook mode (no card required, avoids needing a persistent 24/7 server)
- Google Drive (existing Gmail, no new card) for storage/backups instead of a cloud VM's disk
**Status:** Not yet incorporated into System_Architecture.md — Section 7 (Google Cloud Deployment Notes) still reflects the old plan. This is a pending v3.3 revision.
**Affects:** System_Architecture.md Section 7 (pending update).

### 2026-08-06 — AI_Development_SOP Upgraded to v2.0 (AI Collaboration Workflow Merged)
**Context:** A detailed "AI Collaboration Workflow" document was proposed as a separate file (originally suggested filename referenced a new "MASTER_CONTEXT_v3.3.md").
**Decision:** Merged into `AI_Development_SOP.md` as v2.0 rather than kept as a separate parallel file, to avoid two overlapping "source of truth" documents (Role Matrix and Prompt Templates existed in both). `System_Architecture.md` remains the sole Master Context — no separate MASTER_CONTEXT file was created.
**New content added:** Task Status Flow (TODO→DONE), AI-to-AI Handoff Format, Bug/Feature Report formats, Architecture Change Rule (formalizes the existing Section 16.6 freeze policy for day-to-day AI behavior), Conflict Resolution rules, expanded AI Role Matrix (Grok/Perplexity added as research AIs).
**Note:** This introduces reliance on several additional AI tools (Grok, Perplexity, DeepSeek) coordinated manually by the operator (no API automation between them) — worth monitoring whether this adds too much manual overhead for a solo operator in practice.
**Affects:** `Architecture/AI_Development_SOP.md` (now v2.0, replaces v1.0 entirely).

### 2026-08-10 — Hosting Decision Finalized: v3.3 (Google Cloud Removed, Oracle Made Optional)
**Context:** Two rounds of independent multi-AI infrastructure research were conducted (Claude's own reports, plus comparison against Grok/Perplexity/other AI reports) per `AI_Development_SOP.md`'s Research role and Conflict Resolution process. Key conflict identified and resolved: Oracle's Always Free Ampere A1 allowance was reported as both 4 OCPU/24GB and 2 OCPU/12GB across different sources — resolved as **2 OCPU/12GB, current as of the June 15, 2026 (unannounced) Oracle change**; older figures are stale.
**Decision:** 
- Google Cloud **removed entirely** from infrastructure (card-verification status never confirmed resolved).
- **GitHub Actions** = primary execution/scheduling (no card, sufficient compute per research).
- **Cloudflare Workers** = Telegram webhook receiver only (not suitable for FFmpeg/rendering — 10ms free CPU-time limit).
- **Google Drive** = storage/backup (uses existing Gmail, independent of the Google Cloud decision).
- **Oracle Cloud** = optional secondary resource only, per an explicit usage policy table (System_Architecture.md Section 7.4) — never mandatory for normal Shorts/routine production; usable for high-value/research-heavy/genuinely heavy rendering tasks after benchmarking.
**Affects:** System_Architecture.md Section 7 (fully rewritten), version bumped 3.2 → 3.3.
**Open item:** Real GitHub Actions minute consumption for this project's actual FFmpeg workload is still unmeasured — must be checked empirically during Module 07 (`render_video.py`) testing, not assumed from research alone.

---

*Add new entries above this line, most recent first or last — pick one convention and stay consistent. This file is never "frozen" — it grows throughout the project's life.*
