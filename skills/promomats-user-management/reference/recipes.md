# Queries

VQL has no GROUP BY or COUNT. To count rows per user, use count_by: it pages through
every row and counts in the extension. Replace the <placeholders>; check field and value
names against the describe result first. `{{days_ago:N}}` becomes midnight UTC N days ago
as a DateTime; `{{date_days_ago:N}}` the same day as a Date, for Date fields.

## Base query (step 1)

```sql
SELECT id, name__v, company__sys, activation_date__sys, inactivation_date__sys,
       license_type__sys, TOLABEL(license_type__sys) AS license_label
FROM user__sys
```

TOLABEL needs API v24.1 or later. Without it, drop the last column.

## Document actions per user, last 12 months (step 2)

count_by, by `created_by__v`:

```sql
SELECT created_by__v
FROM document_usage__v
WHERE created_date__v >= '{{days_ago:365}}'
```

## Tasks per user, last 12 months (step 3)

count_by, by `assigned_to__v`. Held (created or completed in the period):

```sql
SELECT assigned_to__v
FROM user_task__v
WHERE created_date__v >= '{{days_ago:365}}' OR completion_date__v >= '{{days_ago:365}}'
```

Completed:

```sql
SELECT assigned_to__v
FROM user_task__v
WHERE completion_date__v >= '{{days_ago:365}}'
```

## One user's latest document activity

```sql
SELECT id, activity_type__v, document_type__v, created_by__v, created_date__v
FROM document_usage__v
WHERE created_by__v = '<user id>'
ORDER BY created_date__v DESC
MAXROWS 10
```

## One user's latest tasks

```sql
SELECT
    id,
    name__v,
    status__v,
    created_by__v,
    created_date__v,
    modified_by__v,
    modified_date__v,
    assigned_to__v,
    due_date__v,
    first_assigned_date__v,
    last_assigned_date__v,
    completion_date__v,
    complete__v,
    assigned__v,
    description__v,
    lifecycle__v,
    state__v,
    priority__sys,
    global_id__sys,
    link__sys
FROM user_task__v
WHERE assigned_to__v = '<user id>'
ORDER BY created_date__v DESC
MAXROWS 10
```

Use MAXROWS for the number of rows, not LIMIT: LIMIT is deprecated and only sets the page
size, without changing how many rows the query finds.

The queries below are for follow-up questions about particular users, groups or documents.

## Access overview

```sql
SELECT id, name__v, username__sys, security_profile__sys, license_type__sys
FROM user__sys
WHERE status__v = 'active__v'
ORDER BY security_profile__sys
```

## Dormant users

With the vault's last-login field in place of <last_login_field>, and <cutoff> as an ISO
timestamp (today minus the period, midnight UTC, e.g. 2026-08-21T00:00:00.000Z):

```sql
SELECT id, name__v, username__sys, security_profile__sys, <last_login_field>, created_date__v
FROM user__sys
WHERE status__v = 'active__v'
  AND (<last_login_field> < '<cutoff>' OR (<last_login_field> = null AND created_date__v < '<cutoff>'))
ORDER BY <last_login_field> ASC
```

## Offboard a user

Find them:

```sql
SELECT id, name__v, username__sys, email__sys, status__v, security_profile__sys, license_type__sys
FROM user__sys
WHERE email__sys = '<email>' OR username__sys = '<username>'
```

A name instead: `WHERE name__v CONTAINS ('<name>')`, or `LIKE '<start>%'`.

Their groups:

```sql
SELECT group__sysr.label__v, group__sysr.name__v, group__sysr.system_group__sys
FROM group_membership__sys
WHERE user_id__sys = '<user id>'
```

Their document roles:

```sql
SELECT document_id, role_name__sys, document__sysr.name__v, document__sysr.document_number__v
FROM doc_role__sys
WHERE user__sys = '<user id>'
```

## Document roles

On one document (find its ID first from its number with
`SELECT id FROM documents WHERE document_number__v = '<number>'`):

```sql
SELECT role_name__sys, user__sysr.name__v, user__sysr.username__sys, group__sysr.label__v
FROM doc_role__sys
WHERE document_id = <document id>
```

Everything a group holds:

```sql
SELECT document_id, role_name__sys, document__sysr.name__v
FROM doc_role__sys
WHERE group__sysr.label__v = '<group label>'
```

## Group members

```sql
SELECT user__sysr.name__v, user__sysr.username__sys, user__sysr.status__v, user__sysr.security_profile__sys
FROM group_membership__sys
WHERE group__sysr.label__v = '<group label>'
```

User-managed groups only: `SELECT id, name__v, label__v FROM group__sys WHERE type__sys = 'user_managed__sys'`.
