# Team migration excuses triage

All uploads to the Archive have to go through
[proposed migration](ProposedMigration.md) and all of the Ubuntu teams
are responsible to unblock it - in particular via the
[+1 Maintenance program](https://wiki.ubuntu.com/PlusOneMaintenanceTeam).

But teams own packages they are more responsible for and for these we want to
watch more directly. The link
[migration excuses - server team](https://ubuntu-archive-team.ubuntu.com/proposed-migration/update_excuses_by_team.html#ubuntu-server)
shows all current cases of our team.

## Tooling

For all [triage](Triage.md) input, we use [`startriage`](https://github.com/ubuntu/startriage).


## Constraints

In this view there will always be a lot of noise (recent cases worked on already,
syncs and transitions that are still ongoing, ...) and also known issues (things
taking a bit of extra time to resolve). But in between there is a set that is
benefitting form us to look at and unblock.

To make actions reasonable, worthwhile and reduce the amount of duplicate effort
we apply some basic limits what we look at - which helps to keep the time spent
at triage low.

* only after more than three days take a look at retriggering flaky tests
* only after a week (or if it is our own upload) check if it needs a deeper look

## What to tackle

### Trigger tests

The most common case is infrastructure issues or flaky tests that only need a
little nudge. Those are considered low effort but are helpful to be re-ran once
we look deeper into it (to know the rate of success/fail). Those tests should
be triggered to re-run if the triage thinks they might formerly have been victim
of such and work on a re-run.

### Deeper look

We'd not expect the triager to have a deeper look. If something looks really odd
and seems it needs a deeper look, bring it up at the next post standup discussion.
Chances are high that someone will recognize an ongoing effect of uploads or
infrastructure.

Only if that does not resolve it we will create a proper bug for it, tag it
server-todo, tag it update-excuse and assign it for bug-work. From there on
[housekeeping](Triage.md#weekly-bug-housekeeping) is the continuation.
