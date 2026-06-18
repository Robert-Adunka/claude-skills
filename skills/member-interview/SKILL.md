# Member Interview Processor

This skill automates the full pipeline for processing TRIZ Mastery Hub member interview transcripts. Run all four steps in sequence without pausing for approval between them.

## Input

Two supported formats:

**File path** — user provides a path to the transcript:
```
/member-interview /path/to/transcript.txt
```

**Inline transcript** — user pastes the transcript directly into the chat (with or without the `/member-interview` command). If no date can be extracted from the text or context, use today's date for the filename.

## Step 1 — Correct the transcript

Read the raw transcript file. Apply **minimum necessary corrections** for speech-to-text errors — fix only what is clearly wrong due to ASR mishearing. Do not paraphrase, improve grammar, remove filler words, or change style.

**TRIZ terminology corrections** (only when context clearly refers to the methodology):
- "Tris", "trees", "Chris", "Pris", "twist" → TRIZ
- "Arize", "arise", "a rise", "a riz" → ARIZ
- "inventive principles" / "contradictions" — leave as spoken, usually fine

**Person name corrections** (fix phonetic variants):
- "Henrik Schuler", "Henry Schuler", "Henry Kal Schuler", "Henrich Alter", "Art Schuller" etc. → Genrikh Altshuller
- "Boris Slutton", "Boris Zlorton", "Boris Zlatev" etc. → Boris Zlotin
- "Alla Zusman" variants → Alla Zusman
- "Valery Sushkov", "Valerie Suchkov", "Valery Souchkov" etc. → Valeri Souchkov
- "Sergei Ikovenko" variants → Sergei Ikovenko (usually correct already)
- "Zion L. Bar-L", "Zion Bar-L", "Zvi Bar-El" etc. → Zion Bar-El (founder of Ideation International, deceased)

**Methodology name corrections:**
- "Capnot-Rego", "Kepnot Rego", "Kepner Trigo", "Cat Noctrigo", "Camp Notre Dame", "Catnod tree go", "Cat Nachigo", "Kepno Chigo", "Ketnotrigo" → Kepner-Tregoe
- "Chainin" → Shainin (competing problem-solving methodology)
- "Trismastery Hub", "Trismas Rehub", "Trist Mastery Hub", "Trismastry Hub" → TRIZ Mastery Hub

**Place name corrections:**
- "Petra Zawotsk", "Petro Zavots" etc. → Petrozavodsk
- "Niranda", "Noranda" variants → Noranda
- "Saas Strasbourg", "Strasbourg" → Strasbourg (correct)
- "Min Paris Tech" → Mines Paris Tech

Hold the corrected text in memory — you'll save it in Step 2.

## Step 2 — Save raw and corrected files

**Target folder:** `/Users/robert/Documents/TCG-Ordner/_Adunka/_CircleSo/MemberMeetingInterviews/`

**File naming:** `yymmdd_MemberMeeting_VornameNachname_raw.txt` and `_corrected.txt`
- Extract date: use the date from the original filename if present (format varies); otherwise use today's date in `yymmdd` format
- Extract name: identify the interviewee from the transcript (they are the non-Robert speaker; their name appears as a speaker label like "Yuri Kharlamov:" at the start of lines). Use their full name in CamelCase with no spaces: e.g. `YuriKharlamov`

Save:
1. A copy of the **original raw transcript** as `yymmdd_MemberMeeting_VornameNachname_raw.txt`
2. The **corrected transcript** as `yymmdd_MemberMeeting_VornameNachname_corrected.txt`

## Step 3 — Create Obsidian interview file

**Template location:** `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/TRIZ-Mastery-Hub/Member-Interviews/_Vorlage Member Interview.md`

**Target file:** `/Users/robert/Library/Mobile Documents/iCloud~md~obsidian/Documents/Arbeit-ObsidianVault/TRIZ-Mastery-Hub/Member-Interviews/<Vorname Nachname>.md`

Read the template, then create a new file named after the interviewee (e.g. `Yuri Kharlamov.md`). Fill out all sections **in German** based on the corrected transcript.

**YAML frontmatter fields to extract:**
- `Name:` — full name as used in Circle (= speaker label in transcript)
- `Datum:` — date of interview in `YYYY-MM-DD` format
- `TRIZ-Level:` — their self-described TRIZ training/level (e.g. "Business TRIZ Level 4 (lernt bei Valeri Souchkov)")
- `Beruf:` — job title / role
- `Branche:` — industry/sector
- `Organisation:` — company name, or "Nicht genannt" if not mentioned
- `Land:` — location/country

**Sections to fill (in German, based on what was actually said):**
- `## Motivation` — why did they join the Hub?
- `## Ziele` — what do they want to achieve or learn?
- `## Background` — TRIZ experience, how they got into it
- `## Beitrag` — what can/do they contribute to the community?
- `## Interessen` — topics, methods, industries they care about
- `## Offene Punkte` — open questions, wishes, problems mentioned
- `## Mögliche Vernetzungen` — who in the community might they connect well with?
- `## Aktivierungsidee` — a concrete next step or follow-up action
- `## Notizen` — free notes, memorable quotes, personality observations

Write concisely but completely. If something wasn't discussed, write a short neutral placeholder rather than leaving the section blank.

## Step 4 — Add note to Circle member profile

The member's name in Circle is **exactly the speaker label** used in the transcript (e.g. "Yuri Kharlamov"). Use this exact name to search.

1. Use `search_community_member` with the member's full name
2. Identify the correct member from the results
3. Use `update_community_member` to set or append to their `note` field

**Note content:** Write in **English**, concise bullet points. Include:
- TRIZ level / background
- Professional role and industry
- Why they joined / their main goal
- Suggested activation idea (what to invite them to)

Keep the note under 150 words. If the member already has a note, append below a `---` separator with the new interview date as a header.

## Summary

After completing all four steps, report briefly in one short paragraph:
- Files saved (with filenames)
- Obsidian note created (filename)
- Circle note updated (member name)
- Any issues encountered
