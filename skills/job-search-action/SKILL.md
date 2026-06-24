---
name: job-search-action
description: >
  Handles all job search status changes and logging. Use this skill whenever the user communicates
  a job search event — applying to a role, hearing back from a company, getting an interview,
  being rejected, being ghosted, adding a new lead, or any pipeline status change. Trigger phrases
  include: "I applied to", "heard back from", "got an interview with", "rejected by", "ghosted by",
  "found a role at", "I just submitted", "that one's dead", "move X to interviewing", "follow up on",
  "update the job search", or any phrasing that implies a job application pipeline change. Also
  triggers when the user mentions completing application steps like "sent my CV to", "did the easy apply
  for", "finished the cover letter for". When in doubt, use this skill — it is better to log an
  action than to let pipeline state drift out of sync.
metadata:
  version: "5.0.0"
---

## Configuration (read this first)
All IDs, project names, and paths below are placeholders of the form `{{PLACEHOLDER}}`. Before doing anything, read `config.local.md` at the repo root and resolve every placeholder to the user's real value. If `config.local.md` is missing or a required value is blank, STOP and tell the user to complete `SETUP.md` first — never proceed with empty IDs. Optional values (e.g. `{{TODOIST_SYSTEM_SECTION}}`) may be blank; skip those steps gracefully.

# Job Search Action Skill (v5.0.0 — Notion sole source-of-truth + 19-property schema + read-merge-write pattern + Excitement/Track/Temperature as Notion SELECTs)

You are the single entry point for all job search pipeline changes. When the user tells you something happened with a job application, you update both systems (Notion DB + Todoist) so they never have to touch either manually.

## Why This Exists

The user runs a two-track job search (Track A: targeted, Track B: volume/easy apply). Before v5.0.0, pipeline state was split across Google Sheets (reference layer via Make.com webhooks) and Todoist (action layer with state encoded in labels). v5.0.0 consolidates pipeline state into a single Notion database — **the Opportunities DB is the sole source of truth.** Todoist holds action tasks only; each task references the Notion opportunity row.

This skill makes Claude the runner: one sentence from the user, both systems updated.

## The Two Systems

**Notion** — the source-of-truth layer. All job-search applications live in the **Opportunities DB**. One row per (Company + Role). Per-opportunity interview prep lives as child pages under each row.

```
Database page ID:    {{NOTION_OPPORTUNITIES_DB}}
Data source ID:      {{NOTION_OPPORTUNITIES_DS}}
Parent page:         {{NOTION_PARENT_PAGE}}
```

The DB has 19 properties (17 functional + 2 read-only formula columns: `Days since last contact` and `Days since applied`). See the "Notion property reference" section below.

**Todoist** — the action layer. One task per active application. Tasks live in project "🔍 Job Search" (ID: `{{TODOIST_PROJECT}}`).

> 🚨 **Section IDs in this table are reference values — verify before use.** Todoist section IDs are opaque strings; long opaque IDs in static markdown are drift-prone (typo on rewrite, copy-paste loses a character, runtime copy outdates source). **Before any `add-tasks` or `update-tasks` call that uses a section ID from this table, run `find-sections` against project `{{TODOIST_PROJECT}}` and use the live ID as the source of truth.**

