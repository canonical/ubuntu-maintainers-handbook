Package Building
================

There are multiple ways to build a package, each with advantages and disadvantages. Which one you choose will depend on your circumstances.


Downloading the orig tarball (optional)
---------------------------------------

If you intend to use more manual methods like `sbuild` or `dpkg-buildpackage` directly, you will probably have to download the orig tarball first.  You can do so by using:

    $ git ubuntu export-orig

It will try to use the `pristine-tar` branch to generate the tarball (and will likely fail), and then it will fallback to downloading the tarball directly from Launchpad.  When it finishes, you should be able to see a link to the orig tarball at `../`.


Building Source Packages
------------------------

### Using git-ubuntu

This is by far the easiest method:

    $ git ubuntu build --source -v --sign

Git ubuntu will automatically try to detect which Ubuntu release the build needs, based on the package's changelog file, but you can always specify an image directly, like this:

    $ git ubuntu build --source -v --sign --lxd-image ubuntu-daily:bionic

This will download the LXD image if needed, start a container, build the packages, copy them to `../` and shut down.


### Using dpkg-buildpackage

This method will directly install any dependencies it needs to build, so it's recommended to create an LXD container to do the build. Replace `bionic` with the container image you wish to use.

From within the package repository:

    $ lxc launch ubuntu-daily:bionic builder &&
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

    $ dpkg-buildpackage -S -d


### Signing the Changes File

In order for a source package to be accepted by Launchpad, it must be signed. You can build the sources using `git-ubuntu` with the `--sign` flag, or you can sign it manually with `debsign`:

    debsign $(grep "Source: " debian/control | sed "s/Source: \(.*\)/\1/g")_$(grep "urgency=" debian/changelog|head -1|sed "s/.*(\(.*\)).*/\1/g")_source.changes



Building Binary Packages
------------------------

### Using git-ubuntu

This is by far the easiest method, and will build on your local machine.

From within the package repository:

    $ git ubuntu build -v

Other flags are similar to `git ubuntu build --source`.

### Using sbuild

Assuming you have configured `sbuild` properly, you can use it to build the binary package:

    $ sbuild

Because of https://bugs.launchpad.net/launchpad/+bug/1699763, it is a good idea to disable the inclusion of `.buildinfo` files in the `*_source.changes` file:

    $ sbuild --debbuildopts='--buildinfo-option=-O'


### As a PPA in Launchpad

Launchpad can build binary packages from your signed source packages and store them in PPA archives. This is especially useful because reviewers of your merge proposal will have a ready binary to test from. The disadvantage is that during busy times, it can take awhile for your package to come up in the build queue.


#### Set the Version String

For the PPA, we need to change the version in the changelog that's lower than the version we plan to release. Since the tilde `~` character sorts lower than everything else in launchpad, we can simply append `~ppa1` to the version string in `debian/changelog`. For example:

    -postfix (3.3.0-1ubuntu0.1) bionic; urgency=medium
    +postfix (3.3.0-1ubuntu0.1~ppa1) bionic; urgency=medium

Note: If you're using `git-ubuntu` to build the source package, you must first create a commit with the changed version string using a dummy commit message like "ppa1", but **do not git push this commit!**

#### Create the PPA archive

Go to your launchpad page (https://launchpad.net/~your-username) and click "Create a new PPA". Give it a name such that you'll remember what it's about in a few months. A useful form is `package-type-lpbug-description`:

For example:

 * **URL:** postfix-sru-lp1753470-segfault
 * **Display name:** postfix-fix-lp1753470-segfault
 * **Description:** (leave it empty)

Now click "Activate".

It is also helpful to enable all architectures to ensure no build regressions were introduced. Do so by clicking on `Change Details` in the newly created PPA page, and then selecting the other architectures.

#### Upload the source package

    $ dput ppa:kstenerud/postfix-sru-lp1753470-segfault ../bionic-postfix_3.3.0-1ubuntu0.1~ppa1_source.changes

When it finishes, you should be able to see it e.g. https://launchpad.net/~kstenerud/+archive/ubuntu/postfix-sru-lp1753470-segfault/+packages

Note: You must wait for the package to build server-side before you can use the PPA to install packages. This might take time depending on how busy things are!
