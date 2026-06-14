# Notion template — two ways to get the database

The system runs on one Notion database: **Opportunities** (full spec in
[`opportunities-db-schema.md`](opportunities-db-schema.md)). You can get it two ways — pick one.

## Path A — Let Claude build it (recommended, fully self-serve)

Once your Notion connector is wired (see [`../SETUP.md`](../SETUP.md)), just ask Claude:

> "Read `notion-template/opportunities-db-schema.json` and create this database under
> *[the page you want it on]*. Then tell me the database ID and data-source ID."

Claude creates the database with the exact property names and emoji SELECT options, then hands
you the two IDs. Paste them into `config.local.md`. Done — no copy-paste of option strings, no
typos.

## Path B — Duplicate the published template

If you'd rather click than chat:

> **Template link:** _(to be added — duplicate this into your own Notion workspace)_

Open the link → **Duplicate** (top-right) → it lands in your workspace with the schema and a
couple of sample rows intact. Then ask Claude "what's the ID and data-source ID of my
Opportunities database?" and paste them into `config.local.md`.

**Caveats for Path B:**
- Each duplicate gets its **own** database ID — that's expected; it's why the IDs live in your
  `config.local.md` and not in the skills.
- The `Days since last contact` **formula copies** with the template.
- You still need to grant your Notion connector access to the duplicated page.

---

Either way, the goal is the same: a Notion database whose property names and SELECT options
match the schema **exactly**, plus its two IDs recorded in your config. After that, the skills
do the rest.
