# Quran Video Automation System — Production Architecture (Version 3.3)

**Status:** CODING_PHASE_READY — v3.2 frozen; this is a versioned amendment (v3.3) finalizing the hosting/infrastructure decision (Section 7): Google Cloud removed, GitHub Actions + Cloudflare Workers + Google Drive as primary, Oracle Cloud as optional secondary resource per the usage policy in Section 7.4
**Channels:** Quran With Nature (English) · قرآن اور فطرت (Urdu)
**Hosting:** GitHub Actions (execution/scheduling) + Cloudflare Workers (webhooks) + Google Drive (storage) + Oracle Cloud (optional, high-value tasks only)
**Approval:** Telegram Bot (human-in-the-loop, mandatory before every upload)

---

## 1. Guiding Principles

1. **Quranic Arabic text is never modified, "corrected," or validated by AI.** It is fetched only from trusted sources (Quran.com API, Tanzil.net) and passed through an *integrity check* (checksum/text-match against the source), not an AI rewrite. AI can hallucinate — the Quran text must not touch that risk.
2. **Every audio file must have an explicit, logged license before use.** No file is used on the assumption that it is "probably free."
3. **A human always approves before public upload.** No stage bypasses the Telegram approval gate.
4. **Everything is logged and stored**, so any video's origin (text version, audio version, license, background clip, AI-generated metadata) can be traced later.

---

## 2. Folder Structure

```
quran-channel-system/
├── config/
│   ├── channels.yaml            # channel IDs, language, branding per channel
│   ├── surah_schedule.yaml      # weekly long-video rotation (Yaseen, Rahman, Mulk...)
│   └── secrets.env              # API keys (Gemini, YouTube, Pexels, Telegram) — gitignored
│
├── data/
│   ├── quran_text/               # cached Arabic + translations (per translator)
│   ├── audio_library/            # downloaded + license-verified reciter audio
│   │   └── licenses/             # license proof/metadata per reciter, per file
│   ├── background_library/       # curated local Pexels/Pixabay clips (cached, not re-fetched each time)
│   └── database.sqlite           # main metadata database (see Section 5)
│
├── modules/
│   ├── fetch_text.py             # pulls Arabic + translation from Quran.com/Tanzil
│   ├── integrity_check.py        # verifies fetched text against a second trusted source
│   ├── fetch_audio.py            # pulls reciter audio, checks against license registry
│   ├── fetch_background.py       # pulls/selects nature clip from local cache or Pexels
│   ├── render_text.py            # Arabic shaping (arabic_reshaper + python-bidi) + Pillow overlay frames
│   ├── render_video.py           # FFmpeg assembly: background + audio + text overlay + timing
│   ├── ai_council.py             # Gemini + Groq + DeepSeek — creative/editorial decisions ONLY (titles, tags, SEO, thumbnail concept, visual style); never Quran text/Hadith
│   ├── content_request_handler.py # v3.2 — operator-initiated on-demand requests via Telegram (Section 18); routes into the same verification/AI Council/approval flow
│   ├── telegram_bot.py           # sends preview, handles Approve/Reject/Re-render/Edit/Schedule
│   ├── youtube_upload.py         # uploads as Unlisted → waits for Content ID → publishes per schedule
│   └── logger.py                 # writes structured logs for every stage
│
├── workflows/                    # GitHub Actions YAML — 2x/day Shorts + 2x/week Long triggers
│   ├── shorts_morning.yml        # ~1st daily Shorts slot
│   ├── shorts_evening.yml        # ~2nd daily Shorts slot
│   ├── long_thursday_night.yml   # mandatory Jumu'ah-eve slot
│   └── long_monday.yml           # second weekly slot
│
└── main.py                       # orchestrator — calls modules in sequence
```

---

## 3. End-to-End Pipeline

