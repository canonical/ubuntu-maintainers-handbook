# Triage

## Tooling

We use [`startriage`](https://github.com/ubuntu/startriage) for all triaging.
It displays multiple sources (Launchpad, Discourse, Github, proposed-migration, ...) for triaging decisions.

```bash
snap install startriage
```

For daily triage:

```bash
startriage triage
```

For bug housekeeping:

```bash
startriage todo
```


The tool has many options for filtering and display - please look at its `--help` output or the git repository / readme.


## Triage rotation

According to load we might shift things, but generally every day Tuesday to
Friday one team member is assigned to triage duty - which means they cover
anything that happened the day before.

Monday is often more work since it includes all of the weekend. Therefore,
Monday is rotated on a longer pattern through triagers. Due to that a Monday
every six weeks is roughly about as much effort as a weekday every other week.

This is organised internally in the team's Jira, and automatically creates a
Story with the "bug-triage" label assigned to the person on rotation.

This rotation covers all three aspects:

- [Bug Triage](BugTriage.md)
- [Documentation Triage](DocTriage.md)
- [Forum Triage](ForumTriage.md)
- [Team migration Triage](TeamMigrationTriage.md)

### Can a triage be skipped?

No, the concept is that the triager looks at the cases touched on the day
before. The one following triager will not see what happened two days before.

The tools all allow to catch up, but since people might wonder why there is
no reaction we should never wait too long. If one can't make it for any reason
we should look for timely coverage by someone else.

An exception is the global end of year shutdown of Canoncal, after that we will
as a team look at all that happened once we are back.

### Awareness of the triage

We have several stakeholders to keep up-to-date on things we've found during
triage. We also want to keep the community generally informed, as well as
raising issues within the team to ensure they are not being forgotten.

For the community we send a mail to `ubuntu-server@lists.ubuntu.com` that
summarises how many bugs we've triaged, and touches on the noteworthy cases.
This can also be used to CC additional people that (for case-specific reasons)
should be aware of a case.

An example of that would be if a security fix caused an upgrade-regression
which would make us CC the uploader and/or ubuntu-security. This mail should
also contain relevant information from [documentation triage](DocTriage.md).

Furthermore, on cases that need immediate attention (or at least awareness)
we might:

* Bring them up in the daily standup (if they need a discussion/decision that
  one can't do alone).
* Ping a subject matter expert via IRC/Mattermost.

In some cases, a package maintainer might already be aware and following the
case. To avoid endless re-pings on such a case the agreement is that if the
maintainer is personally subscribed (i.e. with their launchpad username, not
just indirectly via teams like
[Ubuntu Virtualization](https://launchpad.net/~ubuntu-virt))
then we consider the maintainer to be aware and we will not do extra
pings/mentions/CC.

## Weekly bug housekeeping

In addition to the daily triage (and our ongoing dedication to resolving bugs
we picked up that way) the Server Team has introduced a weekly bug housekeeping
session. In that session we go through various lists, ensuring that no bugs or
issues are forgotten, blocked or stalled for too long.

There are a few additional steps we take, which change over time based on
current priorities. For example, looking for good candidates to do MREs in the
future, or trying to assign our remaining cleanup on the Discourse
documentation feedback backlog. While these may change, there is a core
structure to what we always look at in this meeting:

1. Check the `server-todo` tagged bug list:

   1. Get list via `startriage`:

      configure `savebugs_dir` in [`~/.config/startriage.toml`](https://github.com/ubuntu/startriage/blob/main/startriage/data/defaults.toml)
      ```bash
      startriage todo
      ```

      The list of last week's bugs helps to identify new/closed cases and is
      [tracked in a repository](https://code.launchpad.net/~ubuntu-server/+git/triage-savedbugs)

   1. Check size (see min/max above) of the `server-todo` tagged bug list.
   1. Ensure assigned bugs make reasonable progress:

      1. Discuss blockers/reasons if there was no progress.
      1. Notice, enjoy and celebrate progress that was made.

   1. Ensure unassigned bugs find an owner:
      1. Ensure long term unassigned bugs are re-reevaluated (is there a
         reason why they are not tackled?)

1. Look at [update-excuses by team](https://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses_by_team.html#ubuntu-server)
   to spot anything that needs our attention to migrate. If there are any team
   members assigned to analyse the case and ensure things are progressing.
1. Look at our [merges schedule](http://pinot.endarchy.org:4200/merges-schedule)
   to identify if we have fallen behind on any of them.
1. Look at [reviews needed](https://code.launchpad.net/~canonical-server-reporter/+activereviews)
   to ensure nothing is blocked or get stalled.
1. Look at Documentation [issues](https://github.com/canonical/ubuntu-server-documentation/issues) and [pull requests](https://github.com/canonical/ubuntu-server-documentation/pulls)
   to ensure the actionable cases will be resolved

We try to find a balance, ensuring that nothing falls behind too much, while at
the same time not spending all our time in these cases. If the backlogs grow
out of control despite these regular efforts, then roadmap time needs to be
allocated to ensure we make good use of our community contibutions.
