# Package Building

There are multiple ways to build a package, each with advantages and
disadvantages. Which one you choose will depend on your circumstances.
The most common path is building the source with `dpkg-buildpackage`
followed by either `sbuild` for local builds or `dput` to upload
to the Ubuntu Archive or a PPA.

## Download the orig tarball

For almost anything you want to do, other than inspecting code, you'll
need to download the orig tarball. If you are in a `git-ubuntu` based
source tree, you can utilize its `export-orig` helper:

```bash
$ git ubuntu export-orig
```

It will try to check the latest changelog stanza and use the `pristine-tar`
branches to generate the tarball.
This might fail if you e.g. go ahead or any other reason causing the tarball
to not yet be imported into `git-ubuntu`.
The common fallbacks would be to either download the tarball directly
from Launchpad or Debian via `pull-lp-source` or `pull-debian-source`.
If not even there the next step is to download the right one from upstream
directly, if `d/watch` is defined potentially via `uscan --verbose` or
manually on the upstream repository.

## Build source packages with `dpkg-buildpackage`

While you might want an even more clean environment, e.g. in a LXD container,
for most cases it will be sufficient to use `dpkg-buildpackage` to build the
source from within the package repository:

```bash
$ dpkg-buildpackage -S -I -i -nc -d
```

Let us list the arguments used here

* -S = --build=source - we want to build a source (.dsc, .changes)
* -I = --tar-ignore - for the created tarball, without arguments it will add default exclude options to filter out unwanted files
* -i = --diff-ignore - for the diff, without arguments it will add default exclude options to filter out unwanted files
* -nc = --no-pre-clean - do not clean the tree before building
* -d = --no-check-builddeps - Do not check build dependencies and conflicts, ok as we only build the source

When based on git-ubuntu branch and going for an upload to the Ubuntu Archive
please also add the output of `git ubuntu prepare-upload args` which will add
the arguments to allow `git-ubuntu` to properly reference back to this on
importing the new version - thereby retaining the rich history of your branch.

Therefore the final command from within the package repository for an
upload to the Archive is:

```bash
$ dpkg-buildpackage -S -I -i -nc -d $(git ubuntu prepare-upload args)
```


### Sign the `changes` file

In order for a source package to be accepted by Launchpad, it must be signed.
If your GPG keys are properly installed, `dpkg-buildpackage` may automatically
sign for you. If not, you can sign the source package manually with `debsign`
on the changes file:

```bash
$ debsign ../<filename>_source.changes
```

> **Tip**:
> To avoid the chore of determining the changes file, you can create a script
> that extracts the info from the debian/changelog:
> ```bash
> $ source_package=$(dpkg-parsechangelog -n1 --show-field Source)
> $ version=$(dpkg-parsechangelog -n1 --show-field Version)
> $ debsign "../${source_package}_${version}_source.changes
> ```

## Build binary packages via PPA

Arguably the easiest way to build your package is to let Launchpad do it for
you. "Personal Package Archives" (PPAs) are encapsulated build spaces in
Launchpad that are owned and controlled by you. Packages you sign and upload
to a PPA will be built with the same machinery as official Ubuntu packages, so
they're also a great way to verify your work before formally submitting it to
Ubuntu.

Once built, the packages in a PPA are publically available, allowing you to
share them with bug reporters for testing fixes, and to reviewers of your merge
proposals who can readily use them for testing.

The downsides to PPAs are:

1. They share resources with other Launchpad build processes, so during busy
   times it can take a while to come up in the build queue.
2. It's less hands-on than a local build so can be hard to use for highly
   iterative workflows like sorting out dependency issues or git-bisecting
   build failures (if you expect that you'll need to debug the build, like
   going into the environment to modify and retry, a local build is
   recommended - see below).
3. The PPAs are picky about version strings.


### Set the version string

For the PPA, we should change the version in the changelog to one that's lower
than the official version we plan to release. For example:

```diff
-postfix (3.3.0-1ubuntu0.1) bionic; urgency=medium
+postfix (3.3.0-1ubuntu0.1~bionic1) bionic; urgency=medium
```

