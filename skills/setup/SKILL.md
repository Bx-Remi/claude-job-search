---
name: setup
description: >
  First-run setup wizard for the claude-job-search system. Use this skill when the user wants to
  get the system running for the first time — trigger phrases include "set me up", "set up the job
  search system", "get me started", "walk me through setup", "I just cloned this", "help me install
  this", or any first-time-setup intent. Also use it when a starting prompt asks you to clone the
  repo and follow this file. It drives the ENTIRE setup end to end, doing every step Claude can do
  and pausing only when it genuinely needs the user (account sign-ins, OAuth, a name, story content).
  Written for people who have never heard of MCP, OAuth, or Claude Code internals.
metadata:
  version: "1.0.0"
---

# Setup wizard

You are running first-time setup for a **non-technical user**. Assume they have never heard of "MCP",
"OAuth", "connector", or "database ID". Your job is to get them from nothing to a working system,
doing as much as you possibly can yourself and asking them only for the few things that genuinely
require them.

## Operating principles (read before you start)

- **You drive; they confirm.** Do every step you can do through your tools. Stop and ask the user
  ONLY when the step is something only they can do: creating an account, signing in / approving a
  connector in their browser, choosing their name, or providing the content of a STAR story.
- **One phase at a time.** Briefly say what you're about to do, do it, confirm it worked, then move
  on. Never paste the whole plan at them or run several phases silently.
- **Plain English, always.** The first time a piece of jargon is unavoidable, define it in one short
  sentence (see the glossary below). Never show raw JSON, never ask them to hand-edit a file.
- **Verify, don't assume.** Especially for connectors: after the user says they've connected one,
  prove it by actually calling one of its tools. A half-finished setup that looks done is the worst
  outcome.
- **Fail gracefully.** If something's wrong, say in plain English what's wrong and exactly what to
  click, then wait. Don't barrel ahead on a broken step.
- **Never run on empty.** Don't write placeholder/blank IDs into config; resolve real values or stop.

### One-line glossary (use these phrasings if asked)
- **Claude Code** — the AI assistant you're talking to right now; it runs on your computer and can do
  tasks for you, like this setup.
- **Connector (MCP connector)** — a secure bridge that lets me read and write one of your apps (Notion,
  Todoist) using your own login. I never see your password; you approve it in your browser. ("MCP" =
  Model Context Protocol, the open standard these bridges use — only mention that if they ask.)
- **Notion** — a flexible notes-and-tables app; we use it as the tracker that holds every role.
- **Todoist** — a to-do app; it holds your follow-up reminders with due dates.
- **Database / data-source ID** — just the address of the table we create, so I know where to write.

---

## Phase 0 — Welcome & orient (always do this first)

Before doing anything, give a short plain-English overview and get a go-ahead. Cover:

- **What this is:** a job-search assistant that tracks every application for you — who you've contacted,
  who owes you a reply, what's gone cold, what to do next — so nothing slips.
- **What setup takes:** about 15 minutes, and I do most of it. You'll only need to do a couple of quick
  sign-ins in your browser when I prompt you.
- **What you'll need:** a free **Notion** account and a free **Todoist** account. (Optional: a Google
  account — Gmail/Calendar/Drive — makes interview prep richer, but everything works without it.)
- **Your data stays yours:** the connections use your own logins; nothing is shared with me or anyone
  upstream, and your IDs live only on your machine.

Then ask: **"Ready to start?"** Wait for a yes before continuing.

If you were asked to clone the repo and you haven't yet, clone it into the current folder first, then
start at Phase 0.

## Phase 1 — Accounts

Ask whether they already have **Notion** and **Todoist** accounts.
- If they don't, point them to [notion.so](https://www.notion.so) and [todoist.com](https://todoist.com)
  to sign up (both free), and wait until they confirm they're done.
- If they want the optional Google features, no account creation needed — that's just connectors in
  the next phase.

## Phase 2 — Connect the connectors (the important one)

Explain first, in one sentence: *"A connector is a secure bridge that lets me use your Notion and
Todoist with your own login — you approve it in your browser, and I never see your password."*

Then walk them through connecting each one. The path depends on whether they're in the **desktop
app** or the **terminal (CLI)** — figure out which they're using and give them the matching steps:

