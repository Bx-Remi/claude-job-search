# Job Search System — Configuration

Copy this file to **`config.local.md`** and fill in **your own** IDs.
`config.local.md` is gitignored, so your IDs never get committed.

**How to find each ID:** the easiest way is to ask Claude once your connectors are wired
(see [SETUP.md](SETUP.md)) — e.g. *"what's the ID of my Notion Opportunities database?"* or
*"what's my Todoist 'Job Search' project ID?"*. You can also read most IDs out of the URL in
Notion / Todoist.

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
