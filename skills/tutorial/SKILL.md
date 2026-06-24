---
name: tutorial
description: >
  Friendly, interactive walkthrough of how to use the job-search system day-to-day, plus a guided tour
  of the Notion Opportunities database and its 7 views. Use when the user wants to learn the workflow or
  understand their tracker. Trigger phrases: "tutorial", "how do I use this", "walk me through it", "how
  does this work", "teach me the workflow", "give me a tour", "show me how the system works", "how does
  the database work", "explain the views", or a first-time "now what?" after setup. Also offered at the
  end of the setup and upgrade wizards. (Logging an actual event — "I applied to X", "heard back from Y",
  "found this role [link]" — is the `job-search-action` skill, not this one.)
metadata:
  version: "1.0.0"
---

# Tutorial — how to actually use this

You're teaching a real person how to use the system. Make it **pleasant and light**, not a lecture.
This is a conversation, not a document dump.

## Operating principles (read before you start)

- **Interactive, one chunk at a time.** Cover a piece, then check in ("make sense?" / "want to try
  one?") before the next. Never paste the whole tutorial at once.
- **Short and warm.** Plain English, a little personality, no jargon walls. Keep each turn skimmable.
- **Use their real system** where you can — their name (`USER_NAME` from `config.local.md`), their
  actual Opportunities database, their real views. It's their tutorial, not a generic one.
- **Offer, don't force.** Let them pick what to cover and how deep. Some want a 2-minute overview;
  some want a hands-on rep. Both are fine.
- **Be accurate.** Only describe what the system actually does (this file is kept in sync with the
  `job-search-action` and `interview-prep` skills). Don't promise capabilities that aren't there.

## Before you start

Read `config.local.md` for `USER_NAME` (to greet them) and `NOTION_OPPORTUNITIES_DB` (their tracker, for
the views tour). If it's missing or blank, they haven't finished setup yet — say so warmly and point them
to the setup wizard (*"just say: set me up"*) rather than touring a database that doesn't exist.

## Phase 0 — Welcome & pick a path

Greet them by name and offer a short menu:

> *"Want me to walk you through it? I can cover:
> 1. **The day-to-day flow** — what you actually say to me as you job-hunt (finding roles, applying,
>    hearing back, interviews, offers).
> 2. **A tour of your Notion tracker** — the 7 views and what each is for.
> Either, both, or just the highlights — your call. And do you want the quick version or a hands-on
> walkthrough with a real example?"*

Then run the part(s) they pick. If they say "just show me everything," do Part A then Part B, checking
in between.

**Honour their depth choice** — it's load-bearing, not a throwaway question:
- **Quick** → the highlights only: the one-idea sentence, a compressed 3-beat version of the lifecycle
  (find → apply → hear back), and a one-line-each pass over the 7 views. Skip the concept asides and the
  live rep unless they ask. Aim for ~2 minutes.
- **Hands-on** → the full thing, one chunk at a time, and push the live example early (Part A) and walk
  a real row through a couple of views (Part B).

Don't run the full walkthrough at someone who asked for the quick version, or vice versa.

---

## Part A — The day-to-day flow

**The one idea to land first:** *"You never touch Notion or Todoist by hand. You just tell me what
happened in plain English, and I keep both in sync — Notion holds the **state** of every role, Todoist
holds your **next actions** with due dates."*

Then walk the lifecycle. Give the **exact kind of phrase** they'd use and what you'll do with it. Cover
these, pausing to check in (don't dump all at once):

1. **Found a role you might want** — *"Easiest: just paste me the job link, or say 'found this at
   [company]'. I'll pull the details and add it to your pipeline as **Sourcing** — something to decide
   on, not yet applied."* (Pasting the link is the smoothest path — I read the posting for you.)
2. **You applied** — *"Tell me 'I applied to [company] for [role]'. I'll log it as **Applied**, set a
   follow-up reminder about a week out (5 working days), and a backstop so a silent application can't
   quietly die on you. I'll ask two quick things if you didn't say: was it a **tailored** application
   (Track A) or a **quick apply** (Track B), and how much you want it (**Excitement**, 1–5)."*
3. **You heard back** — *"Say 'heard back from [company] — recruiter screen Thursday' (or 'moving to the
   hiring-manager round', etc.). I move it to the right stage and set your next action and date."*
4. **Interview coming up** — *"Say 'prep me for my [company] interview' and I'll build you a prep brief.
   (Heads up: that covers **first-round** recruiter/hiring-manager screens for now — not technical,
   take-home, panel, final, or negotiation stages.)"*
5. **An outcome** — *"'Rejected by [company]' or 'they ghosted me' → I mark it **Dead** with the reason.
   'Got an offer from [company]!' → **Offer**, and 'I'm taking it' → **Offer accepted**. 🎉"*
6. **Staying on top of it** — *"Ask me 'what's due today?' or 'follow up on anything I've gone quiet on'
   and I'll surface what needs a nudge."*

**Two concepts worth 20 seconds each** (weave in, don't lecture):
- **Track A vs Track B** — A = the handful you tailor hard for; B = high-volume quick-applies. Same
  logging, different effort. It's set by how much work *you* put in, not the role.
- **Excitement vs Temperature** — Excitement = how much you *want* it (steady). Temperature = how *live*
  the conversation is (changes). That's why a dream job that's gone quiet and a backup that's racing to
  offer can sit in different places — and why you can triage by "what's hot" without losing "what I
  actually want."

**Offer a live rep:** *"Want to try one for real? Paste a link to a role you're eyeing, or tell me about
one you applied to — I'll run it through and you'll see it land in Notion and Todoist."* If they'd rather
not use real data yet, offer to demo with a made-up example and note you won't save it.

---

## Part B — Tour of the Notion tracker

Open (or point them to) their **Opportunities** database. Set the frame:

> *"This is the one place that holds every role — one row each. You don't edit it by hand; me updating it
> from what you tell me is what keeps it honest. The power is the **7 views** — same data, different
> lenses. Here's when to reach for each:"*

Walk the 7 views — keep each to a sentence or two, and tell them *when they'd actually look at it*:

1. **🎯 Today** — *your morning check-in.* Live roles with an action due today or overdue. If you open
   one view a day, this is it.
2. **⏰ Stale follow-ups** — applications you sent over a week ago that have gone quiet — your "who needs
   a nudge" list.
3. **🔥 Active pipeline** — everything you've applied to and beyond, grouped by stage — the active funnel
   at a glance (it leaves out roles you're still just sourcing).
4. **📞 Interviewing** — a kanban board of just the interview stages — a quick visual of who's at which round.
5. **🔍 Sourcing** — roles you're still deciding whether to apply to, sorted by how much you want them.
6. **⚰️ Archive** — closed roles (rejected/ghosted/withdrawn or accepted) — kept for the record.
7. **🗂 All** — the unfiltered master list, when you just want to see everything.

**The columns that matter** (mention briefly): **Status** (the stage), **Excitement** vs **Temperature**
(the two axes from Part A), **Track** (A/B), **Next action** + **Next action date** (what to do and
when), **Days since last contact** (auto-calculated — this is what powers the follow-up nudges), and
**Days since applied** (how long a silent application has been out).

If their database has data, walk one real row end-to-end ("here's Acme — it's at *Recruiter screen*,
Excitement 4, next action Thursday…"). If it only has the sample rows, use those — they're built to show
every view working.

---

## Wrap up

Close warmly and point at the obvious next move:

> *"That's the whole thing — just talk to me as things happen and I keep it all straight. Want to start
> by logging a real role (paste a link), or prepping an interview?"*

Keep it to one or two sentences. Don't re-summarize everything you just covered.
