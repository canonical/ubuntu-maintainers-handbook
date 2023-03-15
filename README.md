# Ubuntu Maintainer's Handbook

This handbook explains how to do the common tasks of an Ubuntu package maintainer.  In particular, it explains how the git-ubuntu tool-suite is used for working with Ubuntu packages.

Note that this isn't a policy document; the official policies will be linked to where possible and should be referenced for the officially correct ways to do things.  Instead, this is intended to serve as a tutorial style introduction to help new Ubuntu packagers get up to speed.

Similarly, this handbook doesn't attempt to cover topics comprehensively.  There are many ways to do things, and some alternate approaches may be faster or better in special cases, but this will try to provide the reader with a solid working knowledge of one robust process for accomplishing work items.

## Sections

 * [High Level Concepts](Concepts.md)
 * [Setting up Your Environment](Setup.md)
 * Common Tasks
   - [Making Patch Files](DebianPatch.md)
   - [Committing Your Changes](CommittingChanges.md)
   - [Package Building](PackageBuilding.md)
   - [Package Tests](PackageTests.md)
   - [Merge Proposals](MergeProposal.md)
   - [Sponsorship](Sponsorship.md)
   - [Syncing from Debian](Syncs.md)
 * Package Maintenance
   - [Package Fixing & SRUs](PackageFixing.md)
   - [Package Merging](PackageMerging.md)
   - [Main Inclusion](MainInclusion.md)
   - [Proposed Migration](ProposedMigration.md)
   - [Transitions of Libraries and Languages](Transitions.md)
   - [Salsa Package Maintenance](SalsaDualMaintenance.md)
 * Debugging
   - [Apport Crash Files](DebugApportCrash.md)
 * Reviewing
   - [Reviewing Merge Proposals](MergeProposalReview.md)
   - [Bug Triage](BugTriage.md)
   - [Bug Report Responses](BugReportResponses.md)
 * Requesting Upload Rights
   - [PackageSet](MembershipInPackageSet.md)
   - [MembershipInMOTU](MembershipInMOTU.md)
   - [MembershipInCoreDev](MembershipInCoreDev.md)
 * Quick References
   - [Check Lists](CheckListsSheets.md)
     + [Bug Fix Checklist](BugFixingCheckList.md)

## Contributing

We welcome everyone who wants to improve the Ubuntu documentation! Whether you've found a typo, have a suggestion for improving existing content, or want to add new content, we'd love to hear from you.

To contribute, simply submit a pull request with your changes or create an issue to describe your proposed changes. We'll review your contribution as soon as possible, but please note that our Ubuntu maintainers often have full inboxes, so it may take some time before we can get to your request.

If you're interested in becoming a more active member of the Ubuntu community, we encourage you to participate in our relevant meetings and channels. You can attend one of the weekly [#ubuntu-meeting](https://wiki.ubuntu.com/BeginnersTeam/Meetings) sessions on our [IRC](https://wiki.ubuntu.com/IRC/ChannelList) or ask in a different channel that is appropriate for your topic.

### Code of Conduct

Please note that we have a code of conduct in place for all contributors to this repository. By contributing, you agree to abide by the [Ubuntu Code of Conduct](https://launchpad.net/codeofconduct)

Thank you for considering contributing to the Ubuntu documentation!