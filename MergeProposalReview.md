# Merge Proposal (MP) reviewing

Before sponsoring or uploading we established a review culture helping us
to provide consistent good quality uploads. To do so we propose our changes
as [merge proposals aganist `git-ubuntu` branches](MergeProposal.md).

In review we check for:
* Issues in the formal content
* To follow our stylistic (team) choices
* Common issues
* If warranted for the case we provide further testing

We use symbols that we put into a template to check the most common cases based
on lessons learned. The symbols we put in are representing the POV of the
reviewer to a particular aspect.

* `x` - OK
* `-` - Not OK - Reasons will be outlined in the lines below
* `?` - Not neccesarily bad, but worth to clarify - Question will be outlined in the lines below
* `N` - Not applicable to this case, e.g. checks only valid for new delta added which might not always be the case
* `S` - Skipped by the reviewer as it seemed not mandatory or commensurate for this case. An example might be the component mismatch check which is helpful for a complex merge, but there is a tradeoff that goes too far for a SRU bug fix not changing any d/* content.

## Check the new changelog stanza

* Contains a bug pointer like `(LP: #12345678)` to all associated cases
* Correctly formatted entries according to the [changelog policy](https://www.debian.org/doc/debian-policy/ch-source.html#debian-changelog-debian-changelog) and our [hints](PackageMerging.md#fix-the-changelog) for merges
* Lists all changes made - the changelog shall be a complete representation
* Proper version - check against [Version Strings](VersionStrings.md)
* Proper release - `dch` or habits could have selected the wrong one
* Proper author and email

## Ensure Documentation/release Notes are updated

* We always did enqueue things that eventually need to be mentioned in the release notes, this check in the MR review is a reminder about that.
* If the update/merge has implications that need to be documented we always updated documentation alongside changes. Sometimes things get in via a sync or upstream changes, not by us - as we spot these, make sure to spawn a proper backlog tracker for it.

## Check for indirect changes

* Some changes can imply that the packaging needs to be adapted. Check content and release notes if there are any changes like that.
* When merging from Debian or Upstream it is worth checking if there are even newer versions that would be worth it to go for.
  * Bad things happen, check upstream if a release has been withdrawn or needs an immediate fix-up due to unintentional breakage.
* Ensure that the changes in Debian do not imply that we need to update the delta we carry (do not be fooled by applying cleanly)
* Ensure update maintainer has been run

## Check the old delta

* Ensure that everything we dropped really can be dropped
  * vice versa, there could be more that could be dropped (often needs a little test to verify which is suggested to the owner of the MR and only rarely done by the reviewer)

## Check the new delta

* Do the patches [follow DEP-3](http://dep.debian.net/deps/dep3/)
  * You can use dep3changelog to verify the headers, as well as generate a changelog entry.
* Does the patch content and name [follow our additional style choices](DebianPatch.md)?
  * This is not a strict check, but consitency helps to maintain packages together.
* Do the patches match what is (proposed) upstream?
  * To be clear, if a certain case asks for violating this rule it is ok, but it should be a conscious decision not an accident
  * The intention is to avoid deviating the code we maintain too much from how the project continues to evolve (hard to maintain well over long periods).
* Are the patches applied the right way according to `debian/source/format`?
* Are all changes either:
  * Forwarded to Debian or Upstream so that everyone benefits and we can some day make this a sync again.
  * Or if they are Ubuntu only choices, marked like that so the next packager is not wondering if we want to keep or submit it? (See [`UD-forwarded` for commits](CommittingChanges.md#the-commit-message) and [`Forwarded` for patches](DebianPatch.md#the-patchfile-header).
  * If old delta added in the past does not have such info and you spent the effort to check the history and reasoning to make good decisions, please add the same info to that old-delta.

## Check for Git/maintenance

* Changes are logically split into separate commits (to ease future merges and cherry picking to other releases)

## Check for Build and Test

* Ensure the build in the PPA is ok on all architectures it is meant to build.
* If this is an SRU consider checking that the SRU template in the bug is OK. Test instructions especially are often only understandable for the reporter. We want them to be good before SRU review.
* If applicable relative to the changes made, consider if tests should be added, adapted or extended.
* Too many cases have been caught late and then intertwined in proposed-migration. Testing autopkgtest on the PPA helps to be confident before entering the archive.
* Especially for complex merges of packages in main, check if we accidentially or intentionally added new dependencies that would cause component mismatches, so we can adapt or prepare a [MIR](MainInclusion.md) before being entangled in the archive.
* Depending on the case the test PPA might be used to install and verify functionality of the builds.
  * This is very much an optional service the reviewer can, but does not have to, provide.
  * If possible this should not be done by humans, but autopkgtests. And if it isn't consider adding them. But still sometimes there are situations where one wants to do a quick manual verification to be sure.


# Review template

The following template may be useful when submitting reviews:

```
Review Symbols:
x = OK
- = Not OK
? = Question
N = Not applicable
S = Skipped

* Changelog:
  - [ ] Changelog entry has correct version and targeted codename
  - [ ] Correct formatting of changelog items
  - [ ] Bug references correct
  - [ ] Old content and logical tag match as expected (Package Merge)

* Release notes and Documentation
  - [ ] Added, updated or enqueued relevant documentation.
  - [ ] Added, updated or enqueued relevant release notes.

* Package Merge - indirect changes:
  - [ ] No upstream changes that need adapting due to Ubuntu's design
  - [ ] No further upstream version/changes to consider
  - [ ] Debian changes are compatible with the Ubuntu implementation
  - [ ] update-maintainer has been run

* Package Merge - old delta:
  - [ ] Dropped changes are ok to be dropped
  - [ ] Nothing else to drop
  - [ ] Old delta was forwarded to upstream/Debian or marked as Ubuntu-only

* New delta in debian/*:
  - [ ] new changes in debian/* are OK
  - [ ] New delta was forwarded to Debian or marked as Ubuntu-only

* New patches:
  - [ ] No new patches added
  - [ ] Patches match those proposed/committed upstream
  - [ ] Patches correctly included in Debian/patches/series
  - [ ] Patches have correct DEP-3 metadata
  - [ ] Patches follow our style choices
  - [ ] New code not from upstream was forwarded or marked as Ubuntu-only

* Git/maintenance:
  - [ ] Commits are properly split (more important on -dev than on SRUs)

* Build/Test:
  - [ ] Build is OK
  - [ ] This is an SRU, the validation instructions are ok
  - [ ] Testcases added or adapted (N/A if not strictly required or already present)
  - [ ] autopkgtest against the PPA package passes (if possible, evidence was provided already)
  - [ ] Based on PPA builds and the build-log, no new component mismatch expected
  - [ ] Verified PPA package installs/uninstalls
  - [ ] Verified function manually
```
