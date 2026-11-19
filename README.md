# PromoMats user management

A [Vault Dweller](https://github.com/escadesupremo/vault-dweller-extension) skill for Vault
admins: how much each user of a PromoMats vault has used it over the last 12 months, who
needs a look, and an Excel file to work from.

## Install

In Vault Dweller, open **Skills**, paste this link and choose **Import from GitHub**:

```
github.com/escadesupremo/promomats-user-management
```

## What it does

1. Lists every user: company, activation and inactivation dates, license type.
2. Counts each user's document actions over the last 12 months (views, downloads and
   copies, from Document Usage records).
3. Counts the user tasks each user held and completed over the same period.
4. Flags active users who:
   - have no document actions and no tasks, and were activated more than 60 days ago;
   - read documents but have no tasks, on a Full User license.
5. Hands over an Excel file: a Users sheet (grid, frozen header, filters, flagged users
   first and shaded) and an Explanation sheet with the columns, the flag rules and the
   queries behind each count.

Vault Dweller counts and flags every row itself, so the numbers are exact however many
users the vault has; the AI model sees only the first rows and the totals. The skill
reads with VQL (`user__sys`, `document_usage__v`, `user_task__v`) and changes `user__sys`
records only through Vault Dweller's approval step, never because a user was flagged.

## Layout

```
skills/promomats-user-management/
  SKILL.md              the instructions and what the skill may use
  reference/objects.md  the user, usage, task, group and role objects and fields
  reference/recipes.md  the queries behind each step, and follow-ups
  reference/safety.md   the rules for proposing changes
```

See Vault Dweller's [SKILLS.md](https://github.com/escadesupremo/vault-dweller-extension/blob/main/SKILLS.md)
for the format.
