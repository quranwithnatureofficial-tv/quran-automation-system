# AI Development SOP — Quran Automation System (v2.0)

**Status:** Active
**Companion Document:** `System_Architecture.md` (the Master Context / single source of truth for design — see Section 0 below)
**Principle:** System_Architecture.md is the single source of truth for *what* the system does. This SOP governs *how* development happens day-to-day. No AI may redesign the architecture without explicit Human approval.

*(v2.0 note: this version merges and supersedes the original AI_Development_SOP v1.0, incorporating the AI Collaboration Workflow. There is no separate "MASTER_CONTEXT" file — `System_Architecture.md` fills that role, to avoid maintaining two competing sources of truth.)*

---

## 0. Core Philosophy

- Multiple AIs collaborate, but **none have final authority**.
- Every piece of work passes through Review → Test → Human Approval.
- No AI writes directly to production.
- The Human (Project Architect) is the only one who merges to `main` and deploys.
- The "Don't Redesign Without Approval" rule is mandatory for all AIs.
- No AI — regardless of role — ever generates, modifies, or validates Quranic Arabic text, translations, Hadith authenticity, Tafsir, or Islamic rulings. This inherits directly from `System_Architecture.md` Sections 1, 9, 9.1, and 18, and is never superseded by anything in this SOP.

---

## 1. AI Role Matrix

| Role | Preferred AI(s) | Primary Responsibilities | Secondary |
|---|---|---|---|
| **Project Manager / Orchestrator** | Human + Claude | Task breakdown, prioritization, handoffs, final integration | - |
| **Research AI** | Grok, Perplexity, Gemini | Free-tier limits, API changes, new tools, best practices | DeepSeek |
| **Primary Coder** | Claude | Writing clean, production-grade Python modules | - |
| **Code Reviewer** | ChatGPT | Bugs, logic errors, security issues, performance | Claude |
| **Complex Logic / Edge Cases** | DeepSeek | Algorithms, state management, retry logic, data pipelines | - |
| **Islamic / Content Verification** | Claude + Human | Hadith grading check, translation accuracy, sensitive content — verification against trusted sources only, never AI judgment alone | - |
| **Security Reviewer** | ChatGPT + Claude | Secrets handling, injection risks, token exposure | - |
| **Testing AI** | ChatGPT / DeepSeek | Test cases, edge cases, failure scenarios | - |
| **Creative / Metadata (AI Council)** | Gemini, Groq, DeepSeek | Titles, descriptions, tags, thumbnail concepts — per System_Architecture.md Section 9.1 scope only | - |
| **Quick Fixes / Alternatives** | Grok | Fast debugging, alternative approaches | - |

**Note on Creative/Metadata role:** This maps directly to the AI Council defined in `System_Architecture.md` Section 9.1 (Gemini/Groq/DeepSeek, strict JSON format, Agreement Level, human approval mandatory) — it is not a separate or looser process, just this table's summary of that section.

---

## 2. Official Task Status Flow

```
TODO → IN PROGRESS → REVIEW → TEST → APPROVED → DONE
```
- Only the Human can move a task to **APPROVED** or **DONE**.
- Any AI can move a task to IN PROGRESS or request REVIEW.
- Failed REVIEW or TEST sends the task back to IN PROGRESS with comments attached.

---

## 3. AI-to-AI Handoff Format (Mandatory)

Every AI delivers output in this exact format when handing off work:

```markdown
### HANDOFF

**From:** [AI Name]
**To:** [Next AI / Human]
**Task ID:** [e.g. MOD-003-fetch-text]
**Status:** IN PROGRESS / REVIEW / TEST

**What I did:**
- ...

**Files Changed / Created:**
- modules/fetch_text.py
- ...

**Key Decisions:**
- ...

**Open Questions / Risks:**
- ...

**Next Recommended Action:**
- ...
```

---

## 4. Standard Prompt Templates

*(See also `Prompt_Library.md` for the full, maintained set — this section defines the core two used most often in the dev cycle.)*

### 4.1 Coding Prompt (Claude)
```
Master Context: follow System_Architecture.md [cite relevant section number(s)]

Task: Write the complete module `modules/xxx.py`

Requirements:
- Follow exact folder structure and naming from System_Architecture.md Section 2
- Include proper logging via logger.py (Section 8, error codes per Section 16.1)
- Include checkpoint support via state_manager.py (Section 16.5)
- No AI touching Quran text, translation, or Hadith authenticity — ever
- Full error handling + retries as defined in Section 14.2
- Type hints + docstrings

Output only the complete file content.
```