Post v5.0.0 cutover (Ticket #6), the pipeline-state sections (Sourcing/Applied/Interviewing/Offer-Decision/Dead) are deleted. Only the `System` section remains. New action tasks land **sectionless** (no `sectionId` on `add-tasks`); they get organised by `due_date` and `priority` instead.

| Section | ID (verify via `find-sections` before use) | When | In pipeline? |
|---------|-----|------|---|
| System | `{{TODOIST_SYSTEM_SECTION}}` | Future dev work on the job search system itself (new skills, filter tweaks, scan criteria updates, feedback-loop features). **Rolling section.** | ❌ No — see exclusion rule below |

**System section exclusion rule:** The System section holds dev/improvement work, not applications. `job-search-action` MUST NOT:
- Create application tasks in this section (all pipeline events land sectionless in the Job Search project)
- Apply `job-search` label to anything in this section
- Count System tasks in any pipeline aggregation (weekly pipeline review, "X applications in flight", Notion DB reconciliation, etc.)

Tasks in the System section use their own labels (e.g. `system-dev`, `infra`) and live by normal Todoist due-date mechanics — they surface via `morning-briefing`'s "tasks due today" pull only when actually due, which is the correct behaviour for any time-boxed work. Do not mix System tasks with the Notion DB either; the DB represents the application pipeline only.

## Notion property reference

The Opportunities DB has 19 properties. **Always write the exact emoji-prefixed string for SELECT options** — the API rejects unrecognised values. Examples below.

| Property | Type | Value examples |
|---|---|---|
| Company | TITLE | `"Acme Corp"` |
| Role | RICH_TEXT | `"Operations Executive"` |
| Status | SELECT | `"🔍 Sourcing"` / `"📨 Applied"` / `"📞 Recruiter screen"` / `"👤 Hiring manager"` / `"📝 Take-home"` / `"🎯 Final stage"` / `"💌 Offer"` / `"🎉 Offer accepted"` / `"⚰️ Dead"` |
| Excitement | SELECT | `"5 ⭐⭐⭐⭐⭐"` / `"4 ⭐⭐⭐⭐"` / `"3 ⭐⭐⭐"` / `"2 ⭐⭐"` / `"1 ⭐"` |
| Track | SELECT | `"🅰️ A"` / `"🅱️ B"` |
| Temperature | SELECT | `"🔥 Hot"` / `"☀️ Warm"` / `"❄️ Cool"` |
| Source | SELECT | `"LinkedIn"` / `"Reed"` / `"Adzuna"` / `"Otta"` / `"Referral"` / `"Direct"` / `"Other"` |
| Link | URL | bare URL string |
| Salary range | RICH_TEXT | `"your salary floor"` / `"TBD"` |
| Work model | SELECT | `"🏠 Remote"` / `"🔀 Hybrid 1d"` / `"🔀 Hybrid 2d"` / `"🔀 Hybrid 3d"` / `"🔀 Hybrid 4d"` / `"🔀 Hybrid TBD"` / `"🏢 Onsite"` |
| Location | RICH_TEXT | `"London"` / `"Remote UK"` |
| Applied date | DATE | expanded form: `"date:Applied date:start"` = `"2026-05-15"`, `"date:Applied date:is_datetime"` = `0` |
| Last contact | DATE | same expanded form |
| Next action | RICH_TEXT | `"Follow up if no response"` |
| Next action date | DATE | same expanded form |
| Days since last contact | FORMULA (read-only) | auto: `dateBetween(now(), prop("Last contact"), "days")` |
| Days since applied | FORMULA (read-only) | auto: `dateBetween(now(), prop("Applied date"), "days")` |
| Dead reason | SELECT | `"Rejected"` / `"Ghosted"` / `"Withdrew"` / `"Role closed"` / `"Not a fit"` / `"Other"` |
| Notes | RICH_TEXT | free-form |

**Page body convention:** every new row's page body includes a `## Job Description` heading with the JD pasted underneath. The user can convert to a toggle in UI if desired.

**Views are display-only.** The DB ships with 7 views (🎯 Today, ⏰ Stale follow-ups, 🔥 Active pipeline, 📞 Interviewing, 🔍 Sourcing, ⚰️ Archive, 🗂 All — see `notion-template/opportunities-views-spec.md`). You never write to a view: write properties on the row and every view re-filters automatically. Don't try to create or modify views as part of logging an event.

## Track A vs Track B

The Job Search Framework defines two tracks. This distinction matters for pacing and the type of work involved.

**Track A — Targeted** (2–3 per week, 45 min max per application)
Roles the user genuinely wants. Full workflow: triage → fit check → log → tailor CV → write application → final check → log & follow up. Stored as `Track = 🅰️ A` in Notion.

**Track B — Volume** (5–10 per week, 5–8 min per application)
LinkedIn Easy Apply or quick submissions. Master CV, no tailoring. Steps 1, 2, 3, 7 only. Stored as `Track = 🅱️ B` in Notion.

Track A vs B is determined by the effort the user put in at application stage — not by the role type. A Customer Support Specialist role could be Track A if they tailored their CV for it.

If the user doesn't specify the track, infer it from application effort cues:
- If they mention "easy apply", "quick apply", "no tailoring", "just hit apply" → Track B
- If they mention tailoring, cover letter, adjusting CV, or spending time on the application → Track A
- Do NOT infer track from the role title — any role can be either track
- If unclear, ask: "Did you tailor for this one, or was it a quick apply?"

## Excitement vs Temperature — CRITICAL DISTINCTION

These are two **separate** axes. Do not conflate them.

**Excitement (1–5)** = how much the user personally wants the role.
- Based on role type, salary, location, company, culture — things that don't change much between application and offer.
- Stored as Notion `Excitement` SELECT (e.g. `"4 ⭐⭐⭐⭐"`).
- Rule: **5** = strongly want this (dream-tier). **4** = would be happy with this. **3** = fine, I'd take it. **2** = settling / backup. **1** = wouldn't take it.
- Excitement usually stays stable from application to offer. The main things that can push it up: salary revealed to be higher than expected, unexpected benefits disclosed. Don't guess — ask the user if they didn't provide one.

**Temperature (Hot/Warm/Cool)** = how "live" the lead is (like a sales CRM).
- Based on engagement, momentum, and timeliness. Changes over the lifecycle.
- Stored as Notion `Temperature` SELECT (e.g. `"🔥 Hot"`).
- Also mirrored as a Todoist label (`hot`/`warm`/`cool`) on the action task for at-a-glance triage in Todoist.
- Rule of thumb:
  - `🔥 Hot` — active conversation / interview stage / imminent action needed
  - `☀️ Warm` — applied, waiting, still live
  - `❄️ Cool` — volume play, long shot, going cold, or low-priority

A role can be **high excitement + cool temperature** (dream job that's gone quiet) or **low excitement + hot temperature** (backup role that's actively progressing). Both are valid.

If the user doesn't specify excitement, **ask**. Don't guess. They want this data captured accurately for future ML pattern matching.
If the user doesn't specify temperature, default to `☀️ Warm` for Applied, `🔥 Hot` for Recruiter screen / Hiring manager / Take-home / Final stage / Offer, `❄️ Cool` for Sourcing / Dead.

## Todoist Labels (post-v5.0.0)

Every job search task gets the `job-search` label. Additionally:

| Label | When to apply |
|-------|---------------|
| `job-search` | Mandatory on every job search task |
| `hot` | Active conversation or interview stage — mirrors Notion `Temperature = 🔥 Hot` |
| `warm` | Applied, waiting, still live — mirrors Notion `Temperature = ☀️ Warm` |
| `cool` | Going cold or low priority — mirrors Notion `Temperature = ❄️ Cool` |

**Note (v5.0.0 cutover):** `track-a`, `track-b`, `excitement-1` through `excitement-5` labels were deleted as part of Ticket #6. Track + Excitement now live as Notion SELECT properties only. Do NOT recreate or apply these labels.

**Temperature labels apply to the PARENT application task only — never subtasks (follow-ups, interview prep, etc.). Subtasks only get `job-search`.** Exception: interview-specific subtasks can keep `hot` to surface in morning briefings.

## Input Flow — How the User Communicates Events

The user can communicate a job search event in several ways. Claude should handle all of them:

### Primary: Link-first flow (preferred)

The user pastes a job posting URL (LinkedIn, Wellfound, Otta, Workable, Ashby, Cord, etc.) with a short message like "applied to this one" or "found this, looks good" or just the link itself.

When a URL is provided:
1. Fetch the page using the WebFetch tool to extract key details
2. From the page, extract: company name, role title, location, salary (if listed), key requirements, source platform
3. Present a brief summary to the user for confirmation: "Got it — **[Company] — [Role]**, [Location], [Salary if found]. Track A or B?" (skip the confirmation if the user already specified the track and action in the same message, e.g. "easy applied to this one [link]" — just log it)
4. After confirmation, proceed to the relevant action below

### Secondary: Natural language

The user says something like "I applied to Northwind Trading for a French CS role" without a link. Claude extracts what it can from the message and asks for anything critical that's missing (at minimum: company and role title). If a Notion row already exists for that company+role, use its existing data to fill gaps (call `notion-search` + `notion-fetch`).

### Tertiary: Batch summary

The user lists several applications at once: "Did 3 easy applies today — Acme Corp, Customer Service Coordinator at Northwind Trading, and a Tech Support role at Contoso." Process each one individually using the actions below.

## Actions

### 1. New Lead (Sourcing)

The user found a role but hasn't applied yet.

**Notion (`notion-create-pages`):**
- Parent: `{type: "data_source_id", data_source_id: "{{NOTION_OPPORTUNITIES_DS}}"}`
- Properties: `Company`, `Role`, `Source`, `Link`, `Status: "🔍 Sourcing"`, `Excitement`, `Track`, `Temperature: "❄️ Cool"` (default for Sourcing), `Work model` (if known), `Salary range` (if known), `Notes` (summary of fit / why interested), `Next action: "Review and decide — apply or skip"`, `date:Next action date:start = today`, `date:Next action date:is_datetime = 0`
- Page body content: `## Job Description\n\n[paste JD here]`

**Todoist (`add-tasks`):**
- Project: `{{TODOIST_PROJECT}}`, sectionless
- Content: `"[Company] — [Role]"`
- Description: include link to JD (critical — the link is how the user accesses the JD from Todoist), salary range if known, key requirements (2-3 bullets), and the Notion row URL on a single line
- Labels: `job-search`, `cool` (Sourcing default)
- Due date: today (prompt to triage/apply)
- Priority: `p4`

### 2. Application Submitted

The user applied to a role. This might be a new entry or a move from Sourcing.

**If a Notion row already exists in Sourcing for this Company+Role:**
- Update Status to `"📨 Applied"`, set `date:Applied date:start` to today, update `Next action` to `"Follow up if no response"`, `date:Next action date:start` to applied + 5 business days, `date:Last contact:start` to today.
- Use the read-merge-write pattern (see "How to call Notion" below) — `notion-search` to find the page, `notion-fetch` to read current properties, `notion-update-page` to write changes.

**If no Notion row exists yet (common for Track B batch logging):**
- Create the row via `notion-create-pages` with `Status: "📨 Applied"`, `date:Applied date:start = today`, all other properties as for New Lead.

**Todoist task:**
- Content: `"[Company] — [Role]"`, sectionless, project `{{TODOIST_PROJECT}}`
- Labels: `job-search`, temperature label (`warm` default for Applied, `hot` if active recruiter contact already)
- Due date: next follow-up date (see "Follow-up Timing Rules" below)
- Priority: `p3` (Track A) or `p4` (Track B)
- Description format:
  ```
  [Link to JD](https://...) — [Track A/B]
  [Notion: https://www.notion.so/<page_id>]

  **Status:** Applied [date] ([Track A/B], tailored CV / quick apply)
  **Salary:** [range or TBD]

  **Backstop:** 3 weeks post-app. If no human contact by [applied + 3w], confirm Dead.
  ```
  The link must always be the first line. The Notion URL is the second line — critical for cross-linking. The Backstop line is mandatory for silent applications (no recruiter engaged yet).

### Follow-up Timing Rules

When setting the next-follow-up date on any application, apply this priority order:

1. **Recruiter gave a specific timeline** (e.g. "I'll come back Monday", "expecting feedback next week") → next follow-up = that date + 1 day.
2. **Recruiter engaged but no timeline given** → next follow-up = applied/last-contact date + 5 business days.
3. **No human contact yet (silent application)** → next follow-up = applied date + 5 business days.

Once a recruiter engages, the 3-week silent-app backstop no longer applies — the due date tracks the active conversation. The backstop only governs applications that have received zero human contact.

### 3. Response Received / Interview

A company responded positively or scheduled an interview.

**Notion (read-merge-write via `notion-update-page`):**
- Update `Status` to the appropriate interview stage: `"📞 Recruiter screen"` (first call), `"👤 Hiring manager"` (HM round), `"📝 Take-home"` (assessment), `"🎯 Final stage"` (last interview before offer)
- Update `Temperature` to `"🔥 Hot"` (active conversation)
- Update `Next action`, `date:Next action date:start` (interview date or next milestone)
- Update `date:Last contact:start` to today
- Append to `Notes` (interview details, recruiter quotes, format)
- Do NOT change `Excitement` unless the user tells you it changed (e.g. salary revealed higher than expected)

**Todoist (`update-tasks`):**
- Set due date to interview date (or next action date if no date yet)
- Update priority to `p2`
- Set temperature label to `hot` (remove any existing `warm`/`cool`)
- Add comment via `add-comments` with details (interview date, format, who with)

### 4. Rejection

Company said no.

**Notion (`notion-update-page`):**
- Update `Status` to `"⚰️ Dead"`
- Set `Dead reason` — if the user stated a reason ("they rejected me"), use `"Rejected"`. If unclear, prompt once: *"Dead reason? (Rejected / Ghosted / Withdrew / Role closed / Not a fit / Other)"*. Accept short-form answers. Map first, confirm the code implicitly by writing the right value.
- Update `Temperature` to `"❄️ Cool"`
- Update `date:Last contact:start` to today
- Append rejection reason / context to `Notes`
- Clear `Next action` and `Next action date` (write empty string `""`)

**Todoist:**
- Complete the task via `complete-tasks` (it gets archived)
- Add a final comment via `add-comments` with the rejection date + reason

### 5. Silent Application — 3-Week Backstop

Applies ONLY to applications with zero human contact (no recruiter engagement, no reply from the company). Once a recruiter engages, this section no longer applies — treat the role as an active conversation governed by "Follow-up Timing Rules".

**Follow-up cadence for silent applications:**
- Due date = next follow-up date (see Follow-up Timing Rules, rule 3).
- When the task is due and still silent: send a nudge, update `date:Last contact:start` (Notion) + `last_contact` (Todoist comment), add a dated comment, push due date forward another 5 business days.
- Max 2 nudges total.

**3-week backstop (the Dead-confirmation trigger):**
- If an application is still silent 3 weeks past its applied date AND 2 nudges have been sent, the next action is a Dead-confirmation prompt (handled by `morning-briefing` via the `Days since last contact` formula property — not auto-Dead).
- `job-search-action` does NOT auto-complete these. When the user confirms, move to Dead using the Rejection flow (Section 4) with `Dead reason: "Ghosted"` and Notes "Auto-Dead after 3w silence + 2 nudges".

**Rationale:** The 3-week backstop is a soft signal, not an auto-kill. The user wants to be prompted to decide, not have applications quietly disappear from the pipeline. Recruiter-engaged roles are exempt because they have a live next-touch date from the conversation itself.

### 6. Offer Received

**Notion (`notion-update-page`):**
- Update `Status` to `"💌 Offer"`
- Update `Temperature` to `"🔥 Hot"` (stays the same unless the user mentions otherwise)
- Update `Next action` with the decision deadline
- Update `date:Last contact:start` to today
- Append offer details (salary, terms, deadline) to `Notes`

**Todoist:**
- Priority `p1`
- Temperature label `hot`
- Due date: decision deadline if given

### 7. Offer Accepted

The user accepted an offer. Pipeline closes for this role.

**Notion:**
- Update `Status` to `"🎉 Offer accepted"`
- Update `Temperature` to `"☀️ Warm"` (no longer urgent)
- Append acceptance details + start date to `Notes`

**Todoist:**
- Complete the task. The Notion row remains for historical reference.

### 8. Batch Logging (Track B)

The user might say "I did 3 easy applies today" and list them. Process each one as a separate Application Submitted action. For batch logging:
- Create all Todoist tasks
- Create all Notion rows via a single `notion-create-pages` call with an array of pages (the API accepts up to 100 in one call; batch them to stay below ~3 req/s soft rate limit)
- Confirm the batch at the end: "Logged 3 Track B applications: [Company A], [Company B], [Company C]. All set with 5-day follow-ups."

### 9. Skip (permanent exclusion)

The user says "skip [company X]" or "not interested in [company Y]" — the role appeared somewhere (LinkedIn scan, recruiter outreach) but they do not want to apply, and do not want it re-surfaced.

**No Todoist task, no Notion row.** Skip roles never enter the pipeline — they only get recorded in `.skipped-roles.jsonl` so future sourcing filters can exclude them.

Append one JSONL line to `.skipped-roles.jsonl` at the repo root. The canonical key uses the normalisation function below:

```js
const normStr = s => (s||'').toLowerCase().replace(/[^a-z0-9\s]/g,'').trim().replace(/\s+/g,'-');
```

Lowercase → strip all non-alphanumeric except spaces → trim → collapse whitespace runs to a single hyphen. Note this is destructive to punctuation: `O'Reilly` → `oreilly`, `A&B Corp` → `ab-corp`, `Smith, Jones & Co.` → `smith-jones-co`.

**Canonical format:** `{normStr(company)}:{normStr(title)}`

Examples:
- `Acme Corp` + `Operations Executive` → `acme-corp:operations-executive`
- `Northwind Trading` + `Operations Coordinator` → `northwind-trading:operations-coordinator`

(An automated job-board scanner that consumes this file is a planned add-on, not included in v1.)

#### Reason codes (closed vocabulary)

Every skip must carry a `reason_code` from this fixed list. Feeds the weekly retrospective — free-text reasons don't aggregate into signal.

| Code | When |
|------|------|
| `wrong-domain` | Industry/vertical off (e.g. finance ops, logistics ops, HR ops when the user wants SaaS/consumer customer-ops) |
| `seniority-off` | Too junior (IC role when they want lead/manager) OR too senior (head-of when they want IC/senior IC) |
| `commute` | Location impractical (outside commute radius, 5-days-onsite in wrong city) |
| `salary-low` | Below their floor or expected band |
| `stack-mismatch` | Tech stack / tool requirements don't match (e.g. Salesforce Admin cert required, heavy coding) |
| `duplicate` | Already applied elsewhere, or same role surfaced twice |
| `company-pass` | Specific company they don't want to work for (culture, reputation, past experience) |
| `not-interested` | Fits criteria on paper but doesn't excite them — catchall for gut-level "no" |
| `other` | Doesn't fit above codes — MUST be paired with `reason_detail` free text |

**How to capture the reason:**

1. **If the user stated a reason in the message** (e.g. "skip X, too junior", "skip Y — logistics, not my thing"):
   - Map to the closest code. "too junior" → `seniority-off`. "logistics, not my thing" → `wrong-domain`.
   - Also store their exact words in `reason_detail`.

2. **If the user didn't state a reason** (bare "skip X"):
   - Ask once: *"Quick reason? `wrong-domain` / `seniority-off` / `commute` / `salary-low` / `stack-mismatch` / `duplicate` / `company-pass` / `not-interested` / `other`."*
   - Accept short-form answers. Map first, confirm the code implicitly by writing the right value.

3. **Never guess silently.** If the message is ambiguous, ask.

**Append one JSONL line to `.skipped-roles.jsonl`:**

```json
{"canonical":"acme-corp:operations-executive","company":"Acme Corp","role":"Operations Executive","skipped_at":"2026-04-22","reason_code":"wrong-domain","reason_detail":"logistics/not SaaS ops","source_url":"https://..."}
```

Fields:
- `canonical` — the dedup key (mandatory)
- `company`, `role` — for human-readable audit
- `skipped_at` — ISO date
- `reason_code` — one of the codes above (mandatory)
- `reason_detail` — the user's exact words if given, otherwise `""`. Mandatory non-empty when `reason_code` is `other`.
- `source_url` — the URL if known, helpful for manual audit

**Append, never overwrite.** Use `fs.appendFileSync` in Node or `echo '...' >> .skipped-roles.jsonl` in bash. One line per skip. No trailing newline needed on the final line.

**Confirmation:**

> ✅ **Acme Corp — Operations Executive** skipped (`wrong-domain`). Added `acme-corp:operations-executive` to `.skipped-roles.jsonl`. Will not re-surface in future scans.

**Edge cases:**
- If the user skips something already in the pipeline Notion DB: ask whether they also want to mark the Notion row as `⚰️ Dead` (rare — usually means they changed their mind mid-stream).
- If the canonical already exists in `.skipped-roles.jsonl`: no-op, confirm "already skipped on {date} (`{reason_code}`)".
- If the user pushes back on the reason prompt ("just skip it"): use `not-interested` and move on — don't block them on a data-capture step.

## How to call Notion

All pipeline writes go through Notion MCP. Three primary tools: `notion-create-pages`, `notion-update-page`, `notion-fetch` (+ `notion-search` for lookups).

### New row — `notion-create-pages`

```json
{
  "parent": {"type": "data_source_id", "data_source_id": "{{NOTION_OPPORTUNITIES_DS}}"},
  "pages": [
    {
      "properties": {
        "Company": "Northwind Trading",
        "Role": "Senior CS Ops",
        "Source": "LinkedIn",
        "Link": "https://www.linkedin.com/jobs/view/...",
        "Status": "📨 Applied",
        "Excitement": "4 ⭐⭐⭐⭐",
        "Track": "🅰️ A",
        "Temperature": "☀️ Warm",
        "Work model": "🔀 Hybrid 2d",
        "Salary range": "your salary floor",
        "Location": "London",
        "Next action": "Follow up if no response",
        "date:Applied date:start": "2026-05-15",
        "date:Applied date:is_datetime": 0,
        "date:Next action date:start": "2026-05-22",
        "date:Next action date:is_datetime": 0,
        "date:Last contact:start": "2026-05-15",
        "date:Last contact:is_datetime": 0,
        "Notes": "Tailored CV. Salary band confirmed in JD."
      },
      "content": "## Job Description\n\n[Paste the JD from the job posting here]"
    }
  ]
}
```

### Read-then-update pattern (Actions 3-6 — status flips, field changes)

Notion's `notion-update-page` with `command: "update_properties"` is **additive on property level** — fields you don't include stay unchanged. You don't need to copy unchanged fields (unlike the old Sheet's read-merge-write requirement). But you DO need to find the right page first.

