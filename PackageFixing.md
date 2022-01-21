Fixing a Package
================

In this tutorial we walk through the process of evaluating a bug, finding a fix for it, and then packaging the fix for Ubuntu.  Every bug is unique, of course; this is intended to illustrate the mindset and steps one would follow generally.

Required Reading
----------------

 * https://ubuntu.com/blog/git-ubuntu-clone
 * http://dep.debian.net/deps/dep3
 * https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation
 * https://wiki.ubuntu.com/StableReleaseUpdates



Evaluate the Bug
----------------

Bug Report: https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470

#### Description:

The original bug report was filed with just this description:

    Fresh install of 18.04 server. Every 5 minutes postconf segfaults:

    Mar 5 14:30:05 hostname-here kernel: [ 672.082204] postconf[12975]: segfault at 40 ip 0000564d613ff053 sp 00007ffc39e19b90 error 4 in postconf[564d613e7000+25000]
    Mar 5 14:30:06 hostname-here kernel: [ 672.303499] postconf[13004]: segfault at 40 ip 000055b29d0f8053 sp 00007fff72f4b740 error 4 in postconf[55b29d0e0000+25000]

According to the Apport log, the crash is caused by following command line:

    $ postconf -h queue_directory

Running the command in the shell, however, works as expected and lists
the default spool directory (/var/spool/postfix).

    ProblemType: Bug
    DistroRelease: Ubuntu 18.04
    Package: postfix 3.3.0-1
    ProcVersionSignature: Ubuntu 4.15.0-10.11-generic 4.15.3
    Uname: Linux 4.15.0-10-generic x86_64
    ApportVersion: 2.20.8-0ubuntu10
    Architecture: amd64
    Date: Mon Mar 5 14:26:27 2018
    SourcePackage: postfix
    UpgradeStatus: No upgrade log present (probably fresh install)

Note that the metadata at the end of the description is what gets
appended when filing a bug report is automatically triggered, or if the
user uses a bug reporting assistant (i.e. Apport).

Sometimes these types of bug reports will also include an attached
"something.crash" file.  This is created by the Apport process running
on the user's system at the time of segfault, and typically includes the
core dump, logs, and other relevant information.  If the user has
provided a .crash file, you can (examine it manually)[DebugApportCrash.md]
to get a useful stacktrace.


### Try to reproduce the issue

Not all bugs can be easily reproduced, and not all reproducible bugs will be obvious how to reproduce them.  In these cases, some bug work will be needed to isolate the problem ourselves or work with bug reporters to narrow the cause enough to identify a fix.

However, in this case we're lucky.  The bug triagers have identified a way to reproduce the issue, in [comment #12](https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470/comments/12):

    ubuntu@bionic-postfix:~$ postconf virtual_alias_map
    Segmentation fault (core dumped)
    ubuntu@bionic-postfix:~$ dpkg-query -W postfix
    postfix 3.3.0-1
    ubuntu@bionic-postfix:~$ ll /etc/postfix/valiases.cf
    -rw-r----- 1 root root 169 May 7 14:08 /etc/postfix/valiases.cf
    ubuntu@bionic-postfix:~$

Let's see if we can reproduce the issue as well, using these directions.

Before that, we need to set up an environment for doing the testing.  There's many options for where to do your testing, and different developers have their own preferences.  Here's a couple:

#### Make a container for testing:

    $ lxc launch images:ubuntu/bionic tester
    $ lxc exec tester -- bash

#### Alternatively: Make a VM for testing:

The user's password in this tester VM will be "ubuntu"

    $ uvt-simplestreams-libvirt --verbose sync --source http://cloud-images.ubuntu.com/daily release=bionic arch=amd64
    $ uvt-simplestreams-libvirt --verbose sync release=bionic arch=amd64
    $ uvt-kvm create tester release=bionic arch=amd64 label=daily --password ubuntu
    $ uvt-kvm wait tester
    $ uvt-kvm ssh tester

#### Get up to date and install postfix:

    root@tester:~# apt dist-upgrade
    root@tester:~# apt install -y postfix

#### Tell postfix to use a map file:

    root@tester:~# echo "virtual_alias_maps = pgsql:/etc/postfix/valiases.cf" >> /etc/postfix/main.cf

