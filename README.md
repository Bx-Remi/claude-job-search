# claude-job-search

A two-track job-search and interview-prep system that runs inside **[Claude Code](https://claude.com/claude-code)**.

You talk to it in plain English — *"I applied to Acme for a Support Lead role"*, *"heard back
from Northwind"*, *"prep me for the Contoso interview"* — and it keeps a **Notion** pipeline and a
**Todoist** action list in sync, and scaffolds your interview prep from your calendar, email, CV
and a quick web look. No forms, no spreadsheet-wrangling, no "wait, did I ever follow up with
them?".

It's the system I built for my own search. It landed me a role in about six weeks, and I'm
sharing the genericized version here so you can adapt it.

---

## Get started — one prompt, about 15 minutes

**New to Claude Code? You don't need to understand any of this.** You set it up once, mostly by
letting Claude do the work for you.

1. **Install Claude Code** — download it from [claude.com/claude-code](https://claude.com/claude-code)
   and open it. (It's an AI assistant that runs on your computer and can do tasks for you — like this
   setup.)
2. **Make a new empty folder** anywhere on your computer (call it `job-search`), and open Claude Code
   in it — start a fresh chat there.
3. **Paste this in and send it:**

   ```
   Clone https://github.com/Bx-Remi/claude-job-search into this folder, then read and follow
   skills/setup/SKILL.md to set everything up with me. Explain what we're doing in plain English
   and ask me whenever you need me to do something.
   ```

4. **Follow Claude's lead.** It walks you through the whole thing and only stops to ask when it needs
   *you* — basically a couple of quick sign-ins to **Notion** and **Todoist** (two free apps) in your
   browser. Everything else it handles itself. (Those sign-ins happen in your browser the normal way —
   you're never asked to paste any passwords or keys, and you never have to edit a settings file.)
5. **When it's done, Claude explains how the system works**, and you can start using it — just tell it
   things like *"I applied to Acme for a Support Lead role"* or *"prep me for my Contoso interview."*

That's the whole setup. No spreadsheets, no config files to edit by hand, nothing to memorise.

> Prefer to do it step-by-step yourself? The manual walkthrough is in **[SETUP.md](SETUP.md)**.

---

## Why it's built this way

Job hunting is an admin problem wearing a motivation costume. The hard part isn't the
applications — it's the *state-keeping*: who you've contacted, who owes you a reply, what's gone
cold, what to do next. Spreadsheets and sticky notes fall apart within a week. So the design
goal was simple: **kill the friction between "I should do the thing" and "the thing is done."**

Three ideas do most of the work:

- **One source of truth.** Every role is one row in a Notion database — stage, excitement, *next
  action*, days since last contact. Todoist holds only the *actions* (with due dates); Notion
  holds the *state*. They never drift because you never touch them by hand — Claude does.
- **Two tracks.** **Track A** = a handful of roles you genuinely want, tailored hard (≈45 min
  each). **Track B** = high-volume quick-applies for income insurance (≈5 min each). Different
  effort, different cadence — logged the same way.
- **Two separate axes: excitement vs temperature.** How much you *want* a role (stable) is kept
  apart from how *live* the conversation is (changes daily). That's what lets you triage by
  "what's hot now" without losing "what I actually want."

## What's in the box

| Piece | What it does |
|---|---|
| **`skills/job-search-action`** | Logs every pipeline event from one sentence — applied, heard back, interview, rejected, ghosted, offer — into Notion + Todoist. |
| **`skills/interview-prep`** | Scaffolds a first-round prep doc: pulls the role, the recruiter thread, your CV and a web snapshot into one brief, plus a glance-page in Notion. |
| **`notion-template/`** | The Opportunities database — the backbone. Build it from the schema (Claude does it) or duplicate a template. |
| **`templates/star-story-template.md`** | A STAR-story bank template so you stop improvising the same behavioural answers badly. |
| **`config.example.md`** | Where you put your own IDs. Copy to `config.local.md` (gitignored) — nothing personal is committed. |

## Requirements

- **Claude Code** (these are Claude Code "skills").
- **Notion** and **Todoist** accounts, connected as MCP connectors. *(Required.)*
- **Gmail / Google Calendar / Google Drive** connectors. *(Optional — only `interview-prep` uses
  them, and it degrades gracefully without them.)*

## Quickstart (manual path)

Most people should use the one-prompt setup above. If you'd rather drive it yourself:

1. Clone this repo and open it in Claude Code.
2. Connect your Notion + Todoist connectors.
3. Build the Notion database (ask Claude to build it from `notion-template/opportunities-db-schema.json`).
4. `cp config.example.md config.local.md` and let Claude fill in your IDs.
5. Try it: *"I applied to Acme Corp for a Support Lead role — quick apply."*

Full step-by-step, including which steps Claude can do for you, is in **[SETUP.md](SETUP.md)**.

## How your data stays yours

- Your real IDs live only in `config.local.md`, which is **gitignored** — they never get
  committed.
- The skills reference everything by placeholder (`{{NOTION_OPPORTUNITIES_DB}}` etc.).
- Connectors use **your own** OAuth — Claude acts through your accounts, nothing is shared
  upstream.

## Not included (yet)

An automated **job-board scanner** (pulls roles from job boards, filters to your criteria, dedupes,
and emails you a shortlist) is part of my own setup but isn't in this v1 — it needs your own API
keys and is fiddlier to set up. It may land as an advanced add-on later.

## License

[MIT](LICENSE) — use it, adapt it, make it yours.

---

*Built with Claude Code. If this helped your search, I'd love to hear about it.*
