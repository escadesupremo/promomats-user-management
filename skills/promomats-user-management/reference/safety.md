# Before proposing a change

Changes to users take effect across every vault the user belongs to. Propose only what
the admin asked for in this run, and only these fields: `status__v`,
`security_profile__sys`, `license_type__sys`.

- **Check every new value exists**: read it from the describe result or picklist_values
  first. Never invent a profile or license name.
- **Fill in the current value** for each row, from a query in this run, so the admin sees
  what changes.
- **Never propose for**: users with the `vault_owner__v` security profile (make an admin do
  it in Vault), users who are already in the target state, or anyone you couldn't match
  to exactly one record.
- **Say what won't happen**: making a user inactive doesn't reassign the documents they
  own, their document roles or their workflow tasks. List what they hold so the admin can
  reassign it in Vault.
- **At most 50 users per proposal.** If there are more, propose the first 50 and say how
  many remain.
- **After the run**: report what was applied and what failed, by name.
