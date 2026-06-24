---
name: upgrade
description: >
  Upgrades an EXISTING claude-job-search user to the current version — specifically to the multi-view
  Opportunities database (the 7 ready-made views). Pulls the latest code, then gets the user onto the
  published template and moves anything they've already logged into it. Use this when someone set the
  system up before the views/template existed. Trigger phrases: "update my job search system", "upgrade
  the job search", "get me the new views", "migrate to the new template", "run the update", "I set this
  up a while ago, update it", or when a starting prompt asks you to git pull and follow this file.
  It is non-destructive: it never deletes the user's existing data — it copies it into the new database
  and lets the user remove the old one themselves once they're happy.
metadata:
  version: "1.0.0"
---

# Upgrade wizard — move an existing user to the multi-view template

You are upgrading someone who **already uses** this system to the current version. The big change:
the Opportunities database now ships as a duplicatable **template with 7 views** (the two date-based
views use rolling-date filters the API can't create, so duplication is the only way to get them). Your
job is to pull the update, get them their own copy of the template, move any existing applications into
it, and re-point their config — pausing for the two things only they can do (duplicating in Notion,
confirming).

## Operating principles (read before you start)

- **You drive; pause only when you must.** The only steps the user performs are: duplicating the
  template in their browser, and telling you when that's done. Everything else is you.
- **Non-destructive, always.** Never delete or overwrite the user's existing database or rows. You
  *copy* data into the new database. The user deletes the old database themselves, only after they've
  confirmed the new one looks right. (You also can't delete Notion pages through the connector — so
  never claim you did; hand deletions to the user.)
- **One thing at a time, in plain English.** Say what you're about to do, do it, confirm, move on.
- **Verify, don't assume.** After migrating, compare row counts old vs new before telling them it's safe
  to delete the old one.

---

## Step 1 — Pull the latest code

Run `git pull` in the repo root. Confirm it succeeded and report one line on what changed (you can read
`CHANGELOG.md`). If the folder isn't a git repo or the pull fails, tell them plainly how to fix it
(e.g. re-clone, or `git pull` manually) and wait — don't continue on a stale copy. If the pull fails on
**local changes or conflicts**, don't force or discard anything — offer to `git stash` then pull, or to
re-clone into a fresh folder. (`config.local.md` is gitignored, so their IDs are safe either way.)

> If this file changed in the pull, re-read it before continuing so you're following the current SOP.

## Step 2 — Check what they have now

Read `config.local.md` at the repo root.

- **If `NOTION_OPPORTUNITIES_DB` is set** → they have an existing database. `notion-fetch` it (on the
  database ID — that returns its views) and note: (a) how many rows it has, and (b) whether it already
  has the 7 views (🎯 Today, ⏰ Stale follow-ups, 🔥 Active pipeline, 📞 Interviewing, 🔍 Sourcing,
  ⚰️ Archive, 🗂 All). If the view list comes back empty or unclear, don't guess — ask the user to look
  at their Opportunities database and tell you whether they see those 7 view tabs, and decide on their
  answer (this branch chooses migrate-vs-stop, so get it right).
  - **Already has all 7 views** → they're already current. Tell them, offer the **tutorial** skill, and
    stop. Nothing to migrate.
  - **Missing the views** → continue to Step 3. Tell them honestly: *"Your database has your data but
    not the new views. The clean way to get the views is to duplicate the new template and move your
    data across — I'll handle the move."*
- **If there's no `config.local.md` or `NOTION_OPPORTUNITIES_DB` is blank** → they never finished setup.
  Don't run the migration path; point them to first-time setup instead (the one-prompt in `README.md`,
  which runs the `setup` skill) and stop.

Tell them, in one or two sentences, what you found and what you're about to do.

## Step 3 — Give them the template to duplicate (PAUSE here)

Tell them:

> *"Open this template and click **Duplicate** (top-right) to copy it into your own Notion workspace —
> then tell me when it's done:
> https://thin-goat-86e.notion.site/Opportunities-Shareable-Template-388f26eedb6a8021a6d4db8f4fa3312f"*

**Wait for them to confirm it's duplicated before continuing.** Don't proceed on assumption.

## Step 4 — Find their new copy (and don't confuse it with the old one)

After they confirm, locate the duplicated database with `notion-search` for "Opportunities".

⚠️ **There are now two databases named "Opportunities"** — their old one and the fresh duplicate. Tell
them apart by **ID**, which is unambiguous:
- The **old** one's ID is the one already in `config.local.md` (`NOTION_OPPORTUNITIES_DB`).
- The **new** one is the match whose ID is **different** from that — it sits under a page called
  "Opportunities — Shareable Template" and has fictional sample rows (e.g. Acme, Northwind) plus the 7 views.

