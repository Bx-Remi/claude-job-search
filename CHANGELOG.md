# Changelog

All notable changes to claude-job-search. Check here when you `git pull` to see what changed and
whether you need to do anything.

## v1.1.0 — Multi-view Opportunities database (duplicate-a-template setup)

The Opportunities database now comes with **7 views**, each a different lens on the same pipeline:

- **🎯 Today** — live roles with a next action due today or overdue
- **⏰ Stale follow-ups** — applications you applied to over a week ago that have gone quiet
- **🔥 Active pipeline** — everything in flight, grouped by stage
- **📞 Interviewing** — a kanban board of the interview stages
- **🔍 Sourcing** — the "decide whether to apply" shortlist
- **⚰️ Archive** — closed roles (dead or accepted)
- **🗂 All** — the unfiltered master list

**What changed**

- **Setup is now "duplicate a template."** You get the database by duplicating a published Notion
  template (all 7 views, both formulas, sample data) into your workspace; Claude then finds it and
  writes the IDs into `config.local.md`. This replaces the old "Claude builds the database from a
  schema" flow — see [`notion-template/README.md`](notion-template/README.md). It's now the single,
  standard way to get the database.
- **Why a template instead of building it:** two of the views (🎯 Today, ⏰ Stale follow-ups) use
  *rolling* date filters ("on or before today" / "one week ago"). Notion's connector API can't set
  rolling-date filters — but duplicating a template preserves them. So the template is the only way to
  get those two views working, and the setup wizard now leads with it.
- Added a **`Days since applied`** read-only formula (parity with the reference setup). The schema is
  now 19 properties (17 functional + 2 formulas).
- Select-option **colours** are documented per option, so every copy of the database looks the same.
- New reference docs: [`opportunities-views-spec.md`](notion-template/opportunities-views-spec.md)
  (what each view shows) and the updated schema reference.

**If you already set up before this release**

Easiest path: **duplicate the new template** ([`notion-template/README.md`](notion-template/README.md))
and tell Claude to re-point `config.local.md` at it. If you've already logged **real** applications in
an older database, tell Claude instead — it'll migrate them into the new database rather than starting
fresh.

## v1.0.0 — Initial public release

Two-track job-search + first-round interview-prep system for Claude Code: log any pipeline event from
one plain-English sentence into a Notion Opportunities database + a Todoist action layer; scaffold
first-round interview prep from calendar, email, CV, and a web snapshot. Ships the Notion schema, a
STAR-story template, and a one-prompt setup wizard.
