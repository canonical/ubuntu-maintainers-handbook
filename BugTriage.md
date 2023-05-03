# Bug Triage

New bugs get reported every day, and existing tickets in Launchpad get
updated with comments, patches, and status changes.  The triager's job
is to review the traffic, deal with invalid or trivial issues, ignore
stuff that's just noise, and flag items of importance, urgency, and/or
readiness.

Triaging work is shared across the team, with the goal of successfully
reviewing every bug filed against Ubuntu Server related packages.  Each
triaging session will focus on a subset of tickets that changed during a
specific time period, such as the previous day.  The triager isn't
responsible for solving the reported problem, just to ensure it is
getting appropriate attention it needs.

All newly reported issues will need a triager's review.  A review involves
analyzing a bug to determine if the bug is valid and if sufficient
information was provided, and then marking it 'Triaged'.  Otherwise,
it's set to a more appropriate state with a comment explaining why, and
what next actions are, if any.

Older issues generally require no triager action if they're progressing
through their normal workflow. But watch for comments providing new
information that may make the issue more actionable.


## Types of Issue Tickets

Items in the triaging queue tend to fall into a few categories, that are
handled in different ways:

* Process Tickets.  
  Bug reports are often used to track status of
  packaging workflow tasks:

  - Sync requests
  - Merge requests
  - Stable release updates (SRU)
  - Main Inclusion Requests (MIR)
  - Freeze exception request (FFe, UI-FFe, et al.)
  - Package promotion/demotion ("Seed management/changes")

  These will generally either be filed by or assigned to a team member;
  if not, investigate further.  Generally, for properly owned tickets
  'No action is required' by the triager, unless something unusually
  weird is going on in which case 'Raise with the Team' (that *raising* can
  be "bringing it up in post-standup", "bringing it up in the weekly bug
  meeting" or "bringing it up in a chat channel pinging related developers"
  TL;DR do not stay alone with the weirdness - group experience and group
  decisions win).

* SRU Regression Bugs.  
  Problems that look like the result of an SRU
  update to a server package need to receive top priority.  For such
  bugs, 'Add to Server-Todo Queue' and 'Raise with the Team'.  If
  possible, verify rolling back to the prior version of the package
  fixes the issue, and re-installing the update brings the issue back.
  In this case, also tag the bug 'regression-update'.  Be aware that
  bugs are sometimes described as regressions simply because they're new
  to the user, not due to being actually introduced from an SRU update.
  For example, the user could have had a bad config file already, but
  the SRU triggers a restart of the service an that's when the user
  notices the problem and files a bug, thinking it was the update that
  introduced it.

* Severe Bugs.  
  Urgently important issues such as ones with potential
  for widespread breakage, should immediately 'Add to Server-Todo
  Queue'.

* High Profile Bugs.  
  Issues which involve an important customer or VIP,
  or that turn up frequently in search results, can generate a lot of
  "repeat visits" so to speak.  Just 'Raise issue with the team', unless
  its already well known in which case 'No action required' by the
  triager.

* Support Requests.  
  Sometimes users report problems that are really
  just a misunderstanding of how to use their system.  Kindly redirect
  them to more appropriate venues for help, and/or lend any obvious
  advice, but otherwise close the ticket as "Invalid" due to being 'Not
  a Bug'.

* Unclear.  
  If the ticket is missing log files, version details, or
  other information necessary to decide how to handle the bug, ask the
  user for what's needed, set the ticket to "Incomplete" due to 'Not
  Enough Info.'

* Duplicate.  
  If the issue is clearly the same as another report
  - mark one as duplicate of the better reported (or older) issue.
  - If there are existing tickets that sound very similar, make sure
    they are mentioned in a comment.  Ask the reporter(s) to review
    those and identify if it is indeed a dupe, or if not to then
    elaborate on how this new one differs.

* Non-actionable Comment.  
  If an existing ticket pops up in the queue due
  to a comment, check if it is adding information.
  - If it's just noise (e.g. "me too!"), 'No action is required' by the
    triager.
  - If it seems to describe a legitimate problem, but one completely
    unrelated to the originally reported issue, recommend filing a new
    bug report.
  - Otherwise suggest logical next steps, to put current interest to
    work at helping the bug make forward progress.

* Unreproducible Bug.  
  If the issue can't be reproduced (e.g. in an lxc
  container) then request the reporter provide the 'Missing Steps to
  Reproduce' it.

* Reproducible Bug.  
  If there seems to be enough information to
  reproduce the bug, try to do so in an lxc container, and then itemize
  the steps to follow, and how to identify that the bug has indeed
  occurred.  If it all looks good, subscribe the server team, or if the
  issue looks urgent and/or important 'Add to Server-Todo Queue'.