If more than one candidate remains, or you're at all unsure, **have the user paste the new database's
URL** rather than guessing. **Confirm before migrating** — e.g. *"Your new copy is the one under
'Opportunities — Shareable Template' with the sample rows — that's where I'll move your applications,
right?"* Migrating into the wrong database is the one thing to avoid here.

`notion-fetch` the new database to capture its **database ID** and **data-source ID**.

> If search returns nothing, the connector likely can't see the new page. Tell them: open the duplicated
> page → "•••" → **Connections** → add their Notion connection, then say "try again".

## Step 5 — Migrate their applications (only if the old DB has real rows)

If the old database has rows the user wants to keep (ask if unsure — skip this whole step for an empty
or sample-only old DB):

1. Read **all** rows from the old database (page through the old data source; capture every property).
   Make a ledger of every old row by **Company + Role** — you'll tick it off as you go.
2. **Idempotency (in case this is a re-run):** before creating each row, search the new database for that
   Company + Role; if it's already there, skip it. Never create a duplicate.
3. For each old row not already present, create a matching row in the **new** database with
   `notion-create-pages` (parent = the new `data_source_id`), copying every property across by exact name
   — including the emoji-prefixed SELECT values (`Status`, `Excitement`, `Track`, `Temperature`,
   `Source`, `Work model`, `Dead reason`) and the expanded date fields (`date:Applied date:start`, etc.).
   Don't copy the two formula columns (`Days since last contact`, `Days since applied`) — they recompute.
   - **If an old SELECT value isn't a valid option in the new template** (e.g. a legacy status from an
     older version, or a custom Source the user added), the create is rejected. Don't silently drop the
     row — map it to the closest valid option and tell the user, or ask them. Keep a running list of any
     rows that didn't migrate cleanly.
4. If an old row's page body has a `## Job Description` (or other notes), copy that into the new row's
   `content` so the JD travels with it.
5. **Interview-prep child pages** (if any old rows have child pages from `interview-prep`) do **not**
   migrate automatically. Tell the user which rows have them and offer to either leave them on the old
   database (kept until they delete it) or copy the prep text into the new row's body — their call.
6. Work in batches until **every** old row is present in the new database (your ledger is fully ticked).
   Surface any rows that failed so none are lost.

## Step 6 — Re-point the config

Update `config.local.md`: set `NOTION_OPPORTUNITIES_DB` and `NOTION_OPPORTUNITIES_DS` to the **new**
database's IDs. Leave everything else (Todoist, Drive, USER_NAME, paths) unchanged. From now on the
skills write to the new database.

## Step 7 — Verify, then hand off the cleanup

- **Verify by set-membership, not arithmetic:** fetch the new database and confirm that **every**
  Company + Role from your old-database ledger is now present. Don't rely on an exact row count — the
  template ships with its own sample rows you didn't create, so a count won't line up. Confirm the 7
  views are present too. If any old row is missing or failed to migrate (see Step 5), surface it and
  **do not** tell the user it's safe to delete the old database.
- **Once every real row is accounted for, tell them what to clean up themselves** (you can't delete
  Notion pages through the connector):
  1. Delete the fictional **sample rows** in the new database — the made-up companies (Acme, Northwind,
     and the like) that aren't from their real search.
  2. Only **after** they've eyeballed the new database and confirmed all their real applications are
     there, delete the **old** database. That ordering is the safety net.
- Confirm success in plain English: *"You're upgraded — your tracker now has all 7 views, your
  applications are in it, and your config points at it."*

## Step 8 — Offer the tour

Offer the walkthrough: *"Want a quick tour of the new views and how the day-to-day flow works? Just say
'give me the tutorial'."* (That runs the `tutorial` skill.)

---

## Troubleshooting

- **`git pull` fails / not a git repo** → have them confirm they're in the cloned repo folder, or
  re-clone; don't proceed on a stale copy.
- **Can't find the duplicated database** → connector lacks access to the new page (page "•••" →
  Connections → add the connection), then re-search. Make sure they actually clicked **Duplicate**.
- **Two "Opportunities" databases, unsure which is new** → the old one's ID is in `config.local.md`; the
  new one has the sample rows and sits under "Opportunities — Shareable Template". Ask the user to
  confirm before migrating.
- **Don't:** delete or edit the user's old data, write the new IDs into config before the new database is
  confirmed, or claim you deleted pages you can't delete.
