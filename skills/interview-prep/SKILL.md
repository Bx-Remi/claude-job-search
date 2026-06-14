---
name: interview-prep
description: >
  First-round interview prep scaffolding. v0.2 covers ONE stage only: first-round screens —
  recruiter screens AND hiring-manager intro/fit calls (~30 min, conversational, non-technical).
  Use when the user says "prep for interview", "help me prepare for [company]", "interview with
  [company] tomorrow", "[company] screen", "[company] interview prep", "prep [company]". Auto-pulls
  context from Google Calendar (event), Gmail (recruiter chain), Google Drive (Master CV +
  any local JD PDF), and a tight web snapshot (company news + interviewer LinkedIn). Prompts the
  user to paste the Todoist task body — Cowork cannot reach Todoist directly, and Todoist is the
  spine of the prep. Produces TWO deliverables: (1) Claude-facing markdown prep doc at
  `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/interview-prep.md` (per-interview folder,
  12 sections, exploratory working surface for Claude), and (2) user-facing Notion page at the
  matching Opportunity row in the Opportunities DB, mirroring the template (status header + prep
  checklist + Prep Notes section with H3 + callout blocks per checkbox; Prep Notes filled
  interactively during prep conversations). Refuses out-of-scope stages (technical interviews,
  take-homes, case studies, panel/final/multi-stage day, salary negotiation, post-interview
  thank-you/follow-up automation) with a generic-offer fallback. Runs in Cowork OR Code; in Code,
  can also fall through to the local Todoist MCP instead of asking for the paste.
metadata:
  version: "0.3.0"
---

## Configuration (read this first)
All IDs and paths below are placeholders of the form `{{PLACEHOLDER}}`. Before doing anything, read `config.local.md` at the repo root and resolve every placeholder to the user's real value. If `config.local.md` is missing or a required value is blank, STOP and tell the user to complete `SETUP.md` first. Optional sources (Google Drive, Gmail, Calendar) may be unconfigured — when a source is missing, note "(not configured — context skipped)" and continue; never block on an optional source.

# Interview Prep — v0.3.0 (Notion Opportunities DB as cross-check source; per-opportunity child pages replace legacy parent)

This skill lowers the user's activation energy before a first-round interview. Without it, the user sits down to prep, does not know where to start, and procrastinates or under-preps. With it, they get TWO paired artefacts: (1) a Claude-facing 12-section markdown doc used as a working surface during prep conversations, and (2) a user-facing Notion page they can glance at the morning of the interview, with verbatim answers and anchor beats filled in as prep conversations progress.

## Why This Exists

Interview prep often feels scattered: no clear starting point, no defined "enough" line. Every previous prep ends in either burnout (over-research) or under-prep (could not get going). This skill encodes a fixed ordered process so the cognitive load is "follow the steps" rather than "decide what to do." The MD is Claude's working brain (comprehensive, exploratory); the Notion page is the user's glance surface (curated, ready-to-skim). Two artefacts, paired, not ten scattered files.

## Scope (v0.2)

**IN scope:**
- First-round recruiter screens (~30 min, conversational, motivation/expectations/logistics).
- First-round hiring-manager (HM) intro/fit calls (~30 min, conversational, role specifics + team fit + why-you).

**OUT of scope (v0.3+):**
- Technical interviews (system design, SQL, coding, take-home tasks, case studies).
- Final-round / panel / multi-stage day logistics + storytelling depth.
- Salary negotiation prep + offer-evaluation framework.
- Post-interview thank-you / follow-up automation. The pipeline status flip is `job-search-action`'s job, not this skill's.

**Out-of-scope detection branch (run BEFORE Step 1).** If the user's invocation message OR the matched calendar event mentions: `technical`, `take-home`, `case`, `case study`, `panel`, `final round`, `multi-stage`, `offer`, `negotiation`, `thank you`, `follow-up`, `post-interview` — STOP and respond with the generic-offer fallback below. Do NOT silently produce a first-round-shaped prep doc for a stage it is not sized for.

**Generic-offer fallback script:**

> "This looks like a [technical / panel / final / negotiation] stage. v0.1 of `interview-prep` only covers first-round screens (recruiter + HM intro/fit). Here's what I can offer generically right now without running the full flow: [3-5 bullets tailored to the stage from the table below]. For full prep on this stage, treat as ad-hoc this round and I'll flag it for v0.2."

| Stage | Generic bullets to offer |
|---|---|
| Technical | Write down your loudest 2-3 system-design / SQL / data-model wins from the last 18 months. Re-skim the JD's tech bullets and rate self 1-5 on each. Pick one to "warm up" on with a 30-min refresher. Practice reasoning out loud, not silently. |
| Panel / final | List each interviewer + their angle. One STAR story per angle. One sharp question per interviewer. Test the link/location 24h before. Plan a 15-min decompression buffer between back-to-back blocks. |
| Negotiation | Write down: target / floor / walk-away. Get the offer in writing first. Don't counter same-call. Use silence as a tool. Anchor with one number, not a range. |
| Follow-up | Send within 24h. One specific thread from the conversation, NOT a generic thank-you. Cc the recruiter if HM did not include them. |

