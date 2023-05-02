Committing Your Changes
=======================

Add all of your changed files to git. For example:

    $ git add debian/patches/my-changes.patch debian/patches/series



The Commit Message
------------------

Structure your git commit message in changelog format to make later steps easier:

      * changed/or/added/files: [how it was fixed], [thanks to original author]
        (LP: #12345678)

Use "Thanks to" when you are not the author of the code being submitted.

Maximum column width is 79.

Example:

```
      * debian/patches/my-changes.diff: Reroute the inertial dampeners
        through the auxiliary power relay. Thanks to Miles O'Brien
        <miles.obrien@ds9.fed>. (LP: #19999999)
```

This style of commit message makes integration with the other tools you'll be using easier because it's already in changelog format. The `(LP: #12345678)` at the end will auto-close that bug once the package is published to updates.

For brevity, you'll often see well-known directories abbreviated to their initial letter, such as `d/p/my-changes.diff` for the previous example.  A change affecting multiple files can list them as comma-separated paths, as shown in the next example.  Sub-bullets '-' and '+' can be used to help organize the information.

Here is a more complex example:

```
      * Re-calibrate the long range sensors to detect quasi-particle anomalies.
        - d/p/important.patch: Reroute the inertial dampeners
          through the auxiliary power relay. Thanks to Miles O'Brien
          <miles.obrien@ds9.fed>. (LP: #19999999)
        - d/control, d/rules: Adjust long range sensor power levels
          + Increase power input draw from auxiliary power relay.
          + Set scan range to subatomic frequencies.
```


Modifying the Changelog
-----------------------

For the changelog entry, often you can reuse or even cut and paste text from the git commit or commits you've already made.  But there are some specific changes that need done unique to the changelog.

Some guidelines for writing an effective changelog entry:

  * For each thing you change, briefly explain *why* as well as *what*.  Remember that *what* can always be determined by a future reader by digging into the diff, but *why* may be unknowable except from your message.  This can assist decision-making about if your change is still relevant in new merges, or if it is important enough to backport to other Ubuntu releases
  * Use nested bullet points indented by two spaces with hanging indentation on wrapped lines. Bullet characters are *, - and + for levels 1, 2 and 3.
  * The contents of bullet points are free-form English text, so use normal grammar, punctuation, spaces, full stop at the end, etc. Exception: for technical specifics like filenames, matching technical case exactly, or otherwise breaking grammar rules to avoid ambiguity is appropriate. Brevity in grammar is fine, but not at the cost of losing information. If in doubt, more verbose is better, although if you wish you can always summarize entries and put longer explanations in the relevant bugs.
  * Use exactly "LP: #NNNNNNN" or "(LP: #NNNNNNN)" to reference bugs that are being fixed by the upload. The version using brackets is useful to neatly fit the bug references into standard English sentences. Bug references must be whitespace-perfect as they are picked up by regular expressions in the tooling to ultimately auto-close the bugs.
  * If you want to mention a bug but that wasn’t fixed by this change you can break the automation that would auto-close it with removing the colon  "(LP #NNNNNNN)".
  * For Debian the same is “Closes: #NNNNNN” and again automation can be “avoided by breaking the regular expression like "Closes #NNNNNN".  If a Ubuntu contribution also fixes a related Debian bug it is good practice to tag the closing of the Debian bug as well. That way if they pick our change it automatically closes their bug as well then.
  * For standard (non-merge) uploads, one bullet point per logical thing changed is appropriate. Use sub-items for more detail or if this otherwise helps with clarity. If the set of changes is large, consider categorizing the entries with top level bullet points.
  * For merge uploads, the convention is:
    - One top level bullet point to introduce the merge with sub-items documenting each logical item that was present in the previous Ubuntu delta which is still present in the new Ubuntu delta.
    - If items have been dropped from the previous Ubuntu delta, then one top level bullet point with sub-items describing what was dropped.
    - Further top level bullet points for each new change made that is not a carry-over or drop from the previous delta.


### Version String Format

See https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation#Update_the_packaging

Most version changes, except upstream merges, will follow this format:

| Previous version             | Security update                               |
| ---------------------------- | --------------------------------------------- |
| 2.0-2                        | 2.0-2ubuntu0.1                                |
| 2.0-2ubuntu2                 | 2.0-2ubuntu2.1                                |
| 2.0-2ubuntu2.1               | 2.0-2ubuntu2.2                                |
| 2.0-2build1                  | 2.0-2ubuntu0.1                                |
| 2.0                          | 2.0ubuntu0.1                                  |
| 2.0-2 in two releases        | 2.0-2ubuntu0.11.10.1 and 2.0-2ubuntu0.12.04.1 |
| 2.0-2ubuntu1 in two releases | 2.0-2ubuntu1.11.10.1 and 2.0-2ubuntu1.12.04.1 |

For example, suppose the current version in bionic is `3.3.0-1` and the current version in cosmic is `3.3.0-1ubuntu1`. Versions in bionic should always be "lower" than versions in cosmic so that upgrading to cosmic uses the cosmic version of the package. In this case, we can use `3.3.0-1ubuntu0.1`.


### Reconstructing the Changelog with Git Ubuntu

Optionally, you can use `git-ubuntu.reconstruct-changelog [base branch]` to construct a changelog entry:

    $ git-ubuntu.reconstruct-changelog pkg/ubuntu/bionic-devel

This modifies `debian/changelog` like so:

    mypackage (3.3.0-1ubuntu1) UNRELEASED; urgency=medium

      * debian/patches/my-changes.diff: Reroute the inertial dampeners
        through the auxiliary power relay. Thanks to Miles O'Brien
        <miles.obrien@ds9.fed>. (LP: #19999999)

     -- Karl Stenerud <karl.stenerud@canonical.com>  Mon, 20 Aug 2018 07:58:52 -0700

You'll have to manually fix up anything that reconstruct-changelog got wrong:

 * Version is wrong (should be `3.3.0-1ubuntu1.1`).
 * Name and email may be wrong if you don't have `DEBFULLNAME` and `DEBEMAIL` set in your env.
 * `UNRELEASED`: Change this to the release this change is for (example, `bionic`).

Note: In case a package was as of today just synced over from Debian to Ubuntu and never really touched,
      but is now modified for the very first time (e.g. to include a patch for a SRU)
      and is with that becoming Ubuntu specific from now on,
      the "update-maintainer" script needs to be called after the changelog entry was created.
      This will update the 'Maintainer' entries in the debian/control file.
      Hence in such a case not only the debian/changelog needs to be committed, but also the modified debian/control. 


### Reconstructing the Changelog with DCH

Simply run `dch` from inside of the repository and follow instructions.

Note: Use either git-ubuntu.reconstruct-changelog or dch!


Committing the Changelog
------------------------

    $ git commit -m changelog debian/changelog



Pushing Your Changes
--------------------

    git push [launchpad name] [branch name]

For example:

    $ git push kstenerud mypackage-fix-lp19999999-inertial-dampeners-bionic

To see the repository in launchpad, go to your code page using this template: https://code.launchpad.net/~your-username/+git
