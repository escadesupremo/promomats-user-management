# PromoMats user management

A [Vault Dweller](https://github.com/escadesupremo/vault-dweller-extension) skill for Vault
admins: who has access to a PromoMats vault, who hasn't used it, and what a user holds
before they leave. It can propose status, security profile and license changes, which
the admin approves row by row before anything changes.

## Install

In Vault Dweller, open **Skills**, paste this link and choose **Import from GitHub**:

```
github.com/escadesupremo/promomats-user-management
```

## What it does

| Task | |
| --- | --- |
| Access overview | Active users by security profile and license type; admins by name |
| Dormant users | Active users who haven't logged in for 30 to 180 days (needs a last-login field in the vault) |
| Offboard a user | Their profile, groups and document roles, then a proposal to make them inactive |
| Document roles | Who holds which role on a document, or what a user or group holds |
| Group members | A group's members, or a user's groups |
| Change users | Status, security profile or license changes the admin lists |

It reads with VQL (`user__sys`, `group__sys`, `group_membership__sys`, `doc_role__sys`)
and changes `user__sys` records only through Vault Dweller's approval step. It can't
reassign document ownership, roles or tasks; it lists them so you can do that in Vault.

## Layout

```
skills/promomats-user-management/
  SKILL.md              the instructions and what the skill may use
  reference/objects.md  the user, group and role objects and fields
  reference/recipes.md  the query behind each task
  reference/safety.md   the rules for proposing changes
```

See Vault Dweller's [SKILLS.md](https://github.com/escadesupremo/vault-dweller-extension/blob/main/SKILLS.md)
for the format.