If the user confirms they want to proceed despite the out-of-scope flag ("yeah just do first-round-style for it"), proceed but mark the doc header `⚠️ Stage was [X] — first-round template applied per user's request, may not be fully fit-for-purpose`.

## Cowork compatibility note

This skill is designed to run in Cowork. Cowork cannot reach the Todoist MCP — that is a hard technical limit, not a preference. Therefore Step 3 prompts the user for a narrow paste-in. If running in Code (where the Todoist MCP IS reachable), the skill can call `mcp__todoist__find-tasks` directly instead of prompting; either path is fine, but never substitute Notion/Sheet content as a replacement for Todoist content (the pipeline DB often has no row for referral-sourced opportunities and is missing prep-relevant fields).

**Detection:** if the Todoist MCP tools (`mcp__todoist__find-tasks` etc.) are listed in the available tools, you are in Code — use them. Otherwise you are in Cowork — use the paste-in prompt.

## Trigger phrases

Fire on any of:

- "prep for interview"
- "help me prep[are] for [company]"
- "interview with [company] tomorrow"
- "[company] screen" / "[company] interview prep"
- "prep [company]" / "let's prep [company]"
- "prep [company] interview"
- "/interview-prep" (slash form, optional)

Do NOT fire on:

- "I just had an interview with [company]" — that is `job-search-action` (status flip)
- "rejected by [company]" — that is `job-search-action`
- "follow up on [company]" — that is `job-search-action`'s next-action handling
- "what's coming up this week" — that is `morning-briefing`

## Inputs (data sources)

| Prep-doc section | Source | Auto / paste | Notes |
|---|---|---|---|
| Logistics (date / time / Meet link / interviewer) | Google Calendar | Auto | Read offset from `dateTime`, NOT `start.timeZone`. Domain fallback if title does not contain company. |
| Opportunity snapshot (salary / IC vs PAYE / match score / excitement) | Todoist task body | Paste (Cowork) / Auto (Code) | The richest single source; without it the doc is materially degraded. |
| Application log / referral chain | Todoist task body + Gmail | Paste + Auto | Cross-check Todoist log against Gmail thread dates. |
| Pipeline row (cross-check) | Notion Opportunities DB | Auto | Often empty for referral-sourced roles; flag once but do not block. |
| Recruiter / HM identification | Gmail thread | Auto | Apply sender blocklist (no-reply, marketing, transactional). |
| JD content | local PDF path (from Todoist) then Gmail attachment then Drive then user paste | Auto with fallbacks | |
| Interviewer background | Web search (LinkedIn) | Auto | One search, top 3 hits. Do not deep-research. |
| Company snapshot (recent news) | Web search | Auto | One search, top 3-5 headlines. |
| Master CV (for STAR matching) | Google Drive | Auto | File ID `{{DRIVE_MASTER_CV}}`. Only used to fill gaps Todoist has not already mapped. |
| Pre-drafted prep angles + questions to ask | Todoist task body | Paste / Auto | Lift verbatim; do NOT re-invent. |

Reference IDs (resolve all of these from `config.local.md` before using):

- **Notion Opportunities DB** (single source of truth for pipeline state, v5.0.0+):
  - Database page ID: `{{NOTION_OPPORTUNITIES_DB}}`
  - Data source ID: `{{NOTION_OPPORTUNITIES_DS}}`
  - Parent page: `{{NOTION_PARENT_PAGE}}`
- Master CV (Google Doc): `{{DRIVE_MASTER_CV}}`
- Job Search Drive folder: `{{DRIVE_JOBSEARCH_FOLDER}}`
- Job Search Todoist project: `{{TODOIST_PROJECT}}` (post-v5.0.0 cutover: no pipeline-state sections — only `System` remains; new tasks land sectionless)

## Hard constraints (every invocation)

These are the rules with the highest cost of violation. Re-check before every prep-doc write.

