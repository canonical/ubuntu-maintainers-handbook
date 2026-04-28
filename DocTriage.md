# Documentation triage

Documentation is hosted in [github](https://github.com/canonical/ubuntu-server-documentation)
and from there rendered to PDF and the online view at [Ubuntu Server docs](https://documentation.ubuntu.com/server).
This kind of structure gave us better interaction rates and contributions than
on Discourse, but to make them worthwhile we need to ensure we pick up and act
on what we get there

## Tooling

For all [triage](Triage.md) input, we use [`startriage`](https://github.com/ubuntu/startriage).


## Lists

For the daily triage rotation you only have to look at those issues or pull
requests created or updated in the time your duty covers.

- [Pull requests](https://github.com/canonical/ubuntu-server-documentation/pulls?q=is%3Apr+is%3Aopen+sort%3Aupdated-desc)
- [Issues](https://github.com/canonical/ubuntu-server-documentation/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc)

Check what the interaction is about and help them to progress and land or resolve.

## Tag usage in Doc triage

Right now we have a few tags defined that help us to quickly see in what state
an issue/PR was last time as well as later on allowing volunteers to better find
how they could help us.

* Review needed
  * `review: technical`: used by our Author, submitter and volunteer triagers, flagging that to go forward this needs a technical or even subject matter expert reviewer.
  * `review: wording`: used by our Engineers, submitter and volunteer triagers, flagging that to go forward this needs a review of an Author.
* For:
  * `Open Documentation Academy`: used by triagers to declare a case as a good candidate to be resolved by volunteers of the Open Documentation Academy.
  * `server: ToDo`: mostly matching the [server-todo](BugTriage.md#tagging-server-todo) launchpad tag, representing it is "valid and we should work on it"
* `state:*` tags used to represet the state of a case 
  * `state:incomplete`: this isn't actionable without further into being provided. This closely resembles an incomplete state in launchpad [Bug triage](BudTriage.md).
  * `state:wip`: This PR is up for awareness, but not ready to be reviewed yet.
  * `state:duplicate`: This issue or pull request already exists, used represent why a case was closed.
  * `state:invalid`: This doesn't seem right, used represent why a case was closed.
* Classifiations like `code:*` and `content:*` can be helpful more easily pick cases one can help with, adding them is optional
