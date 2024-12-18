# Making a patchfile

All changes from the Debian version of a package (with the exception of files
inside the `debian` directory) must be applied in the form of a patch.

## Generate a patchfile with Quilt

You can use the `quilt` command to generate a patchfile like so:

```bash
$ quilt new my-changes.patch
$ quilt add src/some_source_file.c
(modify the file)
$ quilt refresh
$ quilt pop -a
```

Unfortunately, this requires you to add the file to Quilt BEFORE you do any
modifications, so if you forget to do so, you're stuck.


## Generate a patchfile with Git

This method allows you to make your changes first, but you must be careful not
to include any changes in the `debian` dir.

Create the patch file:

```bash
$ git diff -- path/to/file1 path/to/file2 >debian/patches/my-changes.patch
```

Add the patchfile to the end of the series file:

```bash
$ echo my-changes.patch >> debian/patches/series
```

Check the patchfile to make sure it's correct, because the next command will
wipe out your changes:

```bash
$ git checkout path/to/file1 path/to/file2
```

### The patchfile header

The patch file must have [a DEP3 header](http://dep.debian.net/deps/dep3). The
basic structure is:

```text
Description: <short description, required>
 <long description that can span multiple lines, optional>
Author: <name and email of author, optional>
Origin: <upstream|backport|vendor|other>, <URL, required except if Author is present>
Bug: <URL to the upstream bug report if any, implies patch has been forwarded, optional>
Bug-<Vendor>: <URL to the vendor bug report if any, optional>
Forwarded: <URL|no|not-needed, useless if you have a Bug field, optional>
Applied-Upstream: <version|URL|commit, identifies patches merged upstream, optional>
Reviewed-by: <name and email of a reviewer, optional>
Last-Update: 2018-05-10 <YYYY-MM-DD, last update of the meta-information, optional>
---
This patch header follows DEP-3: http://dep.debian.net/deps/dep3/
```

* **Description**:
  While this can be multi-line, all subsequent lines after the first must be
  indented by a whitespace character, and empty lines must start with
  whitespace and a period ` .`.

* **Author**:
  Refers to code authorship, not patch authorship. Use the name of whoever
  wrote the original code in the patch.

* **Origin**: Where the code came from:

  * `upstream` = Points to code on the original software site or repo.
  * `backport` = You had to change the code to fit an Ubuntu requirement.
  * `vendor` = The code came from some vendor like Red Hat, SUSE, etc.
  * `other` = The code came from somewhere else.

* **Bug**:
  Bug report on the upstream site, if any.

* **Bug-[Vendor]**:
  Bug report on a vendor's site. We should have one for Ubuntu, but often
  there will also be reports from other vendors like Debian, Red Hat, etc. Add
  them if they're readily available, but you don't need to go hunting for them.

* **Forwarded**:
  Information about where you sent the patch if it hasn't been upstreamed. Only
  useful for vendor-specific patches.

* **Applied-Upstream**:
  If the patch has already been applied upstream, link to it. This is not
  needed if you have Origin upstream.

#### Check your changelog

You can use `dep3changelog` to verify the headers, as well as generate a
changelog entry.


#### Automatic header generation

You can use `quilt`` to automatically add a DEP-3 header:

```bash
$ quilt header -e --dep3 my-changes.patch
```

### Best practice to maintain more easily over a long time

These are not required, but can be rather helpful in packages with many
patches and long patch series. Yet on the other hand it isn't hurting
at all even in simple packages and so it can become muscle memory.

First - To be able to quickly follow patches back to the bug without opening
the file and reading the header. And furthermore to group sets of patches well
the following pattern for the file name emerged:

```
debian/patches/lp-<bugnumber>-<patchindex>-<description>.patch`
```

The one case where this naming scheme can be harmful is that you'd usually
not want to upstream this file name to Debian as the launchpad bug does
not mean much there.

There is yet another way that developers often need to travel, which is
from the changelog to the patch file. Sure that can be determined by
checking all the changed files in a commit, but it turns out to be rather
helpful to make the associated changelog entry related to a patch that was
added to follow the following pattern:

```
  - d/p/<patch-name>: <what was fixed> (LP: #<bugnumber>)
```

With both in place, even very complex packages are easy to navigate from
changelog to patches and from files in `debian/patches/` to the
associated bugs.

### Verify the patchfile

Use `quilt`` to apply the patches in order:

```bash
$ quilt push -a
...
Applying patch 70_some-check.patch
patching file conf/some-script

Applying patch fix-something.patch
patching file src/a_source_file.c
patching file src/another_source_file.c

Applying patch my-changes.patch
patching file path/to/file1
patching file path/to/file2

Now at patch my-changes.patch
```

Now revert the patches:

```bash
$ quilt pop -a
```

From here, `git status` should show the following:

```bash
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   debian/patches/series

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    .pc/
    debian/patches/my-changes.patch
```

`.pc` is the control directory for Quilt patches. Remove it manually before
committing.

> **Note**:
> At this stage only the patch itself (here `debian/patches/my-changes.patch`)
> is committed.

The updated changelog (and maybe an updated control file, in case
`update-maintainer` needs to be run) will be committed later (all separately).
This simplifies a later rebase.

#### About update-maintainer

`update-maintainer` is a script that checks the `Maintainer` field of a
`debian/control` file and (if needed):

* Sets the `Maintainer` field to:
  `Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>`
* Adds a property `XSBC-Original-Maintainer` with the original value of the
  `Maintainer` field.

If a debian package introduces a change that does not work on Ubuntu, we need
to change the package to fix the bug. Now when someone has a problem with the
Ubuntu package they could contact the Debian maintainer and report the bug to
them, although they have nothing to do with our modification. Therefore we
change the value of the `Maintainer` field.


## Further reading

* https://wiki.debian.org/UsingQuilt
* https://wiki.ubuntu.com/DebianMaintainerField