#### To reproduce, the file must be unreadable by the current user:

    root@tester:~# touch /etc/postfix/valiases.cf
    root@tester:~# chmod 0600 /etc/postfix/valiases.cf

#### Reproduce the issue:

    root@tester:~# su - ubuntu
    ubuntu@tester:~$ /usr/sbin/postconf virtual_alias_map
    Segmentation fault (core dumped)

Now we have confirmed the bug.

Note: Keep track of the commands you used to reproduce the bug. You'll need them later.



Check if it's already been fixed
--------------------------------

Often we can save time by leveraging someone else's work, so it's always worth doing some research upfront.  Fixes for bugs can be found in newer versions of Ubuntu, Debian, or upstream, and sometimes in external forums or bug trackers.

### Was it fixed in a newer Ubuntu?

Easiest thing to review the package's status in Ubuntu:

    $ rmadison postfix
     postfix | 2.9.1-4           | precise         | source, amd64, armel, armhf, i386, powerpc
     postfix | 2.9.6-1~12.04.3   | precise-updates | source, amd64, armel, armhf, i386, powerpc
     postfix | 2.11.0-1          | trusty          | source, amd64, arm64, armhf, i386, powerpc, ppc64el
     postfix | 2.11.0-1ubuntu1.2 | trusty-updates  | source, amd64, arm64, armhf, i386, powerpc, ppc64el
     postfix | 3.1.0-3           | xenial          | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
     postfix | 3.1.0-3ubuntu0.3  | xenial-updates  | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
     postfix | 3.3.0-1           | bionic          | source, amd64, arm64, armhf, i386, ppc64el, s390x
     postfix | 3.3.0-1ubuntu1    | cosmic          | source, amd64, arm64, armhf, i386, ppc64el, s390x

Debian can also be worth checking:

    $ rmadison -u debian postfix
    postfix    | 3.3.0-1          | testing        | source, amd64, arm64, armel, armhf, i386, mips64el, mipsel, ppc64el, s390x
    postfix    | 3.3.0-1          | unstable       | source, amd64, arm64, armel, armhf, i386, mips64el, mipsel, ppc64el, s390x
    ...

We see that 3.3.0-1ubuntu1 exists under cosmic, so postfix has been modified there. Let's see what was changed.

#### Clone the Package

Find the repository name:

    $ apt-cache show postfix | grep Source:

In this case, there is no Source field, so we just use "postfix".

    $ git ubuntu clone postfix postfix-gu

This will create a new git clone of the postfix repo named "postfix-gu", with a remote of "pkg". The current branch will be ubuntu-devel, and the various versions for each distribution version will be under `pkg/ubuntu/version`.

Notes:

 * Due to https://launchpad.net/bugs/1761821, you may get: `fatal: could not read Username for 'https://git.launchpad.net': terminal prompts disabled.` It's safe to ignore this.
 * First time will add a git-ubuntu entry to .gitignore
 * Sometimes it can also be helpful to checkout the git repositories for the package maintained by Debian and/or upstream.  These would be checked out to "postfix-debian" and "postfix" respectively.