```
1. TRIGGER
   GitHub Actions cron:
     - Shorts: 2 per day, at two different times (e.g. one morning/midday window, one evening/night window) to cover a global audience across time zones
     - Long videos: 2 per week — Thursday night (so it's live as Friday begins, timed for Jumu'ah-related viewing such as Al-Kahf) is mandatory; second slot on Monday (spaces content evenly across the week rather than clustering near Friday)
        │
2. FETCH TEXT
   fetch_text.py → Quran.com API → Arabic + selected translation (EN or UR per channel)
        │
3. INTEGRITY CHECK
   integrity_check.py → cross-match text against Tanzil.net copy already cached locally
   → if mismatch: STOP, alert via Telegram, do not proceed
        │
4. FETCH AUDIO
   fetch_audio.py → check data/audio_library/licenses/ registry first
   → if reciter/ayah not yet license-verified: STOP, alert via Telegram for manual check
   → if verified: pull cached file or download
        │
5. FETCH BACKGROUND
   fetch_background.py → pull from local curated library first (fast, no API dependency)
   → fallback to Pexels/Pixabay API only if local library lacks a fresh clip
        │
6. RENDER TEXT FRAMES
   render_text.py → Pillow + arabic_reshaper + python-bidi
   → produces per-ayah text overlay images, correctly shaped RTL Arabic + LTR translation
        │
7. RENDER VIDEO
   render_video.py → FFmpeg composites background + audio + text overlays
   → Shorts: 1080x1920, 30fps · Long: 1920x1080, 30fps
   → output saved locally + path logged to database
        │
8. GENERATE METADATA (AI — non-Quran content only)
   ai_council.py → Gemini + Groq + DeepSeek (all free tier) generate title/description/tags/thumbnail concept via the AI Council process (Section 9.1)
   → optional ChatGPT cross-check for metadata only, not for Quran text
        │
9. SAVE TO DATABASE
   Full record inserted (see Section 5)
        │
10. TELEGRAM PREVIEW
    telegram_bot.py sends: video file/clip, Arabic + translation text, title, tags, license info
    User replies:
      ✅ Approve      → go to Step 11
      ❌ Reject        → archive with reason, stop
      🔁 Re-render     → return to Step 6/7 with adjustment note
      ✏️ Edit          → user sends correction text, re-runs Step 6 onward
      🕒 Schedule      → set specific publish time, hold until then
        │
11. UPLOAD (Private/Unlisted first)
    youtube_upload.py → uploads via YouTube Data API as Unlisted
        │
12. CONTENT ID / COPYRIGHT CHECK
    Video sits as Unlisted. Automated polling of Content ID claim status is NOT available
    through the standard YouTube Data API — that requires a Content Owner/CMS account,
    which individual creator channels do not have.
    → Operator manually checks YouTube Studio → Copyright notices tab before publishing
    → Telegram bot sends a reminder prompt ("Check Studio for copyright notices, then reply
      Publish when clear") rather than an automatic pass/fail
        │
13. PUBLISH (per schedule)
    Video set to Public per config/surah_schedule.yaml timing
        │
14. ARCHIVE + LOG
    Final state, YouTube URL, Content ID result written to database + logs/
```

---

## 4. Word-by-Word Highlighting — Open Technical Question

This was the biggest gap across all reviewers' recommendations and needs a decision before implementation:

**Problem:** To highlight each Arabic word in sync with recitation, you need precise timestamps for where each word starts/ends in the audio.

**Options:**
1. **Use a source that already provides word-level timing** — some Quran audio APIs (e.g., certain EveryAyah-linked datasets) include word-by-word timestamp files. This is the simplest path *if* your chosen reciter has this data available and licensed.
2. **Forced alignment** — run the Arabic audio + known transcript through a forced-alignment tool (e.g., Gentle, aeneas) to algorithmically generate word timestamps. This works but adds a processing stage and is not 100% accurate for Quranic Arabic without tuning.
3. **Skip word-by-word for now** — start with ayah-by-ayah sync only (simpler, still looks professional, matches most reference channels you found), and add word-level highlighting later as a v4 feature once the core pipeline is stable.

**Recommendation:** Start with Option 3 for launch, evaluate Option 1 for your specific chosen reciters, keep Option 2 as a future upgrade.

---

## 5. Database Schema (SQLite)

```sql
CREATE TABLE videos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    job_id TEXT UNIQUE,            -- globally unique ID, checked before every upload to prevent duplicate uploads on restart
    channel TEXT,                  -- 'english' or 'urdu'
    video_type TEXT,               -- 'short' or 'long'
    surah_number INTEGER,
    ayah_range TEXT,               -- e.g. '1-4'
    translation_source TEXT,       -- e.g. 'Sahih International', 'Fateh Jalandhri'
    translation_version_id TEXT,
    audio_reciter TEXT,
    audio_file_path TEXT,
    audio_license_status TEXT,     -- 'verified', 'pending', 'rejected'
    audio_license_proof_path TEXT,
    background_clip_source TEXT,   -- 'Pexels' / 'Pixabay' / 'local_cache'
    background_clip_id TEXT,
    render_output_path TEXT,
    ai_title TEXT,
    ai_description TEXT,
    ai_tags TEXT,
    ai_council_summary TEXT,
    ai_council_agreement_level INTEGER,  -- 0-100
    telegram_decision TEXT,        -- 'approved','rejected','re-rendered','edited','scheduled'
    scheduled_publish_time TEXT,
    youtube_video_id TEXT,
    youtube_url TEXT,
    content_id_result TEXT,        -- 'clean','claimed','pending' — set MANUALLY by operator via YouTube Studio, not polled via API
    upload_status TEXT,            -- 'unlisted','public','rejected'
    config_version TEXT,
    pipeline_version TEXT,
    render_engine_version TEXT,
    ai_council_version TEXT,
    metadata_version TEXT,
    translation_revision TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    published_at TIMESTAMP
);

CREATE TABLE license_registry (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    reciter_name TEXT,
    source TEXT,
    license_type TEXT,             -- e.g. 'CC-BY', 'free-for-creators', 'verified-permission'
    proof_url_or_path TEXT,
    verified_by TEXT,              -- who/what verified it
    verified_date TIMESTAMP
);

CREATE TABLE hadith_registry (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    content_type TEXT,             -- 'quran_only','hadith_fadail','scholarly_opinion','common_practice'
    quran_reference TEXT,          -- Surah:Ayah this entry relates to, if applicable
    hadith_reference TEXT,         -- Book, chapter, hadith number
    hadith_grade_primary TEXT,     -- 'Sahih','Hasan','Da'if','Mawdu'' etc.
    graded_by_primary TEXT,
    hadith_grade_secondary TEXT,   -- if scholars differ
    graded_by_secondary TEXT,
    notes TEXT,
    source_url TEXT,
    date_verified TIMESTAMP,
    verified_by TEXT,
    verification_status TEXT       -- 'Verified','Needs Review','Disputed','Weak','Excluded'
);

CREATE TABLE logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    stage TEXT,                    -- 'fetch_text','render','upload', etc.
    video_id INTEGER,
    status TEXT,                   -- 'success','error','warning'
    message TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (video_id) REFERENCES videos(id)
);
```

