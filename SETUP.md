# Setup — get running in about 15 minutes

This walks you through it end-to-end. Each step is tagged **[you]** (only you can do it — usually
because it's your account/OAuth) or **[Claude]** (let Claude do it for you in Claude Code).

> The whole philosophy is "let Claude do the work." Where a step says **[Claude]**, just ask —
> the prompts are written out for you.

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

## Step 3 — Create your Notion database **[Claude]**

Pick a Notion page to hold it, then ask Claude:

> "Read `notion-template/opportunities-db-schema.json` and create this database under
> *[page name]*. Then tell me its database ID and data-source ID."

Claude builds it with the exact properties and emoji options. *(Prefer to click? See the
duplicate-a-template path in [`notion-template/README.md`](notion-template/README.md).)*

## Step 4 — Create your Todoist project **[you, 30s — or Claude]**

Make a Todoist project (e.g. "🔍 Job Search"). Then ask:

> "What's the ID of my Job Search project in Todoist?"

Claude reads it back via the connector.

## Step 5 — Fill in your config **[you + Claude]**

```bash
cp config.example.md config.local.md
```

Then ask Claude:

> "Fill in `config.local.md` — find my Notion Opportunities database ID and data-source ID, my
> Todoist Job Search project ID, and (if connected) my Drive master-CV ID. Ask me for anything
> you can't find."

Claude resolves what it can through the connectors and asks you for the rest. Set `USER_NAME` and
leave the optional values blank if you're not using them.

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
| Build the Notion database | **Claude** (or duplicate a template) |
| Create the Todoist project | **You** (or Claude) |
| Find IDs, fill `config.local.md` | **Claude** (asks you for gaps) |
| Write your STAR stories | **You** (Claude drafts) |
| Run the skills, day to day | **Claude** |
