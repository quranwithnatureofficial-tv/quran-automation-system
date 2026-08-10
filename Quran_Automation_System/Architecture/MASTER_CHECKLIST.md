# MASTER CHECKLIST — Quran Automation System
### Follow top to bottom, in order. Don't skip ahead. Check items off as you go.

**Current version:** System_Architecture.md v3.3 — CODING_PHASE_READY
**Last synced:** 2026-08-10

---

## ✅ STAGE 0 — Already Done (Confirmed)
- [x] Gmail (English channel)
- [x] YouTube Channel — Quran With Nature (@QuranWithNature-TV)
- [x] Facebook Page
- [x] Instagram Business (linked to Facebook Page)
- [x] Threads
- [x] TikTok
- [x] Pinterest
- [x] Logo + Banner (YouTube)
- [x] System_Architecture.md (v3.3, frozen/coding-ready)
- [x] Content_Selection_Lists.md
- [x] Surah_Ayat_Hadith_References.md
- [x] AI_Development_SOP.md (v2.0)
- [x] Prompt_Library.md
- [x] Developer_Notes.md
- [x] Roadmap.md
- [x] Hosting decision finalized (GitHub Actions + Cloudflare + Google Drive + optional Oracle)
- [x] OPAIS project deferred and safely documented separately (not part of this checklist)

---

## 🔲 STAGE 1 — Remaining Setup (Do These Next, In Order)

1. [ ] **Bitwarden fully populated** — every credential so far (Gmail, YouTube, Facebook, Instagram, Threads, TikTok, Pinterest) saved with 2FA on the Gmail
2. [ ] **GitHub account created** (same Gmail, no card needed)
3. [ ] **GitHub repository created** — name: `quran-automation-system`, Private
4. [ ] **Project folder pushed to GitHub** — the `Quran_Automation_System/` structure you already have (Architecture/, Source/, Docs/, etc.)
5. [ ] **Telegram Bot created via BotFather** — token saved in Bitwarden
6. [ ] **Cloudflare account created** (no card needed) — Workers project created (empty, for now)
7. [ ] **Google Drive access confirmed** — just verify you can create/access a folder there for backups (uses existing Gmail, nothing new to set up)
8. [ ] *(Skip Oracle Cloud for now — optional, only needed later per Section 7.4's policy)*

---

## 🔲 STAGE 2 — First Module (Prove the Core Works)

9. [ ] **`fetch_text.py`** — fetch Arabic + English translation for Surah Al-Ikhlas from Quran.com API. Test locally (not on any cloud yet).
10. [ ] **`integrity_check.py`** — cross-check that same text against Tanzil.net. Confirm it correctly catches a mismatch (test with a deliberately wrong value).
11. [ ] **`state_manager.py`** — basic checkpoint read/write to a local SQLite file. Test that it survives a simulated crash (kill the script mid-run, restart, confirm it resumes correctly).
12. [ ] **`main.py`** — orchestrator skeleton that calls the above two modules in sequence, with a working `DRY_RUN` flag.

**Milestone check:** At this point you should be able to run `main.py` locally and see correctly-fetched, verified Quran text for Al-Ikhlas printed out, with no uploads happening.

---

## 🔲 STAGE 3 — First Real Video (Manual, End-to-End)

13. [ ] **`fetch_audio.py`** + license registry — pick and verify 2-3 reciters via EveryAyah (Unlisted-upload test for Content ID claims, per Section 3)
14. [ ] **`render_text.py`** — Arabic + English text overlay rendering (Pillow + arabic_reshaper + python-bidi)
15. [ ] **`render_video.py`** — full Short for Surah Al-Ikhlas: background + audio + text, 1080x1920, quality settings per Section 12
16. [ ] **Measure actual render time/resources used** — record this in Developer_Notes.md (this resolves the open question about GitHub Actions minute consumption)
17. [ ] **`telegram_bot.py`** — approval loop (Approve/Reject/Re-render/Edit/Schedule) + the two-message verification workflow (Section 14.14)

**Milestone check:** One real, correct, good-quality Short video for Surah Al-Ikhlas, approved by you via Telegram, sitting as a file — not uploaded yet.

---

## 🔲 STAGE 4 — First Real Upload

18. [ ] **`youtube_upload.py`** — upload the Al-Ikhlas Short as **Unlisted** first
19. [ ] **Manual Content ID check** in YouTube Studio (Copyright notices tab) — confirm clean
20. [ ] **Publish it** — your first real, live video
21. [ ] **`ai_council.py`** — Gemini + Groq + DeepSeek generating title/tags/thumbnail concept, with the fallback rule (Section 9.1) tested (simulate one service failing)

**Milestone check:** Your channel has its first real, properly-produced, human-approved video, live.

---

## 🔲 STAGE 5 — Automate the Loop

22. [ ] **`watchdog` module** — basic monitoring wired in (Section 14.11)
23. [ ] **Database + logging** fully wired through every module used so far
24. [ ] **GitHub Actions workflow** — get the full pipeline (fetch → render → AI Council → Telegram) running on a schedule, not manually triggered
25. [ ] **Cloudflare Worker webhook** — wire up so Telegram replies trigger the GitHub Actions continuation, instead of you running things by hand

**Milestone check:** You send an ayah request or wait for the schedule, get a Telegram preview, tap Approve, and the video publishes — without you touching a terminal.

---

## 🔲 STAGE 6 — Scale Content (Real Videos, Not Dummy Tests)

26. [ ] Produce 5-10 more real Shorts this way — covering your priority list (Al-Fatiha, Ayat al-Kursi, Al-Kahf, etc. from Content_Selection_Lists.md)
27. [ ] Produce your first real Long video (both versions — Arabic-only, and bilingual alternating narration per Section 12.1)
28. [ ] `content_request_handler.py` — test the manual on-demand request feature (Section 18)
29. [ ] Continue scaling toward the fuller stress-test pass (30 Shorts + 10 Long) — these are your real launch content, not throwaway tests (Section 14.13)
30. [ ] Progressively test failure scenarios: one simulated API failure, one network interruption, one GitHub Actions failure — spread out, not all at once

---

## 🔲 STAGE 7 — Full Multi-Platform + Multi-Language

31. [ ] Add upload modules for Facebook, Instagram, Threads, TikTok, Pinterest (per Section 10's per-platform verification — check each platform's current API docs before building)
32. [ ] Multi-language titles/descriptions (Section 11.1)
33. [ ] Multi-language subtitles (Section 11.2)

---

## 🔲 STAGE 8 — Urdu Channel (Clone, Not Rebuild)

34. [ ] New Gmail for Urdu channel
35. [ ] YouTube + Facebook + Instagram + Threads + TikTok + Pinterest (Urdu, same process as Stage 0)
36. [ ] Clone the proven pipeline config — swap translation source, language, TTS voice only

---

## 🔲 STAGE 9 — Post-Launch (Whenever You're Ready, No Rush)

37. [ ] Word-by-word highlighting (Section 4 — needs a timing-source decision first)
38. [ ] Analytics dashboard, playlist automation, performance reporting, additional branding themes

---

## How To Use This List

- Work through it **in order** — later stages assume earlier ones actually work, not just that the code exists.
- After finishing each numbered item, also check the matching box in `Roadmap.md` (that file tracks the same progress by phase/category, this one tracks it as a strict sequence).
- If you get stuck on any item, that's a good moment for a Bug Report (AI_Development_SOP.md Section 5) rather than skipping ahead.
- Don't jump to Stage 8 (Urdu) or Stage 9 (Post-Launch) early — the whole point of this order is proving the English pipeline works before multiplying it.
