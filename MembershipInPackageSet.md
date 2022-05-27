Membership in Package Set
-------------------------

Upload rights can be given for certain package-sets, such as 'server' or 'desktop'.  Its original design intent was for individuals who will only be working on a very small set of packages.

For this reason, some people skip this level and head straight for MOTU or core-dev instead; if you already have strong packaging experience via another distro you certainly can consider doing similarly.

That said, even if you intend to eventually apply for core-dev, gaining package-set first can be an effective way to build towards those roles, allowing you to upload your own work (within limits) and participate in reviews and [sponsorship of co-workers](Sponsorship.md).


Application Process
-------------------

0.  *Important*  [Check the DMB agenda](https://wiki.ubuntu.com/DeveloperMembershipBoard/Agenda) to see when the DMB next meeting is, and to check the queue of applications.  Only 2 applications are considered each meeting so if there's a queue, make sure to reserve a spot for when you think you'll be ready for consideration.

1.  Create a short wiki page for yourself, at https://wiki.ubuntu.com/FirstnameLastname/, that introduces yourself and your past Ubuntu-related work, and includes a Contact Information section with at least your IRC nick and Launchpad ID.  You can look at other people's pages for other ideas of things to include.  You'll be reusing this text in your membership applications later.

2.  Training and Preparation, as needed.  See the following section for details.

3.  Prepare your Application Form.  Load https://wiki.ubuntu.com/FirstnameLastname/PackagesetDeveloperApplication, replacing FirstnameLastname with what you used in step 0.  Select "UbuntuDevelopment/DeveloperApplicationTemplate" from the list of templates, and follow its directions on how to fill it out; you can look at [past applications](https://wiki.ubuntu.com/Home?action=fullsearch&context=180&value=DeveloperApplication&titlesearch=Titles) such as [Paride's](https://wiki.ubuntu.com/ParideLegovini/UbuntuServerDeveloperApplication) as examples.  Specify at the top that you're "applying for upload rights to the Ubuntu Server package set", replacing "Ubuntu Server" as appropriate, or listing out the exact, specific packages.

4.  Collect Endorsements from people who have sponsored your packages.  Ask on your team's regular discussion channels, and individually contact each of your sponsors not in your team.  It's good form to only ask people who have sponsored multiple packages for you, or that have worked with you on particularly tricky packaging efforts.  You want to strike a good balance between quality and quantity here.

5.  [Apply for team membership](https://wiki.ubuntu.com/DeveloperMembershipBoard/ApplicationProcess) by announcing on the devel-permissions@ mailing list and adding your name to the DMB agenda (if you haven't already).

6.  Review past meeting logs to get a sense for what to expect.

7.  Attend meeting, answer questions, and receive your votes.


Training and Preparation
------------------------

We're going to describe an idealized training program here, however no application is exactly the same, and as such there is a lot of flexibility in expectations.  This is particularly true for packageset given that it is by definition of limited scope - just be prepared to give extra justification if you diverge substantially.

Ideally, you should have a solid mastery of the [basic packaging skills](https://packaging.ubuntu.com/html/) for Debian/Ubuntu distributions, including the following:

  * [Fixing bugs in packages](PackageFixing.md)
  * Building binary packages from source using sbuild or debuild in a chroot or lxc environment
  * Creating the initial packaging for new software
  * Merging updates from Debian
  * Backporting patches
  * Forwarding bugs and patches to Debian and to upstream maintainers
  * Stable Release Update process

Make sure you've done each of the above items at least a couple of times.  If you're not comfortable with any of these, plan on doing some additional practice with them.

You should also work towards understanding some more advanced packaging topics:

  * Purpose of the [different files in debian/](https://packaging.ubuntu.com/html/debian-dir-overview.html)
  * [Debian policy](http://www.debian.org/doc/debian-policy/)
  * [Ubuntu's release process](https://wiki.ubuntu.com/UbuntuDevelopment/ReleaseProcess), including the
    [freeze exception process](https://wiki.ubuntu.com/FreezeExceptionProcess)
  * Running [Autopkgtest](PackageTests.md)
  * Troubleshooting [migration of packages](https://wiki.ubuntu.com/ProposedMigration) from -proposed

While you may not have direct experience with some or most of these topics, you should at least be conversant in all of them conceptually.

In addition to Ubuntu packaging, you will need to have some technical expertise with the subset of software you'll be working on, and the processes and procedures standardized for them.  For example, for the Ubuntu server team you would need to have experience with technologies such as systemd and networking, and processes such as using git-ubuntu for package maintenance.

Keep in mind the DMB's perspective will follow the "Need to unblock" principle:  They want to approve applications that will help either reduce contributor friction or to save work of sponsors.  Establishing yourself as an area expert that is a resource for contributors helps prove the former, and your volume and frequency of uploads justifies the latter.

Collaboration with the upstream(s) related to the software within your packageset is important as well.  Gain experience with your upstreams' bug reporting processes, patch review processes, and testing/CI systems if you haven't so far.  In general, the narrower the number of packages you plan to focus on, the stronger your collaboration with upstream you should be able to show.

And on the flip side, if you are a Canonical employee you will need to be able to negotiate the distinction between Ubuntu governance and Canonical priorities, within your area of focus.  Sometimes these can appear to collide and require a certain level of diplomacy to find solutions that work well for both sides.  The less experience you have in Debian or other open source communities, the more thought you'll want to put into this.

Finally, you'll know you're past ready for applying if anyone ever asks, "How do you not already have upload rights??"


Further Information
-------------------

  * https://wiki.ubuntu.com/RobieBasak/DMB/CoreDev
  * https://wiki.ubuntu.com/UbuntuDevelopers

Once you've been granted packageset upload permissions, there are at least three distinct directions you could embark on, depending on your goals:

  * MOTU, if you want to get depth into the Ubuntu packaging world.
  * Membership in packaging teams at other distributions, if you want to broaden the reach of your software.
  * Upstream maintainership in your project(s) of interest, if you want to pursue more focus in the development of the software itself.