**Rendering rule:** The render pipeline only queries `hadith_registry` entries where `verification_status = 'Verified'`. Entries marked Needs Review, Disputed, Weak, or Excluded are never auto-included in a video; Disputed/Weak entries require explicit manual approval (with on-screen wording noting the scholarly disagreement) if an operator chooses to include one anyway. If no Verified hadith entry matches a selected ayah, the video presents the Quranic verse alone (`content_type = 'quran_only'`) rather than forcing a pairing — see Content_Selection_Lists / Surah_Ayat_Hadith_References for the editorial policy in full.

---

## 6. Telegram Bot — Interaction Flow

```
Bot sends:
  🎬 [video preview/thumbnail]
  📖 Surah [name], Ayah [range]
  🕌 Arabic: [text]
  🌍 Translation: [text]
  🎙️ Reciter: [name] — License: ✅ Verified
  🏷️ Title: [AI-generated]
  📝 Tags: [AI-generated]

User replies (buttons or text):
  ✅ Approve       → uploads immediately (or per schedule)
  ❌ Reject        → archived, no upload
  🔁 Re-render     → user optionally adds a note ("change background", "audio too fast")
  ✏️ Edit text     → user sends corrected text, system re-renders
  🕒 Schedule      → user sends date/time, video held until then

If user does not reply within X hours:
  Bot sends a gentle reminder (does NOT auto-publish without approval)
```

**Note:** This is the core approval flow. See Section 14.14 for the expanded two-message design (Preview + Copy-Friendly Verification Block) that supports free manual cross-checking with ChatGPT or another AI before approval.

---

## 7. Hosting & Infrastructure (v3.3 — supersedes the original Google Cloud plan)

**Status:** Finalized after independent multi-AI research and comparison (see Developer_Notes.md entries for 2026-08-06 through 2026-08-10). Google Cloud is **removed** from the infrastructure plan — its card-verification requirement was never confirmed resolved, and Google Drive (used for storage below) does not depend on it.

### 7.1 Primary Execution & Scheduling: GitHub Actions
- All pipeline modules (`fetch_text.py` through `youtube_upload.py` and other platform uploads) run as GitHub Actions jobs, triggered on cron (Section 3 pipeline schedule: Shorts 2x/day, Long 2x/week).
- No credit card required for free-tier minutes (2,000 min/month private repos, unlimited public). Runners: 2-core/7GB RAM, max 6 hours/job — sufficient for this project's FFmpeg rendering workload based on research; actual minute consumption should be measured during Module 07 testing rather than assumed.
- SQLite database and `pipeline_state.json` persist between runs via GitHub Actions artifacts/cache, backed up to Google Drive after each run (Section 7.3).

### 7.2 Webhooks: Cloudflare Workers
- Hosts the Telegram bot's webhook endpoint only — receives Approve/Reject/Re-render/Edit/Schedule replies instantly (Section 6, 14.14) without needing a persistent 24/7 process.
- On receiving a decision, the Worker triggers a GitHub Actions `workflow_dispatch` (via the GitHub API) to continue the pipeline (e.g., proceed to upload).
- Never used for FFmpeg or any rendering — Cloudflare's free-tier CPU-time-per-request limit (10ms) makes this technically unsuitable, confirmed via research.

### 7.3 Storage & Backup: Google Drive
- Uses the project's existing Gmail account — no new account or card needed, separate from the (removed) Google Cloud infrastructure decision.
- Stores: rendered video backups, SQLite database backups (periodic), `hadith_registry`/`license_registry` exports for redundancy.

### 7.4 Oracle Cloud — Optional Secondary Resource (Not Mandatory)

**Final usage policy (decided 2026-08-10):**

| Work Type | Primary Workflow | Oracle |
|---|---|---|
| Normal Shorts | Lightweight/standard workflow (GitHub Actions) | ❌ Not used by default |
| High-volume Short production | Lightweight/standard workflow | ❌ Not used by default |
| Important/high-value videos | Standard workflow + deeper processing where needed | ✅ Use when beneficial |
| Research-heavy videos | Standard workflow + advanced processing | ✅ Use |
| Heavy long-form FFmpeg/rendering tasks | Standard infrastructure first; benchmark before moving work | ✅ Optional, when justified |
| True always-on workloads | Evaluate separately after benchmarking | ✅ Optional |

**Operating principle:** Oracle is never a mandatory dependency for normal content production. It is retained as an advanced resource for work that genuinely benefits from additional compute — never required for the pipeline to function.

**One-line rule:** *Normal Shorts → lightweight workflow (GitHub Actions). High-value / research-heavy / genuinely heavy work → Oracle when justified.*

