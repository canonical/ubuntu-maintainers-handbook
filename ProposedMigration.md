Proposed Migration
==================

Uploads of fixed or merged packages don't automatically get released to Ubuntu users, but rather go into a special pocket called 'proposed' for testing and integration.  Once a package is deemed ok, it 'migrates' into the release pocket for users to consume.  This is called the ["Proposed Migration" process](https://wiki.ubuntu.com/ProposedMigration).

Here is the typical lifecycle for an upload:

  0.  Issue identified, fixed, and packaged
  1.  *source.changes uploaded
  2.  If the fix is an SRU, or if we're in a [freeze](https://wiki.ubuntu.com/FreezeExceptionProcess):
      *  Upload goes into an 'unapproved' queue
      *  SRU team reviews and approves... (goto 3)
      *  ...or disapproves (goto 0)
  3.  Upload goes into [codename]-proposed queue
  4.  Launchpad builds binary package(s) from source package
  5.  Run [autopkgtests](https://packaging.ubuntu.com/html/auto-pkg-test.html)
      * Autopkgtests for the package itself
      * Autopkgtests for the reverse{-build}-dependencies, as well
  6.  Verify other archive consistency checks
      *  Are binary packages installable?
      *  Are all required dependencies at the right version?
      *  Does it cause anything to become uninstallable?
      *  etc.
  7.  If (and only if) the fix is an SRU:
      *  [SRU bug](https://wiki.ubuntu.com/StableReleaseUpdates#Verification) updated with request to verify
      *  Reporter or developer verifies fix, and updates tags
  8.  Release package from [codename]-proposed to [codename]

Often the migration proceeds automatically, but when there are issues our involvement is needed to sort things out.  Sometimes the package having the problem is one we've uploaded ourselves, and responsibility generally lays with the uploader to analyze and solve the issue.

However, there are numerous other situations that lead to migration trouble, for which there is not a specific responsible party.  These types of issues are focused on as part of the "Plus-One Maintenance" effort, where individuals across the distro team are tasked to work on examining and solving them.  Some of these types of problems arise from items auto-sync'd from Debian, or via a side-effect of some other change in the distro.

Following are tips and tricks for solving migration issues, and some guidance for people just starting the proposed migration duty.


Update Excuses Page
-------------------

All current migration issues for the current devel release are displayed on the [Update Excuses Page](https://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html).  [Similar pages](https://people.canonical.com/~ubuntu-archive/proposed-migration/) exist for stable releases (these items generally relate to particular SRUs).  These pages are created by a software service named "Britney", which updates the page after each batch of test runs, typically every 2-3 hours depending on load.

The page is ordered newest to oldest, and so the items at the top of the page may still be processing (indicated by "Test in progress" marked for one or more architectures).  In general, two things to watch for are "missing build", which can indicate a failure to build the source package (FTBFS), and "autopkgtest Regression", which indicates a test failure.  Both of these situations are described in more depth below.

Many of the items on the page are not actually broken, they're just blocked by something else that needs resolution.  This often happens for new libraries or language runtimes, or really any package that a lot of other things depend on to build, install, or run tests against.

If you are just coming up to speed on proposed migration duty, you'll likely be wondering "Where do I even start?!"  A good suggestion is to look for build failures towards the upper-middle of the page affecting only one architecture; single-arch build failures tend to be more localized and more deterministic as to cause, and items towards the top of the page but after all the "Test in progress" items will have had fewer eyeballs on them since they're newer, and so have a higher chance of being something simple, yet not so new that it's still being actively worked on by someone.  If you suspect there may already be eyeballs on the issue, the #ubuntu-devel IRC channel can be a good place to ask if anyone's already looking at it.

But don't worry too much about where to start; the Update Excuses page is much like a wool sweater in that you start pulling at one thread and end up unraveling the whole affair.


Bug Reports for Migration Problems
----------------------------------

If you file a bug report about a build or test failure shown on the update excuses page, tag the bug report `update-excuse`.  Britney will include a link to the bug report next time it refreshes the update_excuses.html page.  This helps other developers see the investigation work you've already done, and can be used to identify the next-action.  Mark yourself as the Assignee if you're actively working on the issue, and leave the bug unsubscribed if you aren't so that others can carry things forward.



Failure to Build From Source (FTBFS)
------------------------------------

The build step can hit failures for several general classes of problems.  First are regular build problems that can be easily reproduced locally (and that should have been addressed before uploading).  Second are platform-specific issues that may have been overlooked due to being present only on unfamiliar hardware.  Third are intermittent network or framework problems, where simply retrying the build once or twice can result in a successful build.  Fourth are dependency problems, where the task is to get some other package to be the right version in the release pocket so that the build will succeed.  Fifth are ABI changes with dependencies; these often just need a no-change rebuild.  And last are all the other myriad package-specific build problems, that typically require some particular tweak to the package itself.

The first case covers all the usual normal build issues like syntax errors and so on.  In practice these are typically quite rare because developers tend to be very diligent at making sure their packages build before uploading.  Fixes for these is the usual standard development work.

The second case is similar to the first but pertains to issues that arise only on specific platforms.  Common situations are tests or code that relies on [endianness](https://en.wikipedia.org/wiki/Endianness) of data types and so breaks on big-endian systems (e.g. s390x), code expecting 64-bit data types that breaks on 32-bit (e.g. armhf), and so on.  [Canonistack provides hardware](https://wiki.canonical.com/InformationInfrastructure/IS/CanoniStack-BOS01) that Canonical employees can use to attempt reproduction of these problems.  Take note with i386 particularly in that it often fails less due to data type issues but more because it is a partial port and has numerous limitations that require special handling, as described on the [i386 troubleshooting page](https://wiki.ubuntu.com/i386).

In the third case, local building was fine but the uploaded build may have failed on Launchpad due to a transitory issue such as a network outage, or a race condition, or other stressed resource.  Or it might have failed due to a strange dependence on some variable like the day of the week.  In these cases, clicking on the "Retry Build" button in Launchpad once or twice can sometimes cause the build to randomly succeed, thus allowing the transition to proceed (if you're not yet a core-dev, ask for a core-dev on the #ubuntu-devel IRC channel if they can click the link for you).  Needless to say, we don't wish for such unpredictabilities in the build process, so if the problem reoccurs and if you can narrow down the cause, it can be worthwhile to find the root cause and figure out how to fix it properly.

For the fourth case, where a dependency is not the right version or is not present in the right pocket, the question becomes one of identifying what's wrong with the dependency and fixing it.  Be aware there may be some situations where the problem really is with the dependency itself, and the solution is to change the version of the dependency, or adjust the dependency version requirement, or removing invalid binary packages, or so forth.  These latter solutions often require asking for an archive admin's help on the #ubuntu-release IRC channel.

The fifth case of ABI changes comes up particularly when Ubuntu is introducing new versions of toolchains, language runtime environments or core libraries; i.e. new glibc, gcc, glib2.0, ruby, python3, phpunit, et al.  This happens when the release of the underlying libraries/toolchain is newer than the consuming project.  Your package may fail to build because one of its dependencies was built against a different version of glibc, or with less strict gcc options [Ubuntu Defaults can be checked here](https://wiki.ubuntu.com/ToolChain/CompilerFlags#Notes), or whatever, and needs (no-change) rebuilt (and/or patched) to build with the new version or stricter options.  Assuming the upstream project has already made the changes to adapt to the changed behavior or function-prototypes and produced a release, then if we already have that release in Ubuntu a simple no-change rebuild may suffice; if we don't have the release, then it will take a sync or merge, or patching in the changes to what we're carrying.  If upstream has not released the changes, then you could also consider packaging a snapshot of their git repo.  If upstream has not yet made the changes, and there aren't already existing bug reports or pull requests, it may be necessary to make the changes on our end.  Communication with Debian and upstream can be effective here, and where it isn't filing bug reports can be worth the effort.

With this fifth case, it's worth noting that quite often these updates cause the same kind of errors in many places.  It's worth asking in the #ubuntu-devel IRC channel in case standard solutions are already known that you can re-use, such as ignoring a new warning via a `-W...` option or switching to the old behavior via a `-f...` option.  It's also worth reporting your findings in ways others can easily re-use when they run into more cases.

Finally, there are also a miscellania of other problems that can result in build failures.  Many of these will be package-specific or situation-specific.  As you run into situations that crop up more than a couple times please update these docs.


Autopkgtest Regressions
-----------------------

After a package has successfully built, the [autopkgtest infrastructure](https://autopkgtest.ubuntu.com/) will run its [DEP8 tests](https://packaging.ubuntu.com/html/auto-pkg-test.html) for each of its supported architectures.  Failed tests can block the migration, and "Regression" listed for the failed architecture(s), worded like:

### "Migration status for aaa (x to y): <reason>"

This means package "aaa" has a new version y uploaded to -proposed, to replace the existing version x, but the change has not yet been permitted.  The various reasons typically seen are outlined in the items below.

### "Waiting for test results, another package or too young (no action required now - check later)"

This means one or more tests still need to be run.  These will be noted as 'Test in progress' under the 'Issues preventing migration' line item.

### "Will attempt migration (Any information below is purely informational)"

This is good, it indicates the item is currently in process and likely will migrate soon (within a day or so).

### "Waiting for another item to be ready to migrate (no action required now - check later)"

This can be good if it's recent, but if it's more than a few days old then there may be a deeper issue going on with a dependency.  In the former case, just give it some time (maybe a day or so).  In the latter case, refer to the items listed under the 'Issues preventing migration:' line to see what specifically has gone wrong, and then see the "Issues Preventing Migration" section (below) for diagnostic tips.

### "BLOCKED: Cannot migrate due to another item, which is blocked (please check which dependencies are stuck)"

The package itself is likely fine, but it's being blocked by some issue with some other package.  Check beneath the 'Invalidated by dependency' line to see what package(s) need attention.


### "BLOCKED: Maybe temporary, maybe blocked but Britney is missing information (check below)"

One situation where this can occur is if one of the package's dependencies is failing to build, which will be indicated by a line stating "missing build on <arch>".  Once the dependency's build failure is resolved, this package should migrate successfully; if it doesn't within a day or so, a retrigger may be in order.

### "BLOCKED: Rejected/violates migration policy/introduces a regression"

This indicates an "Autopkgtest Regression" with the given package, and is by far the most common situation where your attention will be required.  A typical like will look something like:

* autopkgtest for [package]/[version]: amd64: <span style="background-color:darkred">Regression</span>, arm64: <span style="background-color:yellow">Not a regression</span>, armhf: <span style="background-color:green">Pass</span>, ppc64el: <span style="background-color:#99DDFF">Test in progress</span>, ...

The 'Regression' items indicate problems needing solved.  See the 'Issues Preventing Migration' section (below), for diagnositic tips.

Sometimes you'll see lines where all the tests have passed (or, at least, none are marked Regression).  Usually passing packages are omitted from the display.  When they're not omitted, it generally means that the test ran against an outdated version of the dependency.  For example, you may see:

* python3-defaults (3.10.1-0ubuntu1 to 3.10.1-0ubuntu2)
    - Migration status for python3-defaults (3.10.1-0ubuntu1 to 3.10.1-0ubuntu2): BLOCKED: Rejected/violates migration policy/introduces a regression
    - Issues preventing migration:
    - ...
    - autopkgtest for netplan.io/0.104-0ubuntu1: amd64: Pass, arm64: Pass, armhf: Pass, ppc64el: Pass, s390x: Pass.
    - ...

In this case, check rmadison:

    $ rmad netplan.io
    netplan.io | 0.103-0ubuntu7   | impish
    netplan.io | 0.104-0ubuntu2   | jammy

    netplan.io | 0.103-3       | unstable

We can see our test ran against version 0.104-0ubuntu1, but a newer version 0.104-0ubuntu2 is in the archive.  If we look at the autopkgtest summary page for one of the architectures, we see:

    ## https://autopkgtest.ubuntu.com/packages/n/netplan.io/jammy/amd64
    0.104-0ubuntu2    netplan.io/0.104-0ubuntu2   2022-03-10 11:38:36 UTC     1h 01m 59s  -   pass    log   artifacts
    ...
    0.104-0ubuntu1    python3-defaults/3.10.1-0ubuntu2    2022-03-08 04:26:47 UTC     0h 44m 47s  -   pass    log   artifacts  
    ...

Notice that version 0.104-0ubuntu1 was triggered against the python3-defaults version currently in -proposed, but version 0.104-0ubuntu2 was only ran against netplan.io itself, and thus ran against the old python3-defaults.  We need to have a test run against both these new versions (and against any other of netplan.io's dependencies also in -proposed.)

The ['excuses-kicker' tool](https://code.launchpad.net/~bryce/+git/excuses-kicker) is a convenient way to generate the retrigger URLs:

    $ excuses-kicker netplan.io
    https://autopkgtest.ubuntu.com/request.cgi?release=jammy&arch=amd64&package=netplan.io&trigger=netplan.io%2F0.104-0ubuntu2&trigger=symfony%2F5.4.4%2Bdfsg-1ubuntu7&trigger=pandas%2F1.3.5%2Bdfsg-3&trigger=babl%2F1%3A0.1.90-1&trigger=python3-defaults%2F3.10.1-0ubuntu2&trigger=php-nesbot-carbon%2F2.55.2-1
    ...

Notice how it's picked up not only python3-defaults but also several additional packages that netplan.io has been run against in the recent past.  These aren't necessarily direct dependencies for netplan.io, but serve as an informed guess.


Issues Preventing Migration
---------------------------

Packages can fail to migrate for a vast number of different reasons.  Sometimes the clues listed under the 'Issues preventing migration:" line item on the update_excuses page will be sufficient to indicate the next action necessary; that will be the focus of this section.  In other cases, the autopkgtest log file will need analyzed; that is covered in a latter section.


### Main/Universe Binary Mismatch

A given source package may have some binaries in main and others in universe, but this can get mixed up for various reasons.  This genre of issues, known as ["component mismatches"](https://people.canonical.com/~ubuntu-archive/component-mismatches.txt) is spotted from migration excuses like:

  - "php8.1-dba/amd64 in main cannot depend on libqdbm14 in universe"

Check the prior release to see if php8.1-dba and libqdbm14 were both in main, or both in universe.  If both were in universe before, then most likely the package in main needs demoted to universe.  Ask an archive admin to do this, and/or file a bug report (for example, see [LP: #1962491](https://bugs.launchpad.net/ubuntu/+source/php8.1/+bug/1962491).)

Conversely, if the dependency in universe actually needs to be in main, such as for example a newly introduced binary package, then a 'Main Inclusion Request' (MIR) bug report will need to be filed.  See https://wiki.ubuntu.com/MainInclusionProcess for details on this procedure.

If both packages were in main previously, then some additional investigation should be done to see why the universe package moved out of main.  The resolution may be to move one or the other package so they're both in the same pocket, or find a way to remove the dependency between them.

One very special case of these is unintended dependencies due to extra-includes.  Please be aware that while most Dependencies seem obvious (Seeds -> packages -> packages) there is an aspect of germinate which will [automatically include](https://git.launchpad.net/~ubuntu-core-dev/ubuntu-seeds/+git/ubuntu/tree/supported#n124) all -dbg, -dev, -doc* packages in a source archive that is in main.  In [Germinate](https://people.canonical.com/~ubuntu-archive/germinate-output/ubuntu.jammy/all) these will appear as `Rescued from <src>`. In case a merge is affected, the solution without adding delta usually is to add an `Extra-exclude` like in this [example with net-snmp](https://code.launchpad.net/~sergiodj/ubuntu-seeds/+git/ubuntu/+merge/414063).

For more information on component mismatches, from an archive administrative perspective, see:

  - https://wiki.ubuntu.com/ArchiveAdministration#Component_Mismatches_and_Changing_Overrides


### Impossible Depends

An excuse like this:

   - "Impossible Depends: aaa -> bbb/x/<arch>"

means package 'aaa' has a dependency on package 'bbb', version x, for architecture 'arch', but it is not possible to satisfy this.

One reason this can occur is if package 'bbb' is in universe whereas 'aaa' is in main.  It may be that some of package aaa's binaries need to be moved to universe.  See 'Main/Universe Binary Mixups' (above) for directions.


### Migrating Makes Something Uninstallable

A migration excuse in this format:

   - "migrating [aaa]/[aaa-new-version]/[arch] to testing makes [bbb]/[bbb-version]/[arch] uninstallable"

means that package bbb has a versioned build dependency against the old version of package aaa.  A no-change rebuild can be used in these cases to force a rebuild.

### Depends: aaa [bbb] (not considered)

Package "aaa" is blocked because it depends on "bbb", however "bbb" is either invalid or rejected.

If the dependency itself is not valid, this line will be followed by an 'Invalidated by dependency' line.  In this case, look at the situation with package "bbb" and resolve it first, in order to move package "aaa" forward.

If there is no 'Invalidated by dependency' line, then the dependency may be rejected.  There are three reasons why a rejection can occur:  a) it needs approval, b) cannot determine if permanent, or c) permanent rejection.


### Implicit dependency: aaa [bbb]

An implicit dependency is a pseudo dependency where Breaks/Conflicts creates an inverted dependency.  For example, pkg-b Depends on pkg-a, but pkg-a=2.0-1 breaks pkg-b=1.0-1, so pkg-b=2.0-1 must migrate first (or they must migrate together).  A way to handle this is to re-run pkg-b's autopkgtest (for 2.0-1) and include a trigger for pkg-a=2.0.  For example, run:

    $ excuses-kicker -t pkg-a pkg-b

This can also occur if pkg-b has "Depends: pkg-a (<< 2.0)", due to use of some non-stable internal interface.

It can also occur if pkg-a has a Provides that changes from 1.0-1 to 2.0-1, but pkg-b has a Depends on the earlier version.


### Implicit dependency: aaa [bbb] (not considered)

Similar to above, "aaa" and "bbb" are intertwined, but "bbb" is also either invalid or rejected.  For these cases, attention should first go to resolving the issue(s) for "bbb", and then re-running the autopkgtest for it with a trigger included against package "aaa".


### Has no binaries on arch <arch>

Usually this means the package has either failed to build or failed to upload the binaries for a build.  Refer to the 'Failed to Build From Source' section for tips on how to handle this class of problem.


### Has no binaries on any arch (- to x.y.z)

If the package doesn't have a current version, this error can indicate the package is not (yet) in the archive, or it can mean its binaries were removed previously but not sync-blacklisted and thus reappeared.

If the package should not be sync'd into the archive, on #ubuntu-release ping "ubuntu-archive" with request to remove the packages' binaries and add them to sync-blacklist.txt

Otherwise, there are several things worth checking (note: These links are for jammy; modify them for the current development release):

  - [Stuck in New queue](https://launchpad.net/ubuntu/jammy/+queue?queue_state=0)

  - [Stuck in Unapproved queue](https://launchpad.net/ubuntu/jammy/+queue?queue_state=1)


Autopkgtest Log Files
---------------------
You can view the recent test run history for a package's architecture by clicking on the respective architecture's name.

Tests can fail for myriad reasons.

Flaky tests, hardware instabilities, and intermittent network issues can cause false positive failures.  A simple retriggering of the test run (via the '♻ ' symbol) is typically all that's needed to resolve these.  Before doing this, it is worthwhile to check the recent test run history to see if someone else has already tried.

When examining a failed autopkgtest's log, start from the end of the file, which typically will either show a summary of the test runs, or an error message if a fault was hit.  For example:

```
done.
done.
(Reading database ... 50576 files and directories currently installed.)
Removing autopkgtest-satdep (0) ...
autopkgtest [21:40:55]: test command1: true
autopkgtest [21:40:55]: test command1: [-----------------------
autopkgtest [21:40:58]: test command1: -----------------------]
command1             PASS
autopkgtest [21:41:03]: test command1:  - - - - - - - - - - results - - - - - - - - - -
autopkgtest [21:41:09]: @@@@@@@@@@@@@@@@@@@@ summary
master-cron-systemd  FAIL non-zero exit status 1
master-cgi-systemd   PASS
node-systemd         PASS
command1             PASS
```

Here we see that the test named 'master-cron-systemd' has failed.  To see why it failed, do a search on the page for 'master-cron-systemd', and iterate until you get to the last line of the testrun, then scroll up to find the failed test cases:

```
autopkgtest [21:23:39]: test master-cron-systemd: preparing testbed
...
...
autopkgtest [21:25:10]: test master-cron-systemd: [-----------------------
...
...
not ok 3 - munin-html: no files in /var/cache/munin/www/ before first run
#
#	  find /var/cache/munin/www/ -mindepth 1 >unwanted_existing_files
#	  test_must_be_empty unwanted_existing_files
#
...
...
autopkgtest [21:25:41]: test master-cron-systemd: -----------------------]
master-cron-systemd  FAIL non-zero exit status 1
autopkgtest [21:25:46]: test master-cron-systemd:  - - - - - - - - - - results - - - - - - - - - -
autopkgtest [21:25:46]: test master-cron-systemd:  - - - - - - - - - - stderr - - - - - - - - - -
rm: cannot remove '/var/cache/munin/www/localdomain/localhost.localdomain': Directory not empty
```

All autopkgtests follow this general format, although the output from the tests themselves varies widely.

Beyond "regular" test case failures like this one, autopkgtest failures can also occur due to missing or incorrect dependencies, test framework timeouts, and other issues.  Each of these is discussed in more detail below.


Special Cases
-------------

## Circular Build Dependencies ##

When two or more packages have new versions that depend on each other's new versions in order to _build_, this can lead to a [circular build dependency](https://wiki.debian.org/CircularBuildDependencies).  There's a few different ways these come into being, which have unique characteristics that can help find a way to work through them.

### 1.  Build-Dependencies for Testsuites

If package A's build process invokes a testsuite as part of the build (i.e. in debian/rules under override_dh_auto_test), and the testsuite requires package B, then if package B requires package A to build, this creates a situation where neither package will successfully build.

The workaround in this case is to (temporarily) disable running the testsuite during build.  This is usually ok when the package's autopkgtests (defined in debian/tests/control) also run the testsuite.  For instance:

    override_dh_auto_test:
        echo "Disabled: phpunit --bootstrap vendor/autoload.php"

You may also need to delete test dependencies from debian/control, and/or move them to debian/tests/control.

With package A's testsuite thus disabled during build, it will build successfully but then fail its autopkgtest run.  Next, from package B's launchpad page request rebuilds for each binary in a failed state.  Once package B has successfully built on all supported architectures, package A's autopkgtests can be rerun using package B as a trigger.

Once everything's migrated successfully, a cleanup step would be to re-enable package A's testsuite and verify it now passes.

### 2.  Bootstrapping

In this situation, packages A-1.0 and B-1.0 are in the archive.  We're introducing new versions A-3.0 and B-3.0 (skipping 2.0); A-3.0 depends on B-3.0, and B-3.0 requires A-2.0 or newer.  Both A-3.0 and B-3.0 will fail to build.

One example of where this can occur is if code common to both A and B is refactored out to a new binary package that is provided in A starting at version 2.0.

The most straightforward solution to this situation is to ask an archive admin to delete both A-3.0 and B-3.0 from the archive, then upload version A-2.0 and allow it to build (and ideally migrate).  Next reintroduce B-3.0, and then once that's built, do the same for A-3.0.

With even larger version jumps, e.g. from A-1.0 to A-5.0, it may be required to do multiple bootstraps, and some experimentation to see which intermediary version(s) need to be jumped to.  Another common complication can be where the cycle involves more than two packages.


## Test Dependency Irregularities ##

The package's [debian/tests/control file defines what gets installed](https://salsa.debian.org/ci-team/autopkgtest/blob/master/doc/README.package-tests.rst) in the test environment before executing the tests.  You can review and verify the packages and versions in the DEP8 test log, between the lines 'autopkgtest...: test integration: preparing testbed' and 'Removing autopkgtest-satdep'.

A common issue is that the test should be run against a version of a dependency present in the -proposed pocket, however it failed due to running against the version in -release.  Often this is straightforward to prove by running the autopkgtests locally in a container.  Another easy way to test this is to re-run the test but set it to preferentially pull packages from -proposed -- this is done by appending '&all-proposed=1' to the test URL.  If that passes, but the package still does not migrate, then look in the test log for all packages that were pulled from -proposed and include those as triggers.  [Excuses Kicker](https://git.launchpad.net/~bryce/+git/excuses-kicker) and [retry-autopkgtest-regressions](https://bazaar.launchpad.net/~ubuntu-archive/ubuntu-archive-tools/trunk/view/head:/retry-autopkgtest-regressions) are handy tools for generating these URLs.

As with rebuilds, these retriggers also require core-dev permissions, so if you're not yet core-dev give the links to someone who is for assistance.


## Test Framework Timeouts and Out of Memory ##

The autopkgtest framework will kill tests that take too long to run.  In some cases it makes sense to just configure autopkgtest to let the test run longer.  This is done by setting the ```long_tests``` option.  Similarly, some tests may need more CPU or memory than in a standard worker.  The ```big_packages``` option directs autopkgtest to run these on workers with more CPU and memory.  Both these options are explained on the [ProposedMigration](https://wiki.ubuntu.com/ProposedMigration#autopkgtests) page.

It is worth to mention that Debian test sizing is currently (as of 2021) equivalent to our big_packages.

The configuration that associates source packages to either `big_packages` / `long_tests` and the actual deployment code was recently split.
[The new docs](https://autopkgtest-cloud.readthedocs.io/en/latest/administration.html#give-a-package-more-time-or-more-resources) explain this and link [a repository](https://code.launchpad.net/~ubuntu-release/autopkgtest-cloud/+git/autopkgtest-package-configs) which is now mergeable by any release team member.

## Disabling/skipping tests ##

While ideally failing tests should receive fixes to enable them to pass properly, sometimes this is infeasible or impossible.  In such extreme situations it may be necessary to explicitly disable a test case or an entire testsuite.  For instance, if an unimportant package's test failure blocks a more important package from transitioning.

As a general rule, try to be as surgical as possible and avoid disabling more than is absolutely required.  For example, if a test has 10 sub-cases and only one sub-case fails, prefer to comment out or delete that sub-case rather than the entire test.

There is no single method to disabling tests, unfortunately, since different programming languages take differing design approaches for their test harnesses.  Some test harnesses have provisions for marking tests "SKIP".  In other situations it may be cleaner to add a debian patch that simply deletes the test's code from the codebase.  Sometimes it works best to insert an early return with a fake PASS in the particular code path that broke.

Take care when disabling a test to include a detailed enough explanation for why you're disabling it, that would inform a future packager as to when the test can be re-enabled.  For instance, if a proper fix will be available in a future release, say so and indicate what version will have it.  Or, if the test is being disabled just to get another package to transition, indicate what that package's name and expected version.  Bug reports, DEP3 comments, and changelog entries can be good places to document these clues.


## Disabling/skipping/customizing tests for certain architectures ##

If you need to disable the entire testsuite for a specific architecture, such as an arch that upstream doesn't include in their CI testing, and that sees frequent and ample failures on our end, then you can skip the testing via checks in the debian/rules file.  For example (from util-linux-2.34):

    override_dh_auto_test:
    ifneq (,$(filter alpha armel armhf arm64,$(DEB_HOST_ARCH)))
            @echo "WARNING: Making tests non-fatal because of arch $(DEB_HOST_ARCH)"
            dh_auto_test --max-parallel=1 || true
    else ifeq ($(DEB_HOST_ARCH_OS), linux)
            dh_auto_test --max-parallel=1
    endif

If the issue is the integer type size, here's a way to filter based on that (from mash-2.2.2+dfsg):

    override_dh_auto_test:
    ifeq (,$(filter nocheck,$(DEB_BUILD_OPTIONS)))
    ifeq ($(DEB_HOST_ARCH_BITS),32)
            echo "Do not test for 32 bit archs since test results were calculated for 64 bit."
    else
            dh_auto_test --no-parallel
    endif
    endif

Or based on the CPU type itself (from containerd-1.3.3-0ubuntu2.1):

    override_dh_auto_test:
    ifneq (arm, $(DEB_HOST_ARCH_CPU)) # skip the tests on armhf ("--- FAIL: TestParseSelector/linux (0.00s)  platforms_test.go:292: arm support not fully implemented: not implemented")
            cd '$(OUR_GOPATH)/src/github.com/containerd/containerd' && make test
    endif


## Skipping autopkgtesting entirely ##

If an autopkgtest is badly written, it may be too challenging to get it to pass.  In these extreme cases, its possible to request that test failures be ignored for purposes of package migration.

Checkout [https://git.launchpad.net/~ubuntu-release/britney/+git/hints-ubuntu](lp:~ubuntu-release/britney/hints-ubuntu)

File a MP against it with a description indicating the lp bug#, rationale for why the test can and should be skipped, and explanation of what will be unblocked to migration.

Reviewers should be 'canonical-server', 'ubuntu-release', and any archive admins or foundations team members you've discussed the issue with.


## Other Common Issues ##

Autopkgtest runs tests in a controlled network environment, so if a test case expects to download material from the internet, it will likely fail.  If the test case is attempting to download a dependency (e.g. via PIP or Maven), sometimes this can be worked around by adding the missing dependency to ```debian/tests/control```.  If it is attempting to download an example file, then it may be possible to make the test case use a local file, or to load from the proxy network.