Since the tilde `~` character sorts lower than everything else in Launchpad, we
can append `~<string>1` to the version string in `debian/changelog`. See more
details about the sorting algorithm here:
[`deb-version(7)`](https://manpages.ubuntu.com/manpages/deb-version.7.html))

Having a numeric digit in this suffix is important because once Launchpad has
accepted your upload, it won't accept another one with the same version number
(nor any earlier version number). So if you need to fix something in your
upload -- even just copyediting your changelog entry -- you need an
incrementally higher version number. Incrementing the suffix allows you to do
this without needing to modify the official version number.

For the text, you can use any string as desired; often people use their
username, or just 'ppa'. An advantage of using the release codename, however,
is that if you later intend to port the same package to multiple releases (e.g.
you're doing an MRE, or an SRU that has the same official version in multiple
Ubuntu releases), using the codename ensures each has a unique version (for
Launchpad) while also indicating which package to use for which Ubuntu release
(for users).

As an aside, you'll sometimes run across the suffix style `"~18.04.1"` which is
adopted for reasons similar to the codename, and tends to be a preferred choice
in semi-official PPAs such as ones used for official customer deliveries or
formally maintained backports to the wider userbase. To avoid confusion, the
`'~<codename>N'` style may be better for the one-off testing-oriented PPAs
being discussed here.


### Modify the version for PPA

The command below can be used to modify the version for PPA usage:

```bash
$ codename="bionic"
$ dch -l "~${codename}1" --distribution "${codename}" "Build for PPA"
```

If a PPA is used to build the package and the version string was changed as
described above, make sure to rebuild and resign the source package:

```bash
$ dpkg-buildpackage -S -I -i -nc -d
```


### Create the PPA archive

First, install `ppa-dev-tools` from the snap store:

```bash
$ sudo snap install ppa-dev-tools
```

Next, follow the directions in `INSTALL.md` to install prereqs and to install
the tool. Then, to use it:

```bash
$ ppa create <ppa-name>
```

This creates the PPA for you, and enables all available build architectures.
The first time you run it, it'll ask for authentication via the web.

You can use whatever you want for your `ppa-name`, so long as it's unique in
your own namespace. For consistency, you may want to use a standard naming
style, such as:

```bash
$ ppa_name="<package>-<type>-<lpbug>-<desc>"
# or
$ ppa_name="lp-<lpbug>-<package>-<desc>"
```

It isn't important which you use, just be consistent.
What is important though is that this has to be all lower case.
Many of us even keep the associated git-ubuntu branch names consistent with the PPA names.

So for example, you might have PPAs named `apache2-sru-lp12345678`, `clamav-fix-lp1920217`, and `clamav-fix-lp1920217-alternative`.


### (Optional) Create the PPA archive via web

Alternatively, you can create PPAs directly via Launchpad's web interface.

Go to your launchpad page (https://launchpad.net/~your-username) and click
"Create a new PPA". Give it a name such that you'll remember what it's about
in a few months' time. A useful form is `package-type-lpbug-description`:

For example:

* **URL:** `postfix-sru-lp1753470-segfault`
* **Display name:** `postfix-fix-lp1753470-segfault`
* **Description:** `(leave it empty)`

Now click "Activate".

It is also helpful to enable all architectures to ensure no build regressions
were introduced. Do so by clicking on `Change Details` in the newly-created
PPA page, and then selecting the other architectures.


### Upload the source package

```bash
$ dput ppa:kstenerud/postfix-sru-lp1753470-segfault ../postfix_3.3.0-1ubuntu0.1~bionic1_source.changes
```

When it finishes, you should be able to see it, e.g.:
https://launchpad.net/~kstenerud/+archive/ubuntu/postfix-postconf-segfault-1753470/+packages

> **Note**:
> You must wait for the package to build server-side before you can use the
> PPA to install packages. This might take anywhere from a few minutes to a
> few hours depending on how busy things are!
> 
> It'll first build the binaries for each architecture, then publish the
> source and binary packages to be publically downloadable.

#### Check progress with `ppa`

You can use the `ppa` tool to poll launchpad for progress status:

```bash
$ ppa wait ppa:kstenerud/postfix-sru-lp1753470-segfault
```

It will exit with `0` once the PPA packages have fully built.

Launchpad also sends "status updates" notification mails, so monitor your
inbox.


## Build binary packages locally with `sbuild`

Assuming you [have configured `sbuild` properly](Setup-md#sbuild),
you can use it to build the binary package. Usually you'd want to specify
the following arguments:

* set `DEB_BUILD_OPTIONS`, if nothing else then to gain some speed via `parallel=<number>`
* -A = --arch-all - specify that you also want arch-all
* -d = --dist - set the distribution for which you'd use a named schroot created in [sbuild](Setup-md#sbuild)
* finally the `.dsc` file

Before doing so it is also helpful to update the schroot to the latest
state via `sudo sbuild-update -udcar <name of schroot>`

Which overall means that you'd go `cd ..` one directory out of the source tree,
to the directory where the `.dsc` got created when building the source and run:

```bash
$ sudo sbuild-update -udcar <schroot>
$ DEB_BUILD_OPTIONS="<options>" sbuild -Ad<schroot> <dsc file>
```

And more concrete as an example building qemu in noble with a concurrency of 3:
```bash
$ sudo sbuild-update -udcar noble-proposed-amd64
$ DEB_BUILD_OPTIONS="parallel=3" sbuild -Adnoble-proposed-amd64 qemu_8.2.2+ds-0ubuntu2.dsc
```

> **Note**:
> From Ubuntu 23.04 (Lunar Lobster), the `series-proposed` suite is [disabled by default](https://wiki.ubuntu.com/Testing/EnableProposed) via [APT Preferences](https://wiki.debian.org/AptConfiguration#apt_preferences_.28APT_pinning.29). This affects schroots created with `sbuild-launchpad-chroot`, so proposed packages [may not be used in the build process](https://bugs.launchpad.net/ubuntu/+source/sbuild-launchpad-chroot/+bug/1996205) in this case.

For more information, see:

* https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/Setup.md#software-sbuild
* https://wiki.debian.org/sbuild