1. **Find the page:** `notion-search` with `query: "<Company> <Role>"` and `data_source_url: "collection://{{NOTION_OPPORTUNITIES_DS}}"`, `page_size: 3`. Returns matching page IDs.
2. **Confirm match:** if multiple results, surface the top 3 to the user: "Found 3 rows — which? 1. [Company A] – [Role A] (Status), 2. ...". If single high-confidence match, proceed.
3. **Read current state (optional but recommended for Notes appends):** `notion-fetch <page_id>` returns full property map. Read existing `Notes` if you're appending.
4. **Update:** `notion-update-page` with `page_id`, `command: "update_properties"`, `properties: {...}` containing only the fields that change.

Example — Apply event response to Recruiter screen:

```json
{
  "page_id": "<from notion-search>",
  "command": "update_properties",
  "properties": {
    "Status": "📞 Recruiter screen",
    "Temperature": "🔥 Hot",
    "Next action": "Prepare for screen",
    "date:Next action date:start": "2026-05-20",
    "date:Next action date:is_datetime": 0,
    "date:Last contact:start": "2026-05-15",
    "date:Last contact:is_datetime": 0,
    "Notes": "<existing notes>\n\n2026-05-15: A recruiter reached out, 30-min recruiter video booked..."
  }
}
```

### Date property format

