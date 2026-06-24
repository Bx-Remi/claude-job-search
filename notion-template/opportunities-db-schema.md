# Opportunities database — schema

This is the backbone of the system: **one Notion database, one row per (Company + Role)**,
the single source of truth for your pipeline. Todoist holds the *action* tasks; Notion holds
the *state*.

You don't build this by hand — you [duplicate the ready-made template](README.md), which has all of
this plus the 7 views. This page is the human-readable reference for what that template contains;
[`opportunities-db-schema.json`](opportunities-db-schema.json) is the machine-readable version.

> ⚠️ **The SELECT option strings are emoji-prefixed and matched exactly.** The skills write
> values like `"📨 Applied"` (not `"Applied"`); a mismatch makes the Notion API reject the
> write. This is why you duplicate the published template instead of hand-building the database or
> hand-typing the options.

## Properties (17 editable + 2 formulas)

| Property | Type | Options / notes |
|---|---|---|
| **Company** | Title | The row's title. |
| **Role** | Text | Job title. |
| **Status** | Select | `🔍 Sourcing` · `📨 Applied` · `📞 Recruiter screen` · `👤 Hiring manager` · `📝 Take-home` · `🎯 Final stage` · `💌 Offer` · `🎉 Offer accepted` · `⚰️ Dead` |
| **Excitement** | Select | `5 ⭐⭐⭐⭐⭐` · `4 ⭐⭐⭐⭐` · `3 ⭐⭐⭐` · `2 ⭐⭐` · `1 ⭐` — how much *you* want it (fairly stable). |
| **Track** | Select | `🅰️ A` (targeted/tailored) · `🅱️ B` (volume/quick-apply). |
| **Temperature** | Select | `🔥 Hot` · `☀️ Warm` · `❄️ Cool` — how *live* the lead is (changes over time). |
| **Source** | Select | `LinkedIn` · `Reed` · `Adzuna` · `Otta` · `Referral` · `Direct` · `Other` |
| **Link** | URL | The job posting. |
| **Salary range** | Text | Free-form, e.g. "£45–55k" or "TBD". |
| **Work model** | Select | `🏠 Remote` · `🔀 Hybrid 1d`–`4d` · `🔀 Hybrid TBD` · `🏢 Onsite` |
| **Location** | Text | e.g. "London", "Remote UK". |
| **Applied date** | Date | When you applied. |
| **Last contact** | Date | Last touch either way — drives the follow-up clock. |
| **Next action** | Text | The single next thing to do. |
| **Next action date** | Date | When to do it. |
| **Days since last contact** | Formula | `dateBetween(now(), prop("Last contact"), "days")` — read-only; powers stale-lead nudges. |
| **Days since applied** | Formula | `dateBetween(now(), prop("Applied date"), "days")` — read-only; how long since you applied. |
| **Dead reason** | Select | `Rejected` · `Ghosted` · `Withdrew` · `Role closed` · `Not a fit` · `Other` |
| **Notes** | Text | Free-form running narrative. |

## The two axes that make it work

Most trackers collapse "do I want it?" and "is it live?" into one column. This one keeps them
**separate**, because they move independently:

- **Excitement** = how much you want the role. Set once, rarely changes.
- **Temperature** = how active the conversation is. Changes constantly.

A dream job that's gone quiet is *high excitement, cool temperature*. A backup that's racing to
offer is *low excitement, hot temperature*. Keeping them apart is what lets you triage by "what's
hot right now" without losing sight of "what I actually want".

## Page body convention

Each row's page body starts with a `## Job Description` heading and the JD pasted underneath —
so the full posting travels with the row even if the original link rots.

## Views

The properties above are only half the system — the database ships with **7 views** (🎯 Today,
⏰ Stale follow-ups, 🔥 Active pipeline, 📞 Interviewing, 🔍 Sourcing, ⚰️ Archive, 🗂 All), each a
different lens on the same pipeline. They're documented in
[`opportunities-views-spec.md`](opportunities-views-spec.md) and come with the template you duplicate.
Select-option **colours** are specified per option above, so every copy of the database looks the same.