### 4.2 Code Review Prompt (ChatGPT)
```
Review the following code against System_Architecture.md.

Check for:
1. Bugs & logical errors
2. Security issues (secrets, injection)
3. Deviation from the architecture
4. Performance problems
5. Missing error handling / checkpoints

Provide:
- Critical issues
- Suggested improvements (with exact code)
- Overall score (1-10)
```

### 4.3 Research Prompt (Grok / Perplexity / Gemini)
```
Research the current (2026) free-tier limits and reliability of [tool].
Focus only on information relevant to the Quran Automation System.
Distinguish verified facts from assumptions — cite sources.
```

---

## 5. Bug Report Format

```markdown
**Bug ID:** BUG-001
**Severity:** Critical / High / Medium / Low
**Module:**
**Steps to Reproduce:**
**Expected:**
**Actual:**
**Logs / Screenshots:**
**Suggested Fix (optional):**
```

---

## 6. Feature Request Format

```markdown
**Feature ID:** FEAT-001
**Priority:** Critical / High / Medium / Low
**Description:**
**Why needed:**
**Does it break existing architecture?** Yes/No
**Proposed solution:**
```

---

## 7. Architecture Change Rule

No AI may change the architecture unilaterally.

If an AI believes a change is needed:
1. It writes a detailed proposal: Problem → Impact → Proposed Solution → Complexity
2. The Human approves
3. `System_Architecture.md` is updated (version bump per Section 16.6 — e.g., v3.2 → v3.3)
4. Only then does implementation begin

This is the same Documentation Freeze policy already defined in `System_Architecture.md` Section 16.6 — this section restates it here because it governs day-to-day AI behavior, not because it's a separate rule.

---

## 8. GitHub Workflow

- `main` branch → Production (only Human merges)
- Feature branches: `feature/MOD-003-fetch-text`
- Every PR gets at least one AI review + Human final approval
- Conventional commit messages preferred (builds on the format already defined in the original SOP's Git Rules)
- `.gitignore` must continue to exclude secrets/`.env`/`data/` per the existing rule

---

## 9. Conflict Resolution

1. **Technical disagreement** → get analysis from DeepSeek + ChatGPT both, compare
2. **Architecture disagreement** → Human makes the final call
3. **Islamic/content disagreement** → Human + trusted external sources only — never resolved by AI vote or majority opinion (this is the same principle as the AI Council's exclusion from Islamic content, System_Architecture.md Section 9.1)

---

## 10. Example Full Cycle

```
System_Architecture.md (Master Context)
         │
         ▼
   PROJECT MANAGER (Human + Claude)
         │
    ┌────┼────┐
    ▼    ▼    ▼
Research Coding Content Verification
    │    │         │
    └────┼─────────┘
         ▼
    REVIEW AI (ChatGPT)
         │
         ▼
    TESTING AI
         │
         ▼
   HUMAN APPROVAL
         │
         ▼
   GitHub main branch
         │
         ▼
     PRODUCTION
```

---

## 11. Final Authority

- `System_Architecture.md` = Law
- Human (Project Architect) = Supreme Court
- All AIs = Specialized Engineers

No AI can change production directly.

---

## 12. Module Development Cycle (from original SOP — still applies)

For each module in the Build Order (`System_Architecture.md` Section 13):
```
1. Confirm the module's spec against System_Architecture.md
2. Assign per the AI Role Matrix (Section 1 above) — Claude codes, ChatGPT reviews, etc.
3. Follow the Handoff Format (Section 3) between each AI involved
4. Test against a real case, not a mock, where feasible
5. If it touches Islamic content handling: verify against trusted sources, never AI-only
6. If it's creative/editorial: route through the AI Council (Section 9.1)
7. Commit to GitHub with a clear message
8. Update Roadmap.md checklist
9. Log any issues/decisions in Developer_Notes.md
```

---

## 13. Quality & Coding Rules (from original SOP — still applies)

- No Quran/Hadith content ships without passing verification (System_Architecture.md Sections 1, 5, 16, 18)
- No module marked "done" without at least one real end-to-end test run
- Any third-party API capability claim must be verified against current official docs before being written as fact
- Python, module interface standard: Input → Process → Output → Status → Log
- No hardcoded values — config files only
- `DRY_RUN` mode respected by every module touching upload/publish

---

## 14. Testing Rules (from original SOP — still applies)

- Follow System_Architecture.md Section 14.13's phased approach: 5-10 real videos first, then scale toward the fuller pre-launch stress pass
- Every test video is real content, never throwaway dummy data
- Simulated failure tests happen progressively, not all before the very first real video

---

*This SOP is a living document — update it as the actual development process reveals what works, without needing to reopen System_Architecture.md unless a genuine design flaw is found (see Section 7 above for that process).*
