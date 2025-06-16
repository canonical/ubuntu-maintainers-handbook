# Sponsorship for package uploads

Ubuntu encourages contributions from any person in the wider community.
However, direct uploading to the Ubuntu archives is restricted, for obvious
reasons. General contributions for any particular package need to be reviewed
and uploaded by a **sponsor**, which is a person who has upload rights for that
package.

Contributors with proven packaging skills can get upload rights for:
* [certain sets of packages](MembershipInPackageSet.md),
* [all universe packages](MembershipInMOTU.md), or
* [the full archive](MembershipInCoreDev.md).

Canonical employees are treated no differently than general community members
and must follow the same processes for gaining upload rights. Sponsors are
also able to review and upload contributions from others as well.

This page provides guidance on how to use the sponsorship process to get your
changes into Ubuntu.


## Prepare changes for sponsorship

If you follow the guidance elsewhere in this handbook, your changes will be
properly prepared for sponsorship. In general, just keep in mind that someone
else needs to understand what you've done - so "show your working" as they say.

For anything non-trivial, it can be good practice to discuss the change you're
planning with a potential sponsor after you think you know what needs done
but before you've done it. Often, an experienced developer can offer
alternative approaches that may save you time or provide a better result.


## Find a sponsor

There are two formal ways to seek sponsorship, and one informal.

The first is by filing a [Merge Proposal (MP)](MergeProposal.md) with 
`canonical-<your-team>` (e.g. `canonical-server-reporter` for server team 
members) set as a reviewer. Make sure to mention in your MP comments that 
you're also in need of sponsorship.
If the reviewer has upload rights they can take care of sponsoring the upload
as well.

A second, more traditional approach is to
[file a bug report in Launchpad](https://bugs.launchpad.net/ubuntu/+filebug),
attach your changes as a
[debdiff](https://manpages.ubuntu.com/manpages/en/man1/debdiff.1.html),
and then subscribe `ubuntu-sponsors` (or `ubuntu-security-sponsors` for
security issues). This approach is generally used only if a package is not in
`git-ubuntu` or if an MP can't be generated for some reason.

Informally, you can also try approaching possible sponsors via chat or email
and directly asking for sponsorship. This can be helpful if you get no
response from formal requests, or in the case of urgent issues, or if you want
to find sponsors outside your usual circle.

Canonical employees will typically have ready sponsors from their team mates.
However, sponsors can also be found elsewhere in Canonical or in the larger
community. Having a diversity of sponsors can be useful when
[applying](Reference/PathToUploadRights.md) for [MOTU](MembershipInMOTU.md)
and [core-dev](https://github.com/canonical/ubuntu-maintainers-handbook/blob/main/MembershipInCoreDev.md),
since it will demonstrate breadth of your experience and trustworthiness.


## Tracking for endorsements

You should also keep good notes of who has
[sponsored for you](https://udd.debian.org/cgi-bin/ubuntu-sponsorships.cgi),
which packages they sponsored, and what team or part of the distribution they
work on. These notes will be helpful both for finding sponsors in the future,
and as endorsers on your future applications for upload rights.

## Sponsor a package

This is not very different than uploading your own `.changes` file, but after
ensuring that the upload follows all quality standards you want to sign the
content of the proposing person with your key.

To achieve that, the tools used in package build (like `dpkg-buildpackage`
and `debsign`) need to know about that desire. Otherwise these tools would
identify the person that is referred in the changelog stanza and try to sign
with the key of this person.

To define the key you want to be used for signing you can:

* Set the environment variable `DEB_SIGN_KEYID`, see the [dpkg-buildpackage man page](https://manpages.ubuntu.com/manpages/noble/man1/dpkg-buildpackage.1.html) for more.
* Or where desired add the argument `-k${GPGKEY}` assuming you have your key in that variable and in long (>=16) format.
