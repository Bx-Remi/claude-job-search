# Setup — get running in about 15 minutes

> **Just want it done for you?** Skip this and use the one-prompt setup in the
> [README](README.md#get-started--one-prompt-about-15-minutes) — Claude runs the whole thing and only
> asks when it needs you. This file is the manual walkthrough for people who'd rather drive each step.

This walks you through it end-to-end. Each step is tagged **[you]** (only you can do it — usually
because it's your account/OAuth) or **[Claude]** (let Claude do it for you in Claude Code).

> The whole philosophy is "let Claude do the work." Where a step says **[Claude]**, just ask —
> the prompts are written out for you.

> **Already set up before the 7 views existed?** Easiest is to **duplicate the new template** (Step 3)
> and have Claude re-point your config at it. If you've already logged **real** applications, tell Claude
> instead — that's a quick data migration rather than a fresh start. (See the [CHANGELOG](CHANGELOG.md).)

---

## Step 0 — Prerequisites **[you]**

- **[Claude Code](https://claude.com/claude-code)** installed.
- A **Notion** account and a **Todoist** account.
- *(Optional, for full interview-prep)* a Google account with **Gmail / Calendar / Drive**.

## Step 1 — Get the repo **[you]**

```bash
git clone https://github.com/Bx-Remi/claude-job-search.git
cd claude-job-search
```

Open the folder in Claude Code.

## Step 2 — Connect your MCP connectors **[you — OAuth can't be delegated]**

In Claude Code, connect:

- **Notion** — required
- **Todoist** — required
- **Gmail**, **Google Calendar**, **Google Drive** — optional (only `interview-prep` uses them;
  it works without them, just with less context)

The OAuth handshake happens in **your** browser — Claude can't log into your accounts for you.

> The skills refer to connector tools by their short names (e.g. `notion-create-pages`,
> `add-tasks`). If Claude ever says it can't find a tool, the connector probably isn't connected
> yet — reconnect and retry.

## Step 3 — Get your Notion database (duplicate the template) **[you, 2 clicks + Claude]**

The database, its 7 views (🎯 Today, ⏰ Stale follow-ups, 🔥 Active pipeline, 📞 Interviewing,
🔍 Sourcing, ⚰️ Archive, 🗂 All), and the formulas all come ready-made. You duplicate them; Claude
connects them.

1. Open the template:
   **https://thin-goat-86e.notion.site/Opportunities-Shareable-Template-388f26eedb6a8021a6d4db8f4fa3312f**
2. Click **Duplicate** (top-right) and pick your own workspace.
3. Tell Claude:

> "I duplicated the Opportunities template into my Notion. Find my Opportunities database and write its
> database ID and data-source ID into `config.local.md`."

Claude finds your copy and wires it up — it'll ask which page if you have more than one "Opportunities".
The sample rows are fake; delete them whenever you like. More detail (and what to do if Claude can't
find it) is in [`notion-template/README.md`](notion-template/README.md).

## Step 4 — Create your Todoist project **[you, 30s — or Claude]**

Make a Todoist project (e.g. "🔍 Job Search") — or ask Claude to make it:

> "Create a Todoist project called '🔍 Job Search'."

You don't need to copy its ID; the next step wires it into your config for you.

## Step 5 — Fill in your config **[you + Claude]**

```bash
cp config.example.md config.local.md
```

Then paste this to Claude, filling in the **[bracketed]** names:

> "Fill in `config.local.md` for me. My Notion database is **Opportunities** (under the page **[your
> page name]**) and my Todoist project is **[🔍 Job Search]**. Find their IDs — the Notion database ID
> and data-source ID, and the Todoist project ID — and write them in. If Google Drive is connected, add
> my master CV **[your CV file name]** too. Set USER_NAME to **[your first name]**. Ask me for anything
> you can't find."

Claude finds every ID through the connectors and writes them into the file — you don't hand-edit IDs or
copy them out of URLs. Leave optional values blank if you're not using them.

> `config.local.md` is gitignored — your IDs stay on your machine.

## Step 6 — Seed your STAR stories **[you, Claude drafts]**

Copy [`templates/star-story-template.md`](templates/star-story-template.md) and fill in 3–5 of
your own stories (kept in `STORIES_DIR`, default `./templates`). Don't have them written? Tell
Claude the rough situations and ask it to draft them in STAR shape for you to edit. They never
leave your machine.

## Step 7 — Smoke test **[you + Claude]**

Try a couple of real sentences:

- **Logging:** *"I just applied to Acme Corp for a Support Lead role — quick apply."*
  → expect a new Notion row (`📨 Applied`, Track B) **and** a Todoist task with a follow-up date.
- **Prep:** *"Prep me for my Acme interview."*
  → expect a prep doc under `PREP_OUTPUT_DIR` and a glance-page in Notion. (If Gmail/Calendar
  aren't connected, it'll note the missing context and carry on.)

If a skill ever can't find a value, it'll stop and point you back here — it won't run on empty
IDs.

## Step 8 — Daily use **[Claude]**

That's it. From now on, just talk to Claude as things happen:

> *"heard back from Northwind — recruiter screen Thursday"*
> *"Contoso rejected me"*
> *"got an offer from Acme!"*
> *"follow up on anything I've gone quiet on"*

The skills trigger on the natural phrasing — no commands to memorise.

---

## Who does what — at a glance

| Step | Who |
|---|---|
| Install Claude Code, create Notion/Todoist accounts | **You** |
| Connect each MCP connector (OAuth) | **You** (browser) |
| Get the Notion database (duplicate the template) | **You** (2 clicks) + **Claude** wires it up |
| Create the Todoist project | **You** (or Claude) |
| Find IDs, fill `config.local.md` | **Claude** (asks you for gaps) |
| Write your STAR stories | **You** (Claude drafts) |
| Run the skills, day to day | **Claude** |