- **Do not use em dashes anywhere in the prep doc.** The doc is interview-adjacent (it shapes the user's external speech and answers). Use commas, semicolons, colons, parentheses. The em-dash ban does NOT apply to this SKILL.md itself or to internal Claude-to-user conversation.
- **No fabrication.** If a metric, name, date, or company fact is not sourced from Todoist / Gmail / JD / Web / CV, mark `[research required]` rather than guess. Applies especially to interviewer LinkedIn details and recent-news headlines.
- **Loud and confident voice in pitch + STAR stories.** No "small", "modest", "rather than aspirational", "haven't held this title before". Frame progression, not gap.
- **Preserve the user's pre-drafted content.** If Todoist already lists prep angles, questions to ask, or a tool prep plan, lift them verbatim into the doc. Do NOT re-write or "improve" them. Only ADD what is missing.
- **Two paired deliverables.** (1) Claude-facing markdown at `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/interview-prep.md` (per-interview folder, 12 sections, exploratory). (2) User-facing Notion page **as a child page under the matching Opportunity row in the Opportunities DB** (`{{NOTION_OPPORTUNITIES_DB}}`), mirroring the template (status header + prep checklist + Prep Notes section). No interim notes file, no separate pitch file, no separate questions file. The two artefacts pair: MD is Claude's working brain; Notion is the user's glance surface. They evolve together during prep conversations.
- **Reduce decision load.** Skill picks defaults (which thread to pull, which angles to surface, which Web headlines to keep). It does not ask the user which to choose.

## Skill flow

### Out-of-scope detection (Step 0)

Before anything else, scan the user's invocation message AND (once Step 1 has run) the calendar event title/description for the out-of-scope keywords listed above. If any match: emit the generic-offer fallback and stop, unless the user explicitly opts in.

### Step 1 — Identify the interview

1. Read the system clock: today's date.
2. Search Google Calendar for events in the next 7 days where:
   - Event title or description contains the company name the user mentioned, OR
   - An attendee email's domain matches a known company-domain map (build a per-company map as you encounter domain patterns; note that recruiters sometimes email from a parent-company or legacy domain — confirm the sender's domain rather than assuming it matches the brand name).
3. If multiple events match, pick the soonest. If still ambiguous, ask the user: "Found multiple matches: [list]. Which one?"
4. Extract:
   - **Date + time:** read the `+HH:MM` offset from `dateTime`, NOT the `start.timeZone` field (it can lie — a calendar event may declare one timezone while the actual UTC offset differs). Render as both the user's local time AND the interviewer's local time (inferred from organiser timezone).
   - **Conference URL** (Meet / Zoom / Teams) — capture for the Logistics section.
   - **Organiser email** — this is the interviewer (or recruiter; Step 2 disambiguates).
   - **Other attendees** — sometimes hints at panel; flag if >2 attendees other than the user (could be out-of-scope).

If no event found within 7 days, ask the user: "I can't find a calendar event for [company] in the next 7 days. Is the interview booked yet, or different timing?" If not yet booked, stop — there is nothing to prep against; suggest booking first.

### Step 2 — Confirm stage (recruiter vs HM intro)

If the organiser's email signature mentions:

- "Senior Manager", "Manager", "Director", "Head of", "VP", "Lead" — **HM intro/fit** (most likely; hiring managers usually have IC reports).
- "Recruiter", "Talent Acquisition", "Sourcing", "TA", "People & Culture" — **recruiter screen**.
- Ambiguous or no signal — ask the user once: "Is this a recruiter screen or HM intro? (affects which angles I emphasise)"

If the user has already specified the stage in the invocation, skip the ask.

Stage influences emphasis in the prep doc:

- **Recruiter screen:** emphasise compensation expectations, motivation / why-this-role / why-now, logistics, simpler pitch. De-emphasise deep team-fit angles.
- **HM intro/fit:** emphasise team fit, role specifics, why-you-for-this-team, smart team-process questions. De-emphasise "what's the salary range" (already covered with recruiter, or save for end).

### Step 3 (PRIMARY INPUT) — Pull Todoist task body

If the Todoist MCP is available (Code session): call `mcp__todoist__find-tasks` with `searchText: "<company>"` and `limit: 5`, projectId `{{TODOIST_PROJECT}}`. Pick the matching task. Lift the full description.

If not (Cowork session): emit this exact prompt and wait:

> **Paste the Todoist task body for [Company]**
>
> This skill runs in Cowork which cannot reach your Todoist. Open Todoist, search "[Company]", copy the FULL task body (description + any comments), and paste below.
>
> This is the spine of the prep doc. Without it, several sections will be marked "(missing — Todoist context not provided)".
>
> Paste here, or type "skip" to continue degraded:

If the user types "skip", proceed without it; mark all Todoist-derived sections as "(missing — Todoist context not provided)".

If pasted, parse the body for these structured fields (do not be brittle — task bodies use markdown-like prose, look for the labels):

- **Status / interview details:** "Status:", "Interview details:" — time / format / link / interviewer
- **Salary:** "Salary:", "Asking", "£", "$", "day rate" — target compensation + structure (PAYE vs IC vs contract)
- **Type / Location:** "Type:", "Location:", "Remote", "Hybrid", "On-site"
- **Match / Excitement:** "Match:", "Excitement:" — numeric scores
- **Company:** "Company:" — 1-line description
- **Referral:** "Referral:" — who referred + their relationship to the company
- **Recruiter / Hiring Manager:** "Recruiter", "Hiring Manager" — name + title + email
- **Application log:** bulleted timeline with dates — preserve verbatim for the Application log section
- **JD summary:** "JD summary", "You Will:", "You Have:", "Bonus Points:" — preserve verbatim
- **Interview prep angles:** "Interview prep angles:", "prep angles" — numbered list, preserve verbatim
- **Gap-closing prep plan:** "prep plan", any "gap" / "self-study" mentions — preserve verbatim
- **Key questions:** "Key questions for [interviewer]", "Questions to ask" — preserve verbatim
- **JD source file:** local file path — use in Step 6