Notion dates use the **expanded form** on writes:
- `date:<Property name>:start` — ISO date or datetime string (`YYYY-MM-DD` for date-only)
- `date:<Property name>:end` — for date ranges, omit for single dates
- `date:<Property name>:is_datetime` — `0` for date-only (default), `1` for datetime

To **clear** a date: write `null` to `date:<Property name>:start`.

### Pre-flight self-check (run mentally before EVERY Notion write)

1. Am I using the correct `data_source_id` (`{{NOTION_OPPORTUNITIES_DS}}`)? → If creating a new row, the parent must be a `data_source_id`, not the `page_id` of the DB itself.
2. Are all SELECT values **exact emoji-prefixed strings** matching the property options? → Common gotcha: writing `"Applied"` instead of `"📨 Applied"` will error.
3. Are dates in expanded form (`date:X:start` + `date:X:is_datetime`)? → Inline date objects won't parse.
4. For status flips: did I find the correct page via `notion-search` before calling `notion-update-page`? → A search miss + create_pages creates a duplicate row. When in doubt, narrow the search query.
5. For Dead transitions: am I setting `Dead reason`? → Required by the AC, gives weekly retro signal.
6. For Notes appends: did I read the existing Notes via `notion-fetch` before writing? → `update_properties` replaces Notes wholesale unless I include the existing text in the payload.

