# Surah & Ayat List — With Hadith References (For Cross-Check)

# Surah & Ayat List — With Hadith References (For Cross-Check)

## Database Structure (To Be Used in `license_registry`-style table: `hadith_registry`)

Each entry stores:
- `content_type` — 'quran_only' / 'hadith_fadail' (virtue narration) / 'scholarly_opinion' / 'common_practice'
- `quran_reference` — Surah:Ayah
- `hadith_reference` — Book, chapter, hadith number
- `hadith_grade_primary` — Sahih / Hasan / Da'if / Mawdu' (fabricated)
- `graded_by_primary` — scholar/source who gave that grading
- `hadith_grade_secondary` — if scholars differ, the alternate grading
- `graded_by_secondary`
- `notes` — any dispute or nuance
- `source_url`
- `date_verified`
- `verified_by`
- `verification_status` — one of: **Verified / Needs Review / Disputed / Weak / Excluded**

**Rendering rule:** The video-generation pipeline only pulls entries marked **Verified**. Needs Review / Disputed / Weak / Excluded entries are never auto-included — Disputed and Weak entries additionally require explicit manual approval if an operator wants to include them at all (with the on-screen wording clearly noting "scholars have differed on this narration's authenticity"), per the editorial policy below.

**Editorial policy:** Never state a virtue as an established fact unless the supporting narration is reliably authenticated. Where authenticity is disputed, either omit the narration entirely, or clearly state on-screen that scholars have differed regarding its authenticity — never present it as settled.

**Shorts policy:** Not every ayah needs a paired hadith. If no authentic, directly-relevant hadith exists for a selected verse, the Short presents the Quranic verse alone — a hadith is never forced onto a verse just to fill a template.

---

## Part A: Long Video Surahs — Hadith References

### 1. Al-Mulk
- **content_type:** hadith_fadail
- **Virtue:** Intercedes for its reciter until forgiven; protects from punishment of the grave
- **Entry 1:**
  - hadith_reference: Sunan al-Tirmidhi 2891 (also Abu Dawud 1400, Ibn Majah 3786)
  - hadith_grade_primary: Hasan — graded_by_primary: al-Tirmidhi
  - hadith_grade_secondary: Sahih — graded_by_secondary: Ibn Taymiyyah (Majmu' al-Fatawa 22/277); also al-Albani (Sahih Ibn Majah 3053)
  - verification_status: **Verified**
- **Entry 2 (tent-on-grave narration):**
  - hadith_reference: Sunan al-Tirmidhi 2890
  - hadith_grade_primary: disputed — some editions record "Hasan Gharib," others note chain weakness
  - notes: Recommend using only Entry 1 in video content; Entry 2's grading is inconsistent across sources
  - verification_status: **Disputed** — excluded from default rendering pool

### 2. Al-Kahf
- **content_type:** hadith_fadail
- **Entry 1 — Light between two Fridays:**
  - hadith_reference: via al-Hakim al-Mustadrak 2/399; al-Bayhaqi 3/249; also al-Darimi 3407
  - hadith_grade_primary: Sahih — graded_by_primary: al-Albani (Sahih al-Jami' 6471)
  - verification_status: **Verified**
- **Entry 2 — Protection from Dajjal (first/last 10 verses):**
  - hadith_reference: Sahih Muslim; also Sunan al-Tirmidhi
  - hadith_grade_primary: Sahih (Muslim) / Hasan Sahih (Tirmidhi)
  - verification_status: **Verified**
  - notes: Exact Sahih Muslim hadith number to be confirmed before database entry is finalized

### 3. Ayat al-Kursi (2:255) — status: Needs Review
- content_type: hadith_fadail
- Commonly cited virtue: protection when recited before sleep — narration involves Abu Hurairah, referenced in the chain of Sahih al-Bukhari discussions
- notes: exact Bukhari book/hadith number must be confirmed before use

### 4. Yaseen — status: Weak
- content_type: hadith_fadail
- Commonly cited as "the heart of the Quran"
- notes: widely discussed as da'if (weak) by multiple hadith scholars — **recommend Excluded status** unless a scholar identifies a specific authenticated wording

### 5. Ar-Rahman — status: Needs Review
- content_type: hadith_fadail
- No specific narration researched yet

### 6. As-Sajdah + Al-Insan — status: Needs Review
- content_type: hadith_fadail
- Commonly cited: Prophet ﷺ recited these two Surahs in Fajr prayer on Fridays — well-attested pattern in Sahih al-Bukhari and Sahih Muslim
- notes: exact reference numbers to be confirmed

### 7. Al-Waqiah — status: Weak
- content_type: hadith_fadail
- Commonly cited virtue relating to protection from poverty
- notes: widely considered weak/fabricated by hadith scholars — **recommend Excluded status**

### 8. Al-Muzzammil — status: Needs Review
- content_type: hadith_fadail
- No specific narration researched yet

### 9. Ad-Duha + Ash-Sharh — status: Needs Review
- content_type: hadith_fadail
- No specific narration researched yet

### 10. Al-Ikhlas, Al-Falaq, An-Nas, Al-Kafirun — status: Needs Review
- content_type: hadith_fadail
- Al-Ikhlas: commonly cited as "equal to one-third of the Quran" — well-attested in Sahih al-Bukhari, exact reference number to be confirmed
- Al-Falaq + An-Nas (Al-Mu'awwidhatayn): recitation before sleep — well-attested in Sahih al-Bukhari, exact reference number to be confirmed

**Note:** All Surahs can also be presented as **content_type: quran_only** (verse text/audio with no accompanying hadith) at any time — this is always a safe default, per the Shorts Policy above, and applies equally to Long videos where no Verified hadith exists yet.

---

## Part B: Shorts — Ayat + Hadith Pairing Template

| Category | Ayat/Surah | content_type | Hadith Reference | Grade | Status |
|---|---|---|---|---|---|
| Hope & Mercy | (to be selected) | quran_only or hadith_fadail | | | ⬜ Pending |
| Dua (Prophets') | (to be selected) | quran_only or hadith_fadail | | | ⬜ Pending |
| Stories | (to be selected) | quran_only or hadith_fadail | | | ⬜ Pending |
| Reflection/Signs of Allah | (to be selected) | quran_only or hadith_fadail | | | ⬜ Pending |

*Default to quran_only unless a Verified hadith is directly and clearly relevant — never force a pairing.*

---

## Recommended Next Step

Please have Gemini/ChatGPT:
1. Confirm the two **Verified** entries (Al-Mulk Entry 1, Al-Kahf Entries 1 & 2) are accurately transcribed, and supply the missing exact reference numbers (Sahih Muslim hadith number for Al-Kahf Entry 2)
2. Research and grade the **Needs Review** entries (Ayat al-Kursi, Ar-Rahman, As-Sajdah+Al-Insan, Al-Muzzammil, Ad-Duha+Ash-Sharh, the four short Surahs)
3. Confirm whether Yaseen and Al-Waqiah entries should remain **Weak/Excluded** or if a specific authenticated wording exists that changes that assessment
4. Flag any additional commonly-circulated-but-weak narrations relevant to this project's Surah list

