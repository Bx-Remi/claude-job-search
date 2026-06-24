# Get your Opportunities database — duplicate the template

The whole system runs on one Notion database, **Opportunities**, with **7 views** (🎯 Today ·
⏰ Stale follow-ups · 🔥 Active pipeline · 📞 Interviewing · 🔍 Sourcing · ⚰️ Archive · 🗂 All) and two
read-only formulas. You don't build any of it by hand — you **duplicate a ready-made template** and
Claude wires it up. This is the only way to set up the database (it's also the only way to get the
two rolling-date views, whose filters can't be created through the Notion API).

## Duplicate it (the whole setup)

1. **Open the template:**
   **https://thin-goat-86e.notion.site/Opportunities-Shareable-Template-388f26eedb6a8021a6d4db8f4fa3312f**
2. Click **Duplicate** (top-right) and choose your own workspace. You now have your own copy of the
   **Opportunities** database — all 7 views, both formulas, and a handful of **fake sample rows** so you
   can see each view working.
3. **Let Claude connect it** — paste this (and say where you put it if it asks):
   > "I duplicated the Opportunities template into my Notion. Find my Opportunities database and write
   > its database ID and data-source ID into `config.local.md`."

   Claude finds it through the connector and fills in your config. If more than one page is called
   "Opportunities", it'll ask which one — or you can paste the database's URL.
4. The sample rows are fictional — delete them whenever you like (keep them while you learn the views,
   then clear them).

That's it: the database is ready and your config points at it.

## Notes

- Each duplicate gets its **own** database ID — that's expected; it's why the IDs live in your
  `config.local.md` and not in the skills.
- The **7 views and both formulas copy** with the template, including the rolling-date filters on
  🎯 Today and ⏰ Stale follow-ups. (Those relative-date filters can't be created through the connector
  API — which is exactly why the database is shipped as a duplicatable template rather than built from
  a schema.)
- **If Claude can't find your copy**, the connector probably lacks access to the new page: open the
  duplicated page → "•••" → **Connections** → add your Notion connection, then ask Claude to try again.

## What's inside (reference)

You don't need these to set up — they just document what you're duplicating:
- [`opportunities-db-schema.md`](opportunities-db-schema.md) — every property, with colours and formulas.
- [`opportunities-views-spec.md`](opportunities-views-spec.md) — what each of the 7 views filters and shows.

---

**Already set up before this template existed?** Same path: duplicate the template, then tell Claude to
re-point `config.local.md` at the new database. If you've already logged **real** applications in an old
database, tell Claude — that's a quick data migration rather than a fresh start.