### Error handling

- **API rate limits:** Notion soft-limits at 3 req/s. For batches >5 rows, space requests ~200ms apart. `notion-create-pages` accepts up to 100 pages per call — prefer one big call over many small ones.
- **Property not found errors:** the property name changed. Re-fetch the data source via `notion-fetch collection://{{NOTION_OPPORTUNITIES_DS}}` and verify against `config.local.md`.
- **Multi_select validation errors:** the SELECT option doesn't exist. Re-fetch and check exact spellings (emoji-prefixed). If you need a new option, ask the user rather than silently adding one.

## Comments as Progress Log

Every status change should include a Todoist comment on the task. This creates a timestamped activity log that the user can scan later without reading the full Notion Notes field. Use the `add-comments` tool with the task ID.

Format: `**[DD Mon]** — [What happened]`

Examples:
- `**16 Apr** — Applied via LinkedIn Easy Apply. Track B.`
- `**18 Apr** — Recruiter pre-screen done. Waiting for CV to be shared with the hiring team.`
- `**22 Apr** — Interview scheduled for 25 Apr, 2pm, video call with hiring manager.`
- `**25 Apr** — Rejected. Feedback: went with an internal candidate.`

Keep comments concise — one or two sentences max. The Notion `Notes` field holds the structured narrative; Todoist comments tell the story timeline-style on the action layer.

