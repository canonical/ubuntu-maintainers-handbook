Setting up for Ubuntu Development
=================================

The following is a short guide to getting set up for Ubuntu development.

Prerequisites
-------------

You must have a Launchpad ID. To get an ID:

 * Go to https://launchpad.net/
 * Click `Log in / Register`


Software
--------

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
        qemu \
        qemu-kvm \
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


Caching packages
----------------

Package downloading can be a bottleneck, so it helps to set up a local cache:

    $ echo 'Acquire::http::Proxy "http://127.0.0.1:3142";' | sudo tee /etc/apt/apt.conf.d/01acng


Configuration
-------------

### Groups

Your user should be a member of the following groups:

 * adm
 * kvm
 * libvirt
 * lxd
 * sbuild
 * sudo

    $ groups my_user
    my_user : my_user root adm cdrom sudo dip plugdev lpadmin

    $ sudo groupadd lxc
    $ sudo groupadd sbuild
    $ sudo groupadd libvirt


### Profile

Your `.profile` should include entries for `DEBFULLNAME` and `DEBEMAIL`:

    export DEBFULLNAME="Your Full Name"
    export DEBEMAIL=your@email.com

You can also set the `DEBSIGN` variables:

    export DEBSIGN_PROGRAM="/usr/bin/gpg2"
    export DEBSIGN_KEYID="0xMYKEYHASH"

A fix for "clear-sign failed: Inappropriate ioctl for device":

    $ export GPG_TTY=$(tty)

If you're operating from a GUI, this can be useful:

    $ eval `dbus-launch --sh-syntax`


### Software: GnuPG

Install and set up GPG normally. List the keys and make sure the email you want for publishing is associated.

    $ gpg --list-secret-key
    /home/karl/.gnupg/pubring.kbx
    -----------------------------
    sec   rsa4096 2018-08-15 [SC]
          7C177302572849D84A5048349E9C224744EF2A5A
    uid           [ultimate] Karl Stenerud <kstenerud@gmail.com>
    ssb   rsa4096 2018-08-15 [E]

In this case, my canonical address isn't in there, so I need to add it:

    $ gpg --edit-key 7C177302572849D84A5048349E9C224744EF2A5A
    ...
    gpg> adduid
    Real name: Karl Stenerud
    Email address: karl.stenerud@canonical.com
    Comment: 
    You selected this USER-ID:
        "Karl Stenerud <karl.stenerud@canonical.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

Then save and quit:

    gpg> save

And push to the keyserver:

    $ gpg --keyserver keyserver.ubuntu.com --send-keys 7C177302572849D84A5048349E9C224744EF2A5A

Make sure to note the key strength of your gpg key.  In this case its rsa4096, but if you have an older key it may be a weaker 2048-bit or 1024-bit key.  If so, create a new 4096-bit one and deprecate the old one in Launchpad, github, etc.


### Software: Git

Installing git-ubuntu will modify your `.gitconfig`. Make sure it got your launchpad username correct:

    [gitubuntu]
        lpuser = your-launchpad-username

You must also ensure that the `[user]` section has your name and email:

    [user]
        name = Your Full Name
        email = your@email.com

You may also want to add the following to your .gitconfig:

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


### Software: Quilt

Quilt is a CLI used to manage patch stacks.

A working `.quiltrc`:

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

This configures quilt for use with Debian packages, with default settings that conform to standard Debian practices.


### Software: DPut

dput is used to upload a software package to the Ubuntu repository, or to a personal package archive (PPA).

A working `.dput.cf`:

    [DEFAULT]
    default_host_main = unspecified

    [unspecified]
    fqdn = SPECIFY.A.TARGET
    incoming = /

    [ppa]
    fqdn            = ppa.launchpad.net
    method          = ftp
    incoming        = ~%(ppa)s/ubuntu

This configures dput for safety, such that if you accidentally forget to specify a destination, it'll default to doing nothing.


### Software: SBuild

Assuming user `my_user`. Replace with your username where appropriate.

Make mount points:

    $ mkdir -p ~/schroot/build
    $ mkdir -p ~/schroot/logs

Set up scratch dir (replace `my_user` user with your own):

    $ mkdir -p ~/schroot/scratch
    $ echo "/home/my_user/schroot/scratch  /scratch          none  rw,bind  0  0" >> /etc/schroot/sbuild/fstab

Optionally, you can mount your homedir inside the container:

    $ echo "/home/my_user  /home/my_user          none  rw,bind  0  0" >> /etc/schroot/sbuild/fstab


A template `.sbuildrc`:

Replace the following:
 * `$maintainer_name='Your Full Name <your@email.com>';`
 * `$build_dir='/home/my_user/schroot/build';`
 * `$log_dir="/home/my_user/schroot/logs";`


Template:

    # Name to use as override in .changes files for the Maintainer: field
    # (mandatory, no default!).
    $maintainer_name='Your Full Name <your@email.com>';

    # Default distribution to build.
    $distribution = "bionic";
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
    $build_dir='/home/my_user/schroot/build';

    # Directory for writing build logs to
    $log_dir="/home/my_user/schroot/logs";

    # don't remove this, Perl needs it:
    1;

A working `.mk-sbuild.rc`:

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

Configure GnuPG for sbuild:

    $ sbuild-update --keygen

More Info: https://wiki.ubuntu.com/SimpleSbuild


### Software: LXD

LXD is a powerful container system similar in concept to Dropbox and other container software.

Install and setup LXD using the standard installation directions.

Create some helper aliases for common LXD tasks:

    $ lxc alias add ls 'list -c ns4,user.comment:comment'

    $ lxc alias add login 'exec @ARGS@ --mode interactive -- bash -xac $@my_user - exec /bin/login -p -f '

(The trailing space after the -f is important).  Replace 'my_user' with 'ubuntu' or whatever username you use in your containers.


More Info:  https://help.ubuntu.com/lts/serverguide/lxd.html
