# Queries for each task

VQL has no GROUP BY or COUNT: fetch the rows and count them yourself. Replace the
<placeholders>; check field and value names against the describe result first.

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
