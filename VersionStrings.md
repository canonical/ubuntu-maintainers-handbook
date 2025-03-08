Version string format
=====================

Choosing the appropriate [version](https://manpages.ubuntu.com/manpages/man7/deb-version.7.html) can be complex due to numerous conditions to consider. The following paragraphs first provide links to essential foundational information, shared with Debian. Subsequently, each paragraph addresses specific scenarios that may arise when uploading to Ubuntu.


## The Version field

Since Ubuntu's versioning specifics are derived from Debian, it is beneficial to begin by thoroughly reviewing the
[Debian control field "Version"](https://www.debian.org/doc/debian-policy/ch-controlfields.html#version)
to understand the foundational principles.
The key takeaway is that Debian versions are formatted as `[upstream_version]-[debian_revision]`, using the `-` to split the upstream version from the Debian packaging revision.

When we modify a package compared to what's shipped by Debian, we indicate the change by adding 'ubuntu' to the Debian revision.
One can think of a Ubuntu version number consisting of three components:
`[upstream_version]-[debian_revision_component]ubuntu[ubuntu_revision_component]`.
The `ubuntu` string then marks that whatever follows it is related to changes added in Ubuntu.
(`ubuntu` isn't the only string used here, but more on that shortly.)

This distinction allows each party involved in providing a package to the users — Upstream, Debian and Ubuntu — to modify and iterate on their section of the version number without interfering with others.
When everyone abides by these conventions we can guarantee package upgradability, but also allow that one can understand some of the package history by just looking at the package version.


## Version: Adding a change in the current Ubuntu development release

As shown above `[upstream_version]-[debian_revision]ubuntu[ubuntu_revision]` is the basic pattern.
Changes within the development release require adding or incrementing the number immediately following the `ubuntu` string.
Consequently, the first Ubuntu modification would have `ubuntu1` appended to the Debian revision.
Subsequent Ubuntu changes would increment this suffix, i.e. `ubuntu1 -> ubuntu2`.

> Example in detail: _Adding a change to `2.0-2` in the Ubuntu development release will use `2.0-2ubuntu1`_

> Example in detail: _Adding another change to `2.0-2ubuntu1` in the Ubuntu development release will use `2.0-2ubuntu2`_

As alluded to earlier, there are a number of special cases, and situations where we use a string other than `ubuntu` in Ubuntu version numbers.
One such case is `build`, typically used for _[no change rebuilds](VersionStrings.md#version-no-change-rebuilds)_ that will be discussed in more depth below.
The key thing to know when incrementing from a `buildN` is that the package is considered unchanged from the debian revision component, and incremented as a *new* Ubuntu change.

Here's a listing of examples of the application of these rules:

| Previous version             | Modification type | Recommended version   |
| ---------------------------- | ----------------- | --------------------- |
| 2.0-2                        | Ubuntu change     | 2.0-2ubuntu1          |
| 2.0-2ubuntu1                 | Ubuntu change     | 2.0-2ubuntu2          |
| 2.0-2ubuntu2                 | Ubuntu change     | 2.0-2ubuntu3          |
| 2.0-3                        | Re-Sync w/ Debian | 2.0-3                 |
| 2.0-3                        | No-change rebuild | 2.0-3build1           |
| 2.0-3build1                  | No-change rebuild | 2.0-3build2           |
| 2.0-3build2                  | Ubuntu change     | 2.0-3ubuntu1          |

While these examples cover common scenarios, it's important to note that they apply specifically to the current Ubuntu development release.
Stable releases have different policies, but first we'll explore some special cases for development releases.


## Version: Syncing from Debian

Sometimes the desired change has already been packaged by Debian in a newer version of the package.

During the Ubuntu development cycle, the archive tooling automatically copy or '[sync](Syncs.md#why-a-sync-is-better)' new package versions from Debian.
Only packages with no Ubuntu modifications or those with no-change updates (i.e., versions with `build` rather than `ubuntu` in the revision), are automatically synced.
If these updates pass Ubuntu's automated testing, they do not require manual intervention.
However, there are scenarios where the automation cannot handle the process.

First, once the development release reaches [Debian import freeze](https://wiki.ubuntu.com/DebianImportFreeze), the sync automation is disabled.
Therefore, if an update from Debian *is* required, it must be [manually synced](Syncs.md).
Version numbering in this case is trivial:  It's the exact same version as in Debian.

A second case is when the package does have Ubuntu changes,
but our modifications no longer need to be carried as Ubuntu delta with the new Debian package.
For example, perhaps the Ubuntu changes were cherrypicked patches from upstream that are now available in a newer release now available from Debian,
or perhaps Debian has adopted all of the Ubuntu changes into their own packaging.
These situations will also require a [manual re-sync with Debian](Syncs.md).
The version number also remains identical to Debian (see the 'Re-Sync w/ Debian' example in the previous section).


## Version: Merging from Debian

If Ubuntu has delta to the package from Debian, it can not be
[automatically synced](Syncs.md#why-a-sync-is-better) otherwise that delta would be lost.
In that case an Ubuntu developer regularly [merges the delta](PackageMerging.md#merging) (usually at least once per Ubuntu release)
that is in Ubuntu with the content from Debian and indirectly also with the changes by upstream.

The counter for Ubuntu increments resets to 1, therefore the version for the new upload to the Ubuntu development is the version from Debian with a suffix of `ubuntu1`

> Example in detail: _Uploading `3.1` to Ubuntu -Devel based on Debian `3.1-2` retaining former Ubuntu delta of `2.1-1ubuntu2` would become `3.1-2ubuntu1`_

List of this and further related examples:

| Old-Debian   | Old-Ubuntu           | New-Debian     | Ubuntu Devel          |
| ------------ | -------------------- | -------------- | --------------------- |
| 2.1-1        | 2.1-1ubuntu2         | 3.1-2          | 3.1-2ubuntu1          |
| 1:7.0+dfsg-7 | 1:7.0+dfsg-7ubuntu14 | 1:8.0.4+dfsg-1 | 1:8.0.4+dfsg-1ubuntu1 |

## Version: Merging from Upstream

The mentioned special case of an _upstream merge_ is triggered when Ubuntu merges directly from upstream ahead of Debian
upstream. In this case the new version shall:

* Reflect that it was not yet packaged in Debian (at this point in time) and thereby is not _really_ based on a Debian version by using `-0` as the Debian version suffix.
* Represent that this is the first devel upload via `ubuntu1` which also serves as a place increment for further uploads to the development release.

> Example in detail: _Uploading `3.1` to Ubuntu -Devel while Debian is not yet at `3.1` so we pick `3.1` followed by `-0` representing it is not based on Debian and `ubuntu1` to index and iterate changes to Ubuntu to an overall `3.1-0ubuntu1`._

List of this and further related examples:

| Old-Debian   | Old-Ubuntu           | New-Debian     | New-Upstream | Ubuntu Devel          |
| ------------ | -------------------- | -------------- | ------------ | --------------------- |
| 2.1-1        | 2.1-1                | _unchanged_    | 3.1          | 3.1-0ubuntu1          |
| 2.1-1        | 2.1-1ubuntu2         | _unchanged_    | 3.1          | 3.1-0ubuntu1          |
| 2.1-1        | 2.1-1ubuntu2         | _unchanged_    | 2.3          | 2.3-0ubuntu1          |

## Version: Adding a change in Ubuntu as a [stable release update](https://canonical-sru-docs.readthedocs-hosted.com/)

After a version of Ubuntu is released, changes to packages follow a slightly different versioning scheme which ensures upgradability to later releases and allows the history of changes to be recognized from just looking at the version.

The only change from the normal ubuntu version scheme is to the _ubuntu revision_ section of `[upstream_version]-[debian_revision]ubuntu[ubuntu_revision]`.

* Increment Y in the numeric `ubuntuX.Y` suffix, like: `ubuntu3.1 -> ubuntu3.2`.
* Never increment X in the numeric `ubuntuX.Y` suffix. An Ubuntu delta is always captured in Y.
* If this is the first change via the SRU process, add the `.1` suffix, like: `ubuntu3 -> ubuntu3.1`.
* If no `ubuntuX` was present before, then set `ubuntu0.1` which will represent that there was no Ubuntu delta (`0`) before this upload which is the first to add a change (`.1`).
* If two releases have the same version, then this would make the package non upgradeable and cause a version conflict (two builds, but not the same). To resolve this, add the numeric `YY.MM` release version in between `ubuntuX` and the increment `.1`. For Jammy that might look like `...ubuntu3.22.04.1`

Compare these examples with the _[adding a change in the current Ubuntu development release](VersionStrings.md#version-adding-a-change-in-the-current-ubuntu-development-release)_ section above to better see the subtle difference.

> Example in detail: Adding a change to `2.0-2` in an Ubuntu stable release update will use `2.0-2ubuntu0.1`.

> Example in detail: Adding an Ubuntu delta to `2.0-2ubuntu2.4` in an Ubuntu stable release update will use `2.0-2ubuntu2.5`.

List of these and further related examples:

| Previous version             | Recommended version                           |
| ---------------------------- | --------------------------------------------- |
| 2.0-2                        | 2.0-2ubuntu0.1                                |
| 2.0-2ubuntu0.1               | 2.0-2ubuntu0.2                                |
| 2.0-2ubuntu2                 | 2.0-2ubuntu2.1                                |
| 2.0-2ubuntu2.1               | 2.0-2ubuntu2.2                                |
| 2.0-2build1                  | 2.0-2ubuntu0.1                                |
| 2.0                          | 2.0ubuntu0.1                                  |
| 2.0-2ubuntu0.22.04.1         | 2.0-2ubuntu0.22.04.2                          |
| 2.0-2 in two releases        | 2.0-2ubuntu0.11.10.1 and 2.0-2ubuntu0.22.04.1 |
| 2.0-2ubuntu1 in two releases | 2.0-2ubuntu1.11.10.1 and 2.0-2ubuntu1.22.04.1 |

## Version: native packages

There exist packages in which Debian or Ubuntu are themselves the upstream.
This is due to the classification as _native_ or _non-native_ package as outlined by the referred [Debian control field version](https://www.debian.org/doc/debian-policy/ch-controlfields.html#version) and in more detail [Debian source packages](https://www.debian.org/doc/debian-policy/ch-source.html#s-source-packages).
To quote "Presence of the `debian_revision` part indicates this package is a non-native package. Absence indicates the package is a native package."

Due to that in a Debian native package, there is no `-debian_revision`.

This continues into native Ubuntu packages, which have neither `-debian_revision` nor `ubuntu[ubuntu_revision]`.

> Example in detail: _A package native to Debian `2.0` (no `-`), getting an Ubuntu change in the devel release would use `2.0ubuntu1`_

Furthermore native package versioning is package dependent; whether if it uses only _major_, or a _major.minor_, or any other version pattern is the maintainer's choice.
Just as usually upstream can and will version software the way they consider the best.
Due to that selecting the right subsequent version requires you to check the package history or confer with its maintainer.

> Example in detail: _A package native to Ubuntu `2.0` (no `-` and no `ubuntuX`) getting an Ubuntu change in the devel release would use `2.1` or `3.0`_

Yes that means, from just the version, you can not differentiate between a _native Debian package that was synced_ and a _native package in Ubuntu._

List of these and further related examples:

| Previous version       | Devel upload  |  SRU upload   |
| ---------------------- | ------------- | ------------- |
| 2.0 (native in Debian) | 2.0ubuntu1    |  2.0ubuntu0.1 |
| 2.0 (native in Ubuntu) | 2.1 or 3.0    |  2.0ubuntu0.1 |
| 2   (native in Debian) | 2ubuntu1      |  2ubuntu0.1   |
| 2   (native in Ubuntu) | 3             |  2ubuntu0.1   |
| 2.0ubuntu2             | 2.0ubuntu3    |  2.0ubuntu2.1 |
| 2.0build1              | 2.0ubuntu1    |  2.0ubuntu0.1 |
| 2.0build2              | 2.0ubuntu1    |  2.0ubuntu0.1 |

Note: The rule of multiple releases having the same version requiring to add also a per-release YY.MM to differentiate and ensure upgradability might apply here as well.

## Version: no change rebuilds

Ubuntu continuously [syncs](Syncs.md#why-a-sync-is-better) changes from Debian
until the [Debian import freeze](https://wiki.ubuntu.com/DebianImportFreeze) date is met, then syncs are deactivated to stabilize the upcoming Ubuntu release.
As outlined in these references there _"will be automatically imported from Debian where they have not been customized for Ubuntu, that is when the version number of the
package in the current Ubuntu development branch does not contain the substring ubuntu"_.

And as we learned in _[Adding a change in the current Ubuntu development release](VersionStrings.md#version-adding-a-change-in-the-current-ubuntu-development-release)_
above adding a change in Ubuntu will add a `ubuntuX` suffix, which prevents the automatic synchronisation process, which thereby will ensure Ubuntu is not unintentionally losing these changes.

But what if one needs to rebuild a package as-is, for example due to a library transition to build against new versions of other packages,
but does not want to prevent future automatic syncing of new versions from Debian?
Well, just rebuilding is not considered a change that has to be retained, so in this special case one would:

* Not add `ubuntuX` and instead add a `buildX` suffix.
* If there is already one such `buildX` suffix present, increment it.
* If there is already an `ubuntuX` suffix there would be no auto-sync anyway, so increment the existing suffix.

So far it is weakly defined if a no change rebuild for _[native packages](VersionStrings.md#version-native-packages)_ shall be a version increment or adding a `buildX` suffix.
Both styles are present in the archive and both will work just fine.
Until this is properly defined one should try to follow what the particular package used so far.

> Example in detail: _A Debian package `2.0-2` needs a rebuild in Ubuntu which would use `2.0-2build1`_

List of this and further related examples:

| Previous version             | No change rebuild in devel    |
| ---------------------------- | ----------------------------- |
| 2.0-2                        | 2.0-2build1                   |
| 2.0-2ubuntu2                 | 2.0-2ubuntu3                  |
| 2.0-2build1                  | 2.0-2build2                   |
| 2.0 (native in Ubuntu)       | 2.1 or 3 or 2.0build1         |
| 2   (native in Ubuntu)       | 3 or 2build1                  |
| 2.0 (native in Debian)       | 2.0build1                     |
| 2   (native in Debian)       | 2build1                       |

It is unlikely, but possible that one needs a no change rebuild as part of a [Stable release update](https://canonical-sru-docs.readthedocs-hosted.com),
but in this situation auto-syncing isn't active anyway and the need for upgradability applies.
So a no-change-rebuild in regard to picking a version number in this case is identical to any other change
(see the section _[adding a change in Ubuntu as a stable release update](VersionStrings.md#version-adding-a-change-in-ubuntu-as-a-stable-release-update)_ above).


## Version: Backport from upstream

There are more special cases, especially related to backports (as in backports of new versions via SRU, not as in https://wiki.ubuntu.com/UbuntuBackports).

In the rare case of a _new upstream release being pushed to all stable releases_
it will lose all its former version suffixes. This is a common practice on some
minor release exception processes, where we pick up the most recent upstream
content, but avoid regressions/changes due to packaging changes like moving files,
splitting or removing binary packages etc. To represent this the upload should:

* Signal that it is not based on a Debian packaging version by using `-0` as the Debian packaging revision.
* Signal that it was not yet packaged before in this Ubuntu release before via `ubuntu0.`.
* Add a suffix per Ubuntu release like `.22.04`.
* Add an increment that can be used in subsequent individual per release SRU uploads like `.1`.

> Example in detail: _Uploading `3.1` but packaging-wise using what is already in LTS and LTS+1 -- not with all the Ubuntu changes that might be in `3.1-1ubuntu2`_.

List of this and further related examples:

|       | Old          | New                  |
| ----- | ------------ | -------------------- |
| LTS   | 2.0-2        | 3.1-0ubuntu0.22.04.1 |
| LTS+1 | 2.7-2ubuntu1 | 3.1-0ubuntu0.22.10.1 |
| LTS+2 | 2.7-2ubuntu1 | 3.1-0ubuntu0.23.04.1 |

The new version is independent of the former version of the target release, here is a list of examples targeting 22.04:

| Previous       | New upstream | Upload for LTS       |
| -------------- | ------------ | -------------------- |
| 2.0-2          | 3.1          | 3.1-0ubuntu0.22.04.1 |
| 2.0-2ubuntu2   | 3.1          | 3.1-0ubuntu0.22.04.1 |
| 2.0-2ubuntu2.1 | 3.1          | 3.1-0ubuntu0.22.04.1 |
| 2.0-2build1    | 3.1          | 3.1-0ubuntu0.22.04.1 |

## Version: Backport from Ubuntu devel

But if instead the backport is actually without _meaningful differences_ to what is in the current Ubuntu development release (a common
practice for some packages that keep the same version everywhere) with only minimal backporting adaptations it would instead:

* Signal that is it basically _identical but backported_ by starting with the same version as the one in the development release.
* Add a `~` to ensure it sorts earlier than the version in -devel.
* Add a suffix per Ubuntu release like `22.04`.
* Add an increment that can be used in subsequent individual per release SRU uploads like `.1`.

> Example in detail: _Uploading `3.1` to LTS and LTS+1 being more or less identical to what is in -devel in `3.1-1ubuntu2`_.

List of this and further related examples:

|       | Old          | New                  |
| ----- | ------------ | -------------------- |
| LTS   | 2.0-2        | 3.1-1ubuntu2~22.04.1 |
| LTS+1 | 2.7-2ubuntu1 | 3.1-1ubuntu2~22.10.1 |
| LTS+2 | 2.7-2ubuntu1 | 3.1-1ubuntu2~23.04.1 |

The new version is again independent of the former version that was present in the target release, here examples targeting 22.04:

| Previous       | New Dev      | Upload for LTS       |
| -------------- | ------------ | -------------------- |
| 2.0-2          | 3.1-1ubuntu2 | 3.1-1ubuntu2~22.04.1 |
| 2.0-2ubuntu2   | 3.1-1ubuntu2 | 3.1-1ubuntu2~22.04.1 |
| 2.0-2ubuntu2.1 | 3.1-1ubuntu2 | 3.1-1ubuntu2~22.04.1 |
| 2.0-2build1    | 3.1-1ubuntu2 | 3.1-1ubuntu2~22.04.1 |

Note: If in this case the package in the Ubuntu development release is a native package one
would still use _the same version_ as in the development release.

> Example in detail: _Uploading `3.1` to LTS and LTS+1 being more or less identical to what is in -devel in `3.1`_.

List of this and further related examples:

|       | Old          | New         |
| ----- | ------------ | ----------- |
| LTS   | 2.0-2        | 3.1~22.04.1 |
| LTS+1 | 2.7-2ubuntu1 | 3.1~22.10.1 |
| LTS+2 | 2.7-2ubuntu1 | 3.1~23.04.1 |

## Version: almost native packages

Let us be clear, _almost native packages_ is not a clear definition, but some have evolved that way.
An example is snapd, which is packaged like a native package in Ubuntu with versions like `2.67+25.04`,
the content matching 1:1 what is upstream including the debian/* being there, and being packaged as `3.0 (native)`.
But the packages versions are nowadays no more _defined_ by these native uploads to Ubuntu.
There actually is a [separate upstream project with proper releases](https://github.com/canonical/snapd/tags).

Furthermore the Debian packaging switched over to proper upstream/downstream packaging with
orig tarball, debian additions and a version like `2.67-1` and format being `3.0 (quilt)`.

Due to that some of the normal rules no more apply.
Changing the example `2.67` to `2.67.1` is no more just a Ubuntu native version choice.
It represents a new upstream version in the upstream project which might not exist or not match.

Furthermore the example of snapd is also usually backported with the same content to all active releases.
Combined that makes this an uncommon mix of the rules usually applied to:

* native package
* backport from upstream

Let us define how such a case could be handled for the example of snapd:

* The first element of the version matches the upstream version it represents, like `2.67`
* Since this is backported to multiple releases at the same version, but being native can't have an `-` like in the usual `-X` or `-XubuntuY`. But it can have the suffix is added via `+ubuntuYY.MM`
  * Note: former versions just added `+YY.MM` but for symmetry and to avoid e.g. conflicts with the auto-sync process `+ubuntuYY.MM` should be the way to go
* If iterations on the same upstream version and for the same target Ubuntu release are needed add an increment counter `.x" as suffix

| Previous                  | New upstream 2.67 | Changes inside of 2.66 |
| ------------------------- | ----------------- | ---------------------- |
| LTS: 2.66+ubuntu24.04     | 2.67+ubuntu24.04  | 2.66+ubuntu24.04.1     |
| Devel: 2.66+ubuntu25.04   | 2.67+ubuntu25.04  | 2.66+ubuntu25.04.1     |
| LTS: 2.66+ubuntu24.04.1   | 2.67+ubuntu24.04  | 2.66+ubuntu24.04.2     |
| Devel: 2.66+ubuntu25.04.1 | 2.67+ubuntu25.04  | 2.66+ubuntu25.04.2     |

> Yes, no more being a _real_ native package means this could also become
> `2.68-0ubuntu1` and thereby the more "normal" rules would apply, but
> until the release process can handle it the above is an example how we can
> apply the same rules outlined here to almost any case and still ensure
> upgradability and some reasonable instant insight looking at only the
> version number.

## Version: Undoing mistakes

In a very rare case where an upgrade to a new upstream version of a package
caused major regression and the only way out is rolling back, the convention is
to insert `+really` followed by the actual version number.

To be clear, both should be used sparingly and only if really needed.
Please also have a look at [+really in control fields](https://www.debian.org/doc/debian-policy/ch-controlfields.html#epochs-should-be-used-sparingly).

> Example in detail: _Uploading `3.1` caused a regression, so rolling back to the previous version `2.0-2ubuntu2` would use `3.1+really2.0-2ubuntu2`_

List of this and further related examples:

| Before         | Current      | Upload                                               |
| -------------- | ------------ | ---------------------------------------------------- |
| 2.0-2ubuntu2   | 3.1-2ubuntu1 | 3.1+really2.0-2ubuntu2                               |
| 7.80+dfsg1-5   | 7.91+dfsg1-1 | 7.91+dfsg1+really7.80+dfsg1-1ubuntu1 (Ubuntu upload) |
| 7.80+dfsg1-5   | 7.91+dfsg1-1 | 7.91+dfsg1+really7.80+dfsg1-1 (Debian upload)        |
| 7.80+dfsg1-5   | 7.91+dfsg1-1 | 7.91+dfsg1+really7.80+dfsg1-1ubuntu0.1 (Ubuntu SRU)  |


## Version: Further reading

Right at the beginning we had the link to the underlying concepts in Debian
referring to the [Debian control field "Version"](https://www.debian.org/doc/debian-policy/ch-controlfields.html#version)
for overall topic awareness.

But of course there are more special cases, one that you might encounter is package uploads based on repository content.
This is most commonly done:

* If upstreams do not yet have a new release, but we'd like to have all the content up to a certain point in time/git.
* In cases where the upstream release is imminent, but to meet freeze deadlines one uploads the state from git (but going to final before the release)
* If upstream is only using nightly builds and you need to pick one of them

At a high level you'd either go before the next version `{upcoming_version}~git...` or
if this seems more appropriate to the case ahead of the current `{current_version}+git...`.
The former is more common, the latter a special case for example considered if nobody knows if/when/what the new revision will be.
The underlying principles are outlined in more detail in the [Debian Wiki](https://wiki.debian.org/Versioning)
