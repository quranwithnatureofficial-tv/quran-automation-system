# Project Roadmap — Quran Automation System

## Phase 1 — Foundation & Accounts
- [x] Gmail Ready (English channel)
- [x] YouTube Channel Ready (Quran With Nature)
- [x] Facebook Page Ready
- [x] Instagram Business Ready (linked to Facebook Page)
- [x] Threads Ready
- [x] TikTok Ready
- [x] Pinterest Ready
- [ ] Password Manager (Bitwarden) fully populated with all current credentials
- [ ] GitHub Repository Ready
- [ ] Google Drive confirmed accessible for backups (existing Gmail — no new setup needed)
- [ ] Cloudflare account + Workers project created (for Telegram webhook)
- [ ] Oracle Cloud account (OPTIONAL — only if/when a high-value task per Section 7.4's policy needs it; not a blocker for Phase 1 completion)
- [ ] Telegram Bot Created & Ready

## Phase 2 — Architecture & Documentation
- [x] System_Architecture.md (v3.2) — frozen, coding-phase-ready
- [x] Content_Selection_Lists.md
- [x] Surah_Ayat_Hadith_References.md
- [x] AI_Development_SOP.md
- [x] Roadmap.md (this file)
- [x] Prompt_Library.md
- [x] Developer_Notes.md

## Phase 3 — Core Modules (English Channel First)
- [ ] Module 01 — `fetch_text.py`
- [ ] Module 02 — `integrity_check.py`
- [ ] Module 03 — `state_manager.py`
- [ ] Module 04 — `main.py` (orchestrator, DRY_RUN support)
- [ ] Module 05 — `fetch_audio.py` + license registry (first 2-3 verified reciters)
- [ ] Module 06 — `render_text.py`
- [ ] Module 07 — `render_video.py` (first working Short: Surah Al-Ikhlas)
- [ ] Module 08 — `telegram_bot.py` (approval loop + two-message verification workflow)
- [ ] Module 09 — `ai_council.py` (Gemini + Groq + DeepSeek)
- [ ] Module 10 — `youtube_upload.py` + Content ID manual-check step
- [ ] Module 11 — `watchdog` monitoring
- [ ] Module 12 — Database + logging wired throughout
- [ ] Module 13 — `content_request_handler.py` (v3.2 manual request feature)

## Phase 4 — Testing
- [ ] 5-10 real end-to-end test videos (become actual launch content)
- [ ] Scale toward 30 Shorts + 10 Long stress pass
- [ ] Simulated API failure
- [ ] Simulated VM/server restart
- [ ] Simulated network interruption
- [ ] Simulated disk-full condition
- [ ] Simulated API quota exceeded
- [ ] Simulated Telegram offline
- [ ] Simulated GitHub Actions failure

## Phase 5 — Automation & Deployment
- [ ] GitHub Actions scheduling (Shorts 2x/day, Long 2x/week — Thursday night + Monday)
- [ ] Multi-platform upload modules (Facebook, Instagram, Threads, TikTok, Pinterest)
- [ ] Multi-language titles/descriptions (Section 11.1)
- [ ] Multi-language subtitles (Section 11.2)

## Phase 6 — Urdu Channel (Clone of Proven English Pipeline)
- [ ] New Gmail for Urdu channel
- [ ] YouTube Channel (قرآن اور فطرت)
- [ ] Facebook, Instagram, Threads, TikTok, Pinterest (Urdu)
- [ ] Clone pipeline config: translation source, language, TTS voice

## Phase 7 — Post-Launch Roadmap
- [ ] Word-by-word highlighting (requires timing-source decision, System_Architecture.md Section 4)
- [ ] Multi-language expansion beyond English/Urdu
- [ ] Analytics dashboard
- [ ] Playlist automation
- [ ] Performance reporting
- [ ] Multiple branding themes

---

*Update this checklist as each item is completed — check items off with [x], keep incomplete as [ ]. This file reflects real progress, not the plan itself (that's System_Architecture.md).*
