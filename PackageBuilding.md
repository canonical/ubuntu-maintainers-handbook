Package Building
================

There are multiple ways to build a package, each with advantages and disadvantages. Which one you choose will depend on your circumstances.

Note: git-ubuntu no longer supports the build argument (neither for source nor for binary builds).


Downloading the orig tarball (optional)
---------------------------------------

If you intend to use more manual methods like `sbuild` or `dpkg-buildpackage` directly, you will probably have to download the orig tarball first.  You can do so by using:

    $ git ubuntu export-orig

It will try to use the `pristine-tar` branch to generate the tarball (and will likely fail), and then it will fallback to downloading the tarball directly from Launchpad.  When it finishes, you should be able to see a link to the orig tarball at `../`.

Building Source Packages
------------------------

### Using dpkg-buildpackage

This method will directly install any dependencies it needs to build, so it's recommended to create an LXD container to do the build. Replace `focal` with the container image you wish to use.

From within the package repository:

    $ lxc launch images:ubuntu/focal builder &&
      sleep 5 &&
      lxc exec builder -- mkdir -p /root/build/package &&
      tar cf - . | lxc exec builder -- tar xf - -C /root/build/package &&
      lxc exec builder -- sh -c 'apt update &&
      apt dist-upgrade -y &&
      apt install -y ubuntu-dev-tools &&
      cd /root/build &&
      pull-debian-source -d $(grep "Source: " package/debian/control | sed "s/Source: \(.*\)/\1/g") $(grep "unstable; urgency=" package/debian/changelog |grep -v ubuntu|head -1|sed "s/.*(\(.*\)).*/\1/g") &&
      cd /root/build/package &&
      apt build-dep -y ./ &&
      dpkg-buildpackage -S' &&
      lxc exec builder -- tar cf - --exclude=package -C /root/build . | tar xf - -C .. &&
    $ lxc delete -f builder

Even though the recommended way to build a source package is to use a pristine environment inside an LXD container, you can also use `dpkg-buildpackage`'s `--no-check-builddeps` option and build the source package locally:

    $ dpkg-buildpackage -S -I -i -nc --no-check-builddeps


### Signing the Changes File

In order for a source package to be accepted by Launchpad, it must be signed. Depending on if your GPG keys are properly installed, `dpkg-buildpackage` may automatically sign for you.  Or you can sign it manually with `debsign` on the changes file:

    $ debsign ../<filename>_source.changes

#### Protip

To save the chore of determining the changes file, you could create a script that extracts the info from the debian/changelog:

    $ source_package=$(dpkg-parsechangelog -n1 --show-field Source)
    $ version=$(dpkg-parsechangelog -n1 --show-field Version)
    $ debsign "../${source_package}_${version}_source.changes


Building Binary Packages via PPA
--------------------------------

Arguably the easiest way to build your package is to let Launchpad do it for you.  "Personal Package Archives", or "PPAs" are encapsulated build spaces in Launchpad but owned and controlled by you.  Packages you sign and upload to a PPA will be built with the same machinery as the official Ubuntu packages, so they're also a great way to verify your work before formally submitting it to Ubuntu.

Once built, the packages in the PPA are publically available, allowing you to share them with bug reporters for testing fixes, and to reviewers of your merge proposals who can readily use them for testing.

The downsides to PPAs are first, they share resources with other Launchpad build processes so during busy times it can take a while to come up in the build queue; second, it's less hands-on than a local build so can be hard to use for highly iterative workflows like sorting out dependency issues or git-bisecting build failures (if you expect that you'll need to debug the build, like going into the environment to modify and retry, a local build is recommended - see below); and third, the PPAs are picky about version strings.


#### Set the Version String

For the PPA, we need to change the version in the changelog that's lower than the official version we plan to release. Since the tilde `~` character sorts lower than everything else in launchpad, we can simply append `~<string>1` to the version string in `debian/changelog`. For example:

    -postfix (3.3.0-1ubuntu0.1) bionic; urgency=medium
    +postfix (3.3.0-1ubuntu0.1~bionic1) bionic; urgency=medium

