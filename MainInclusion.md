Main Inclusion
==============

Packages in the Ubuntu archive are released in one of four components: main, restricted, universe and multiverse. Of these, most Ubuntu package maintenance work takes place in main and universe, and the work of Canonical's staff is focused on the main component.

All packages in main must be supported by a relevant Ubuntu development team. We use Launchpad's "team bug subscription" feature to communicate this status. For example, the `apache2` package is in main, and supported by the Ubuntu Server Team, so in Launchpad the `~ubuntu-server` team has a team bug subscription on the `apache2` package.

See https://wiki.ubuntu.com/MainInclusionProcess for details about main inclusion generally in Ubuntu.

Ubuntu Server Team subscription maintenance
-------------------------------------------

On the Ubuntu Server Team, we use git to maintain the list of packages for which we wish to indicate support through Launchpad's team bug subscription mechanism.

The use of git, instead of just adding and removing the subscriptions in Launchpad, allows us to:

 * Avoid accidental additions and deletions.
 * Maintain records about what we did and didn't support in the past.
 * Note our rationale when we make changes.
 * Examine the past when considering future changes.

### Process for making changes to the Ubuntu Server Team package subscription list

This process is to be used when the [main inclusion process](https://wiki.ubuntu.com/MainInclusionProcess) calls for a package to be subscribed by the Ubuntu Server Team.

The git repository can be found [here](https://git.launchpad.net/~canonical-server/+git/team-subscriptions).

First, prepare the change locally with a git commit that makes the change. Include the rationale for the change in the commit message, using links to other resources (eg. the MIR bug) as appropriate. Make sure that the commit message only contains information that can be made public.

If the change is mechanical and uncontroversial, then go ahead and push the change with no merge proposal required. For example, if `php7.2` was already in the list and `php8.0` needs to be added, or if a source package has been split into two with no other significant changes.

If the change fundamentally changes the set of packages that we support from a user perspective, then file a merge proposal for the change, and request a review from `~canonical-server`.

If in doubt, please file a merge proposal anyway.

Once a change has landed into the git repository, a cron job will automatically enact the change in Launchpad itself within 24 hours. All automated changes made are logged to the [ubuntu-server mailing list](https://lists.ubuntu.com/mailman/listinfo/ubuntu-server).

#### Reviewing a merge proposal to change the set of supported packages

Before the merge proposal is approved, the change should be discussed by the Canonical Server Team. When adding packages, they must consider if they are prepared to take on the additional work of supporting that package over many releases and therefore many years (since main inclusion for just one release is generally inappropriate). When removing packages, they must consider if the loss of support from the Ubuntu Server Product is appropriate when balanced against the burden of supporting those packages.

### Further information and rationale

In the future, we might be able to enrich the subscription data further, such as to indicate specific releases that we support for particular packages. This becomes relevant when we change support status for a package. For example, after we have decided to stop supporting a particular package in main, moving it to universe, we must still retain the subscription until the final release in which the package was supported reaches EOL.

We use `~canonical-server` as the owner for this repository because:

 * We have automation driven from the data, which means we must be able to trust the input.
 * `~ubuntu-server`, which would be the logical place to keep data about `~ubuntu-server` subscriptions, has an open membership policy, which means anybody on the Internet might be able to subvert our script if we had stored it there.
 * Ultimately, Canonical have the authority in Ubuntu to decide what is and isn't in main, so restricting changes to Canonical staff makes sense.
 * We don't perceive a need to restrict maintenance to only uploaders.

Note that this repository is public because the set of packages in main is public. Rationale, however, sometimes might contain private information. For example, we might decide to put a package into main on the basis of customer demand, but specific customer identities should generally remain private. When writing git commit messages, you should avoid including private information. However, references to internal resources such as ticket numbers are acceptable in git commit messages in this repository (unlike most other Ubuntu processes). This is because main inclusion decisions in Ubuntu are owned and driven by Canonical specifically, unlike most other development processes in Ubuntu.