**Known constraints on Oracle (from research — re-verify before relying on it for any specific task):**
- Always Free Ampere A1 allowance was cut from 4 OCPU/24GB RAM to 2 OCPU/12GB RAM, effective June 15, 2026, without official announcement — treat 2 OCPU/12GB as the current planning figure.
- Provisioning a new Always Free Ampere A1 instance frequently fails with "out of capacity" errors, region-dependent — this is an ongoing, not one-time, issue.
- Card-verification status for this project's account was not independently re-confirmed — verify directly before depending on Oracle for any task.

### 7.5 Security
All API keys/tokens (YouTube, Telegram bot token, Google Drive OAuth, Gemini/Groq/DeepSeek, Oracle credentials if used) live in GitHub Secrets and/or Cloudflare Workers secrets — never committed to any repository, matching the existing `.gitignore` rule (Section 16.4, AI_Development_SOP.md Section 8).

---

## 8. Logging Strategy

Every stage writes to the `logs` table and a local log file:
- `fetch_text.log` — source, timestamp, integrity check result
- `fetch_audio.log` — reciter, license check result
- `render.log` — render time, output path, any FFmpeg warnings
- `ai_metadata.log` — what was sent to Gemini/ChatGPT, what was returned (never includes Quran text as "verified/corrected" — only metadata generation calls are logged here)
- `upload.log` — upload status, Content ID result, publish time
- `telegram.log` — every decision made, by whom, when

This makes every published (or rejected) video fully traceable months later.

---

## 9. What AI Is and Isn't Used For

| Task | AI Used? | Notes |
|---|---|---|
| Quranic Arabic text | ❌ Never | Sourced directly from Quran.com/Tanzil, integrity-checked against each other, not AI-modified |
| Translation text | ❌ Never | Pulled as-is from a named, cited translator (e.g., Sahih International, Fateh Jalandhri) |
| Hadith authenticity/Tafsir | ❌ Never (no AI voting) | Governed strictly by `hadith_registry` verification_status (see Section 5) — never decided or influenced by AI consensus |
| Video titles/Descriptions/Tags/SEO | ✅ Yes | Generated via the AI Council (Section 9.1) — Gemini, Groq, DeepSeek |
| Thumbnail concept, background visual style, motion effects, color scheme | ✅ Yes | Generated via the AI Council (Section 9.1) |
| License-checking judgment calls | ❌ No | Human decision only — AI can suggest but final call is manual, logged in `license_registry` |

---

## 9.1 AI Council (`ai_council.py`)

**Scope — strictly limited to creative/editorial decisions:**
- Thumbnail concept
- Background visual style
- Motion effects
- Color scheme
- Tags/SEO/titles/descriptions

**Explicitly out of scope:** Quran text, Hadith authenticity, and Tafsir are never decided or influenced by AI voting/consensus — these remain governed solely by the trusted-source and `hadith_registry` rules in Sections 1, 4, and 5.

### Selected Free Models
| Model | Role |
|---|---|
| Gemini | Primary — visual concepts & SEO |
| Groq | Fast brainstorming & alternative ideas |
| DeepSeek | Logical reasoning & consistency check |

### Process
```
Editorial decision needed (e.g. thumbnail concept for this week's Surah)
        │
        ▼
Identical prompt sent to Gemini, Groq, and DeepSeek
        │
        ▼
Each model required to respond in strict JSON format:
{
  "title": "...",
  "reason": "...",
  "confidence": 0-100,
  "pros": ["...", "..."],
  "cons": ["...", "..."]
}
        │
        ▼
ai_council.py compares the three JSON responses:
  - Computes "Agreement Level %" (how many models converge on the same
    recommendation — this is a consensus/agreement metric, NOT an
    accuracy or correctness score, since there is no single "correct"
    creative answer)
  - Summarizes where all three agree, and where they diverge
        │
        ▼
Telegram bot sends the combined report:
  "All 3 models suggest: ___  (Agreement: 100%)"
  or
  "Gemini + Groq suggest: ___ | DeepSeek differs: ___  (Agreement: 66%)"
        │
        ▼
Mandatory Final Decision button (Human Approval) — operator picks one
suggestion, blends ideas, or overrides all three. No editorial output
is ever auto-applied without this human confirmation.
```

### Key Rules
1. **Strict JSON enforcement** — identical prompt structure to all three models ensures a fair, comparable response for the Agreement Level calculation.
2. **Agreement Level, not accuracy** — the percentage reflects how many models converged, not a claim about which suggestion is objectively "correct." Editorial/creative choices don't have a single correct answer.
3. **Human Control is mandatory** — the Telegram report always ends with a required Final Decision step; the council only informs the operator's choice, it never finalizes it.
4. **Graceful fallback (important for reliability):** If any one of the three models fails to respond (API error, timeout, rate limit), the council proceeds with the remaining models rather than blocking the pipeline — Agreement Level is then calculated out of 2 (or even 1, with a note that no comparison was possible). This is applied via the existing retry policy (Section 14.2) before falling back, so a single hiccup never stalls video production.

**Note on load:** This adds text-only API calls (not rendering/compute load), so it does not slow down FFmpeg rendering or uploads. The trade-off is: (a) more free-tier quota used per video (3 calls instead of 1), and (b) more external dependencies that could each individually fail — mitigated by the fallback rule above.

---



## 10. Additional Platforms (Beyond YouTube)

