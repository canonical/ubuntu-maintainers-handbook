Ubuntu Maintainer's Handbook
============================

This handbook explains how to do the common tasks of an Ubuntu package maintainer.  In particular, it explains how the git-ubuntu tool-suite is used for working with Ubuntu packages.

Note that this isn't a policy document; the official policies will be linked to where possible and should be referenced for the officially correct ways to do things.  Instead, this is intended to serve as a tutorial style introduction to help new Ubuntu packagers get up to speed.

Similarly, this handbook doesn't attempt to cover topics comprehensively.  There are many ways to do things, and some alternate approaches may be faster or better in special cases, but this will try to provide the reader with a solid working knowledge of one robust process for accomplishing work items.

Sections
--------

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
   - [Doc Triage](DocTriage.md)
   - [Bug Report Responses](BugReportResponses.md)
 * Requesting Upload Rights
   - [PackageSet](MembershipInPackageSet.md)
   - [MembershipInMOTU](MembershipInMOTU.md)
   - [MembershipInCoreDev](MembershipInCoreDev.md)
 * Quick References
   - [Check Lists](CheckListsSheets.md)
     + [Bug Fix Checklist](BugFixingCheckList.md)

