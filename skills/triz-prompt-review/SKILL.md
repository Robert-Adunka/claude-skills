---
name: triz-prompt-review
description: Use this skill when the user wants to improve, review, or update a TRIZ skill — including the XML primer prompt, SKILL.md routing description, and n8n MCP Server entry. Triggers on "TRIZ prompt verbessern", "skill verbessern", "TRIZ review", "prompt review", "lass uns heute triz_xyz verbessern", or any TRIZ tool name mentioned with improvement intent.
version: 1.0.0
---

# TRIZ Prompt Improvement Ritual

Guides a structured improvement session for TRIZ tools. Each session covers all three artifacts together: **XML primer + SKILL.md + n8n MCP Server entry**. One skill per session.

## The Three Artifacts

Every TRIZ tool has three interdependent artifacts that must be kept in sync:

| Artifact | Repo / Location | Purpose |
|---|---|---|
| `SKILL.md` | `triz-skills` repo | LLM routing — tells Claude when to invoke the tool |
| XML primer | `triz-prompt-engineering` repo | Full prompt behavior and process definition |
| n8n node | TRIZ MCP Server workflow | `description` field + `ai_tool` connection |

---

## Session Ritual

### Step 1 — Pick the skill
User names the skill to improve, or choose from the priority list below.

### Step 2 — Read current state
Read existing SKILL.md + XML from `/Users/robert/Downloads/TRIZ-Skills-new/[skill]/` or directly from the GitHub repos.

### Step 3 — Run both checklists
Work through the XML checklist and SKILL.md checklist. Report all findings before touching anything.

### Step 4 — Plan & confirm
Present planned changes clearly. Wait for user approval before implementing.

### Step 5 — Implement (all three artifacts)
Edit in staging → deploy to GitHub → update n8n. See deployment pipeline below.

### Step 6 — Test recommendation
Suggest a concrete semi-automatic test scenario the user can run immediately.

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

## Checklist: SKILL.md

| Check | Rule |
|---|---|
| `name` field | Present in frontmatter |
| `description` field | Contains specific trigger phrases users would actually say |
| Trigger keywords | Precise, non-overlapping with other TRIZ skills |
| Negative abgrenzung | Add "Does NOT handle X" when another skill covers similar territory |
| Sync with n8n | SKILL.md description = n8n node description (must be identical) |

---

## Checklist: n8n MCP Server

| Check | Rule |
|---|---|
| Node description | Matches SKILL.md description exactly |
| Connection type | `ai_tool` — never `main` |
| Fix syntax if wrong | `removeConnection` then `addConnection` with `sourceOutput: "ai_tool"` |
| Workflow ID | `QDDXjTgvUQOpJqou` (TRIZ MCP Server) |

---

## Deployment Pipeline

```
Staging:   /Users/robert/Downloads/TRIZ-Skills-new/[skill]/

GitHub 1:  /Users/robert/Documents/GitHub/triz-prompt-engineering/
           prompts/technical_triz/[skill]/[skill].xml
           ⚠ Branch protection on main → create branch + PR via gh

GitHub 2:  /Users/robert/Documents/GitHub/triz-skills/[skill]/SKILL.md
           ✓ Direct push to main OK

n8n:       Workflow QDDXjTgvUQOpJqou (TRIZ MCP Server)
           → Update description on tool node (must match SKILL.md)
           → Verify ai_tool connection
```

---

## Priority List

Skills with clear process logic that benefit most from the three-mode pattern:

1. `triz_contradiction_solver` — Widerspruchsmatrix + ARIZ, klare Schrittfolge
2. `triz_function_analysis` — mehrstufiger Prozess, sehr häufig genutzt
3. `triz_cause_effect_chain_analysis` — iterative Analyse
4. `triz_trimming` / `triz_interactive_trimming` — Dopplung prüfen, evtl. zusammenführen
5. `triz_resource_analysis`
6. `triz_ideality`
7. `triz_76_standard_solutions`
8. Weitere nach Bedarf

---

## Key Technical Facts

| Fact | Value |
|---|---|
| TRIZ MCP Server workflow ID | `QDDXjTgvUQOpJqou` |
| triz-prompt-engineering | Branch protection on `main` → always PR |
| triz-skills | Direct push to `main` OK |
| Staging area | `/Users/robert/Downloads/TRIZ-Skills-new/` |
| n8n ai_tool connection | `addConnection` needs `sourceOutput: "ai_tool"` |
| triz-prompt-engineering path | `prompts/technical_triz/[skill]/[skill].xml` |
| triz-skills path | `[skill]/SKILL.md` |
