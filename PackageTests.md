# Running package tests

Packages will have their own tests under `debian/tests`. We need to run those
to ensure there are no regressions.

We can have Launchpad do it against a PPA, or use an LXC container, or a VM to
run the autopkgtests locally. Each approach has its benefits, so they're all
worth learning, but the first option (PPA-based testing) produces results most
similar to what occurs in the archive itself, so we'll start there.


## 1) PPA-based autopkgtest testing

First, if you haven't already, install `ppa-dev-tools`:

```bash
$ sudo snap install ppa-dev-tools
$ ppa --help
usage: ppa [-h] [-C CONFIG_FILENAME] [-D] [-V] [--dry-run] [-v] [-q]
       {create,desc,destroy,list,set,show,status,tests,wait} ...
```

Next, you'll need to
[set up a PPA and build your package in it](PackageBuilding.md) as described
in the "Build binary packages via PPA" section. Once it has built binaries for
the architecture(s) you intend to test:

```bash
$ ppa tests \
  --show-url ppa:kstenerud/postfix-postconf-segfault-1753470 \
  --release bionic
```

This prints to the console a bunch of lines like:

```text
 Using Release Packages ♻️ 
 http://autopkgtest.ubuntu.com/request.cgi?release=bionic&arch=amd64&package=postfix&ppa=kstenerud/postfix-postconf-segfault-1753470&trigger=postfix/3.3.0-1ubuntu0.1~ppa1
 http://autopkgtest.ubuntu.com/request.cgi?release=bionic&arch=s390x&package=postfix&ppa=kstenerud/postfix-postconf-segfault-1753470&trigger=postfix/3.3.0-1ubuntu0.1~ppa1
  ...
```

The autopkgtest requests require special permissions to run; as a new
developer you'll need to ask your co-workers or a coredev to load them. If you
don't know where else to ask, the #ubuntu-devel IRC channel is suitable. The
`--showurl` parameter causes these URLs to be printed out so they're easier to
cut-and-paste into email or chat channels.