**Desktop app (easiest):** click the **+** button next to the message box → **Connectors** → find
**Notion** in the list (it's there by name) → click it → a browser window opens → sign in to Notion
and click **Allow**. Then do exactly the same for **Todoist**.

**Terminal / CLI:** run **`/mcp`** to add and authenticate a connector. If they need the official
server addresses, they are:
- Notion: `claude mcp add --transport http notion https://mcp.notion.com/mcp`
- Todoist: `claude mcp add --transport http todoist https://ai.todoist.net/mcp`
Then run `/mcp` and authenticate — a browser opens for each.

*(Optional)* Same flow for **Gmail / Google Calendar / Google Drive** if they want richer prep.

**Required:** Notion + Todoist. Optional: the Google three.

**Credential safety — tell them this plainly:** the official connectors sign you in through your
browser (the same as logging into Notion or Todoist normally), and you'll **never be asked to paste
an API key, token, or password** — not into the chat, not into a file. If any setup tells you to
paste a token, it's an unofficial third-party server; don't use it. Stick to the named connector in
the app, or the two official URLs above.

Then **verify before moving on**. After they say a connector is done, actually call one of its tools
to confirm it works:
- Notion: do a small search or list (e.g. search for any page).
- Todoist: list their projects.

If a call fails with "tool not found" or similar, the connector isn't actually connected yet — tell
them plainly and have them redo the connect step (and, if needed, fully reopen Claude Code so the new
connector loads). **Do not proceed to Phase 3 until both required connectors respond.**

## Phase 3 — Build the Notion tracker

Do this yourself. Tell them: *"I'll now create the table that holds every role you track."*
- Read `notion-template/opportunities-db-schema.json` (the exact spec).
- Ask which Notion page to put it under — or offer to create a fresh top-level page for it.
- Create the database with the exact property names and emoji options from the schema.
- **Select colours must come from Notion's 10-colour palette only:** `default, gray, brown, orange,
  yellow, green, blue, purple, pink, red`. The schema doesn't specify colours, so you choose them —
  but any value outside this list (e.g. `teal`, `sky`, `lime`) makes the create call fail. Colours
  are cosmetic; only the option *names* must match the schema exactly.
- Capture its **database ID** and **data-source ID** (you'll need them for the config).

If creation fails because the connector can't access the chosen page, explain that they need to share
that page with the Notion connection (in Notion: the page's "•••" menu → Connections → add it), then retry.

## Phase 4 — Create the Todoist project

Do this yourself via the connector: create a project called **"🔍 Job Search"** and capture its ID.
Tell them: *"This is where your follow-up reminders will live."*

## Phase 5 — Write the config

Tell them: *"Last setup step — I'll save the addresses of everything we just made so the assistant
knows where to write. This file stays on your machine."*
- Copy `config.example.md` to `config.local.md` (it's gitignored, so nothing personal is committed).
- Fill in every value you resolved: the Notion database + data-source IDs, the Todoist project ID,
  and any optional Drive IDs if Google was connected. Leave optional values blank if unused.
- Ask the user only for **`USER_NAME`** (their first name — used to phrase pitches in the first person).
- Do not leave any required value blank.

## Phase 6 — STAR stories (optional)

Explain in one line: *"A STAR story is a short interview answer structured as Situation, Task, Action,
Result — having a few ready means you stop improvising the same behavioural questions badly."*

Offer: *"Want to seed a few now, or skip and add them later?"*
- If yes: ask for the rough situations and draft them into `templates/star-story-template.md`'s shape,
  saved in `STORIES_DIR` (default `./templates`). They stay on their machine.
- If skip: move on — they can always do it later.

## Phase 7 — Smoke test

Prove it works with one real example. Run:
> *"I just applied to Acme Corp for a Support Lead role — quick apply."*

Expect a new Notion row (e.g. `📨 Applied`, Track B) **and** a Todoist task with a follow-up date.
Confirm both appeared and report what you see. If they connected Google, you can also offer a quick
*"prep me for an Acme interview"* run.

If anything didn't land, debug it (usually a connector or a config ID), fix it, and re-run before
declaring success.

## Phase 8 — How the system works (do this ONLY once setup is verified)

Now — and only now — explain the system in plain English so they know what they just got:

- **You never touch Notion or Todoist by hand again.** You just talk to me, and I keep both in sync.
- **Notion holds the state** (every role and where it stands); **Todoist holds the actions** (your
  next steps, with due dates). They never drift because I update both for you.
- **Two tracks:** *Track A* = a handful of roles you really want, tailored hard (~45 min each);
  *Track B* = high-volume quick-applies for income insurance (~5 min each). Same logging, different effort.
- **Day to day, just say what happened**, for example:
  - *"I applied to Northwind for a Support Lead role."*
  - *"Heard back from Acme — recruiter screen Thursday."*
  - *"Contoso rejected me."* / *"Got an offer from Acme!"*
  - *"Prep me for my Northwind interview."*
  - *"Follow up on anything I've gone quiet on."*
- The skills trigger on natural phrasing — **no commands to memorise.**
- Your settings live in `config.local.md`; interview-prep docs are written to `PREP_OUTPUT_DIR`.

Close with something like: *"You're all set. From here on, just tell me what happens in your search
and I'll keep everything straight."*

---

## Troubleshooting quick-reference

- **"Tool not found" / I can't see Notion or Todoist** → the connector isn't connected. Go back to
  Phase 2, connect it, and reopen Claude Code if it still doesn't appear.
- **Notion database won't create** → the connection can't access the target page. Share the page with
  the Notion connection (page "•••" → Connections), then retry.
- **A skill says config values are missing** → finish Phase 5; the skills refuse to run on empty IDs.
- **Don't:** ask the user to edit JSON or markdown by hand, dump all phases at once, proceed past
  Phase 2 before a connector call succeeds, or invent IDs.
