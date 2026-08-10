# Quran Automation System

Automated, human-approved Quran content pipeline for YouTube, Facebook, Instagram, Threads, TikTok, and Pinterest — starting with the English channel "Quran With Nature," with an Urdu channel ("قرآن اور فطرت") to follow as a clone of the proven pipeline.

## Core Principle

AI never generates, modifies, or validates Quranic Arabic text, translations, Hadith authenticity, Tafsir, or Islamic rulings. These come only from trusted sources (Quran.com, Tanzil.net, and the verified `hadith_registry`). AI is used only for creative/editorial decisions (titles, tags, thumbnail concepts, visual style) via the AI Council (Gemini + Groq + DeepSeek), and every publish action requires human approval via Telegram.

## Project Structure

```
Quran_Automation_System/
│
├── Architecture/
│   ├── System_Architecture.md      # Frozen design document (v3.2) — the source of truth
│   ├── AI_Development_SOP.md       # Living process document — how development happens day-to-day
│   ├── Roadmap.md                  # Progress checklist across all phases
│   ├── Prompt_Library.md           # Reusable prompts per AI tool
│   └── Developer_Notes.md          # Running log of decisions, problems, solutions
│
├── Source/
│   ├── modules/                    # Python modules (fetch_text.py, render_video.py, etc.)
│   └── config/                     # Config files (channels.yaml, pipeline.yaml, etc. — no secrets)
│
├── Tests/                          # Test scripts and test-run records
├── Docs/
│   ├── Content_Selection_Lists.md  # Surah priority order + Shorts ayat categories
│   └── Surah_Ayat_Hadith_References.md  # Hadith verification registry (source list)
├── Config/                         # Deployment-level config (kept separate from Source/config)
├── Logs/                           # Runtime logs (gitignored in practice — local/cloud only)
└── README.md                       # This file
```

## Where To Start

1. Read `Architecture/System_Architecture.md` first — it's the frozen design.
2. Check `Architecture/Roadmap.md` for current progress.
3. Follow `Architecture/AI_Development_SOP.md` for how to develop each module.
4. Use `Architecture/Prompt_Library.md` for consistent prompts across AI tools.
5. Log anything notable in `Architecture/Developer_Notes.md` as you go.

## Status

**Current phase:** Phase 3 — Core Modules (see Roadmap.md). Architecture is frozen at v3.2, CODING_PHASE_READY.