### Step 4 — Pull pipeline row (cross-check)

Query the Notion Opportunities DB for the matching row:

1. `notion-search` with `query: "<Company> <Role>"`, `data_source_url: "collection://{{NOTION_OPPORTUNITIES_DS}}"`, `page_size: 3`.
2. If a single high-confidence match returns, `notion-fetch <page_id>` to read the row's properties.
3. Cross-check Notion `Status` / `Salary range` / `Next action` / `Excitement` / `Track` / `Temperature` against the Todoist body. If they conflict (rare), trust Todoist for active opportunities (Todoist is the action-layer truth on a live application timeline; Notion is the structured snapshot).
4. Capture the Notion row's page ID — needed in Step 11 to parent the prep page under it.

If no row exists: surface ONE quiet flag in the prep doc's "Open gaps" section ("Notion Opportunities DB has no row for this opportunity — referral-sourced log gap"). Do NOT prompt the user to fix it now — that is a separate `job-search-action` task and not blocking for the interview.

If multiple rows match (e.g. the user applied to two different roles at the same company): surface the top 3 results and ask which row to use for the prep cross-check + Step 11 parent. Default to the most recently-touched row if the user does not reply.

### Step 5 — Pull Gmail correspondence

Search Gmail threads for the company name. Apply this sender blocklist:

- `no-reply@*`
- `noreply@*`
- `purchase-noreply@*`
- `notifications@*` (most platforms)
- Anything containing `marketing`, `newsletter`, `promotions` in the local-part.

Note: recruiters sometimes email from a parent-company or legacy domain rather than the brand domain — confirm the sender's actual domain matches the expected company rather than assuming it. Build a per-company domain map (e.g. brand name + any parent or historical domains) and check all of them as you encounter them.

`newer_than:6w` combined with a free-text term has been observed to silently return empty in Cowork (query parser quirk). Drop `newer_than` or use absolute `after:YYYY-MM-DD` instead.

Surface from the threads:

- Recruiter / HM name and signature title (cross-check Step 2).
- Referral chain: any cc'd or forwarded internal contacts (preserve the chain).
- JD attachment location if present (use in Step 6).
- Anything in correspondence not already in the Todoist body (e.g. last-minute reschedule notes, recruiter's pre-interview brief).

### Step 6 — Locate and parse the JD

Priority order:

1. **Local file path from Todoist body.** If the path exists, read the file using the Read tool with the `pages` parameter if needed.
2. **Gmail attachment** from the matched thread (use Gmail MCP tools to fetch attachment).
3. **Google Drive search** for the JD as a standalone file (uncommon).
4. **Prompt the user to paste** the JD text inline ("JD not in any auto-source — paste the text or skip").

Use the parsed JD to:

- Verify Todoist's "You Have" mapping against actual JD bullets.
- Extract 3-5 key requirements for the Role recap section.
- Cross-reference against the Master CV (Step 7) for STAR-story matching.

If JD is unavailable, mark the Role recap section "(JD not retrieved — work from Todoist's JD summary if logged)".

### Step 7 — Pull Master CV (gap-fill only)

Read Master CV from Google Drive (file ID `{{DRIVE_MASTER_CV}}`). DO NOT use it to override the user's Todoist-logged prep angles. Only use it to:

- Identify any obviously missing competency from the JD that Todoist's prep angles do not cover.
- Pull the underlying STAR stories that Todoist's prep angles reference.

If Todoist had no prep angles (degraded mode), use the CV plus JD to draft 3-5 prep angles from scratch — but mark the section "(drafted by skill, no Todoist input — verify before relying on these)".

Cross-reference for richer context: `{{STORIES_DIR}}/*.md` if available — these may have fuller story context than the CV bullets.

### Step 8 — Web snapshot (capped)

Two web searches max:

1. `"<company>" news <current year>` — top 3-5 headlines for Company snapshot. Filter out spam / SEO listicles. Prefer official company posts, reputable trade press.
2. `"<interviewer name>" "<company>" LinkedIn` — top 3 hits for Interviewer background. Prefer the LinkedIn profile result + 1-2 other signals (talks, podcasts, articles they have written). If no clear LinkedIn match in the top 3, mark "(LinkedIn not auto-located — search manually before the call)".

Hard cap: 2 searches. If you find yourself wanting a 3rd, mark the section "[research required]" and move on.

### Step 9 — Create the per-interview folder

Build the folder path: `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/`

Folder naming rules:
- `<Company>` and `<Role Title>` use the values exactly as logged in Todoist / pulled from the JD, preserving capitalisation and spacing.
- Join with ` — ` (space + em-dash + space). Em-dash is allowed in folder names; the em-dash ban applies to prep-doc content, not file system names.
- Examples: `Northwind Trading — Customer Operations Analyst/`, `Westbrook Health — Operations Coordinator/`.

If the parent `{{PREP_OUTPUT_DIR}}/` directory does not exist, create it. If the per-interview folder already exists (re-running the skill for a known company), do not overwrite — ask the user: "Folder already exists for this interview. Refresh the MD in place, append a `interview-prep-v2.md`, or skip?" Default to refresh in place if the user does not reply.

### Step 10 — Write the Claude-facing MD

Write the file to: `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/interview-prep.md`

Use the prep-doc template below (12 sections, MD format). This is Claude's working surface — comprehensive, exploratory, may include speculation/options. It is not what the user reads tomorrow morning (that is the Notion page); it is what Claude refers to when filling the Notion page during prep conversations.

### Step 11 — Create the user-facing Notion page (as child of the Opportunity row)

Create a child page under the **matching Opportunity row** in the Notion Opportunities DB (page ID captured in Step 4). Use the `mcp__b7e16cc0-427c-48d8-8ad2-fa00234ac06e__notion-create-pages` tool with `parent: {type: "page_id", page_id: "<opportunity_row_id>"}`.

Page properties:
- **Title:** `<Company> — <Role Title>` (matching the folder name); for multi-round interviews, add a round suffix e.g. `<Company> — <Role> — Stage 2`
- **Icon:** company emoji if obvious (otherwise leave default)
- **Parent:** the matching Opportunity row's page ID

Use the Notion page template below (status header + prep checklist + empty Prep Notes section). The Prep Notes section starts empty — it is filled interactively during prep conversations, not at scaffold time.

If a child page with the same title already exists under the Opportunity row, ask the user: "Prep page already exists for this interview. Update the existing page, or create a new one (e.g. for Stage 2)?" Default to "update existing" if the user does not reply.

### Step 12 — Surface to user

After writing both artefacts, emit a one-line summary:

```
Scaffolded prep for <Company> — <Role Title>:
   - MD: {{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/interview-prep.md (<X>/12 sections complete, <gaps flagged>)
   - Notion: <page URL>
   Suggested read-time: morning of the interview, allow ~15 min on the Notion page.
```

Then stop. Do NOT auto-create a calendar block, do NOT auto-add a Todoist task, do NOT auto-open either artefact. The user reads on their own schedule.

## Interactive prep mode (post-scaffold)

After scaffolding, the user may invoke Claude conversationally to fill the Notion page's Prep Notes section with verbatim answers, anchor beats, and competency coverage. This is a separate mode from scaffold-time generation:

- Each prep checklist item gets its own H3 in Prep Notes, followed by a callout block (icon "speak", colour `gray_bg`) containing the verbatim answer, then delivery notes / anchor beats in bold-labelled bulleted lists.
- Add an empty block between sub-sections (callout / delivery notes / anchor beats) for breathing room.
- Working flow per item: Claude drafts, user reviews/redirects, Claude ships to Notion + ticks the checkbox, then move on. One item at a time.
- Re-fetch the Notion page before any multi-op `notion-update-page` call if there has been an interactive gap (the user may have edited the page directly; stale `old_str` values fail silently).
- All canonical STAR stories live at `{{STORIES_DIR}}/*.md` — always check those first before drafting a story from scratch.

## Prep-doc template

Write the file with this exact structure. Section count: 12. Section order matters (Logistics first because that is what the user opens to verify "do I have the link, what time").

```markdown
# Interview Prep — <Company> — <Role>

**Date:** <Day, DD Mon YYYY>
**Time:** <HH:MM> <user TZ> / <HH:MM> <interviewer TZ abbrev>
**Stage:** <Recruiter screen | HM intro/fit>
**Doc generated:** <YYYY-MM-DD HH:MM> by `interview-prep` v0.3.0

---

## 1. Logistics

- **Conference link:** <URL> <- TEST THIS BEFORE THE CALL
- **Backup contact:** <email or phone if logged anywhere>
- **Who you're meeting:** <Name>, <Title>
- **Length:** <30 min, etc.>
- **Format:** <Google Meet / Zoom / Teams / phone>
- **What to wear / setup:** <video on, business casual default for fit calls; check lighting + audio 30 min before>

---

## 2. Opportunity snapshot

- **Match score:** <X/10> | **Excitement:** <hard X / soft X>
- **Salary target:** <target or day rate or range>
- **Role type:** <Permanent / IC / contract length>
- **Location:** <Remote / Hybrid / On-site + city>
- **Key contract caveats:** <e.g. "12-month contract, day rate to clarify">

---

## 3. Company snapshot

<1-2 sentences on what they do, stage, ownership.>

**Recent news (last 6 months):**
- <Headline 1 + 1-line context>
- <Headline 2 + 1-line context>
- <Headline 3 + 1-line context>

---

## 4. Role recap

Top requirements from the JD (lifted, not paraphrased):

1. <Requirement 1>
2. <Requirement 2>
3. <Requirement 3>
4. <Requirement 4>
5. <Requirement 5>

<If JD not retrieved: "JD not retrieved. Working from Todoist's JD summary if logged.">

---

## 5. Interviewer

**<Name>** | <Title> | <Company>

- **Background:** <2-3 lines from LinkedIn / web search>
- **Tenure at <Company>:** <X years if visible>
- **Referral context:** <if applicable>

**2 specific questions to ask THEM (warmer than generic "tell me about your team"):**
- <Question tied to their background or recent role>
- <Question tied to a specific company / team event>

---

## 6. Application log

<Lifted verbatim from Todoist's application log. Bulleted, dated, terse.>

- <Date> — <event>
- <Date> — <event>

---

## 7. 60-90 sec pitch

<Drafted from CV + JD + Todoist match angles. First-person, conversational, no em dashes. Loud and confident.>

> "Hi <Interviewer first name>, I'm [name]. <2 sentences on background — most recent role + the through-line that connects to this role>. <2 sentences on why this role / why this company specifically — tie to JD angle 1>. <1 sentence on what you're looking for — tie to role type / any gap closing if relevant>."

<If Todoist already had a pitch logged: lift verbatim, mark "(from Todoist — review before delivering")".>

---

## 8. Prep angles + STAR stories

<Lift Todoist's numbered prep angles VERBATIM if present. Add 1-2 only if a JD requirement is uncovered.>

1. **<Angle 1>** — STAR story: <Situation> | <Task> | <Action> | <Result with numbers>
2. **<Angle 2>** — ...
3. ...

---

## 9. Gap-closing prep plan

<Lift verbatim from Todoist if present.>

- <Plan item 1>
- <Plan item 2>

<If no gap logged: "No specific gap flagged. Spend 30-45 min on Glassdoor reviews + recent product launches the morning of.">

---

## 10. Likely questions + prepared angles

<5-8 items. Skill-drafts unless Todoist already has them. First-round screens skew to: motivation, fit, salary, logistics, role specifics, "tell me about a time...">

- **"Why <Company>?"** — <2-sentence angle tied to JD + company news>
- **"Why this role?"** — <angle tied to career arc + role type>
- **"What's your current situation?"** — <progression framing — never "between jobs">
- **"Walk me through a time you handled high-volume work"** — STAR story <ref to angle 1>
- **"What are your salary expectations?"** — <"In line with the market for this role type — happy to discuss specifics if you have a range in mind">
- ...

---

## 11. Questions to ask them

<Lift Todoist's pre-drafted set first. Add 1-2 stage-appropriate.>

- <Question 1>
- <Question 2>
- <Question 3>
- <Question 4>
- <Question 5>

---

## 12. Open gaps / things to confirm before walking in

<Anything the auto-pull and paste-in left blank. Be honest.>

- [ ] <Gap 1 — e.g. "LinkedIn for interviewer not auto-located, search manually">
- [ ] <Gap 2 — e.g. "Day rate range not confirmed, ask in opening minutes">
- [ ] Test the conference link 30 min before
- [ ] Re-read the JD bullet by bullet 60 min before
- [ ] One bio break + glass of water immediately before
```

## Notion page template

Write the Notion page with this exact structure. Status header values come from the Todoist task body + JD; the prep checklist is a starter set (the user will customise); Prep Notes starts empty.

```markdown
**Status:** Interviewing — <Day DD Mon> <HH:MM>-<HH:MM> <user TZ> (<HH:MM> <interviewer TZ>) <format> booked.
**Salary:** <range or TBD or "Asking [target]">
**Location:** <Remote / Hybrid / On-site + city>
**Type:** <Permanent / Independent Contractor / Contract (~Xmo)>
**Company:** <Company> (<context, e.g. "B2B SaaS, logistics analytics platform">)
**Referral:** <Name (relationship to company)> — if applicable, otherwise omit
**Interviewer:** Hiring Manager — <Name>, <Title> (<email if known>)
**Team:** <team context lifted from Todoist / JD>

---

## **To prepare for this interview specifically:**

- [ ] Introduction/Presentation
- [ ] Why this role / Why this company
- [ ] Work gap answer (if applicable)
- [ ] Power Story A — high-volume / queue / ops scaling probe
- [ ] Power Story B — process improvement OR cross-functional probe
- [ ] Salary expectations (recruiter screens) OR compensation deflection (HM screens)
- [ ] Questions for the interviewer
- [ ] <JD-specific item 1, lifted from JD bullets>
- [ ] <JD-specific item 2>
- [ ] ... (user extends with role-specific items)

---

## Prep notes

<Empty at scaffold time. Filled interactively during prep conversations using the Interactive Prep Mode pattern (H3 per checkbox + callout for verbatim + delivery notes + anchor beats + empty-blocks for breathing room).>
```

**Starter checklist rationale.** The 7 fixed items (Intro, Why this role, Gap, two power stories, Salary, Questions) cover the universal first-round-screen probes. The JD-specific items come from parsing the JD's "You Have" / "You Will" bullets and asking the user which 3-5 to anchor on. The user can add / remove / reword freely — the starter is scaffolding, not a contract.

**Power story selection.** When the JD has both a high-volume operational probe AND a process-improvement probe, use Power Story A for high-volume and Power Story B for either process improvement or cross-functional. Always check `{{STORIES_DIR}}/*.md` first to see which canonical stories are already polished.

## Worked example: Northwind Trading (smoke test + v0.2 migration)

The original v0.1 smoke test used the fictional Northwind Trading / Sam Carter interview (Thu 2026-04-10 14:00 BST) as the canonical reference. v0.2 retroactively restructured it into the new convention.

This is the smoke test that built the skill. Use as the canonical example for the v0.1 behaviour.

**Inputs found:**

- GCal: "Interview — Operations" (NOT "Northwind Trading") on 2026-04-10 14:00-14:30 BST, Google Meet `abc-defg-hij`, organiser `s.carter@northwindlogistics.com` (Sam Carter). Calendar event title did not contain the brand name — domain match on `@northwindlogistics.com` was the only way to find the event.
- Stage: HM intro/fit (Sam's signature: "Head of Customer Operations, Northwind Trading").
- Todoist body: rich — salary target GBP 42K PAYE, match 7/10 / excitement hard 6, full JD summary, 5 pre-drafted prep angles, CRM gap-closing plan (self-study module + peer walkthrough booked for the weekend), pre-drafted questions for Sam, JD local path saved in task notes, application log January 2026 to April 2026.
- Notion Opportunities DB: NO row for Northwind Trading (referral-sourced, never formally logged in the pipeline) — quietly flagged in Open gaps section.
- Gmail: 3 signal threads (Sam's intro email, calendar invite, referral introduction from a mutual contact); several noise threads filtered (automated delivery notifications from northwindlogistics.com marketing system).
- Note: recruiters sometimes email from a parent-company or legacy domain — confirm the sender's domain rather than assuming it matches the brand name. In this case all signal threads came from `@northwindlogistics.com` directly.
- Web: Northwind Trading news 2026 (new Southeast Asia logistics corridor announced March 2026, Q1 earnings beat, sustainability report release); Sam Carter LinkedIn search returned 1 strong hit (Head of Customer Ops, 4 years tenure).
- Master CV `{{DRIVE_MASTER_CV}}`: only used to fill the CRM gap — pulled relevant transferable experience bullets to support the gap-closing narrative.

**Output (v0.1 original, since restructured for v0.2):**
- MD at `{{PREP_OUTPUT_DIR}}/Northwind Trading — Customer Operations Analyst/interview-prep.md` (migrated 2026-04-11 from original flat path). All 12 sections filled, 1 gap flagged (confirm day-rate vs PAYE structure before the call), pitch and STAR stories drawn from Todoist's pre-drafted angles verbatim plus 1 added angle for CRM gap closing.
- Notion page created as a child under the Northwind Trading Opportunity row in `{{NOTION_OPPORTUNITIES_DB}}`. Status header + 14-item user-curated checklist + Prep Notes section filled across two interactive prep conversations on 2026-04-10 and 2026-04-11 (Intro, Gap answer, Power Story A = peak-volume queue story, all dropped in as H3 + callout + delivery notes + anchor beats).

**What the skill did NOT do:** re-write the user's prep angles, suggest different STAR stories than what the user already curated, fabricate company news headlines, recommend a salary number outside the user's logged target.

**Fictional interviewer note.** Jordan Ellis (recruiter, initial screening call on 2026-03-15) passed the candidate to Sam Carter (hiring manager). Jordan used the title "Talent Acquisition Partner" in their email signature, which correctly triggered the recruiter-screen classification at Step 2 for the first call; the April 10 call with Sam triggered the HM intro/fit path.

## Gotchas (live notes — extend as encountered)

These are real failure modes the smoke-test agents discovered. Re-read before each invocation.

| # | Gotcha | Mitigation |
|---|---|---|
| 1 | GCal `start.timeZone` field can lie (an event may declare one timezone while the actual UTC offset differs). | Always read the `+HH:MM` offset off `dateTime`, not the `timeZone` string. |
| 2 | Searching GCal by company name misses events whose title does not contain the company (e.g. "Interview — Operations"). | Combine date-range scan + email-domain match on attendees. Build a per-company domain map as you encounter patterns. |
| 3 | Gmail `newer_than:6w` combined with a keyword silently returned empty. | Drop `newer_than` or use absolute `after:YYYY-MM-DD`. |
| 4 | Gmail company-name search drowns in `no-reply` / automated notification noise. | Sender blocklist applied at filter time, not after. Extend the list per company as encountered. |
| 5 | Recruiters sometimes use a parent-company or legacy domain, not the customer-facing brand domain. | Build a per-company domain map and check all of them. Never assume the sender domain matches the brand name. |
| 6 | JD often only exists as a Gmail attachment, never saved to Drive. | Priority chain: local file path (from Todoist) then Gmail attachment then Drive then user paste. |
| 7 | Pipeline DB often has no row for referral-sourced opportunities. | Quiet flag in Open gaps; do not block, do not prompt. Todoist is the source of truth. |
| 8 | Two pipeline trackers may exist (one current, one stale). | Sort search hits by `modifiedTime desc`; prefer most recent file. |
| 9 | Referral chain may span multiple Gmail threads over months. | Pull all matching threads, surface chain as a list, not just the most recent. |
| 10 | Drive `read_file_content` for a spreadsheet returns concatenated tab dumps; row checks require text grep over the whole output. | Either accept that, or use a Sheets-API-specific cell query if/when added. |
| 11 | Some environments (e.g. Cowork) can't reach the Todoist connector. | Fall back to the Step 3 paste-in pattern (ask the user to paste the task body) rather than failing. |

## Notes for Claude

- This skill is the contract for how prep docs get built — keep edits deliberate, and test on a real interview before relying on a change.
- Default output location is `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/` — this directory may not exist on first run; create it. `{{PREP_OUTPUT_DIR}}` is defined in `config.local.md` (default: `./output/interview-prep`). Per-interview subfolders are new in v0.2.
- The Master CV file ID and the Notion Opportunities DB ID are referenced in a few places here for convenience — all of them resolve from `config.local.md`, which is the single place you set them.
- Interview prep pages are children of the **matching Opportunity row** in the Opportunities DB (`{{NOTION_OPPORTUNITIES_DB}}`).
- Cowork-vs-Code detection: check whether `mcp__todoist__*` tools are listed. If yes — Code path. If no — Cowork path with paste-in.
- v0.3 candidates (do NOT build into v0.2):
  - Technical interview branch (different shape: write down loudest wins, refresh on JD's tech bullets, practice reasoning out loud).
  - Panel / final-round branch (per-interviewer angle table, 15-min decompression buffer planning).
  - Salary negotiation branch (target / floor / walk-away; "anchor with one number, not a range").
  - Todoist to pipeline DB sync that removes the paste-in step entirely.
  - Multi-stage prep within a single per-interview folder (`interview-prep.md` for round 1, `interview-prep-stage-2.md` for round 2, etc.) once the user hits a real multi-round flow.

## Changelog

### v0.3.0 — 2026-05-15

**Notion pipeline migration.** The job-search pipeline moved from Google Sheets + webhooks to a single Notion Opportunities DB (`{{NOTION_OPPORTUNITIES_DB}}`). This skill updated accordingly:

- **Step 4 rewrite:** pipeline cross-check now queries Notion (`notion-search` + `notion-fetch`) instead of a sheet webhook. Schema is 19 properties; cross-check fields are now Status, Salary range, Next action, Excitement, Track, Temperature.
- **Step 11 reparent:** prep pages now created as child pages under the **matching Opportunity row** in the Opportunities DB, not under a legacy flat parent page. Any previously created prep pages should be re-parented to their respective Opportunity rows.
- **Reference IDs section updated:** Sheet ID dropped. Notion DB ID + data source ID added. Master CV + Drive folder unchanged.
- **Todoist project unchanged:** still `{{TODOIST_PROJECT}}`. Pipeline-state Todoist sections were deleted in the same migration; only `System` section remains. The Cowork paste-in pattern (Step 3) is unaffected.

### v0.2.0

**Output model change.** Per-interview folder convention replaces the flat-file layout. Two paired deliverables instead of one:

1. **Claude-facing MD** at `{{PREP_OUTPUT_DIR}}/<Company> — <Role Title>/interview-prep.md` — the same 12-section structure as v0.1, just relocated into a per-interview folder. Still Claude's exploratory working surface.
2. **User-facing Notion page** under the matching Opportunity row, mirroring the template (status header + prep checklist + empty Prep Notes section). New in v0.2.

Step 9 expanded to Steps 9 (folder), 10 (MD write), 11 (Notion page creation). Step 10 became Step 12 (surface to user).

Added new sections: **Notion page template** (status header + starter checklist + empty Prep Notes), **Interactive prep mode (post-scaffold)** documenting the H3 + callout + empty-block pattern that emerged from the live smoke-test prep.

Hard constraint changed: "One deliverable" — "Two paired deliverables". Em-dash ban clarified: applies to prep doc CONTENT, not folder/page names (which use ` — ` as separator).

### v0.1.0

Initial scaffold. First-round screens only (recruiter + HM intro/fit). Cowork-compatible (Todoist paste-in) with Code fallthrough (direct Todoist MCP read). Out-of-scope detection branch refuses technical / panel / final / negotiation / follow-up with a generic-offer fallback. ONE deliverable: markdown file with 12 sections. Eleven gotchas captured from the smoke-test exploration.
