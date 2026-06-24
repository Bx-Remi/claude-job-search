# Job Search System — Configuration

Copy this file to **`config.local.md`** and fill in **your own** IDs.
`config.local.md` is gitignored, so your IDs never get committed.

**The easy way — let Claude fill this in.** Once your connectors are wired (see [SETUP.md](SETUP.md)),
copy this file to `config.local.md`, then paste the prompt below to Claude, filling in the
**[bracketed]** names. Claude finds every ID through the connectors and writes them into the file — you
never read IDs out of a URL or paste them yourself.

> "Fill in `config.local.md` for me. My Notion job-search database is called **Opportunities** (under
> the page **[your Notion page name]**), and my Todoist project is **[🔍 Job Search]**. Find their IDs
> — the Notion database ID and data-source ID, and the Todoist project ID — and write them into the
> file. If I've connected Google Drive, also find my master CV **[your CV file name]** and add its ID.
> Set USER_NAME to **[your first name]**. Ask me for anything you can't find."

Both skills resolve every `{{PLACEHOLDER}}` from this file. If a required value is missing,
the skill stops and asks you to finish setup — it will **not** run with empty IDs.

## Notion (required)
```
NOTION_OPPORTUNITIES_DB:   <your Opportunities database ID>
NOTION_OPPORTUNITIES_DS:   <your Opportunities data-source ID>
NOTION_PARENT_PAGE:        <the page that holds the database>
```

## Todoist (required)
```
TODOIST_PROJECT:           <your Job Search project ID>
TODOIST_SYSTEM_SECTION:    <optional — a section for system/dev tasks; leave blank if unused>
```

## Google Drive (optional — only used by interview-prep for CV matching)
```
DRIVE_MASTER_CV:           <optional — Google Doc ID of your master CV>
DRIVE_JOBSEARCH_FOLDER:    <optional — Drive folder ID for prep outputs>
```

## Local paths
```
PREP_OUTPUT_DIR:           ./output/interview-prep   # where interview-prep writes its docs
STORIES_DIR:               ./templates               # where your STAR stories live
```

## Identity
```
USER_NAME:                 <your first name>          # used to phrase pitches in the first person
```