* Already Fixed in Development.  
  An issue that can be reproduced in a
  supported Ubuntu release, but not in the current development version
  of Ubuntu, may qualify for SRU processing.
  - Make sure the reproduction steps are clearly outlined
  - If the issue is minor/trivial, it probably won't be worth SRUing,
    so should be closed as already fixed in development.
  - If the issue is a request for a feature (not a bug fix), it may not
    qualify as eligible for SRU.  If that's the case, either close it as
    fixed in development, or add bug tasks for the requested releases
    set as 'Wishlist', and close the main bug task as fixed.
  - If the issue is a bug fix and looks important, then determine which
    supported Ubuntu releases will need the fix and add Bug Tasks as
    appropriate.  Even though the issue is fixed in the development,
    leave the main task *open* in this case because otherwise Launchpad
    may not display it in reports and lists.

* Already Fixed in Debian.  
  If the issue has been solved in Debian, it
  will likely be worth merging and/or sru'ing the fix.
  - Make sure steps to reproduce it are identified.
  - If the issue affects the development release, it is a merge
    opportunity.  If past feature-freeze, decide if is it worth a freeze
    exception.  Make sure there is a merge bug in launchpad for
    the package, and consider to Add to Server-Todo Queue.
  - If the issue affects a stable release and looks SRU-worthy,
    determine which supported Ubuntu releases will need the fix and add
    Bug Tasks as appropriate.

* Already Fixed Upstream.  
  All the steps of the "already fixed in Debian" category above apply,
  here as well -  in addition, help Debian when appropriate by:
  - filing a Debian bug about it or chime in if there is an existing one
  - If you create a PR for Ubuntu that can be used almost as-is consider
    sending one via salsa as well
  - Aligning our solution with Debian not only is kind, but additionally
    helps to avoid long term complex divergence and delta

* Unclear.  
  If in doubt or none of the above applies consider bringing it up via
  chat or if looking for a group discussion and decision tag it
  `server-triage-discuss`. Those bugs we will try to resolve together
  in our weekly meeting.

### Special cases

A few of our packages have common issue patterns or best practise triaging actions. This section shall
list them so that anyone on triage duty can find all of it in one place.

