Backporting a Package
=====================

Backporting is the process of adapting a newer version of a software
package or a specific fix from a newer release (or upstream) to an
older, stable Ubuntu release. This is typically done to deliver critical
bug fixes, security updates, or essential features to users who do not
wish to upgrade the entire software stack to a newer, potentially
disruptive release of the distribution.

The methods described below leverage the git-ubuntu tool-suite and
standard Debian/Ubuntu packaging practices.

Required Reading
----------------

 * https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation
 * https://wiki.ubuntu.com/StableReleaseUpdates


Introduction to Backports
-------------------------

Ubuntu releases are snapshots, and while security updates and critical
bug fixes are routinely applied, major version upgrades of applications
or libraries are generally avoided in non-development releases to
maintain stability. However, there are scenarios where a newer version
or a specific fix is beneficial. This is where backporting comes in.

The word "backport" is used in a number of different contexts, so to
avoid ambiguity, this document is discussing a particular type of
backport that goes through the Stable Release Update (SRU)
process. There is also a *backports pocket* that users can subscribe to
with a variety of different software version updates. The security team
also occasionally releases full version updates of software as part of
their *security updates process*. And, teams and individuals may also use
Launchpad’s *Personal Package Archives (PPAs)* to provide newer versions
of software built for earlier Ubuntu releases. Each of these has their
own policies and caveats.

It is also important to clarify that the SRU process is used for more
than just backports, and indeed it is more typically used for
bugfixes. Bugfix SRUs introduce discrete, targeted patches for
individual, well-understood defects. Backport SRUs, by contrast,
introduce larger code changes that may include a range of (perhaps less
understood) bug fixes and other non-disruptive changes (docs, tests,
refactoring, cleanups), and sometimes even new features. The overriding
principle to follow is that none of these changes should introduce
regressions compared with what already exists. Therefore, backports
require careful consideration, thorough testing, and often specific
policy approvals.


General Backporting Principles
------------------------------

With Backport SRUs, the following principles apply, regardless of the
style used to achieve it:

* **Stability:** The primary goal is to maintain the stability of the
  target release. Backports must be rigorously tested to ensure they
  don't introduce regressions or break other parts of the system.