## Execution Order

Always update Notion first, then Todoist. Reason: the Notion page URL is needed for the Todoist task's description (cross-link). Order:

1. Create or update the Notion row → capture the page URL (`https://www.notion.so/<page_id>`)
2. Create or update the Todoist task → include the Notion URL in the task description
3. Add a progress comment to the Todoist task
4. Confirm to the user what was done

## Confirmation Format

After every action, confirm concisely:

> ✅ **[Company] — [Role]** moved to [Status]. Follow-up set for [date]. Notion + Todoist updated.

Or for new entries:

> ✅ **[Company] — [Role]** logged as [Track]. Applied [date], follow-up [date]. Notion row: [link]. Todoist task created.

Keep it to one or two lines. No verbose summaries.

## Source Format

When logging the source, write the exact Notion SELECT option:
- `LinkedIn` — found and applied directly on LinkedIn
- `Reed` / `Adzuna` / `Otta` — found on the named platform
- `Referral` — referred by someone (capture the person's name in Notes)
- `Direct` — applied via company careers page or other direct channel
- `Other` — anything else (Hackajob, Wellfound, Cord, etc. — capture details in Notes)

## Error Handling

- If Notion update succeeds but Todoist update fails: tell the user the Notion row is updated but the Todoist task needs manual attention. Don't silently drop it.
- **If updating a row that might not exist in Notion:** call `notion-search` first. If the company+role isn't found, use `notion-create-pages` instead of `notion-update-page`. When in doubt, prefer creating — a duplicate row is easier to fix than a silently failed update against a non-existent page.
- If the user gives incomplete info (e.g., "I applied to Northwind Trading" but no role title): check if a Notion row or Todoist task already exists for that company. If yes, use the existing data. If no, ask for the missing field.
- If a company name matches multiple Notion rows or Todoist tasks: list them and ask which one.

## Design Rules (core invariants)

- Every active application must exist in Notion (primary) and have a corresponding Todoist task (action layer).
- No Todoist task without a due date — if there's no due date, there's no next action, and it will rot.
- **Due date = next follow-up date** for all live applications. Driven by recruiter timeline + 1 day if given, else applied/last-contact + 5 business days. See "Follow-up Timing Rules" in Section 2.
- **3-week backstop (silent applications only):** if no human contact 3 weeks post-application + 2 nudges sent → prompt the user to confirm Dead. Not auto-Dead. Recruiter-engaged roles are exempt from the backstop entirely.
- 45-minute ceiling per Track A application, 5–8 minutes per Track B.

## v5.0.0 changelog

- **Notion is sole source of truth.** Replaced Google Sheet (13-column model) with Notion Opportunities DB (19 properties). Make.com webhooks (`add_row`, `update_row`, `read_rows`, `delete_row`) decommissioned and archived.
- **Schema enrichment.** Added properties: `Track`, `Temperature`, `Work model`, `Location`, `Dead reason`, `Days since last contact` (formula). All Track / Excitement / Temperature state moved from Todoist labels → Notion SELECTs.
- **Status taxonomy expanded.** Old 5-state Sheet status (`Sourcing / Applied / Interviewing / Offer / Dead`) replaced with 9-state Notion taxonomy with emoji prefixes (`🔍 Sourcing → 📨 Applied → 📞 Recruiter screen → 👤 Hiring manager → 📝 Take-home → 🎯 Final stage → 💌 Offer → 🎉 Offer accepted / ⚰️ Dead`).
- **Scan-ID batch syntax removed.** Quaternary input flow (scan-ID resolution from job-market-scan reports) retired. Users source directly via LinkedIn; scan-ID resolution is dead code.
- **Todoist labels reduced.** `track-a`, `track-b`, `excitement-1` through `excitement-5` deleted. Only `job-search` + `hot`/`warm`/`cool` remain.
- **System section exclusion rule preserved.** Wording updated to reflect dead labels.
- **`.skipped-roles.jsonl` contract preserved** — appended on skip events for future sourcing filter dedup.