Once you've gained permissions to run autopkgtests, you can load each of these
URLs in your web browser yourself, which will cause the appropriate
autopkgtests to run. If you omit the `--show-url` parameter, `ppa tests` will
instead display clickable links, making it even more convenient. Alternatively,
it is possible to [trigger the tests through the command
line](ProposedMigration.md#triggering-tests-from-the-cli), which is useful when
you need to trigger several tests.

After a while, run `ppa tests` again to see how the tests are coming along:

```bash
$ ppa tests ppa:kstenerud/postfix-postconf-segfault-1753470 --release bionic
...
Results: (from http://autopkgtest.ubuntu.com/results/.../?format=plain)
  postfix @ amd64:
    14.06.22 21:57:01 ✅     Triggers: postfix/3.3.0-1ubuntu0.1~ppa1
...
```

If anything failed, you can load up the log URLs to see details about why.

## Testing with VMs and containers

If you use a container or VM, you'll need an image to test from. `autopkgtest`
will build a suitable image for you. You may want to regenerate the image from
time to time to cut down on the number of updates it must run.

The type of image you can use (chroot, container, or VM) depends on the
restrictions in `debian/tests/control`(see this page
[in the autopkgtest docs](https://salsa.debian.org/ci-team/autopkgtest/-/blob/master/doc/README.package-tests.rst)).

Important restrictions:

* `breaks-testbed`: This test is liable to break the testbed system. VM or container recommended.
* `isolation-machine`: You must use a VM to run these tests.
* `isolation-container`: You must use a VM or container to run these tests.
* `needs-reboot`: The test reboots the machine, so you must use a VM or container.

### The general process

No matter whether you are testing in a VM or a container, the command to run
the tests (and indeed, the general process) is constructed in the same way. 

#### Build the image

First, we will build the image we prepared in the previous section.

* To build a VM image:

  ```bash
  $ autopkgtest-buildvm-ubuntu-cloud -r focal -v \
   --cloud-image-url http://cloud-images.ubuntu.com/daily/server
  ```

  (Replace `focal` with your release of choice)
 
  Copy the resulting image (`autopkgtest-focal-amd64.img`) to the
  `/var/lib/adt-images` directory.

  > **Note**: 
  > Use `-m` to specify a closer mirror or `-p` to use a local proxy
  > if it's slow.

* To build a container image:

  ```bash
  $ autopkgtest-build-lxd ubuntu-daily/oracular
  ```

  You should see an autopkgtest image now when you run `lxc image list`.

#### Constructing the command

For every method, our command will use the same format:

```bash
autopkgtest <options> <what we want to test> -- <where we want to test> 
```

We have already created the image, thing we want to test, so the part of the
command **before** the `--` will be the same in every example, and uses the
following options:

```bash
$ autopkgtest \
  --apt-upgrade \
  --shell-fail \
  --output-dir <package-name>
    <what we want to test>
  -- <where we want to test> </image path/image name.img>
```

For more details on these options you can refer to the
[autopkgtest manpage](https://manpages.debian.org/testing/autopkgtest/autopkgtest.1.en.html).

Then, for each method, the only part that will change is the "where" we want
to test, which will be the part **after** the `--`. 

## 2) Testing with a VM
### Run the tests (manually) against a local directory

Make sure you're one directory up from your package directory and run:

```bash
$ autopkgtest \
  --apt-upgrade \
  --shell-fail \
  --output-dir dep8-mypackage \
    mypackage/ \
  -- qemu /var/lib/adt-images/autopkgtest-focal-amd64.img
```

> Note:
> * `mypackage/`: Put your package name here. The trailing slash tells it to
  interpret this as a directory rather than a package name.
> * `qemu`: Is shorthand for `autopkgtest-virt-qemu`.


### Run the tests (against the PPA)

Make sure you're one directory up from your package directory and run:

```bash
$ autopkgtest \
  --apt-upgrade \
  --shell-fail
  --output-dir dep8-mypackage-ppa \
  --setup-commands="sudo add-apt-repository \
    --yes \
    --update \
    --enable-source \
    ppa:mylaunchpaduser focal-mypackage-fixed-something-1234567" \
  --no-built-binaries \
    mypackage \
  -- qemu /var/lib/adt-images/autopkgtest-focal-amd64.img
```

Note that in the `add-apt-repository` command, the `--update` flag became part
of the default after 20.04 LTS. So if you are running a test on jammy or later
you do not need to include that flag.

> **Note**: 
> In this case, the package name **doesn't** have a trailing slash because we
> want to install the package.


## 3) Testing with a container

### Run the tests (against the PPA)

```bash
$ autopkgtest \
 --apt-upgrade \
 --shell-fail \
 --output-dir dep8-mypackage-ppa \
 --setup-commands="sudo add-apt-repository -y -u -s \
 ppa:mylaunchpaduser/focal-mypackage-fixed-something-1234567" \
 --no-built-binaries \
 mypackage \
 -- lxd autopkgtest/ubuntu/focal/amd64
```

The `setup-commands` options are as described in the previous section.

## 4) Testing in the cloud with Canonistack

This is by far the closest in terms of "similarity" to the real autopkgtests
since they also run in such an environment, but it needs some preparation.

First of all, you must have been *unlocked for* and have set up
[Canonistack](https://wiki.canonical.com/InformationInfrastructure/IS/CanoniStack-BOS01)
for yourself.

In going through the setup process for Canonistack, you'll have created an
OpenStack RC file that sets region, auth and other environment variables. Go
ahead and source this file, if you haven't already.

Then you'll look for the image you want to boot as in the following example:

```bash
$ source ~/.canonistack/novarc_bos01
$ openstack image list | grep -i arm64 | grep hirsute
| 4d24cfbe-b6a5-4d84-8c50-b9f025d0dd43 | ubuntu/ubuntu-hirsute-daily-arm64-server-20201124-disk1.img    | active |
| 1cfeacff-f04a-4bce-ab92-9d8fec7e5edb | ubuntu/ubuntu-hirsute-daily-arm64-server-20201125-disk1.img    | active |
```

Make sure that you have installed `glance`. The `nova` script we will use in
the following example needs it. You can install it by either using `apt`:

```bash
$ sudo apt update && sudo apt install python3-glanceclient
```

Or `pip`:

```bash
$ pip install python3-glanceclient
```

Finally, to
[run the test on Canonistack](https://wiki.ubuntu.com/proposedMigration#Reproducing_tests_in_the_cloud)
is quite similar to the other invocations. Just two things change compared to
"local" `autopkgtest-runner` invocations.

* `--setup-commands setup-testbed` will have autopkgtest execute
  `/usr/share/autopkgtest/setup-commands/setup-testbed` on the target which
  converts any system into a system that is ready for autopkgtest to log in.
* `-- ssh -s nova` achieves two things:
  * First, it selects the SSH virtualisation driver `autopkgtest-virt-ssh` to
    reach out to a remote system.
  * It also selects the setup script `nova` from
    `/usr/share/autopkgtest/ssh-setup/nova`, which happens to know how to deal
    with OpenStack.

```bash
# General pattern
$ autopkgtest \
  --no-built-binaries \
  --apt-upgrade \
  --setup-commands setup-testbed \
  --shell-fail <mypackage>.dsc \
  -- ssh -s nova -- \
  --flavor m1.small \
  --image <image> \
  --keyname <yourkeyname>
```

```bash
# One example
$ autopkgtest \
  --no-built-binaries \
  --apt-upgrade \
  --setup-commands setup-testbed \
  --shell-fail systemd_247.3-1ubuntu2.dsc \
  -- ssh -s nova -- \
  --flavor m1.small \
  --image ubuntu/ubuntu-hirsute-daily-arm64-server-20201125-disk1.img \
  --keyname paelzer_canonistack-bos01
```

You can use all the usual OpenStack terms, e.g. other flavors, sizing the VM
used, or other images to run the same test on different releases or
architectures.


### Armhf is special

Canonistack does not have native armhf nodes. Because of that the autopkgtests
on that architecture actually run in armhf containers on arm64 hosts.
To recreate that environment you'll first need to get a Canonistack arm64
instance and there combine all of the above like:

```bash
$ autopkgtest \
  --no-built-binaries \
  --apt-upgrade \
  --setup-commands setup-testbed \
  --shell-fail <mypackage>.dsc \
  -- lxd ubuntu-daily:mantic/armhf
```

These days normal images mostly work, but for completeness (and because you
read this being cursed by tracking a special case) there is also a form which
creates an image adapted to the use for autopkgtest.

```bash
# prep armhf container image for autopkgtest
$ autopkgtest-build-lxd ubuntu-daily:mantic/armhf

# check the created container
$ lxc image list
...
| autopkgtest/ubuntu/mantic/armhf | d5d93f552340 | yes    | autopkgtest Ubuntu mantic armhf       | armv7l       | CONTAINER | 571.57MB | Aug 25, 2023 at 8:56am (UTC) |
...

# run a test in that container
$ autopkgtest --no-built-binaries --apt-upgrade --setup-commands setup-testbed --shell-fail <mypackage>.dsc -- lxd autopkgtest/ubuntu/mantic/armhf
```

## Common options you'll need

### Run against `-proposed` or its subsets

Quite often, a test fails by running against new packages in the `-proposed`
pocket. In this case, it's helpful to check if the test needs other packages
from `-proposed` to resolve the issue. This can easily be done via the
`--apt-pocket` option.

A test will usually run against all packages in `-release` plus the new
candidate from `-proposed`, which looks like this:

```bash
--apt-pocket=proposed=src:yourpkg
```

To run against all packages in `-proposed`, you can remove the reference to a
specific package.

```bash
--apt-pocket=proposed
```

If instead you need a given set of packages, but not *everything* else from
`-proposed`, you can use a comma-separated list:

```bash
--apt-pocket=proposed=src:srcpkg1,srcpkg2
```

Here are some examples testing various combinations against `octave-parallel`:

```bash
# normal
$ autopkgtest --apt-pocket=proposed \
 --shell-fail octave-parallel_4.0.0-2ubuntu1~ppa1.dsc \
 -- qemu ~/work/autopkgtest-hirsute-amd64.img
# all proposed
$ autopkgtest --apt-pocket=proposed \
 --shell-fail octave-parallel_4.0.0-2ubuntu1~ppa1.dsc \
 -- qemu ~/work/autopkgtest-hirsute-amd64.img
# specific subset
$ autopkgtest --apt-pocket=proposed=src:octave,octave-parallel,octave-struct \
 --shell-fail octave-parallel_4.0.0-2ubuntu1~ppa1.dsc \
 -- qemu ~/work/autopkgtest-hirsute-amd64.img
```

### Size the test VM

One might often wonder "hmm, might this work with more CPU/memory?". At least
in the case of QEMU and nova, that can be controlled.

For `qemu` you can add `--ram-size` and `--cpus`. For example, to run the same
test in different sizes:

```bash
$ autopkgtest --no-built-binaries --apt-upgrade \
 --shell-fail octave-parallel_4.0.0-2ubuntu1~ppa1.dsc \
 -- qemu --ram-size=1536 --cpus 1 ~/work/autopkgtest-hirsute-amd64.img
$ autopkgtest --no-built-binaries --apt-upgrade \
 --shell-fail octave-parallel_4.0.0-2ubuntu1~ppa1.dsc \
 -- qemu --ram-size=4096 --cpus 4 ~/work/autopkgtest-hirsute-amd64.img
```

For `nova`, you use
[OpenStack flavors](https://docs.openstack.org/nova/latest/user/flavors.html).
If you're unsure which ones are defined you can check with
`openstack flavor list`. Here's an example of passing `nova` different sizes:

```bash
$ autopkgtest --no-built-binaries --apt-upgrade \
 --setup-commands setup-testbed --shell-fail systemd_247.3-1ubuntu2.dsc \
 -- ssh -s nova -- \
 --flavor m1.small \
 --image ubuntu/ubuntu-hirsute-daily-arm64-server-20201125-disk1.img \
 --keyname paelzer_canonistack-bos01
$ autopkgtest --no-built-binaries --apt-upgrade \
 --setup-commands setup-testbed --shell-fail systemd_247.3-1ubuntu2.dsc \
 -- ssh -s nova -- \
 --flavor cpu4-ram8-disk20 \
 --image ubuntu/ubuntu-hirsute-daily-arm64-server-20201125-disk1.img \
 --keyname paelzer_canonistack-bos01
$ autopkgtest --no-built-binaries --apt-upgrade \
 --setup-commands setup-testbed --shell-fail systemd_247.3-1ubuntu2.dsc \
 -- ssh -s nova -- \
 --flavor cpu8-ram16-disk50 \
 --image ubuntu/ubuntu-hirsute-daily-arm64-server-20201125-disk1.img \
 --keyname paelzer_canonistack-bos01
```

### Restrict networking

Some DEP-8 test failures occur due to the autopkgtest environment's network
restrictions. A good clue that this has happened is when the tests reliably
pass locally, yet fail after uploading to Launchpad. Other clues include:

* Test logs mentioning port errors
* Inaccessible URLs or IPs
* Upstream language packaging tools like `npm`, `compose`, `pip`, etc.

The network restrictions can be mimicked by invoking autopkgtest locally with
an internal proxy. This won't fully replicate it as there are also firewalls
in place, but when in doubt it often is worthwhile to retry with a local
VM-based repro to check if it fails this way.

To do this, add the internal proxy (this needs the VPN up); or if you want
to, try another proxy of your choice that is rather restrictive. Then add the
following to the call of autopkgtest:

```bash
--env='no_proxy=127.0.0.1,127.0.1.1,localhost,localdomain,novalocal,internal,archive.ubuntu.com,security.ubuntu.com,ddebs.ubuntu.com,changelogs.ubuntu.com,ppa.launchpad.net' \ 
--env='http_proxy=http://squid.internal:3128'
```

Here's an example from when we tracked down an issue with `ulfius`:

```bash
$ autopkgtest \
 --env='no_proxy=127.0.0.1,127.0.1.1,localhost,localdomain,novalocal,internal,archive.ubuntu.com,security.ubuntu.com,ddebs.ubuntu.com,changelogs.ubuntu.com,ppa.launchpad.net' \
 --env='http_proxy=http://squid.internal:3128' \
 --no-built-binaries \
 --apt-upgrade \
 --shell-fail ulfius_2.7.1-3.dsc  \
 -- qemu ~/work/autopkgtest-impish-amd64.img
```

## Save the results

You'll see the tests run:

```text
autopkgtest [11:47:12]: version 5.3.1
autopkgtest [11:47:12]: host karl-tp; command line: /usr/bin/autopkgtest -U -s -o dep8-postfix-ppa '--setup-commands=sudo add-apt-repository -y -u -s ppa:kstenerud/postfix-postconf-segfault-1753470' -B postfix -- lxd autopkgtest/ubuntu/focal/amd64
autopkgtest [11:47:31]: @@@@@@@@@@@@@@@@@@@@ test bed setup

...

----------------------------------------------------------------------
Ran 15 tests in 67.027s

OK
autopkgtest [11:49:51]: test postfix: -----------------------]
autopkgtest [11:49:51]: test postfix:  - - - - - - - - - - results - - - - - - - - - -
postfix              PASS
autopkgtest [11:49:52]: @@@@@@@@@@@@@@@@@@@@ summary
postfix              PASS
```

Save the last part for the description for your merge proposal.
