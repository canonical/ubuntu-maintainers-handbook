Sponsorship for Package Uploads
===============================

Ubuntu encourages contributions from any person in the wider community, however direct uploading to the Ubuntu archives is restricted, for obvious reasons.  These general contributions need to be reviewed and uploaded by a *sponsor*.

Trustworthy contributors with proven packaging skills can obtain upload rights for [certain sets of packages](MembershipInPackageSet.md), [all universe packages](MembershipInMOTU.md), or [the full archive](MembershipInCoreDev.md).  Canonical employees are treated no different from general community members and must follow the same processes for gaining upload rights.  These people are also able to review and upload contributions from others as well.

This page provides guidance in how to use the sponsorship process to get your changes into Ubuntu.


Preparing Changes for Sponsorship
---------------------------------

If you follow the guidance elsewhere in this handbook, your changes will be properly done for sponsorship.  In general, just keep in mind that someone else needs to understand what you've done - so "show your work" as they say.

For anything non-trivial, it can be a good practice to discuss the change you're planning with a potential sponsor after you think you know what needs done but before you've done it.  Often, an experienced developer can offer alternative approaches that may save you time or provide a better result.


Finding a Sponsor
-----------------

There are two formal ways to seek sponsorship.  The first is by filing a Merge Proposal with 'canonical-server' (or other appropriate team) set as a reviewer.  Make sure to mention in your MP comments that you're also in need of sponsorship.  If the reviewer has upload rights they can take care of sponsoring the upload as well.

A second, more traditional approach is to [file a bug report in Launchpad](https://bugs.launchpad.net/ubuntu/+filebug), attach your changes as a [debdiff](http://packaging.ubuntu.com/html/traditional-packaging.html#creating-a-debdiff), and then subscribe *ubuntu-sponsors* (or *ubuntu-security-sponsors* for security issues).  This approach is generally used only if a package is not in git-ubuntu or if a MP can't be generated for some reason.

Informally, you can also try approaching possible sponsors via chat or email and directly asking for sponsorship.  This can be helpful if you get no response from formal requests, or for urgent issues, or if you want to find sponsors outside your usual circle.

Canonical employees will typically have ready sponsors from their team mates.  But sponsors can be found elsewhere in Canonical or in the larger community.  Having a diversity of sponsors can be useful when applying for MOTU and core-dev, since it will demonstrate breadth of your experience and trustworthiness.

Tracking for Endorsements
-------------------------

You should also keep good notes of who has [sponsored for you](https://udd.debian.org/cgi-bin/ubuntu-sponsorships.cgi), which packages they sponsored, and what team or part of the distribution they work on.  These notes will be helpful both for finding sponsors in the future, and as endorsers on your future applications for upload rights.
