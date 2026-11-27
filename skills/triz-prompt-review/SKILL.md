---
name: triz-prompt-review
description: Use this skill when the user wants to improve, review, or update a TRIZ skill — including the XML primer prompt, the SKILL.md in truinorva/triz-skills, and the n8n MCP Server entry. Triggers on "TRIZ prompt verbessern", "skill verbessern", "TRIZ review", "prompt review", "lass uns heute triz_xyz verbessern", or any TRIZ tool name mentioned with improvement intent.
version: 1.1.0
---

# TRIZ Prompt Improvement Ritual

Guides a structured improvement session for TRIZ tools. Each session covers all artifacts of one skill together: **XML primer + SKILL.md (with its references) + n8n MCP Server**. One skill per session.

## Where the skills live (since 2026-11-27)

The skill content is maintained in **[truinorva/triz-skills](https://github.com/truinorva/triz-skills)** (Jens Träger / Truinorva). It is a plugin marketplace for Claude and ChatGPT with two plugins:

- `technical-triz/skills/[skill]/` — technical TRIZ
- `business-triz/skills/[skill]/` — `business-function-analysis`, `perception-mapping`, `solutions-at-system-levels`

The n8n MCP Server and the chat agents on triz-consulting.de load their skills **live from `main` of that repository**. Whatever is merged there reaches MCP users and chat agents immediately (chat agents within their 24 h prompt cache, or at once when the source URL changes).

**Two exceptions** stay in [Robert-Adunka/triz-skills](https://github.com/Robert-Adunka/triz-skills), because only these versions use what the MCP server adds on top of the plain skill:

| Skill | Why it stays in Robert-Adunka/triz-skills |
|---|---|
| `contradiction-solver` | Server-side matrix lookup: second tool call with `matrix_type`, `improving_indices`, `worsening_indices` |
| `function-oriented-search` | Server loads the TRIZ Effects Database vocabulary live from triz-consulting.de |

For these two, a change to the version in truinorva/triz-skills does **not** reach the MCP server. Ask the user whether the change belongs in both versions.

## The artifacts

| Artifact | Repo / Location | Purpose |
|---|---|---|
| `SKILL.md` + `references/` | `truinorva/triz-skills` → `[plugin]/skills/[skill]/` (exceptions: see above) | The skill itself: frontmatter `description` decides when it is used, the body defines the process, `references/` carries the material |
| XML primer | `jenson500/triz-prompt-engineering` → `prompts/technical_triz/[skill]/[skill].xml` | Full prompt behavior and process definition (ccTOPP) |
| n8n tool node | Workflows `QDDXjTgvUQOpJqou` (TRIZ MCP Server) **and** `r9nSwVJPUHINt1MQ` (TRIZ MCP Server Web) | Tool `description` shown to the calling model + `ai_tool` connection |
| n8n sub-workflow | `TRIZ-MCP-[skill]` | Fetches `SKILL.md` and the reference files from GitHub and returns them as `skill_content` |

---

## Session Ritual

### Step 1 — Pick the skill
User names the skill to improve, or choose from the priority list below. Determine the plugin (`technical-triz` or `business-triz`) and whether it is one of the two exceptions.

### Step 2 — Read current state
Read from `main` of the repos (or a local clone, see *Key Technical Facts*):
- `SKILL.md` and every file in `references/`
- the XML primer
- the tool node description in **both** MCP Server workflows
- the `TRIZ-MCP-[skill]` sub-workflow: which files it fetches

### Step 3 — Run all checklists
Work through the XML, SKILL.md and n8n checklists. Report all findings before touching anything.

### Step 4 — Plan & confirm
Present planned changes clearly. Wait for user approval before implementing.

### Step 5 — Implement (all artifacts)
Branch → edit → validate → pull request in truinorva/triz-skills → after merge: update n8n. See *Deployment Pipeline*.

### Step 6 — Test recommendation
Suggest a concrete semi-automatic test scenario the user can run immediately. After the merge, call the MCP tool once to confirm it returns the new content.

---

## Checklist: XML Primer (prompt-format-guide standard)

| Check | Rule |
|---|---|
| Title length | ≤ 50 characters |
| Description length | ≤ 300 characters |
| Starters count | Exactly 2–3 |
| Instruction type attributes | Only `type="general"` and `type="alternative"` — no custom types |
| Required tags present | `<Goal>`, `<Role>`, `<Output_Format>`, `<User_Input>` all exist |
| Correct tag name | `<Output_Format>` not `<Output>` |
| Example structure | Each `<Example title="...">` has `<Problem>` + `<Solution>` children |
| Three-mode pattern | Mode selection as Step 1 in `type="general"`; details in `type="alternative"` |
| Output completeness | All required output elements explicitly stated in `<Output_Format>` |

### Three-Mode Pattern — Reference

**In `Instruction type="general"`, Step 1 must read:**
> Ask the user which working mode they prefer: Automatic (generate immediately using own assumptions, state them), Semi-automatic (ask [N] targeted questions, then generate), or Interactive (step by step with user confirmation at each stage).

**In `Instruction type="alternative"`:**
- Semi-automatic: list the specific questions to ask before generating
- Interactive: describe the step-by-step confirmation behavior

---

## Checklist: SKILL.md (rules of truinorva/triz-skills)

| Check | Rule |
|---|---|
| `name` field | Present, lowercase-hyphenated, ≤ 64 characters, **identical to the folder name** |
| `description` field | ≤ 1024 characters; contains specific trigger phrases users would actually say |
| Trigger keywords | Precise, non-overlapping with other TRIZ skills |
| Negative Abgrenzung | Add "Does NOT handle X" / "use skill Y instead" when another skill covers similar territory |
| References | Every file the body tells the model to read exists in `references/`, and every file in `references/` is mentioned |
| License comment | Keeps the authors and "see the LICENSE file shipped with this Skill" |
| Sync with n8n | SKILL.md `description` = tool node description in both MCP Server workflows (must be identical) |

---

## Checklist: n8n MCP Server

| Check | Rule |
|---|---|
| Node description | Matches SKILL.md `description` exactly — in `QDDXjTgvUQOpJqou` **and** `r9nSwVJPUHINt1MQ` |
| Connection type | `ai_tool` — never `main` |
| Fix syntax if wrong | `removeConnection` then `addConnection` with `sourceOutput: "ai_tool"` |
| Sub-workflow source | `TRIZ-MCP-[skill]` fetches from `https://raw.githubusercontent.com/truinorva/triz-skills/main/[plugin]/skills/[skill]/` (exceptions: Robert-Adunka/triz-skills) |
| Sub-workflow references | Fetches exactly the reference files the SKILL.md names. A new, renamed or removed reference file needs the fetch list updated, otherwise the tool fails or returns half a skill |
| Large references | Bilingual references (DE + EN) are sent in one language only, chosen from `lang` or the query (see `TRIZ-MCP-76-standard-solutions`, `TRIZ-MCP-physical-contradictions`) |
| Published, not only saved | Check with `n8n_get_workflow` mode `active` that the change is live |

---

## Deployment Pipeline

```
truinorva/triz-skills      (skill content — the default case)
  clone:   git clone https://github.com/truinorva/triz-skills /Users/robert/Documents/GitHub/truinorva-triz-skills
  path:    [plugin]/skills/[skill]/SKILL.md (+ references/)
  ⚠ Pull-request workflow: feature branch → PR → merge into main. Keep main releasable:
           ChatGPT workspaces sync it daily, the MCP server and chat agents read it live.
  before the PR:
           python scripts/build_skills.py --check
           claude plugin validate --strict ./technical-triz   (or ./business-triz)
  release: installed plugins (Claude Code, ChatGPT desktop) update only on a new version.
           A release = same new version in all four plugin manifests + tag vX.Y.Z.
           Coordinate releases with Jens; do not cut one per skill change.

Robert-Adunka/triz-skills  (only contradiction-solver, function-oriented-search)
  path:    /Users/robert/Documents/GitHub/triz-skills/[skill]/SKILL.md
  ✓ Direct push to main OK

jenson500/triz-prompt-engineering  (XML primer)
  path:    /Users/robert/Documents/GitHub/triz-prompt-engineering/prompts/technical_triz/[skill]/[skill].xml
  ⚠ Branch protection on main → create branch + PR via gh

n8n (after the skill PR is merged)
  → tool node description in QDDXjTgvUQOpJqou AND r9nSwVJPUHINt1MQ (must match SKILL.md)
  → verify ai_tool connection
  → TRIZ-MCP-[skill]: update the reference fetch list if references changed
```

---

## Priority List

Skills with clear process logic that benefit most from the three-mode pattern:

1. `triz_contradiction_solver` — Widerspruchsmatrix + ARIZ, klare Schrittfolge (exception: MCP version in Robert-Adunka/triz-skills)
2. `triz_function_analysis` — mehrstufiger Prozess, sehr häufig genutzt
3. `triz_cause_effect_chain_analysis` — iterative Analyse
4. `triz_trimming` / `triz_interactive_trimming` — Dopplung prüfen, evtl. zusammenführen
5. `triz_resource_analysis`
6. `triz_ideality`
7. `triz_76_standard_solutions`
8. Weitere nach Bedarf

Known open item (2026-11-27): several n8n tool descriptions still carry the old wording and no longer match the SKILL.md descriptions in truinorva/triz-skills (seen for `ideality` and `sustainability-assessment`). Fix them skill by skill in the sessions.

---

## Key Technical Facts

| Fact | Value |
|---|---|
| Skill source | `truinorva/triz-skills`, branch `main` — PR workflow |
| Skill path | `technical-triz/skills/[skill]/SKILL.md` or `business-triz/skills/[skill]/SKILL.md` |
| Business-TRIZ skills | `business-function-analysis`, `perception-mapping`, `solutions-at-system-levels` |
| MCP-specific exceptions | `contradiction-solver`, `function-oriented-search` in `Robert-Adunka/triz-skills` (direct push to `main` OK) |
| TRIZ MCP Server workflow IDs | `QDDXjTgvUQOpJqou` (with bearer token), `r9nSwVJPUHINt1MQ` (Web, no auth) |
| Sub-workflows | `TRIZ-MCP-[skill]`, one per skill |
| Chat agents loading skills live | `76 Standard Solutions v4`, `Feature Transfer v4`, `Root Conflict Analysis v4`, `Solutions at System Levels v4` |
| triz-prompt-engineering | Branch protection on `main` → always PR |
| triz-prompt-engineering path | `prompts/technical_triz/[skill]/[skill].xml` |
| n8n ai_tool connection | `addConnection` needs `sourceOutput: "ai_tool"` |
