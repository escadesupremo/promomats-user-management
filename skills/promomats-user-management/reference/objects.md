# The objects behind users, groups and document roles

From Veeva's VQL and Vault API reference (v26.2). Field lists vary by vault: confirm with
describe_object on `user__sys` before relying on anything below.

## Users: `user__sys`

Since v18.1 users are `user__sys` records; Veeva recommends updating them as object
records. Fields every vault has:

| Field | |
| --- | --- |
| `id` | Record ID: what propose_changes needs |
| `first_name__sys`, `last_name__sys`, `name__v` | Names |
| `username__sys` | The login, e.g. `jdoe@acme.com` |
| `email__sys` | Email address |
| `status__v` | Active or inactive (`active__v`, `inactive__v` as names) |
| `security_profile__sys` | The security profile, e.g. `document_user__v` |
| `license_type__sys` | e.g. `full__v`, `read_only__v`; vaults on user-based licensing also have application license fields starting `license_` |
| `activation_date__sys` | First day the user can sign in |
| `language__sys`, `locale__sys`, `timezone__sys` | Preferences |
| `created_date__v`, `modified_date__v` | Audit dates |

A last-login date is not in that list. Look for one in the describe result; if there is
none, dormant-user checks can't be done from VQL.

Relationship: `group_membership__sysr` (down) to the user's group memberships.

## Groups: `group__sys` and `group_membership__sys`

`group__sys` fields: `id`, `name__v`, `label__v`, `status__v`, `description__sys`,
`system_group__sys` (true when Vault manages the group), `type__sys` (e.g.
`user_managed__sys`), `created_date__v`, `modified_date__v`.

`group_membership__sys` fields: `id`, `user_id__sys`, `group_id__sys`, with parent
relationships `user__sysr` and `group__sysr`. Both `user__sys` and `group__sys` reach it
through `group_membership__sysr`.

## Document roles: `doc_role__sys` (v21.1 and later)

Fields: `role_name__sys` (e.g. `owner__v`, `reviewer__v`, `approver__v`), `document_id`,
`user__sys`, `group__sys`. Relationships: `user__sysr`, `group__sysr`, `document__sysr`.
From documents: `doc_roles__sysr` (one to many).

Document roles can only be read here; reassigning them is done in Vault.
