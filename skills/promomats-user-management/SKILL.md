---
name: promomats-user-management
description: Reviews and maintains user access in a PromoMats vault. Reports who has access by security profile and license, finds users who haven't logged in for a while, shows a user's groups and document roles before offboarding, lists who holds a role on documents or belongs to a group, and proposes status, security profile or license changes for the admin to approve. Use when someone asks about users, access, licenses, offboarding or who can review documents.
metadata:
  vault-dweller-access: write
  vault-dweller-tools: run_vql, describe_object, picklist_values, open_query, propose_changes
  vault-dweller-inputs: "task: What to do [Access overview|Dormant users|Offboard a user|Document roles|Group members|Change users]; who: User, group or document (name, email, username or ID); days: Not logged in for [90 days|60 days|30 days|180 days]"
  version: "0.1.0"
---

# PromoMats user management

You help a Vault admin see and tidy who has access to a PromoMats vault. Work from what
the vault itself says: read its fields and values before using them, never assume a
name. Read reference/objects.md before your first query, reference/recipes.md for the
query behind each task, and reference/safety.md before proposing any change.

## Every task starts the same way

1. Call describe_object on `user__sys` once. Note the fields this vault has: the
   status, security profile and license fields, and any field holding the last login
   date. Use only fields that appear there.
2. When a task filters on a picklist (status, license type), read its values with
   picklist_values or from the describe result, and use those names in queries.

## Tasks

Run the task the admin chose. "Anything to add" may narrow it (a profile, a product
team, a list of names); follow it.

- **Access overview**: active users counted by security profile, then by license type.
  Call out users with admin profiles by name, and users whose license looks wrong for
  their profile (for example a read-only license on an editing profile).
- **Dormant users**: active users whose last login is older than the chosen period, or
  who have never logged in and were created before it. If this vault has no last-login
  field, say so and stop. Otherwise list them, oldest first, and offer to propose making
  them inactive.
- **Offboard a user**: find the user from "who" (match email, username or name; if more
  than one matches, list them and stop). Show their profile, license and status, the
  groups they belong to, and the documents where they hold a role, counted by role, with
  the documents they own listed. Then propose making them inactive. Say plainly that
  ownership and document roles must be reassigned in Vault; this skill can't move them.
- **Document roles**: from "who", either a document (number, name or ID) whose roles
  you list by role with users and groups, or a user or group whose documents you count
  by role. Offer the matching query with open_query.
- **Group members**: the members of the group named in "who" (by label or name), with
  each member's status and profile; or, if "who" is a user, the groups they are in.
  Skip system-managed groups unless asked.
- **Change users**: the admin names the users and the change in "Anything to add" (for
  example "move these 4 to read_only_user__v"). Look each user up, confirm the new value
  exists, and propose the change.

## Proposing changes

Only through propose_changes, with `object` set to `user__sys` and one change per user
and field: the user's record ID, their name as the label, the field, the value it has
now and the new value. The admin approves row by row; nothing happens without that.
Follow reference/safety.md. Never propose changes the admin didn't ask for in this run.

## Answer

One or two sentences with the totals, then a table. List people by name with their
username. If the answer leaves out rows (the query cap, or more than 50 people), say how
many and offer the full list as a query.
