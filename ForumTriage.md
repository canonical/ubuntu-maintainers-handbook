# Forum triage

In order to maintain a line of communication with the community, forum triage has been added as a task to do during [regular triage](Triage.md).

## Tooling

For all [triage](Triage.md) input, we use [`startriage`](https://github.com/ubuntu/startriage).


## Types of updates

Forum updates fall into a few categories that require different levels
of action.

### New pages

When someone creates a new page, which in Discourse is a new Topic under the
relevant category, this will show up as a `+` next to the page title and author
name. For example:

```text
+Changing package files [Lena Voytek, 2023-01-11]
```

### Page edits

If a given page has been modified in any way, a `*` will show up alongside the
page title and the name of the editor. This looks like:

```bash
*Ubuntu Server tutorials [Sally Makin, 2023-04-12]
```

Such changes usually only happen on wiki style pages and therefore
are rare, but might change the context and therefore is worth to be
re-evaluated as if it would be a new page.

### Community comments

When a community member comments on a page, it will show up with a `+`
alongside a link and author name for the comment. A `*` may show up instead if
the comment was modified. This line will show up as part of a tree of replies
under its given page name.

Multiple updates to a conversation may also have happened, such as:

```bash
SSHd now uses socket-base… [Steve Langasek] 
├─ 77972 [Paride Legovini] 
│  └─ 80966 [Saxl] 
│     └─ 81043 [Leszek A. Szczepanowski] 
│        └─ 83849 [Nebulabox] 
│           └─ 84773 [Michael Utech] 
│              └─ +88070 [Piradix, 2023-04-05] 
├─ +88748 [Dave Ruedeman, 2023-04-19] 
└─ +88948 [Darren Embry, 2023-04-23] 
└─ *88978 [Steve Langasek, 2023-04-23] 
```

Look through the added and updated comments by clicking the attached links.

### Team responses

Responses to community comments by someone on the team will look the same as
other comments. In this case, check to make sure the conversation was resolved,
and provide extra input if needed.

For all kind of interactions the update should be checked to see if we need to
answer or to act on it. For example actions might be needed for these:

- A misdirected bug report -> guide them to report it well
- Something that is a suggestion to be considered in the documentation -> suggest them to contribute there or if it for us file it as documentation idea for us.
- Misbehavior -> please moderate
- Happen to find an unanswered question -> consider to reply with the relevant information if you can help.

Gladly the most common case are healthy community discussions - in which case we
are allowed, but not strictly required, to step in.

If an action is required but can not be taken immediately as it will take
too much time, reply to the community member to inform them that we will get
back to them and thank them for taking the time to leave a comment. This helps
to set expectations and prevents our community members from feeling ignored
(even if we cannot provide an answer straight away).
If you do so, ensure that we have an artifact that allows us to remember, like
a bug, a github issue in documentation or a jira backlog entry.

If any of the above is worthwhile to be aware for a wider audicence, consider
mentioning it in your [triage report](Triage.md#awareness-of-the-triage).