* Mysql  
  MySQL often has low quality bug reports by users not fully aware which dependency brought it onto their system.
  Those often fall into a few common usage errors.  
  Furthermore there are a few long standing issues that affect many users but often are reported as new.
    * Due to that we've found that quite often as a first step in triaging mysql one might want to check for duplicates.
        * In the days of mysql-5.7 (Xenial/Bionic) we tagged the common core bugs that one would dup new bugs to, those are
          available as (mysql-5.7 triage tag](https://bugs.launchpad.net/ubuntu/+source/mysql-5.7/+bugs?&field.tag=triage)
        * Since mysql-8.0 we no more use that tag, instead it turned out to be more reliable to just look at recent
          [mysql-8.0 bugs by heat](https://bugs.launchpad.net/ubuntu/+source/mysql-8.0/+bugs?orderby=-heat&start=0) to spot the
          duplication candidates
    * If not a duplicate, then still please update the bug title from the usual apport "failed on postinst" to whatever the bug
      really is about for better recognizing the issue in any kind of overview that just lists the title.

* Virtualization controlled through libvirt  
  If these reports are about the inability to access devices or *permission denied* issues the user often does not realize that
  libvirt applies an apparmor profile to the guest for enhanced security. If not available in the bug report (dmesg of the time
  of occurance) please make sure to ask to check for apparmor denials at the time the problem triggers.

## Process and Policy

### Direct team subscriptions

We subscribe *~ubuntu-server* directly to a bug to track our community bug backlog when the bug meets the following criteria.
When the bug no longer meets these criteria, we unsubscribe it:

1. Anything that, if the bug turns out to be valid, is something that would be under the *~ubuntu-server* remit to fix (common use
   cases but not obscure ones - but nothing stops an individual volunteering to work on an obscure use case, of course).
2. By definition, if it's something that we wouldn't fix and request volunteers even if we had time, then it doesn't warrant a
   subscription.
3. This subscription is for the Ubuntu Server triage community and is not for tracking of internal Canonical customer requests.   
   Whether a Canonical customer has made a request in relation to a particular bug makes no difference and provides no
   additional priority under this process. A Canonical customer bug may still be subscribed if it qualifies under these
   criteria.
4. If the bug is assigned to someone on our team, leave it subscribed. No need to subscribe, and feel free to unsubscribe the team.

### tagging `server-todo`

This is our tag we use to represent valid and work we should work on (better than just average valid backlog).

We want to assign bugs from this queue regularly. To avoid losing traction
there is a weekly bug housekeeping meeting (see below) to ensure non bug gets
blocked or forgotten for too long.

The goal is to have this list around ~30-40 bugs most of the time, if dropping
below we can refill with candidates from the *~ubuntu-server* subscribed bugs.
But if it grows significantly out of this range it is non-realistic to expect
those issues to be handled in time, we should communicate so to the reporters.

Definition to qualify for server-todo:

* Whatever we think that we want to work on soon. For example:
    * An important new technology for Ubuntu-Server users
    * Great community engagement that provide debugging and/or patches
    * OTOH we don’t want to put in bugs where the next step is significantly larger than one day to complete, unless the bug
      is particularly important. Examples:
        * A feature that is Ubuntu only and important for our users -> ok to be in the list despite likely needing more time
        * A valid crash report, but being a corner case and having just one affected user; All low hanging fruits and obvious
          checks are done, therefore the next debug step is estimated to take at least a week -> this might be ok for the
          backlog, but not really for `server-todo`
    * Make sure it is clear if the bug needs work in development or needs SRUs, by defining bug tasks accordingly.
      (These bug tasks can help in identifying current vs. obsolete bugs.)
* Only bugs that qualify for the backlog qualify here. If they aren’t suitable for the backlog (eg. not actionable by us)
  then they get dropped from both `server-todo` and the backlog.
  * If there are any updates to the case they will come in via triage again and can be reevaluated

### Daily Bug Expiration

The tooling will help to report these to the Triager.

* **Server subscription expiration** - default after **180 days**  
  If nobody touched a bug for 180 days (~= 1 release cycle) it is reasonable to check for changed conditions. Quite often e.g.
  a patch one was waiting on is available now. In other cases a newer release fixed it already. Essentially anything that is
  listed here needs to be fully re-triaged to ensure the list is reflecting the current status. It also can after this time be
  used as a metric how many more people chimed in got dupped on the bug (importance/#affected). Most common cases are, that it
  turns out:
    * that recent releases upstream or even already in Ubuntu have the fix -> re-triage, consider tagging `server-todo` for SRU
    * that the bug should have been supported by the community but nothing happened -> re-triage importance, consider dropping
      *~ubuntu-server* subscription
    * that a bug that was formerly considered a real case is not qualifying anymore (e.g. alternative solutions
      have taken hold as *the* way to do it) -> re-triage importance, consider dropping *~ubuntu-server* subscription
    * If unsure, add the server-triage-discuss tag and bring it up at the next standup

Overall we want to be honest to the bug reporter, try to understand why an
issue was not worked on and explain it if possible. Also if we drop `server-todo`
or the *~ubuntu-server* subscription for any of the reasons above always add a
explanatory comment. If reporters disagree with our re-triage they will report
on the bug and it will show up in the daily triage duty the next day to be
reconsidered with that point of view taken into consideration.

### Weekly Bug Housekeeping

In addition to the daily triage and our ongoing dedication to resolve bugs we
picked up that way, the Server Team has introduced a weekly bug housekeeping
session. In there we go through various lists ensuring that none are
forgotten, blocked or stalled for too long.

There are a few additional steps that we do always changing over time based on
current priorities - for example looking for good candidates to do MREs in the
future or trying to assign our remaining cleanup on discourese documentation
feedback backlog. But while those change there is a core structure of what we
always look at in this meeting:


1. Check the `server-todo` tagged bug list
   1. Get list via ustriage
      1. `clear; ustriage --no-show-triage --extended --show-tagged --tag server-todo -S savebugs/todo-$(date -I'seconds').yaml -C $(ls -1t savebugs/* | head -n 1)`
      2. the list of last weeks bugs helps to identify new/closed cases and is
         [tracked in the helpers repository](https://git.launchpad.net/~ubuntu-server/+git/ubuntu-helpers/tree/savebugs)
   3. Check size (see min/max above) of the `server-todo` tagged bug list
   4. Ensure assigned bugs make reasonable progress
      1. discuss blockers/reasons if there was no progress
      2. Notice, enjoy and celebrate progress that was made
   5. Ensure unassigned bugs find an owner
      1. Ensure long term unassigned bugs are re-reevaluated (is there a reason they are not tackled?)
2. Look at [update-excuses by team](https://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses_by_team.html#ubuntu-server)
   to spot anything that needs our attention to migrate. If there are any assign
   team members to analyze the case and ensure things are progressing.
3. Look at our [merges schedule](http://pinot.endarchy.org:4200/merges-schedule)
   to identify if we have fallen behing on any of them


Future outlook:
> as capacity permits we want in the future to start also having
> a look at the [sponsoring queue](http://reqorts.qa.ubuntu.com/reports/sponsoring/index.html)


## Awareness of the Triage

We have several stakeholders we want to keep up-to-date on things that
we've found on triage. On one hand we want to keep the community generally
informed as well as raising issues within the team to ensure they are not
falling through the cracks.

For the community we send a mail to ubuntu-server@lists.ubuntu.com that
summarizes how many bugs we've triaged and touches on the noteworthy
cases. This can also be used to CC additional people that (for case
specific reasons) should be aware of a case.
An example of that would be if a security fix caused an upgrade-regression
which would make us CC the uploader and/or ubuntu-security. This mail should also contain the relevant information from [documentation triage](DocTriage.md).

Furthermore on cases that need immediate attention or at least awareness
we might:

 * Bring them up in the daily standup (mostly if they need a discussion/decision that one can't do alone)
 * Ping a subject matter expert via IRC/Mattermost

In some cases a package maintainer might already be aware and follow a case.
To avoid endless re-pings on such a case the agreement is that if the maintainer
is personally subscribed (i.e. with his launchpad user, not just indirectly via
teams like [Ubuntu Virtualisation](https://launchpad.net/~ubuntu-virt))
then we consider the maintainer to be aware and will not do extra
pings/mentions/CC.


## Triage Rotation

According to load we might shift things, but generally every day Tue-Fri
has a team member assigned. Monday is often more work and includes more
low quality bugs as it includes all of the weekend - therefore Monday is
a rotation through all eligible bug triagers.
This is organized internally in the teams Jira and automation will create
a Task with the "bug-triage" label assigned to the person on rotation.

## Tooling

The [ustriage](https://snapcraft.io/ustriage) tool is available as a snap
and serves as the tool a triager would use. It is maintained publicly on github
as [ubuntu server triage](https://github.com/canonical/ubuntu-server-triage).
It has options to identify bugs for the triage of the day as well as serving
as a helper to check our tagged bugs ensuring that nothing falls through the
cracks. The Readme.md of the linked project has some more details and use
case example.

## Outdated/Deprecated

These info is kept around for a while in case one encounters older bugs
which still use those tags or processes. Sometime later these sections will
be removed.

### tagging `server-next` (deprecated)

Since early 2022 we have had enough control over the backlog bugs that the
former set of `server-next` + `server-todo` bugs was reduced to just
`server-todo`. The reason for that was that we found the capacity to work even
on bugs that didn't fulfil the rather strict rules we used to have for
`server-next`.

> Since the backlog is bigger than what can be achieved in a short time, there is the extra classification via the tag
> `server-next`. That tag is set by the Triager (or anyone else working on doing the Root-Cause-Analysis or a Fix) to
> reflect that this is an issue that shall be tackled by the Teams resources "next".
>
> Another reason to add `server-next` in some cases is to preserve high quality contributions of the community. An example might be a report that the user already bisected and created a patch for - in those cases the benefit diminishes by bit rot way too fast, so handling that next helps to retain the work the reporters did. And vice versa it might encourage one or the other to provide more high quality bugs.
>
> The goal is to have this list around ~20 bugs most of the time, if dropping below we can refill with candidates from the *~ubuntu-server* subscribed bugs. But if it grows significantly out of this range it is non-realistic to expect those issues to be handled in time, we should communicate so to the reporters.
>
> The rules of the `server-next` tag are as follows:
>
> 1. Must not tag unless bug is actionable. Doesn't mean it must have a patch, only that a developer has enough information to
>    work on the bug, even if it means more debugging.
> 2. Tag only if one of these two things are true:
>     1. Delays will discourage this excellent community contribution.
>     2. If you believe it affects a major use case for Ubuntu server users. In this case you should also set the bug Importance.
> 3. The set of all bugs tagged `server-next` must be kept small. If it grows, the lowest priority bugs tagged `server-next` must
>    be removed until the list isn’t too big.
> 4. This tag is for the Ubuntu Server triage community and is not for tracking of internal Canonical customer requests. Whether a
>    Canonical customer has made a request in relation to a particular bug makes no difference and provides no additional priority
>    under this process. A Canonical customer bug may still be tagged if it qualifies under these criteria.
> 5. If the bug is assigned to or otherwise owned by someone on our team, there is no need to tag it.
> 6. Remove the tag when the bug is assigned to or otherwise owned by someone on our team.

> * **Server-next expiration** - default after **60 days**  
>   If we considered a bug actionable and added it to server-next, but then no update happened in 60 days that usually means
>   something went wrong. Often bugs are blocked on external constraints. This needs to be evaluated as a case-by-case decision.
>   Most common cases are, that it turns out:
>     * that the bug is not solvable/reasonable the way it was planned -> re-triage, maybe drop server-next.
>     * that it is actually fixed or otherwise progressed without update -> update bug
>     * that we failed to give it the required focus -> add the server-triage-discuss tag to the bug and bring it in the next 
>       standup

