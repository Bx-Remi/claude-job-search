# Opportunities database — the 7 views (reference)

The [template](README.md) ships with these **7 views**, in this order — each a different lens on the
same pipeline. You don't build them; they come with the duplicate. This page just documents what each
one does (handy if you want to tweak one or understand the design).

| # | View | Type | Shows |
|---|------|------|-------|
| 1 | 🎯 Today | table | Live roles with a next action **due today or overdue**. |
| 2 | ⏰ Stale follow-ups | table | Applications you applied to **over a week ago** that have gone quiet. |
| 3 | 🔥 Active pipeline | table, grouped by Status | Everything **in flight**, grouped by stage. |
| 4 | 📞 Interviewing | board, grouped by Status | A **kanban** of the interview stages only. |
| 5 | 🔍 Sourcing | table | The **"decide whether to apply"** shortlist. |
| 6 | ⚰️ Archive | table | **Closed** roles — dead or accepted. |
| 7 | 🗂 All | table | The unfiltered **master list**. |

---

### 1. 🎯 Today — table
- **Filter:** Status is not `⚰️ Dead` and not `🎉 Offer accepted`, and `Next action date` is on or before **today** (rolling).
- **Sort:** Next action date, ascending.
- **Columns:** Company, Role, Next action, Next action date, Status, Track, Temperature, Excitement.

### 2. ⏰ Stale follow-ups — table
- **Filter:** Status is `📨 Applied` and `Applied date` is on or before **one week ago** (rolling).
- **Sort:** Last contact, ascending (oldest first).
- **Columns:** Company, Role, Days since last contact, Last contact, Applied date, Next action, Track, Temperature, Excitement.

### 3. 🔥 Active pipeline — table, grouped by Status
- **Filter:** Status is not `🔍 Sourcing`, not `⚰️ Dead`, not `🎉 Offer accepted`.
- **Group by:** Status. **Sort:** Last contact, descending.
- **Columns:** Company, Role, Next action, Next action date, Last contact, Days since last contact, Track, Temperature, Excitement, Salary range.

### 4. 📞 Interviewing — board, grouped by Status
- **Filter:** Status is `📞 Recruiter screen`, `👤 Hiring manager`, `📝 Take-home`, or `🎯 Final stage`.
- **Group by:** Status. **Sort:** Next action date, ascending.
- **Columns:** Company, Role, Next action, Next action date, Track, Excitement.

### 5. 🔍 Sourcing — table
- **Filter:** Status is `🔍 Sourcing`. **Sort:** Excitement, descending.
- **Columns:** Company, Role, Link, Source, Track, Excitement, Salary range, Work model, Location, Notes.

### 6. ⚰️ Archive — table
- **Filter:** Status is `⚰️ Dead` or `🎉 Offer accepted`. **Sort:** Last contact, descending.
- **Columns:** Company, Role, Dead reason, Applied date, Last contact, Excitement, Track, Salary range, Notes.

### 7. 🗂 All — table
- **Filter:** none. **Sort:** Status, ascending.
- **Columns:** Company, Role, Status, Excitement, Track, Temperature, Applied date, Last contact, Salary range, Notes.

---

> **Why the database is shipped as a template** rather than built from a schema: views 1 and 2 use
> *rolling* date filters ("on or before today" / "one week ago"). Notion's connector API can create
> views but **can't set rolling-date filters** — so duplicating a template (which preserves them) is the
> only way to get these two views working. That's why setup is "duplicate the template," not "build it."
