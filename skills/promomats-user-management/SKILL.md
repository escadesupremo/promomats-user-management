---
name: promomats-user-management
description: Reports how active every user of a PromoMats vault has been over the last 12 months (document actions and tasks per user), flags users who have done nothing and Full User licenses that only read documents, and hands over a clean Excel file with an explanation sheet. Use when someone asks about user activity, idle or dormant users, unused licenses, access reviews or offboarding.
metadata:
  vault-dweller-access: write
  vault-dweller-tools: run_vql, count_by, export_excel, describe_object, picklist_values, open_query, propose_changes
  version: "0.5.0"
---

# PromoMats user management

You help a Vault admin see how much each user of a PromoMats vault has used it over the
last 12 months, and who needs a look. The data comes from the admin's live Vault session
through the tools; never assume a field or value the vault hasn't shown you.

The result is an Excel file: one row per user with their document actions and tasks over
the last 12 months, a Flags column, and an Explanation sheet. The extension does the
counting and flagging on every row, so the numbers are exact however many users the
vault has. Your job is to make the right calls and explain the result.

## Step 1: establish the users

Run this base query with run_vql:

```sql
SELECT id, name__v, company__sys, activation_date__sys, inactivation_date__sys,
       license_type__sys, TOLABEL(license_type__sys) AS license_label
FROM user__sys
```

This is the list every later step works from: active and inactive users alike, with
their company, the date their access started, the date it ended (empty while they are
still active) and their license type, by name and by label. Keep its result_id.

- If the query fails on TOLABEL, run it again without `TOLABEL(license_type__sys) AS
  license_label` and use license_type__sys for the License column.
- If company__sys comes back as record IDs rather than names, find its relationship with
  describe_object on user__sys and add `<relationship>.name__v` to the query, so the
  Excel shows company names.
- The result gives the total number of users. If it holds fewer rows than that, say so in
  the summary.

## Step 2: count document actions per user

Vault adds a Document Usage record each time someone views, downloads or copies a
document, or downloads a rendition, in its steady state. Count the last 12 months by
user with count_by:

```sql
SELECT created_by__v
FROM document_usage__v
WHERE created_date__v >= '{{days_ago:365}}'
```

Count by `created_by__v`. Keep the result_id.

## Step 3: count tasks per user

Two counts with count_by, both by `assigned_to__v`:

Tasks the user held in the last 12 months (created or completed in that time):

```sql
SELECT assigned_to__v
FROM user_task__v
WHERE created_date__v >= '{{days_ago:365}}' OR completion_date__v >= '{{days_ago:365}}'
```

Tasks the user completed in the last 12 months:

```sql
SELECT assigned_to__v
FROM user_task__v
WHERE completion_date__v >= '{{days_ago:365}}'
```

If a query fails because completion_date__v is a Date field rather than a DateTime, use
`'{{date_days_ago:365}}'` for it. Keep both result_ids.

## Step 4: the flags

Two flags, applied by the extension to every user. Both look only at users who are still
active (no inactivation date): a user whose access has ended needs nothing more.

1. **No document actions or tasks in 12 months**: no document actions, no tasks, and
   active for at least 60 days (activated more than 60 days ago), so new users aren't
   flagged.
2. **Reads documents but has no tasks, on a Full User license**: document actions but no
   tasks, on a Full User license. Full User is Vault's highest license type, so "full
   license or better" means Full User. The rows from step 1 show license_type__sys next to
   license_label: use the name that goes with "Full User" (usually `full__v`).

## Step 5: make the Excel

Call export_excel once, with the result_ids from steps 1 to 3 in place of `<users>`,
`<docs>`, `<tasks>` and `<tasks done>`, and `full__v` replaced if the vault names Full User
differently:

```json
{
  "filename": "promomats-user-activity",
  "title": "PromoMats user activity, last 12 months",
  "sheet": "Users",
  "noun": "users",
  "summary": "<two or three sentences: how many users, how many flagged for each flag, anything the admin should know>",
  "base": "<users>",
  "key": "id",
  "columns": [
    { "id": "user", "title": "User", "field": "name__v" },
    { "id": "company", "title": "Company", "field": "company__sys" },
    { "id": "license", "title": "License", "field": "license_label" },
    { "id": "activated", "title": "Activated", "field": "activation_date__sys" },
    { "id": "inactivated", "title": "Inactivated", "field": "inactivation_date__sys" },
    { "id": "docs", "title": "Document actions (12 months)", "counts": "<docs>",
      "description": "Views, downloads and copies of steady-state documents, and rendition downloads, the user made in the last 12 months (Document Usage records)." },
    { "id": "tasks", "title": "Tasks (12 months)", "counts": "<tasks>",
      "description": "User tasks assigned to the user that were created or completed in the last 12 months." },
    { "id": "tasks_done", "title": "Tasks completed (12 months)", "counts": "<tasks done>",
      "description": "Of those, the tasks the user completed in the last 12 months." }
  ],
  "flags": [
    { "text": "No document actions or tasks in 12 months",
      "when": { "all": [
        { "col": "docs", "op": "eq", "value": 0 },
        { "col": "tasks", "op": "eq", "value": 0 },
        { "col": "activated", "op": "older_than_days", "value": 60 },
        { "col": "inactivated", "op": "empty" } ] } },
    { "text": "Reads documents but has no tasks, on a Full User license",
      "when": { "all": [
        { "col": "docs", "op": "gt", "value": 0 },
        { "col": "tasks", "op": "eq", "value": 0 },
        { "col": "license_type__sys", "op": "in", "value": ["full__v"] },
        { "col": "inactivated", "op": "empty" } ] } }
  ],
  "notes": [
    "Full User is Vault's highest license type, so \"full license or better\" means Full User.",
    "Users whose access has ended (an inactivation date) are listed but never flagged."
  ]
}
```

Change the License column to `license_type__sys`, or the Company column to the company
relationship's name, if step 1 had to. Write the summary from the counts the tool returns,
not from the rows you saw: you only see the first rows of each result.

## Step 6: answer

Keep it short: the number of users, how many each flag caught, and anything worth a
closer look. Flagged accounts that look like system or integration users (a name such as
System, Integration or API) are worth pointing out: they often hold a Full User license
without doing visible work. Don't list every user; the Excel has them. The download
button appears under your answer.

## Looking closer at one user

When the admin asks about a particular user, read their latest document activity and
tasks with run_vql, with the user's `id` from step 1 in place of <user id>:

```sql
SELECT id, activity_type__v, document_type__v, created_by__v, created_date__v
FROM document_usage__v
WHERE created_by__v = '<user id>'
ORDER BY created_date__v DESC
MAXROWS 10
```

```sql
SELECT id, name__v, status__v, created_by__v, created_date__v, modified_by__v,
       modified_date__v, assigned_to__v, due_date__v, first_assigned_date__v,
       last_assigned_date__v, completion_date__v, complete__v, assigned__v,
       description__v, lifecycle__v, state__v, priority__sys, global_id__sys, link__sys
FROM user_task__v
WHERE assigned_to__v = '<user id>'
ORDER BY created_date__v DESC
MAXROWS 10
```

Open tasks (not complete) and overdue ones (due date passed, not complete) matter most:
they are what would be left behind if the user's access ended. reference/recipes.md has
more follow-up queries: groups, document roles, offboarding.

## Proposing changes

Only through propose_changes, with `object` set to `user__sys`: the user's record ID from
the base query, their name as the label, the field, the value it has now and the new
value. The admin approves row by row; nothing happens without that. Never propose changes
just because a user was flagged: the flags are for the admin to review. Follow
reference/safety.md.