#### View the Commit Log

    $ git log -b pkg/ubuntu/cosmic
    ...
    commit 73cb543efe06a340021cbf538d3ca88abfd96bd8 (tag: pkg/upload/3.3.0-1ubuntu1)
    Author: Andreas Hasenack <andreas@canonical.com>
    Date:   Wed May 9 10:14:49 2018 -0300

        changelog

    commit d4cb4562480496f8a1b25ddc397cef45dd45d855
    Author: Andreas Hasenack <andreas@canonical.com>
    Date:   Wed May 9 09:51:20 2018 -0300

          * debian/patches/fix-postconf-segfault.diff: Fix a postconf segfault
            when map file cannot be read.  Thanks to Viktor Dukhovni <postfix-
            users@dukhovni.org>. (LP: #1753470)

d4cb45 sure looks like a fix for this issue!

    $ git log -b -p pkg/ubuntu/cosmic
    ...
    diff --git a/debian/patches/fix-postconf-segfault.diff b/debian/patches/fix-postconf-segfault.diff
    new file mode 100644
    index 00000000..f8eef6bf
    --- /dev/null
    +++ b/debian/patches/fix-postconf-segfault.diff
    @@ -0,0 +1,25 @@
    +Description: Fix a postconf segfault when map file cannot be read
    +Author: Viktor Dukhovni <postfix-users@dukhovni.org>
    +Origin: https://marc.info/?l=postfix-users&m=152578771531514&w=2
    +Bug-Debian: https://bugs.debian.org/898271
    +Bug-Ubuntu: https://launchpad.net/bugs/1753470
    +Last-Update: 2018-05-09
    +---
    +This patch header follows DEP-3: http://dep.debian.net/deps/dep3/
    +--- a/src/postconf/postconf_dbms.c
    ++++ b/src/postconf/postconf_dbms.c
    +@@ -174,10 +174,10 @@
    +        */
    +       dict = dict_ht_open(dict_spec, O_CREAT | O_RDWR, 0);
    +       dict_register(dict_spec, dict);
    +-      if ((fp = vstream_fopen(cf_file, O_RDONLY, 0)) == 0
    +-          && errno != EACCES) {
    +-          msg_warn("open \"%s\" configuration \"%s\": %m",
    +-                   dp->db_type, cf_file);
    ++      if ((fp = vstream_fopen(cf_file, O_RDONLY, 0)) == 0) {
    ++        if (errno != EACCES)
    ++              msg_warn("open \"%s\" configuration \"%s\": %m",
    ++                           dp->db_type, cf_file);
    +           myfree(dict_spec);
    +           return;
    +       }
    diff --git a/debian/patches/series b/debian/patches/series
    index c2e47271..1f77ec0b 100644
    --- a/debian/patches/series
    +++ b/debian/patches/series
    @@ -15,3 +15,4 @@
     50_LANG.diff
     70_postfix-check.diff
     tls_version.diff
    +fix-postconf-segfault.diff

Here we see the patch and the change to debian/patches/series to include the patch. This is the fix we need!


### Was it fixed in Debian?

Sometimes the fix may have been updated in Debian instead of Ubuntu.  There are many ways to locate fixes from Debian.  Debian maintains its own git repository for many (but not all) of its packages, so having a clone of this can be handy.

For example, let's assume for argument's sake that we had a problem with sshd in xenial, where it would fail to check config files before reloading (https://bugs.launchpad.net/ubuntu/+source/openssh/+bug/1771340).  From Debian's openssh source package page (https://packages.debian.org/source/stretch/openssh), we find the git repository (https://salsa.debian.org/ssh-team/openssh), and can check it out:

    $ git clone https://salsa.debian.org/ssh-team/openssh.git openssh-debian
    $ cd openssh-debian
    $ git branch -av | cat
    * master                               296562ba1 releasing package openssh version 1:8.2p1-4
      remotes/origin/HEAD                  -> origin/master
      remotes/origin/buster                6d9ca74c4 releasing package openssh version 1:7.9p1-10+deb10u2
      remotes/origin/etch                  851625c74 releasing version 1:4.3p2-9etch1
      remotes/origin/experimental          09a03c340 Update contact information for Natalie Amery
      remotes/origin/jessie                9da94db38 Merge branch 'jessie' into 'jessie'
      remotes/origin/master                296562ba1 releasing package openssh version 1:8.2p1-4
      remotes/origin/pristine-tar          5fdaf4d7d pristine-tar data for openssh_8.2p1.orig.tar.gz
      remotes/origin/sarge                 f297a6e07 debconf-updatepo
      remotes/origin/squeeze               faa0b9a59 releasing package openssh version 1:5.5p1-6+squeeze5
      remotes/origin/stretch               0ef21e4e2 Merge branch 'fix-923486-stretch' into 'stretch'
      remotes/origin/ubuntu/saucy          f8daff632 releasing package openssh version 1:6.2p2-6ubuntu0.5
      remotes/origin/ubuntu/trusty         f6ffa5954 releasing package openssh version 1:6.6p1-2ubuntu2
      remotes/origin/ubuntu/xenial         bd9cfb441 releasing package openssh version 1:7.2p2-4ubuntu1
      remotes/origin/upstream              f0de78bd4 Import openssh_8.2p1.orig.tar.gz
      remotes/origin/upstream-experimental 102062f82 Import openssh_8.0p1.orig.tar.gz
      remotes/origin/upstream-jessie       487bdb3a5 Import openssh_6.7p1.orig.tar.gz
      remotes/origin/upstream-stretch      971a76537 Import openssh_7.4p1.orig.tar.gz
      remotes/origin/wheezy                e345e2a5f releasing package openssh 1:6.0p1-4+deb7u3
      remotes/origin/wheezy-backports      1d95da812 Remove now-unnecessary backports-specific version changes.

That's a lot of branches, but the ones of most interest will be master and sometimes experimental.  Master is already checked out, so lets peruse its commit history.  Doing this, we find:

    commit d4181e15b03171d1363cd9d7a50b209697a80b01
    Author:     Colin Watson <cjwatson@debian.org>
    AuthorDate: Mon Jun 26 10:18:26 2017 +0100
    Commit:     Colin Watson <cjwatson@debian.org>
    CommitDate: Mon Jun 26 10:18:26 2017 +0100

        Test configuration before starting or reloading sshd under systemd (closes: #865770).

Our issue would be the same as Debian bug #865770.

It's also possible to search for commits via Debian's web front-end for git, https://salsa.debian.org.  Doing so in this case would bring you to https://salsa.debian.org/ssh-team/openssh/commit/d4181e15b03171d1363cd9d7a50b209697a80b01

Either way, you should also mention the salsa link in the fixed up bug report, and possibly include it in your fix commit message.

Since we can't push new versions of packages to previous Ubuntu releases, you'd need to backport the fix by copying what Debian did into a new commit on xenial.



### Was it fixed upstream?

For bugs that aren't already fixed in Ubuntu or Debian, sometimes the original developers of the software have already found and fixed the issue, or at least are aware of it and may have a proposed solution or workaround available.

From the unpacked package directory, a quick way to see if there's a newer upstream release is via `uscan`:

    $ cd dovecot-gu/
    $ uscan --safe
    uscan: Newest version of dovecot on remote site is 2.3.10, local version is 2.3.7.2
    uscan:    => Newer package available from
      https://dovecot.org/releases/2.3/dovecot-2.3.10.tar.gz

This only works if the package has a debian/watches file.  If it doesn't, look in the package's README or other documentation, and do the research online manually.

Searching the upstream bug tracker, or generally googling on error messages or symptoms can sometimes turn up a patch or bug report of relevance.


### Forwarding issues upstream

If there are no existing fixes for an issue, you can either develop one yourself, or communicate the problem to Debian or the upstream developers.  Sometimes clues can be found "in the wild" via random forum posts or bug trackers, but be aware these can span the full range from high quality to dangerous so treat them only as ideas and don't accept anything blindly.

Each upstream project has its own conventions and expectations for how they can be communicated with.  Check the source tree and development section of the upstream's website for policies, or study other recent bug reports and patch contributions for best practices to follow.

In general though, it is a good idea to make sure you are able to reliably reproduce the issue yourself.  Document the steps you follow in a way that non-Ubuntu users could follow.  If there is a workload or test case, try to simplify it down to the minimal set of commands to reproduce the problem.

When filing the bug report or pull request upstream, do identify yourself as a Ubuntu developer and your role in forwarding an issue reported against the distribution.




Apply the Fix
-------------

Changes to packages are done via patches. The patches themselves are stored in debian/patches/ under the root of the package repository. debian/patches/series lists the order in which the patches should be applied. debian/changelog lists what changes have been made to the package over time.

We use git-ubuntu to make changes to packages.


#### Step 1: Assign the task to yourself

First, go back to https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470

Go to the task (row) that starts with "bionic" and assign the task to yourself and switch the status to "in progress" using the yellow pencil icons. If you don't see yellow pencil icons, you need to get permissions.


#### Step 2: Clone the package (if you haven't already)

Find the repository name:

    $ apt-cache show postfix | grep Source:

In this case, there is no Source field, so we just use postfix.

    $ git ubuntu clone postfix postfix-gu
    $ cd postfix-gu


#### Step 3: Make a branch based on the appropriate ubuntu branch

The affected version of postfix is in bionic, so we branch from `bionic-devel`.
It helps to use a branch name that's descriptive.

    $ git checkout pkg/ubuntu/bionic-devel -b postfix-sru-lp1753470-segfault-bionic


#### Step 4: Make a patch to fix the issue (maybe)

If the only changes you made are within the debian subdir, you don't need a patchfile, and can skip this step.

On the other hand, if you've made changes to the upstream code (anything outside of the debian directory), you'll need to generate a patch in debian/patches.

See [Making a Patchfile](DebianPatch.md)


#### Step 5: Commit the patch

See [Committing your Changes](CommittingChanges.md)



Build a Fixed Package
---------------------

See [Package Building](PackageBuilding.md)



Test the Package
----------------

### Start a bionic container and enter it:

We can name our lxc containers with any scheme we wish, such as 'tester' earlier for a temporary one to test with.  But for bug fixes we'll often need to keep the container around for reference as the bug fix goes through the review, sponsorship, and SRU processes.  So, to keep things consistent let's reuse our git branch name, and just prefix the package name:

    $ lxc launch ubuntu:bionic postfix-sru-lp1753470-segfault-bionic
    Creating postfix-sru-lp1753470-segfault-bionic
    Starting postfix-sru-lp1753470-segfault-bionic

    $ lxc exec postfix-sru-lp1753470-segfault-bionic -- bash
    root@postfix-sru-lp1753470-segfault-bionic:~# 


### Reproduce the Bug

Record your steps as you go (you'll need them later):

    # apt dist-upgrade
    # apt install -y postfix
    # touch /etc/postfix/valiases.cf
    # chmod 0600 /etc/postfix/valiases.cf
    # echo "virtual_alias_maps = pgsql:/etc/postfix/valiases.cf" >> /etc/postfix/main.cf
    # su - ubuntu
    $ /usr/sbin/postconf virtual_alias_map
    Segmentation fault (core dumped)


### Install the fixed package

In this case, I'm using the PPA. Alternatively, if you've built locally, you can copy in the .deb file and install it manually.

    $ sudo add-apt-repository -ys ppa:kstenerud/postfix-sru-lp1753470-segfault
    $ sudo apt update
    $ sudo apt upgrade -y


### Test the Bug Again

    $ /usr/sbin/postconf virtual_alias_map
    /usr/sbin/postconf: warning: virtual_alias_map: unknown parameter

The bug is fixed! Sweet!



Run the Package Tests
---------------------

The DEP8 autopkgtests don't exercise our bug, but are worth running as just-in-case sanity checks and to catch regressions.

See [Running Package Tests](PackageTests.md)

Any change in behavior should be considered priorities to resolve before proceeding.


Start a Merge Proposal
----------------------

See [Merge Proposals](MergeProposal.md)



Update the Bug Report
---------------------

For regular bug fixes and merges, adding a comment about your progress is typically all you'll need.  You might provide some links to your PPA if you'd like to get people to test your fix, or if you want to provide the fix to the userbase swiftly.

### SRU Paperwork

For stable release updates (SRUs), on the other hand, you need to add a bit more detail.

Go back to the bug report (in my case, https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470).

Modify the bug description (yellow pencil icon) and update it to conform with https://wiki.ubuntu.com/StableReleaseUpdates#SRU_Bug_Template  These are normally the [Impact], [Test Case] and [Where problems could occur] sections.  It is good practice to make the Test Case section itemized with explicit steps, "paint by numbers" style.  It is also best practice to include both a [Development Fix] and [Stable Fix]; the former explains the situation with the fix in the current development release, while the latter explains your strategy for addressing (or skipping) it in LTS and other stable releases.

Note: Keep the original description as-is, in a section called `[Original Description]` at the bottom.

Note: You'll see your branch and merge proposal in the `Related branches` because of the (LP: #NNNNNN) in the changelog entry.


SRU Review Process
------------------

There is a distinction between sponsorship and the SRU process. They are possibly a little confused in the SRU wiki page (especially section 6 “Fixing several bugs in one upload).

Consider the process from the point of view of your sponsor and the SRU team. On review, they will start from the diff and expect to see:

  * The diff fully explained by the changelog entry. This means that if there is something in the diff that is not explained by the changelog, then there is a problem.
  * A bug for everything mentioned in the changelog entry. Reviewers are pragmatic: there is no strict rule such as "every bullet point must refer to a bug"”, but more that logically everything mentioned corresponds to a bug, so that a reviewer can go to a bug to find more information on any part of the changelog. For an SRU, even added functionality must refer to a bug. If some part of a changelog entry does not obviously refer to a bug, then there is a problem.
  * Every issue mentioned in an SRU changelog must have a bug task filed against the package.  The same bug # can be mentioned in different SRUs, since a bug may have multiple bug tasks. The Ubuntu Bug Control Team, or other members of the server team can assist if you need help creating bug tasks.
  * The issue should be resolved for the Ubuntu development release.  This is tracked by having a bug task set to Fix Released for the devel series. The goal is to avoid regression from a user’s perspective when they upgrade to the newer Ubuntu release. If the status is not Fix Released but you still want to proceed with the SRU, explain what is going on in a [Development Fix] section.
  * Every LP bug # mentioned in an SRU changelog must have "SRU paperwork" filled out as described in the previous section.

After you or your sponsor have uploaded your package:
  * Set the bug task status to "In Progress"
  * The upload will appear in the "unapproved queue", for example https://launchpad.net/ubuntu/focal/+queue?queue_state=1.  It may take a week or two before its processed.
  * If you find a problem while its still unapproved, ask in the Freenode #ubuntu-release channel for the package to be rejected from the queue.  This is a trivial task for archive admins.  If rejected at this stage then the same version number can be re-used in a subsequent upload.
  * The SRU team will review incoming SRU uploads from the unapproved queue and expect to see the review items completed correctly as above. They will either accept or reject (with a reason) from the unapproved queue. If they reject, then you will need to handle the rejection reason and then start again from the beginning. If they accept, then the bug task will change to Fix Committed, the package will enter the -proposed pocket and then the package binaries will be built.


When the Bugfix is Accepted
---------------------------

### The Acceptance Email

You'll receive an email notification that the bugfix was accepted:

    Accepted postfix into bionic-proposed. The package will build now and be
    available at
    https://launchpad.net/ubuntu/+source/postfix/3.3.0-1ubuntu0.1 in a few
    hours, and then in the -proposed repository.

    Please help us by testing this new package.  See
    https://wiki.ubuntu.com/Testing/EnableProposed for documentation on how
    to enable and use -proposed.Your feedback will aid us getting this
    update out to other Ubuntu users.

    If this package fixes the bug for you, please add a comment to this bug,
    mentioning the version of the package you tested and change the tag from
    verification-needed-bionic to verification-done-bionic. If it does not
    fix the bug for you, please add a comment stating that, and change the
    tag to verification-failed-bionic. In either case, details of your
    testing will help us make a better decision.

    Further information regarding the verification process can be found at
    https://wiki.ubuntu.com/QATeam/PerformingSRUVerification .  Thank you in
    advance!

    ** Changed in: postfix (Ubuntu Bionic)
           Status: In Progress => Fix Committed

    ** Tags added: verification-needed verification-needed-bionic

Follow the build link (https://launchpad.net/ubuntu/+source/postfix/3.3.0-1ubuntu0.1) and make sure that it's publishing to the correct place (bionic), and that the builds completed (green checkmarks).


### IRC Monitoring

Join #ubuntu-ci-eng on the freenode IRC server to get pinged with your name when CI events occur.


### The Excuses Page

Check the "excuses" or "migration" page (for bionic in this case): http://people.canonical.com/~ubuntu-archive/proposed-migration/bionic/update_excuses.html

General page: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html

Eventually, the package with your fixes will appear there (search for postfix in this case). It will show the dep8 tests for postfix and anything that depends on it. Any tests that fail will show in red.

Note: This page is generated every few minutes, and doesn't update real-time.


### SRU Verification

It's best to have the package independently verified (preferably by the person who reported the bug), but if it sits idle too long (2 days or so), you can verify it yourself.  Follow the instructions provided by the SRU team, which usually means changing the verification-needed tag into verification-done.

https://people.canonical.com/~ubuntu-archive/pending-sru.html shows what SRUs are pending, and what their status is.  Note that this includes dep8 test results; if these have failed then it is unlikely the SRU team will release the update, so it's wise to followup if this happens.

Once all of the SRU's bugs have reached verification-done and a 7-day waiting period has elapsed, the SRU team will move the source and binary packages into the -updates pocket and mark the bug task(s) as Fix Released.
