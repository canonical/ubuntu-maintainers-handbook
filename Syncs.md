# Syncs

If a Ubuntu package is the same as the Debian version, we have an automated
mechanism that synchronises the Debian version to our Ubuntu series. 

A **delta** represents the difference between the Ubuntu and Debian versions of
a package. Typically, when we make changes to a package, we
[merge](PackageMerging.md#merging) our delta onto the upstream version. Debian
packages can only be synchronised if there is no delta.

However, there can be occasions where Debian or upstream incorporate our
logical changes before we merge them, meaning that there is essentially no
difference between the Ubuntu and Debian versions. If we were to proceed with
our merge, we would be merging an
[empty commit](PackageMerging.md#empty-commits) (i.e., adding a commit where
nothing has changed).

In this situation, it is better to sync the new version of the Debian
package back to Ubuntu than to manually perform an empty merge.

## Asking for a sync

The automatic syncing of packages from Debian is active for only some of the
Ubuntu release cycle - see
[the Debian Import Freeze](https://wiki.ubuntu.com/DebianImportFreeze)
page for more information.

Let us consider a test case where we have an empty Ubuntu delta before
Debian Import Freeze. You can check the
[Release Schedule](https://wiki.ubuntu.com/ReleaseSchedule)
for current releases in development. The Debian package is on
[`testing`](https://www.debian.org/releases/), so doing an
[explicit sync](https://wiki.ubuntu.com/SyncRequestProcess#Content_of_a_sync_request)
is unnecessary, but we have to fill the MP for the
unfinished-and-not-necessary-merge in the following way:

- Specify that the MP is for a sync request.
- Write down how you discovered it is a sync: changelog entries, step in where
  the empty commit message appeared, point to upstream git repository, etc.
- Change the changelog using `dch -i` to get a new version with the `ubuntu1`
  suffix and check the Ubuntu series for which the package is to be built. The
  text in that new changelog entry should say "build debian version to verify
  before a sync".
- [Build the source package](PackageBuilding.md#using-dpkg-buildpackage)
  and upload to the PPA you're using in this MP.

An example of this case is
[presented here](https://code.launchpad.net/~mirespace/ubuntu/+source/freeipmi/+git/freeipmi/+merge/407014).

For other sync situations, see the
[Ubuntu wiki page](https://wiki.ubuntu.com/SyncRequestProcess).
Outside of the server team process, the common way is to request an explicit
sync via either
[filling a Launchpad Bug](https://wiki.ubuntu.com/SyncRequestProcess#For_people_requiring_sponsorship),
or using the
[**requestsync** tool](https://manpages.ubuntu.com/manpages/lunar/man1/requestsync.1.html).


## How to perform a sync

If you have the permissions to upload the package to Ubuntu, you can issue a
sync request using the
[**syncpackage** tool](https://manpages.ubuntu.com/manpages/lunar/man1/syncpackage.1.html).
The process is described more fully in the
[Ubuntu Wiki page](https://wiki.ubuntu.com/SyncRequestProcess#For_people_with_permission_to_upload_the_package_to_Ubuntu).
To be able to use `syncpackage`, the package needs to be known to Launchpad
and there is a slight delay between a Debian upload and the availability in
Launchpad. You can check the Debian publishing history of a package in
`https://launchpad.net/debian/+source/<name_of_the_package>/+publishinghistory`
like in this example for
[freeipmi](https://launchpad.net/debian/+source/freeipmi/+publishinghistory).

For our example case of `freeipmi`, the sync was done in this way:

```shell
syncpackage -r impish-proposed -d unstable -v freeipmi --force
```


## What's next?

You can check the status of the build as with any other upload, from its
"Overview" page (in the format
`https://launchpad.net/ubuntu/+source/<name_of_the_package>`).
Checking the buildlog:

- In the main part of that page you can see the list of built packages for
  every Ubuntu series. You can click on a package's "version" to get to the
  builds for a specific architecture and see the buildlog - e.g. the
  [freeipmi amd64 buildlog](https://launchpad.net/ubuntu/+source/freeipmi/1.6.6-4/+build/21971101/+files/buildlog_ubuntu-impish-amd64.freeipmi_1.6.6-4_BUILDING.txt.gz).

- Visiting the publishing history of the package (in the format 
  `https://launchpad.net/ubuntu/+source/<name_of_the_package>/+publishinghistory`):
  a link at the top right of the "Overview page" - e.g.
  [for freeipmi](https://launchpad.net/ubuntu/+source/freeipmi/+publishinghistory).