**Verification disclaimer:** Platform-specific automation depends on the current official API capabilities and available developer permissions at the time of implementation. TikTok, Instagram, Threads, Pinterest, and Meta APIs frequently change permissions and publishing capabilities — nothing in this table should be assumed to work until checked against that platform's current official developer documentation immediately before building the corresponding module. Also note: <cite index="6-1">the YouTube Data API v3 free tier grants 10,000 quota units/day, which caps uploads at roughly 6 videos/day</cite> — worth tracking if the Urdu channel later shares GitHub Actions/quota infrastructure with the English channel.

| Platform | Free API? | Setup Requirement |
|---|---|---|
| Facebook | Yes (Meta Graph API) | Facebook Page (not personal profile) |
| Instagram | Yes (Meta Graph API) | Business/Creator account, linked to a Facebook Page |
| Threads | Yes | Uses the same Instagram Business account link |
| TikTok | Yes, but developer approval takes time | Standard developer account |
| Pinterest | Yes | Standard developer account |
| X (Twitter) | Very limited free tier (paid beyond a small monthly quota) | Recommended to skip for now — doesn't fit the "fully free" requirement |

Facebook + Instagram + Threads can be set up together in one session since they're linked under one Meta Business setup. Each platform gets its own upload module (`facebook_upload.py`, `instagram_upload.py`, etc.), but all route through the same Telegram approval gate.

## 11. Thumbnail & Social Post Generation

Auto-generated per video, using the same Pillow pipeline as text rendering:

- **YouTube thumbnail** (1280x720): background frame + Surah name + short attractive text excerpt, fixed brand template (consistent font/color/logo across all videos)
- **Instagram/Facebook post image** (1080x1080 or 1080x1350): same template, resized
- **YouTube Community post text**: No official YouTube Data API v3 endpoint for creating Community Posts was found upon verification (as of this writing, none of Google's documented YouTube APIs — Data API v3, Analytics, Reporting, Live Streaming — expose a community-post-creation resource). **Treat this as a manual task**: Gemini can still draft the post text per video, but the operator posts it manually via YouTube Studio unless/until an official API is confirmed to support it. Re-verify against current developer.google.com/youtube docs before attempting automation here.
- **Instagram/Facebook caption + hashtags**: generated via Gemini per video, pulled partly from a fixed relevant-hashtag list

All of these render alongside the video and appear together in the single Telegram preview — one Approve covers the video, thumbnail, and social posts as a set.

### 11.1 Multi-Language Titles & Descriptions (YouTube Localizations)

YouTube's Data API supports setting localized title/description per video for multiple languages — viewers see the title in their own YouTube app language automatically, without needing separate uploads per language.

**Process:**
1. Gemini generates the primary (e.g., English) title/description as usual
2. Gemini translates title/description into a fixed set of target languages — recommended starting set: Urdu, Arabic, Hindi, Indonesian/Malay, French (covers the largest Muslim-majority and general global audiences)
3. All language versions are set on the video via the YouTube API's `localizations` field in a single update call
4. Telegram preview shows all generated language versions before approval, alongside the video/thumbnail

**New module:** `modules/localize_metadata.py` — handles the Gemini translation calls and the YouTube API localization update.

This is metadata-only (title/description text) and does not require re-rendering the video itself, so it adds negligible cost or processing time.

### 11.2 Multi-Language Subtitles/Captions

Separate from title/description localization (11.1) — this adds selectable CC/subtitle tracks in multiple languages, so a viewer can turn on captions in their own language regardless of what's shown on-screen by default.

**Process:**
1. The ayah translation text (already generated for on-screen display) is translated by Gemini into additional target languages (e.g., Urdu, Hindi, Arabic, French, Indonesian, Spanish)
2. Each language's text is assembled into a timed `.srt` subtitle file, using the same ayah timing already computed for on-screen text sync
3. All subtitle files are uploaded to the video via the YouTube API (`captions.insert`)
4. Viewers select their preferred language from the CC menu independent of the video's default on-screen text

**New module:** `modules/generate_subtitles.py` — builds `.srt` files per language and uploads them via the captions API.

## 12. Video Quality Standards (Professional / Premium Look)

| Factor | Standard Used |
|---|---|
| Resolution | Source clips in 4K, exported at 1080p |
| Bitrate | 8–10 Mbps target (avoids blurry/compressed look) |
| Text rendering | Clean Arabic typeface (e.g., Amiri/Scheherazade) + subtle shadow for readability over video |
| Color grading | Light contrast/saturation correction via FFmpeg filters (avoids flat/dull look) |
| Transitions | Soft fade between ayah text changes, not hard cuts |
| Audio | Normalized loudness across all videos (consistent volume, no clipping/noise) |

These settings are fixed once in the render pipeline and apply automatically to every video.

## 12.1 Audio & On-Screen Text Format — Shorts vs Long Videos

**Shorts (per channel language, e.g. English channel):**
- **Audio/voice:** Only the translation is spoken (TTS narration in the channel's language — e.g., English). No Arabic recitation audio in Shorts.
- **On-screen text:**
  - Top: reference line (Surah name + ayah number/range)
  - Arabic text (written, not spoken)
  - Below: translation text in the channel's language (English for the English channel, Urdu for the Urdu channel)

**Long Videos — Two Versions Per Surah:**
1. **Arabic-only version:** Arabic Qari'at (recitation) audio only. On-screen: Arabic text only, no translation.
2. **Arabic + Translation version (bilingual alternating narration):** For each ayah — Arabic recitation audio plays first, then the translation is spoken aloud in the channel's language (English/Urdu) immediately after, before moving to the next ayah. Pattern: [Ayah 1 Arabic audio] → [Ayah 1 translation audio] → [Ayah 2 Arabic audio] → [Ayah 2 translation audio] → ... On-screen throughout: reference line (Surah + ayah number), Arabic text on top, translation text below — both visible for the full duration that ayah is being covered (both the Arabic and translation audio segments).

This means the rendering pipeline needs: (a) a recitation-only audio track for version 1, and (b) an interleaved dual-audio track for version 2 that stitches Arabic recitation clips with translation-narration (TTS) clips per ayah, with on-screen text synced to cover both audio segments per ayah — rather than one fixed format for every video.

## 13. Build Order (Suggested)

1. `fetch_text.py` — Quran text + translation retrieval, the foundation
2. `integrity_check.py` — cross-source verification before anything else trusts the text
3. `state_manager.py` — checkpoint/recovery system (Section 16.5), built early so every subsequent module writes state correctly from the start rather than being retrofitted later
4. `main.py` — orchestrator skeleton that calls modules in sequence, with `DRY_RUN` support (Section 16.3) from day one
5. `fetch_audio.py` + license registry — build and manually populate with your first 2-3 verified reciters
6. `render_text.py` — Arabic/translation text overlay rendering
7. `render_video.py` — get one full Short (Surah Al-Ikhlas) rendering correctly end-to-end, manually
8. `telegram_bot.py` — approval loop (Section 6) plus the two-message verification workflow (Section 14.14), working on that one manual video
9. `ai_council.py` — add Gemini/Groq/DeepSeek-generated titles/tags/thumbnail concepts once the core render pipeline is proven
10. Upload modules (`youtube_upload.py` first, others per Section 10) + Content ID manual-check step (Section 3, Step 12) — test with the Unlisted-first safety step
11. `watchdog` module (Section 14.11) — wire in monitoring once there's a running pipeline to monitor
12. Database + logging — wired in throughout steps 1-11, not bolted on at the end
13. Stress testing (Section 14.13) — progressively work through the test list; early real videos double as launch content per that section's note
14. GitHub Actions scheduling — automate only after the above work reliably by hand
15. Urdu channel — clone the proven English pipeline, swap translation source and language config

Once steps 1-13 pass reliably, Version 3.1 of this architecture is considered **Production Ready** and is frozen per Section 16.6 — further changes become v3.2+.

---

## 14. Production Hardening (Reliability & Maintainability)

### 14.1 Configuration & Dependency Versioning
Every rendered video's database record includes:
- `config_version`, `pipeline_version`, `render_engine_version`

Dependencies are locked and documented in `requirements.txt` (Python version, FFmpeg version, Pillow version, and MoviePy version if used) so future updates don't silently break the pipeline.

### 14.2 Retry Policy
Every external API call (Quran text, audio, background clips, Gemini/ChatGPT, YouTube upload) follows:
```
Attempt → fail → Retry 1 → fail → Retry 2 → fail → Retry 3 → fail → Telegram Alert (manual intervention)
```

### 14.3 Operational Monitoring via Telegram
Beyond video approvals, the bot also alerts on:
- API request failures
- Rendering failures
- Upload failures
- Database errors
- Storage running low
- GitHub Actions workflow failures

### 14.4 Daily Health Report
An automatic daily summary sent via Telegram:
- Database status
- Disk usage
- Pending jobs / failed jobs
- API availability check
- Last successful upload timestamp

### 14.5 Duplicate Video Protection
Before rendering, the system checks the database for an existing record with the same Surah/Ayah combination for that channel. If found, rendering is skipped and the operator is notified — prevents accidental re-uploads.

### 14.6 Translation Revision Tracking
In addition to `translation_source`, the database stores `translation_revision` — so if a translation provider updates their text later, historical videos remain traceable to the exact version used at the time.

### 14.7 Render Cache & Chunked Rendering

If a render fails partway through, the system reuses already-generated text overlay frames instead of re-rendering everything from scratch — saves processing time and CPU on retries.

**Chunked rendering rule (VM memory constraint handling):** When a Long video render would exceed the VM's available memory/storage (e.g., the free-tier VM's ~1GB limits), the system splits the render into more, smaller segments — it does **not** lower video quality/bitrate to make it fit. Quality (Section 12: 1080p, 8-10 Mbps target) is treated as fixed; the number of chunks is the flexible variable. If 5-6 chunks still strain memory, the system increases to 10-12 (or more) smaller chunks rather than compressing quality down to relieve memory pressure.

**Seamless joining requirement (no visible/audible seam at chunk boundaries):**
- Every chunk is rendered with **identical encoding parameters** — same resolution, frame rate, bitrate, codec, and keyframe interval. Any mismatch between chunks is what causes a visible jump/flicker or audio glitch at the join point.
- Chunks are joined using FFmpeg's concat demuxer in **stream-copy mode** (no re-encoding at join time) — re-encoding at the seam is what typically introduces a detectable quality shift; stream-copy concatenation of identically-encoded chunks produces a mathematically seamless join.
- Chunk boundaries are only placed at points where an ayah's audio and on-screen text have fully completed — never mid-ayah, mid-word, or mid-audio-clip — so there is a natural pause at the join rather than a cut felt mid-sentence.
- Before finalizing, the render pipeline does an automated check across each join point (e.g., comparing the last frame of one chunk and first frame of the next for resolution/format consistency) as a safeguard, in addition to the identical-parameters rule above.

### 14.8 SQLite Transaction Recovery
Recovery covers both `pipeline_state.json` (overall pipeline progress) and SQLite itself: if the database crashes mid-write, the system must detect the incomplete transaction on restart and roll back safely — never leaving a duplicate or corrupted `videos` record. SQLite's built-in WAL (Write-Ahead Logging) mode is used for this, which is the standard way to get crash-safe writes in SQLite.

### 14.9 Idempotency Protection
Every upload job is assigned a globally unique Job ID (stored in the `videos` table). Before any upload call, the pipeline checks whether that Job ID has already been marked "uploaded" — if the pipeline restarts after a partial failure, the same video is never uploaded twice.

### 14.10 Backup Verification
Creating backups alone isn't sufficient — the system periodically performs an actual test-restore of a backup (e.g., monthly) to confirm the backup is genuinely usable, not just that a backup file exists.

### 14.11 Enhanced Watchdog Monitoring
Beyond basic alerts (Section 14.3), the Watchdog continuously tracks:
- GitHub Actions execution age (has a scheduled run silently stopped firing?)
- Disk usage
- Database integrity
- Telegram Bot availability
- External API quota status (especially YouTube's 6-uploads/day cap and each AI Council model's free-tier limits)
- Last successful upload timestamp
- Last successful backup timestamp
- Pipeline heartbeat (a periodic "I'm alive" signal from the running pipeline)

### 14.12 Expanded Configuration Versioning
Every rendered video permanently records, in addition to the fields in 14.1:
- `ai_council_version`
- `metadata_version`

This, together with `config_version`, `pipeline_version`, and `render_engine_version`, gives complete traceability for any published video.

### 14.13 Stress Testing Before Full Automation
Before relying on the system fully hands-off, test beyond normal functional testing — but scaled sensibly for a solo operator rather than all at once:
- Start with 5-10 real end-to-end test videos to prove the pipeline genuinely works, then scale toward a fuller pre-launch stress pass of 30 Shorts + 10 Long videos (covering the priority Surahs from Content_Selection_Lists.md) before beginning fully unattended production publishing
- **Important:** these test videos should not be throwaway/dummy content — since the Telegram approval gate is always active, each one can be a real, correctly-produced video of an actual priority Surah. This means the stress-testing phase doubles as the channel's actual launch content rather than being wasted effort, reconciling thorough pre-launch testing with not wanting to delay real publishing
- Once stable, add: consecutive daily runs, a simulated API failure, a simulated VM/server restart, a simulated network interruption, a simulated disk-full condition, a simulated API quota exceeded, a simulated Telegram-offline period, and a simulated GitHub Actions failure
- Treat this as a gradual hardening process alongside real usage — the full stress-test list above should be worked through before switching off manual oversight of infrastructure health (not before the very first video is ever published, since human approval already gates every publish)

## 14.14 Telegram Manual Verification Workflow (ChatGPT Cross-Check, No API Needed)

To add quality assurance without requiring the paid ChatGPT API, the Telegram bot always sends **two separate messages** per video, so the operator can manually copy the second into ChatGPT (or any other AI) for a free second opinion before approving.

**Message 1 — Preview** (visual inspection only):
- Video preview/thumbnail
- Fazilat (virtue) card, if a Verified hadith is attached

**Message 2 — Copy-Friendly Verification Block** (plain text only — never sent as an image, caption, PDF, or JSON, so it can be selected and copied with one tap):
```
Channel: [English/Urdu]
Video Type: [Short/Long]
Surah: [name]
Ayah Range: [range]
Complete Arabic Text: [full text]
Complete Translation: [full text]
Translation Source: [translator name]
Hadith Reference: [reference, if any]
Hadith Grade: [grade, if any]
Audio Reciter: [name]
Audio License Status: [Verified/Pending]
AI-Generated Title: [title]
Description: [description]
Tags: [tags]
Thumbnail Concept: [concept]
Background Source: [clip source]
AI Council Summary: [summary]
Agreement Level: [%]
```

The bot follows this with a ready-to-paste review prompt asking the operator's chosen AI to evaluate: Quran text presentation, translation formatting, hadith reference accuracy, SEO quality, description, tags, thumbnail concept, and overall presentation quality.

**Critical rule:** Any response from ChatGPT (or another AI) obtained this way is **advisory only**. The Telegram Approve/Reject/Re-render/Edit/Schedule buttons remain the sole publishing gate — nothing is ever auto-published based on an external AI's manual-verification response. This keeps the system fully free while adding a genuine extra quality layer.

## 16. Pre-Coding Engineering Standards (v3.1 Additions)

### 16.1 Global Error Code System
Every module returns a standardized error code on failure, logged via `logger.py`:
```
E001 = Quran Fetch Failed
E002 = Integrity Check Failed
E003 = Audio License Missing
E004 = Render Failed
E005 = Upload Failed
E006 = Telegram Offline
E007 = AI Council Service Unreachable
E008 = Database Write Failed
E009 = Content ID Manual Check Pending (see Section 3, Step 12)
E010 = Manual Request Verification Failed (see Section 18)
```
(Extend this list as new failure modes are identified during development — codes should never be reused for a different meaning once assigned.)

### 16.2 Standardized Module Interface
Every module in `modules/` follows the same shape, so any one module can be replaced or upgraded without touching the rest of the pipeline:
```
Input → Process → Output → Status (success/error code) → Log
```

### 16.3 Dry Run Mode
A config flag, `DRY_RUN = true/false` in `config/channels.yaml` (or a dedicated `config/pipeline.yaml`):
- When `true`: the full pipeline runs (fetch, render, AI Council, Telegram preview) but **no upload and no public publishing occurs** — useful for testing changes safely
- When `false`: normal production behavior

### 16.4 Central Configuration (No Hardcoded Values)
The following must live in config files only, never hardcoded in module code:
- Render resolution, fonts, frame rate, bitrate
- Languages / translation sources per channel
- Upload schedule (times, days)
- Background clip duration rules
- Retry limits and timeouts
- DRY_RUN flag

### 16.5 State Manager Discipline
Every pipeline stage writes to `pipeline_state.json` both **before** starting and **after** completing that stage — not just on success. This ensures that after any crash, the exact stage the pipeline was in is always recoverable (see also Section 14.8 for the SQLite side of this).

### 16.6 Documentation Freeze
This document is frozen as **Version 3.1** once coding begins. Any further architectural changes during development become **Version 3.2**, then **4.0**, etc. — the architecture should not be casually re-opened mid-build unless a critical issue is discovered; incremental learnings get queued as versioned amendments instead.

## 18. Telegram Manual Content Request Module (v3.2)

Beyond the scheduled Shorts/Long pipeline (Section 3), the operator can initiate a video on-demand by sending a request directly to the Telegram bot — for example, a specific ayah, a hadith reference, a topic idea, or a request to regenerate existing content.

**New module:** `modules/content_request_handler.py`

### Supported Request Types
- Quran Ayah reference (e.g., "Surah Al-Mulk 1-5")
- Hadith reference
- Verified Fazilat Card
- Topic suggestion
- Existing content (for regeneration)

### Workflow
```
Telegram Message (operator-initiated)
        │
        ▼
Identify Request Type
        │
        ├── Quran Ayah
        ├── Hadith
        ├── Fazilat Card
        ├── Topic Request
        └── Existing Content (regeneration)
        │
        ▼
Verification (Trusted Sources Only — see rules below)
        │
        ▼
AI Council (Creative Decisions Only — Section 9.1 scope applies unchanged)
        │
        ▼
Generate Preview
        │
        ▼
Telegram Approval (Section 6 / 14.14 flow — unchanged)
        │
        ▼
Upload to Selected Platform(s)
```

### Verification Rules
The AI Council must **never** verify or authenticate: Quranic Arabic text, Quran translations, Hadith authenticity, Tafsir, or Islamic rulings — this is unchanged from Sections 1, 9, and 9.1, and applies identically to manually-requested content as it does to the scheduled pipeline.

Verification always uses trusted sources only:
- **Quran** → Quran.com + Tanzil integrity check (Section 1, `fetch_text.py` + `integrity_check.py`)
- **Translation** → the channel's selected, fixed verified translator (Section "Translation policy")
- **Hadith** → Verified entries only from `hadith_registry` (Section 5, 16's status rules)
- **Fazilat** → Verified registry entries only

**If verification fails, the pipeline stops immediately** and sends a Telegram notification rather than proceeding with unverified content:
```
❌ Verification Failed

Reason: The requested Hadith/Fazilat is not available in the verified registry.

Action Required: Please verify it manually or add it to the verified registry 
before rendering.
```
This maps to error code **E010 = Manual Request Verification Failed** (extending the error code list in Section 16.1).

### AI Council Responsibilities (After Successful Verification Only)
The AI Council may assist only with creative/editorial decisions, identical in scope to Section 9.1:
- Long vs. Short format recommendation
- Background style
- Motion effects
- Thumbnail concept
- Title, description, tags, hashtags
- Platform recommendation (which of the connected platforms suit this content)

The AI Council must never modify or validate any Islamic source — this restriction is identical to and inherits from Section 9.1's scope rules, not a separate policy.

### Final Principle
Human approval remains mandatory before rendering or publishing. No video, image, or social media post generated via a manual content request may be published without Telegram approval — the same gate that governs the scheduled pipeline (Section 6, 14.14) governs this module too, with no shortcut for operator-initiated requests.



- Word-by-word highlighting (see Section 4)
- Multi-language expansion beyond English/Urdu
- Analytics dashboard (views, engagement per Surah/reciter)
- Playlist automation (auto-organize by Surah/Juz)
- Performance reporting (which content types perform best)
- Multiple branding themes/templates to A/B test

---

*This document reflects a synthesis of recommendations across Claude, Gemini, and ChatGPT reviews, resolved where they disagreed (notably: AI must not touch Quranic Arabic text; audio licensing must be explicitly verified and logged, never assumed).*
