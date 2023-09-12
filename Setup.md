# Setting up for Ubuntu development

The following is a short guide to getting set up for Ubuntu development.

## Prerequisites

You must have a Launchpad ID. To get an ID:

* Go to https://launchpad.net/
* Click `Log in / Register`


## Install software

```bash
$ sudo apt update && \
sudo apt dist-upgrade -y && \
sudo apt install -y \
    apt-cacher-ng \
    autopkgtest \
    build-essential \
    debconf-utils \
    debmake \
    devscripts \
    dh-make \
    dpkg-dev \
    git-buildpackage \
    libvirt-clients \
    libvirt-daemon \
    libvirt-daemon-system \
    net-tools \
    ovmf \
    pastebinit \
    piuparts \
    pkg-config \
    qemu-system \
    quilt \
    sbuild \
    snapcraft \
    ubuntu-dev-tools \
    uvtool \
    virtinst && \
sudo snap install lxd && \
sudo snap install --classic ustriage && \
sudo snap install --classic --edge git-ubuntu && \
sudo snap install --classic --beta multipass
```

## Configure software

### GnuPG

[GnuPG](https://gnupg.org/) is an encryption tool that helps manage your
encryption keys. 

* Install and set up GPG normally.
* List the keys and make sure you associate the email you want to use for
  publishing.

  ```bash
  $ gpg --list-secret-key
  /home/karl/.gnupg/pubring.kbx
  -----------------------------
  sec   rsa4096 2018-08-15 [SC]
        7C177302572849D84A5048349E9C224744EF2A5A
  uid           [ultimate] Karl Stenerud <kstenerud@gmail.com>
  ssb   rsa4096 2018-08-15 [E]
  ```

  * In this case, my Canonical address isn't in there, so I need to add it:

    ```bash
    $ gpg --edit-key 7C177302572849D84A5048349E9C224744EF2A5A
    ...
    gpg> adduid
    Real name: Karl Stenerud
    Email address: karl.stenerud@canonical.com
    Comment: 
    You selected this USER-ID:
        "Karl Stenerud <karl.stenerud@canonical.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
    ```

* Then save and quit:
  ```bash
  gpg> save
  ```

* And push to the keyserver:
  ```bash
  $ gpg --keyserver keyserver.ubuntu.com --send-keys 7C177302572849D84A5048349E9C224744EF2A5A
  ```

Make sure you note the key strength of your GPG key. In this case its rsa4096,
but if you have an older key it may be a weaker 2048-bit or 1024-bit key. If
so, create a new 4096-bit one and deprecate the old one in Launchpad, GitHub,
etc.


### Git

Installing `git-ubuntu` will modify your `.gitconfig`. Make sure it got your
Launchpad username correct:

```text
[gitubuntu]
    lpuser = your-launchpad-username
```

You must also ensure that the `[user]` section has your name and email:

```text
[user]
    name = Your Full Name
    email = your@email.com
```

You may also want to add the following to your `.gitconfig`:

```text
[log]
    decorate = short
[commit]
    verbose = true
[merge]
    summary = true
    stat = true
[core]
    whitespace = trailing-space,space-before-tab

[diff "ruby"]
    funcname = "^ *\\(\\(def\\) .*\\)"
[diff "image"]
    textconv = identify

[url "git+ssh://my_lp_username@git.launchpad.net/"]
    insteadof = lp:
```

### Quilt

[Quilt](https://savannah.nongnu.org/projects/quilt) is a CLI used to manage
patch stacks. It can take any number of patches and condense them into a
single patch.

A working `.quiltrc`:

```bash
d=. ; while [ ! -d $d/debian -a `readlink -e $d` != / ]; do d=$d/..; done
if [ -d $d/debian ] && [ -z $QUILT_PATCHES ]; then
    # if in Debian packaging tree with unset $QUILT_PATCHES
    QUILT_PATCHES="debian/patches"
    QUILT_PATCH_OPTS="--reject-format=unified"
    QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
    QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
    QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
    if ! [ -d $d/debian/patches ]; then mkdir $d/debian/patches; fi
fi
```

This configures Quilt for use with Debian packages, with default settings that
conform to standard Debian practices.


### DPut

[DPut](https://packages.debian.org/sid/dput) is the Debian Package Upload Tool.
It's used to upload a software package to the Ubuntu repository, or to a
personal package archive (PPA).

A working `.dput.cf`:

```text
[DEFAULT]
default_host_main = unspecified

[unspecified]
fqdn = SPECIFY.A.TARGET
incoming = /

[ppa]
fqdn            = ppa.launchpad.net
method          = ftp
incoming        = ~%(ppa)s/ubuntu
```

This configures `dput`` for safety, such that if you accidentally forget to
specify a destination, it'll default to doing nothing.


### SBuild

[SBuild](https://wiki.debian.org/sbuild) is a wrapper script around `schroot`.

In these examples, replace `my_user` with your own username.

Make mount points:

```bash
$ mkdir -p ~/schroot/build
$ mkdir -p ~/schroot/logs
```

Set up scratch dir:

```bash
$ mkdir -p ~/schroot/scratch
$ echo "/home/my_user/schroot/scratch  /scratch          none  rw,bind  0  0" \
 >> /etc/schroot/sbuild/fstab
```

Optionally, you can mount your homedir inside the container:

```bash
$ echo "/home/my_user  /home/my_user          none  rw,bind  0  0" \
 >> /etc/schroot/sbuild/fstab
```

In the following template (`.sbuildrc`), replace the following:
 * `$maintainer_name = 'Your Full Name <your@email.com>';`
 * `$build_dir = '/home/my_user/schroot/build';`
 * `$log_dir = "/home/my_user/schroot/logs";`

Template:

```bash
# Name to use as override in .changes files for the Maintainer: field
# (optional; only uncomment if needed).
# $maintainer_name = 'Your Full Name <your@email.com>';

# Default distribution to build.
$distribution = "focal";
# Build arch-all by default.
$build_arch_all = 1;

# When to purge the build directory afterwards; possible values are 'never',
# 'successful', and 'always'.  'always' is the default. It can be helpful
# to preserve failing builds for debugging purposes.  Switch these comments
# if you want to preserve even successful builds, and then use
# 'schroot -e --all-sessions' to clean them up manually.
$purge_build_directory = 'successful';
$purge_session = 'successful';
$purge_build_deps = 'successful';
# $purge_build_directory = 'never';
# $purge_session = 'never';
# $purge_build_deps = 'never';

# Directory for chroot symlinks and sbuild logs.  Defaults to the
# current directory if unspecified.
$build_dir = '/home/my_user/schroot/build';

# Directory for writing build logs to
$log_dir = "/home/my_user/schroot/logs";

# don't remove this, Perl needs it:
1;
```

A working `.mk-sbuild.rc`:

```bash
SCHROOT_CONF_SUFFIX="source-root-users=root,sbuild,admin
source-root-groups=root,sbuild,admin
preserve-environment=true"
# you will want to undo the below for stable releases, read `man mk-sbuild` for details
# during the development cycle, these pockets are not used, but will contain important
# updates after each release of Ubuntu
SKIP_UPDATES="1"
SKIP_PROPOSED="1"
# if you have e.g. apt-cacher-ng around
DEBOOTSTRAP_PROXY=http://127.0.0.1:3142/
```

Configure GnuPG for SBuild:

```bash
$ sbuild-update --keygen
```

Create an x86\_64 schroot:

    $ mk-sbuild focal --arch=amd64

> **Note**: 
> For more info, see the [Ubuntu wiki page on SBuild](https://wiki.ubuntu.com/SimpleSbuild)


### LXD

[LXD](https://documentation.ubuntu.com/lxd/en/latest/) is a powerful container
system similar in concept to Docker and other container software.

Install and set up LXD using the
[standard installation](https://documentation.ubuntu.com/lxd/en/latest/installing/)
directions.

Create some helper aliases for common LXD tasks:

```bash
$ lxc alias add ls 'list -c ns4,user.comment:comment'

$ lxc alias add login 'exec @ARGS@ \
--mode interactive -- bash -xac $@my_user - exec /bin/login -p -f '
```

Note that the trailing space after the `-f` is important. Replace 'my_user'
with 'ubuntu' or whatever username you use in your containers.

> **Note**:
> For more info, see the
> [LXD documentation]((https://documentation.ubuntu.com/lxd/en/latest/))


## Caching packages

Downloading packages can be a bottleneck, so it helps to set up a local cache:

```bash
$ echo 'Acquire::http::Proxy "http://127.0.0.1:3142";' \
| sudo tee /etc/apt/apt.conf.d/01acng
```

## Configuration

### Groups

Your user should be a member of the following groups:

* adm
* libvirt
* lxd
* sbuild
* sudo

Ensure you have installed the packages listed above, which will be the trigger
to create most of these groups. For group membership to be activated one
usually needs to re-login. Then, one can double check group membership via:

```bash
$ groups my_user
my_user : my_user adm cdrom sudo dip plugdev lpadmin sambashare \
 libvirt sbuild lxd
```

If any of the following groups is missing for your user you can fix it via
`adduser`, like this:

```bash
$ sudo adduser my_user lxd
$ sudo adduser my_user sbuild
$ sudo adduser my_user libvirt
```


### Profile

Your `.profile` should include entries for `DEBFULLNAME` and `DEBEMAIL`:

```text
export DEBFULLNAME="Your Full Name"
export DEBEMAIL=your@email.com
```

You can also set the `DEBSIGN` variables:

```text
export DEBSIGN_PROGRAM="/usr/bin/gpg2"
export DEBSIGN_KEYID="0xMYKEYHASH"
```

A fix for "clear-sign failed: Inappropriate ioctl for device":

```bash
$ export GPG_TTY=$(tty)
```

If you're operating from a GUI, this can be useful:

```bash
$ eval `dbus-launch --sh-syntax`
```