* **Policy Adherence:** Always refer to the
  [Ubuntu Stable Release Updates (SRU) Policy](https://wiki.ubuntu.com/StableReleaseUpdates).
  As well, if the software being backported has an
  [SRU Package-Specific Exception](https://documentation.ubuntu.com/sru/en/latest/reference/package-specific/), make sure to follow it's guidance.

* **Testing (Distro):** At a minimum the package will need to build
  properly on all of the package's target architectures, it's own
  autopkgtests must pass, and the autopkgtests of any packages dependent
  on it must pass.  The process of uploading the package to the archive
  will automatically cause all these checks to be run, so as an uploader
  you must follow up on any issues that arise.  A good practice is to
  upload your package to a PPA and run autopkgtests against it before
  uploading to the archive so if there are issues they can be solved
  ahead of time.

* **Testing (Package):** If a package has an SRU Package-Specific
  Exception, then additional testing should be performed as defined in
  that document.  If the package doesn't have an exception, then the SRU
  reviewers will likely expect that the uploader has run the testsuite
  (if one exists) and/or performed sufficient manual testing to ensure
  it is safe for inclusion in the archive.

* **Clear Justification:** For the Backport SRU bug reports, clearly
  articulate the rationale for the backport with a link to the
  package-specific exception page (if one exists), what notable changes
  are in the backport, how it was tested, and any irregularities
  encountered during packaging or testing. Crucially, highly any changes
  or findings that might be of concern for regression risk.


Prerequisites and Initial Setup
-------------------------------

Before starting, ensure you have your Ubuntu packaging environment set
up, including:

* **git-ubuntu:** The primary tool for interacting with Ubuntu's Git
  repositories.

* **Packaging Tools:** dpkg-dev, devscripts, build-essential, quilt,
  etc. (for patch management), and git-buildpackage (gbp) if importing
  from Debian's Git.

(Refer to the [Setting up your environment](Setup.md) section of this
document for detailed setup instructions.)

Once your environment is ready, follow these initial steps for any
backport:

1. **Identify the Target Package and Release:**
   Determine the package name, the upstream version you intend to
   backport, and the Ubuntu LTS release(s) you are targeting.
   ```
   package="squid"
   version="6.13"
   codename="noble"
   release="24.04"
   ```

2. **Update the Bug Report:**

   Ensure the relevant Launchpad bug report is updated to reflect the
   target release(s) for the backport. If there isn’t already a bug
   report, then file one yourself for tracking purposes.  This can
   be done through the Launchpad web UI, or with this handy CLI tool
   (available in the [ubuntu-helpers git repository](https://code.launchpad.net/~ubuntu-server/+git/ubuntu-helpers/)):
   ```
   lp_bug_number=2085197
   lp-affects-devel ${lp_bug_number} ${package} ${codename}
   ```

   If this package has been backported before, there may be an
   established template that should be used for the bug report's
   description.  If not, try to follow the standard SRU template, with
   particular attention given to the [Test Plan] section, since a
   package backport may include more extensive changes than an ordinary
   SRU and may thus require more expansive testing assurances.

3. Initialize the Job Directory:
   Create a dedicated directory for your backport work.
   ```
   mkdir -p ~/pkg/${package}/backport-lp${lp_bug_number}/
   cd ~/pkg/${package}/backport-lp${lp_bug_number}/
   ```

4. Checkout the git-ubuntu Code:
   Clone the git-ubuntu repository for the package.
   ```
   git ubuntu clone ${package} ${package}-gu
   cd ${package}-gu
   ```

## **4. Packaging a Software Backport**

There are generally three main styles of backporting, varying in complexity and scope:

* Full Package Backports
* Tarball-only Backports
* Partial Backports (Cherry-picks)

As mentioned previously, all three of these go through the SRU process,
and thus all three are "SRU Backports", but the technical procedure used
is different for each.


### **4.1. Backport the Full Packaging**

This is the most comprehensive type of backport, involving copying the
entire packaging (including a newer upstream tarball and the associated
debian/ directory) from a newer Ubuntu or Debian release to an older
one. This is typically done when a significant update is required, and
the packaging in the newer release is already well-adapted to the new
upstream.

**Use Cases:**

* A major new feature or upstream release is required, and a
  well-maintained package already exists in a newer release of Ubuntu or
  Debian.

* The debian/ directory has undergone substantial changes in the newer
  release, making tarball-only updates (section 4.2) too cumbersome.


**Process:**


1. **Determine Parent Commit:** For a given target release to backport
   to, figure out the base parent commit.

   $ git ubuntu clone ${package} ${package}-gu && cd ${package}-gu
   $ git checkout pkg/ubuntu/${codename}-devel -b ${codename}-devel
   $ git log

   Generally, the parent commit will be `pkg/ubuntu/${codename}-devel`,
   but there are (rare) situations where you will want something different.

   $ parent_commit="pkg/ubuntu/${codename}-devel"

2. **Backport the release:**

   $ branch="backport-lp${lp_bug_number}"  # or as desired
   $ git branch -m ${branch} ${branch}.old  # if necessary
   $ git checkout pkg/ubuntu/devel -b ${branch}
   $ git rebase -i --onto pkg/ubuntu/${codename}-devel "${parent_commit}"

3. **Resolve any merge conflicts.**

   Sometimes, the merge conflicts will merely be to `debian/changelog`,
   `debian/patches/series`, and perhaps `debian/control`, but for more
   complex cases, this can be more involved.

   In this style of backport, you will generally have a single commit
   for the backport itself that represents all the changes (tarball,
   packaging, and changelog) from the original Ubuntu release, and
   subsequent separate commits for each discrete packaging change (if
   any).

4. **Drop changes already included.**

   The old version may have changes that are now superseded by the new
   version you're backporting.  For example, security updates may have
   cherrypicked CVE patches from upstream but are now present in the new
   codebase.  Or fixes developed in Ubuntu and forwarded to Debian or
   upstream may now be available in the backported package without need
   for delta on the Ubuntu side.

5. **Review debian/control and Other Packaging Files:**

   Make sure none of the debian/ changes being backported would be
   inappropriate for that release.  E.g. versioned dependencies that
   won't match, compat version changes, etc.

   $ git diff pkg/ubuntu/${codename}-devel... debian/

   * **Dependencies:** Review Build-Depends and Depends. The
     dependencies from the newer release might not be available or might
     be older in your target release. Adjust them to match the target
     release's available package versions. This is a common point of
     failure.

   * **debian/compat:** Ensure the debhelper compatibility level is
     supported by the target release.

   * **debian/rules:** Check for any specific build flags or targets
     that might be incompatible.

   * **AppArmor/Systemd:** If relevant, ensure new AppArmor profiles or
     Systemd units are compatible with the target release.

6. **Create changelog entry:** The changelog will now reflect the
   history of the newer release. You must add a new entry to the
   debian/changelog file for the backport.

   ```
   ${package} (${version}~ubuntu0.${release}.1) ${distribution}; urgency=medium

   * Backport recent ${package} release v${upstream_version} from
     Ubuntu ${codename} (LP: #${lp_bug_number})
     - <Details about changes included in this release>
   ```

   $ git commit debian/changelog -m changelog

7. **Build and Test the Package:** Build the package and perform
   required testing to ensure all functionality works as expected and no
   regressions are introduced.

   * Building in a PPA is recommended.
   * Autopkgtests can be invoked against the package in the PPA.
   * Follow any additional preparatory testing procedures as defined in
     the *SRU Package-Specific Exception*.

8. **Merge Proposal and Migration:**

   From this point, the process follows the standard process for
   uploading a package for SRU:

   * [Prepare MP](MergeProposal.md)
   * [MP Review](MergeProposalReview.md)
   * [Upload](Sponsorship.mp) the .changes files
   * [SRU team acceptance](PackageFixing.md)
   * Validation testing:  Perform the [Test Plan] section of the
     backport SRU bug report, and post the results as a comment on the
     bug report.  As usual, change the tags from `validation-needed*` to
     `validation-done*`.
   * [Migration](ProposedMigration.md) & any followup



### **4.2. Backport Tarball-only into Existing Stable Packaging**

This method involves updating the source package in a stable release by
swapping its orig tarball out with a newer one while largely retaining
the existing Debian/Ubuntu packaging as-is. Changes to the debian/
directory are performed only to adapt to the requirements of the new
upstream code, and each change must be explicitly listed in the
debian/changelog entry.

**Use Cases:**

* This style is often preferred when upstream produces new releases at a
  frequent rate that we wish to track, and that the packaging changes
  from release to release tend to be stable.

* This is also often preferable when upstream maintains multiple stable
  branches of their releases, and we wish to track one of these stable
  branches that Debian does not package or does not track as closely as
  we wish.

* Policy allowing for minor version bumps (MRE) of upstream releases.

**Process:**

1. **Obtain New Upstream Tarball:** Identify the new upstream tarball (e.g., foo_1.2.3.orig.tar.gz).

  * **Option A: Export from a newer Ubuntu release (if already packaged
    there):**
    ```
    # From your <package-name>-gu directory
    git checkout pkg/ubuntu/devel -b ubuntu/devel
    git ubuntu export-orig
    tarball="${package}_${version}.orig.tar.xz"
    git checkout ${branch_name}
    ```

  * **Option B: Download from upstream and verify:**
    ```
    # From your parent job directory (e.g., ~/pkg/Openldap/backport-lp2085192/)
    pubkey_url="https://www.openldap.org/software/download/OpenLDAP/gpg-pubkey.txt"
    tarball_url="https://www.openldap.org/software/download/OpenLDAP/openldap-release/openldap-2.5.19.tgz"
    signature_url="https://www.openldap.org/software/download/OpenLDAP/openldap-release/openldap-2.5.19.tgz.asc"
    checksum_url="https://www.openldap.org/software/download/OpenLDAP/openldap-release/openldap-2.6.9.sha3-512"
    tarball="${package}_${version}.orig.tar.gz"

    wget -v -O ../openldap-pubkey.txt ${pubkey_url}
    wget -v -O ../${tarball} ${tarball_url}
    wget -v -O ../${tarball}.asc ${signature_url}
    wget -v -O ../${tarball}.sha3-512 ${checksum_url}

    # Verify tarball (adjust key ID and tarball name as needed)
    gpg --keyserver pgpkeys.mit.edu --recv-key <key-id>
    gpg --edit-key <key-id>
      # gpg> trust
      # gpg> save
    gpg --verify ../${tarball}.asc ../${tarball}
    # Also verify checksum:
    sha3sum -c ../${tarball}.sha3-512 --ignore-missing # Or other checksum tool
    ```

2. **Checkout the Target Release and Create New Branch:**
   First, ensure your local git-ubuntu repository is up-to-date and you are on your backport branch.
   ```
   cd ${package}-gu
   git fetch
   branch_name="backport-${codename}-${version}"
   git checkout --no-track pkg/ubuntu/${codename}-devel -b ${branch_name}
   ```

3. **Prepare for Tarball Import:**
   Remove all existing source files in the working directory, keeping
   only the debian/ directory and .git information.
   ```
   rm -vrf $(ls -a1 | grep -v -e '^.git/*$' -e '^./*$' -e '^\.\./*$' -e '^debian/*$' | xargs)
   ```

4. **Import New Upstream Tarball:** Extract the new upstream tarball into the current directory.
   ```
   tar xv --strip-components=1 -f ../${tarball} # Assuming tarball is in parent dir
   git add -f . # Add all new/changed files, including potentially new upstream files
   git commit -s -m "New Upstream release ${version}"
   ```

   This effectively "imports" the new upstream source alongside the
   existing debian/ directory.

5. **Merge and Resolve Conflicts:** The process of importing the new
   tarball might result in merge conflicts between the new upstream
   source and your debian/ directory if debian/ was modified by the
   tarball extraction.

   * Carefully resolve conflicts, prioritizing the new upstream code for
     the application logic and preserving Ubuntu-specific packaging
     changes in debian/.

   * Pay close attention to changes that might affect build dependencies
     (debian/control), build rules (debian/rules), or patches
     (debian/patches).

6. **Verify Existing Patches Apply Cleanly:**
   Use quilt to check and manage patches.
   ```
   quilt push -a && quilt pop -a
   ```

   * If any patches are now applied upstream or are no longer necessary,
     remove the patch file and its entry in debian/patches/series.

   * Commit these changes with a clear message:
     ```
     Drop d/p/<patch-name>.patch
     [Applied in upstream release ${version}]
     ```

7. **Check Debian for Relevant Changes:**
   If the new upstream version is already packaged in Debian, review
   their debian/ directory for any interesting changes or adaptations
   they've made. Apply these as appropriate, ensuring they do not enable
   any new features that are not intended for backporting.

   ```
   # From your parent job directory (e.g., ~/pkg/Openldap/backport-lp2085192/)
   git clone https://salsa.debian.org/openldap-team/openldap openldap-deb
   cd openldap-deb
   git checkout 2.5.19+dfsg-1 -b 2.5.19+dfsg-1
   git log # Review commit history for relevant changes
   # Use 'git diff' or manual inspection to compare debian/ directories
   ```

8. **Adapt Packaging to New Upstream:**
   * **debian/control:** Review and update Build-Depends, Depends,
     Breaks, Replaces based on upstream changes and the target release's
     available package versions.

   * **debian/rules:** Check if upstream build system changes require
     adjustments.

   * **debian/changelog:** Add a new entry indicating the upstream
     version bump and the reason for the backport, referencing the
     bug. Increment the Ubuntu revision number.

     ```
     package-name (${version}) noble; urgency=medium

       * New upstream release ${version} (LP: NNNNNNN)
         - Includes fix for <bug>, ...
         - Other changes from upstream.

     -- Your Name <your.email@example.com>  <Date and Time>
     ```

   * After editing the changelog, commit it:
     ```
     dch -i
     git add debian/changelog
     git commit -m "changelog for ${new\_version}"
     ```

### **4.3. Partial Backports (Cherry-picks)**

For completeness, a third type of backport is effectively just the
regular SRU process, but applied more broadly to a software release.

This type of backport involves selecting and applying specific commits
from a newer branch (e.g., upstream, Debian, or a newer Ubuntu release)
to an older one. It is ideal for situations where the new release
contains a mix of desirable bug fixes with undesirable feature changes,
particularly for packages that do not have a formal SRU exception.

This differs importantly from other styles of backporting in that *each
patch must have its own SRU bug report with a detailed test case to
reproduce the defect and validate the fix*. In other words, if you are
backporting 7 commits from a new release, you need to have 7 defined
test cases documented in 7 bug reports.  If any one of those fails
validation, the entire SRU will be rejected; you'll need to remove the
problematic patch and re-submit the SRU without it.

**Process:**

1. **Identify the Backportable Commits:** Note the specific commit(s) in
   the a newer release that you will be including in the backport.

2. **Convert the Desired Commit(s) to patches:** Use git format-patch to
   extract the desired commits as patches.
   ```
   cd ${package-name}-gu
   git fetch
   git checkout ubuntu/devel
   git format-patch HEAD~N
   ```

3. **Create a New Branch:** Always work on a new branch for your
   changes.
   ```
   branch_name="backport-${codename}-${version}"
   git checkout pkg/ubuntu/${codename}-devel -b ${branch_name}
   ```

   **Check Patch Application:** Apply each patch in turn and verify it
   applies
   ```
   quilt init  # if debian/packages/ doesn't already exist
   quilt push -a  # if debian/packages/ exists already
   quilt new <patch-name>.patch
   patch -p1 < <patch-name>.patch
   quilt add <files>
   quilt refresh
   git add debian/patch/series debian/patches/<patch-name>.patch
   git commit -m "  * d/p/<patch-name>.patch: <patch-description>"
   quilt push -a
   quilt pop -a
   ```

   * **Apply patch:** Resolve each patch carefully, ensuring the fix is
     correctly applied without introducing regressions.  Try to keep
     the patches as close to the original commit as possible, or where
     not possible keep them to the minimum necessary changes to provide
     the desired improvement.

   * **Review Changes:** After ensuring the patch applies, review it to
     verify the intended fix is still being presented properly.

   * **Update Patch Header:** In the metadata at the top of the patch,
     include a note about updates you needed to make to get the patch to
     apply, along with the date and your name.

4. **Update debian/changelog:** Add an entry to debian/changelog using
   `dch -i`, clearly describing the backported fix, referencing the
   Launchpad bug, and noting that it's a backport. Increment the Ubuntu
   [revision number](VersionStrings.md) appropriately.
   ```
   package-name (${version}) noble; urgency=medium

     * Backport fixes from ${version} (LP: NNNNNNN)
       - d/p/<patch-name>.patch: <description> (commit ABCDEF)
       - ...

    -- Your Name <your.email@example.com>  <Date and Time>
   ```

Build the Package
-----------------

In general, the process of building the package follows the same steps
as defined in the
[Ubuntu Maintainer's Handbook](PackageBuilding.md).
It is generally recommended to generate a source package locally inside
a clean LXD container of the target release, and then perform the binary
build in a PPA.

```
ppa create "${package}-backport-lp${lp_bug_number}"
```

Prepare for PPA build:

  1. Checkout the branch you want to test (e.g., your backport branch)
  2. Update changelog for PPA build (optional, but good practice for testing)
  ```
  dch -i  # Add entry with version suffixed with ~${codename}1 and
  changelog entry 'Build for PPA'.
  ```
  3. Build the source package, specifying the version currently in the
     target release.
  ```
  debuild -sa -S -v${prior_version}
  ```
  4. Upload to PPA
  ```
  ppa_address="ppa:<your-launchpad-id>/${package}-backport-lp${lp_bug_number}"
  changes_file="../${package}_${new_version}~${codename}1_source.changes"
  dput ${ppa_address} ${changes_file}
  ```


6. Testing and Quality Assurance
--------------------------------

Regardless of the backport type, rigorous testing is paramount.

* **Trigger Autopkgtests:** Autopkgtests are crucial for verifying the
  package's integrity across various architectures.

* **Install in Clean Environment:** Test the .deb files from the PPA in
  a clean LXD container or VM running the exact target Ubuntu release.

* **Dependency Check:** Verify that all runtime dependencies are met and
  that the package doesn't pull in excessive or undesirable new
  dependencies.

* **Check for Known Test Overrides:** Be aware of any existing test
  overrides for the package and whether they are still valid or need
  adjustment with the backport.

* **Smoketest:** Check the basic functionality particular to the type of
  software.

  * Ensure the package installs and uninstalls correctly.
  * For command line programs, check that its commands display their
    expected **\--help** text.
  * For services, ensure the service starts as expected, and can be shut
    down using its normal mechanisms.
  * For libraries, ensure at least one of its main reverse-dependencies
    builds against it without error.
  * For graphical applications, make sure the UI launches and is visible
    without obvious corruption.

* **Verification:** Perform testing as specified in the SRU bug report's
  [Test Plan], and include the results in a comment to the bug.


Merge Proposal and Sponsorship
------------------------------

Once your backport is thoroughly tested and you are confident in its
stability:

1. **Create a new source package**:  You can use the same steps as done
   above, but make sure to drop any ~${codename}1 suffix (and the "Build
   for PPA" changelog entry).

2. **Push and Create Merge Proposal:** Push your branch to your
   Launchpad repository and create a merge proposal targeting
   pkg/ubuntu/<target-release>-devel (e.g., pkg/ubuntu/noble-devel).

   git push <lp_username> ${changes_file}

3. **Create a Merge Proposal (MP) on Launchpad**:
   The previous step should print a URL for creating an MP.  If not,
   navigate to your branch on Launchpad (e.g.,
   https://code.launchpad.net/~<you>/ubuntu/+source/${package_name}/+git/${branch_name})
   and click "Create merge proposal."

   * **Target the Correct Release:** Ensure the MP targets the correct
     Ubuntu release (e.g., ubuntu/noble/main).
   * **Fill out the MP Description:**
     * Link to the relevant Launchpad bug.
     * Clearly describe the type of backport, what it fixes/introduces,
       and why it's necessary for the stable release.
     * Detail your testing steps and results.
     * State any known limitations or risks.
     * Specify if you'll need your upload sponsored.
     * **Crucially, unmark it as "Ready for Review" initially if you
       need to add more details or if it's part of a larger series of
       changes.** (The "Needs review" checkbox can cause timeouts if
       unchecked immediately after filing, so it's often better to file,
       then edit the MP to uncheck it if needed.)
   * In the merge proposal description, link to the bug report and explain
     the backport.

4. **Request Review and Sponsorship:** Once the MP is ready, you can request
   review and sponsorship for your upload from via the
   ["Patch Pilot" program](https://discourse.ubuntu.com/t/ubuntu-patch-pilots/37705),
   by subscribing the ubuntu-sponsors team.  Your upload will go into the
   [Sponsorship Queue](http://sponsoring-reports.ubuntu.com/general.html).
   A patch pilot will review your code, packaging, and testing before uploading
   to the archive.
