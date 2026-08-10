# Prompt Library — Quran Automation System

Reusable prompt templates for each AI involved in this project. Fill in the bracketed placeholders per use.

---

## Claude — Module Coding Template

```
I'm building [module_name].py for the Quran Automation System.

Reference: System_Architecture.md Section [X] defines this module's spec.

Requirements:
- [specific requirement 1]
- [specific requirement 2]

Constraints (always apply):
- Follow the module interface: Input → Process → Output → Status → Log
- No hardcoded values — use config files
- If this module touches Quran text, translation, or Hadith: it must NEVER 
  generate/modify/validate that content — only fetch from trusted sources and 
  pass through integrity checks
- Respect DRY_RUN mode if this module touches upload/publish actions
- Use the Global Error Code system (Section 16.1) for failure states

Please write this module now, and note any assumptions you're making.
```

---

## ChatGPT — Architecture/Module Review Template

```
Please review the following [architecture section / module code] for the 
Quran Automation System.

[paste content here]

Please check for:
- Technical accuracy (flag anything that assumes an API capability without 
  verification)
- Consistency with the project's core rule: AI never touches Quranic text, 
  translations, Hadith authenticity, or Tafsir
- Production-readiness gaps (error handling, edge cases, security)
- Anything missing that a senior engineer would flag before this goes to production

Please give a final assessment and a clear list of recommended changes.
```

---

## AI Council Template (Gemini / Groq / DeepSeek) — Creative/Editorial Only

**Scope reminder:** This template is ONLY for thumbnail concepts, background style, 
motion effects, color scheme, titles, descriptions, tags/SEO. Never send Quran text, 
translations, or Hadith content through this template for "verification" — the AI 
Council does not verify Islamic content, ever.

```
You are one of three AI models being consulted for a creative/editorial decision 
on a Quran-content video (English/Urdu channel — nature-themed background, 
respectful tone, no clickbait).

Context: [Surah name, ayah range, category — e.g. "Hope & Mercy"]

Task: [e.g. "Suggest a YouTube Shorts title, 3-5 tags, and a one-line thumbnail 
concept"]

Respond ONLY in this exact JSON format, no other text:
{
  "title": "...",
  "reason": "...",
  "confidence": 0-100,
  "pros": ["...", "..."],
  "cons": ["...", "..."]
}
```

---

## Telegram Manual Verification — Copy-Paste Review Prompt

(This is the prompt the operator manually pastes into ChatGPT or another AI 
alongside the copy-friendly verification block from System_Architecture.md 
Section 14.14)

```
Please review this Quran video's details before I approve it for publishing.

[paste the copy-friendly verification block here]

Please evaluate:
- Quran text presentation
- Translation formatting
- Hadith reference accuracy (if any)
- SEO quality (title/tags/description)
- Thumbnail concept
- Overall presentation quality

Your feedback is advisory only — I make the final publish decision myself.
```

---

*Add new templates here as new recurring prompt patterns emerge during development. Keep each template scoped to exactly one AI and one purpose — don't blend Islamic-content verification into any AI Council or general-purpose template.*