Having a numeric digit in this suffix is important because once Launchpad has accepted your upload, it won't accept another one with that same version number (nor any earlier version number).  So if you need to fix something in your upload, even just copyediting your changelog entry, you need an incrementally higher version number.  Incrementing the suffix allows you to do this without needing to modify the official version number.

For the text, you can use any string as desired; often people use their username, or just 'ppa'.  An advantage of using the release codename, however, is if you later intend to port the same package to multiple releases (e.g. you're doing an MRE, or an SRU that has the same official version in multiple Ubuntu releases), using the codename ensures each has a unique version (for Launchpad) while helping indicate which package to use for which Ubuntu release (for users).

As an aside, you'll sometimes run across the suffix style "~18.04.1" which is adopted for reasons similar to the codename, and tends to be a preferred choice in semi-official PPAs such as ones used for official customer deliveries or formally maintained backports to the wider userbase.  To avoid confusion, the '~<codename>N' style may be better for the one-off testing-oriented PPAs being discussed here.


#### Protip

Note, the command below can be used to modify the version for PPA usage:

    $ codename="bionic"
    $ dch -l "~${codename}1" --distribution "${codename}" "Build for PPA"

If a PPA is used to build the package and the version string was changed as described above, make sure to rebuild and resign the source package:

    $ dpkg-buildpackage -S -I -i -nc -d


#### Create the PPA archive

First, install 'ppa-dev-tools' from its git repository:

    $ git clone git://git.launchpad.net/ppa-dev-tools
    $ cd ppa-dev-tools

Next, follow the directions in INSTALL.md to install prereqs and install the tool.

Then to use it:

    $ ppa create <ppa-name>

(The first time you run this it'll ask for authentication via the web.)

This creates the PPA for you, and enables all available build architectures.

You can use whatever you want for your `ppa-name`, so long as its unique in your own namespace.  For consistency, you may want to use a standard naming style, such as:

    $ ppa_name="<package>-<type>-<lpbug>-<desc>"

So for example, you might have PPAs named `apache2-sru-lp12345678`, `clamav-fix-lp1920217`, `clamav-fix-lp1920217-alternative`.


#### (Optional) Create the PPA archive (via web)

Alternatively, you can create PPAs directly via Launchpad's web interface.

Go to your launchpad page (https://launchpad.net/~your-username) and click "Create a new PPA". Give it a name such that you'll remember what it's about in a few months. A useful form is `package-type-lpbug-description`:

For example:

 * **URL:** postfix-sru-lp1753470-segfault
 * **Display name:** postfix-fix-lp1753470-segfault
 * **Description:** (leave it empty)

Now click "Activate".

It is also helpful to enable all architectures to ensure no build regressions were introduced. Do so by clicking on `Change Details` in the newly created PPA page, and then selecting the other architectures.


#### Upload the source package

    $ dput ppa:kstenerud/postfix-sru-lp1753470-segfault ../postfix_3.3.0-1ubuntu0.1~bionic1_source.changes

When it finishes, you should be able to see it e.g. https://launchpad.net/~kstenerud/+archive/ubuntu/postfix-postconf-segfault-1753470/+packages

Note: You must wait for the package to build server-side before you can use the PPA to install packages. This might take anywhere from a few minutes to a few hours depending on how busy things are!  It'll first build the binaries for each architecture, then publish the source and binary packages to be publically downloadable.

#### Protip

You can use the `ppa` tool to poll launchpad for progress status:

    $ ppa wait ppa:kstenerud/postfix-sru-lp1753470-segfault

It will exit with 0 once the PPA packages have fully built.

Launchpad also sends status updates notification mails, so monitor your inbox.


Building Binary Packages Locally
--------------------------------

### Using sbuild

Assuming you have configured `sbuild` properly, you can use it to build the binary package:

    $ sbuild

Because of https://bugs.launchpad.net/launchpad/+bug/1699763, it is a good idea to disable the inclusion of `.buildinfo` files in the `*_source.changes` file:

    $ sbuild --debbuildopts='--buildinfo-option=-O'

For more information see:

  * https://packaging.ubuntu.com/html/setting-up-sbuild.html
  * https://wiki.debian.org/sbuild
