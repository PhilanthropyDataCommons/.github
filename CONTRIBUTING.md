# Contribution Guidelines

These technical guidelines (mostly) apply to (most) [Open Tech
Strategies](https://opentechstrategies.com/) software projects.  A
given project may extend or override these guidelines as needed.

Please see also the [Code of Conduct](CODE_OF_CONDUCT), which applies
everywhere.

## Standard Pull Request Model

We use the typical "[merge
request](https://docs.gitlab.com/ee/user/project/merge_requests/)"
a.k.a. "[pull request
(PR)](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)"
development workflow.  Below, we'll say "PR" for short.

You're welcome to open a PR in any OTS repository.  We encourage you
to first post about your plans either in an issue ticket (an existing
one or one you create) or in the appropriate discussion forum.  That
discussion forum will vary from project to project.  If you're not
sure where to find the right one, please ask on
[chat.opentechstrategies.com](https://chat.opentechstrategies.com/);
folks there will help steer you to the right place.

By checking first, you can get any important design feedback or
suggestions before you start coding.  It's also okay to ask questions
before you've decided whether or not you'll contribute a change.

## Writing Commit Messages

When composing a commit message, please use [these
guidelines](https://chris.beams.io/posts/git-commit/).  The quick
summary is:

* Limit the subject line to 50 characters 
* Capitalize the first letter of the subject line, but...
* ...Do not end the subject line with a period
* Use the imperative mood in the subject line
* Separate subject from body with a blank line
* Wrap the body at 72 characters
* Use the body to explain _what_ and _why_ more than _how_

The reason for the short initial subject line is to support commands
-- such as `git show-branch` -- that print summary lists of changes
showing just the first line of each commit message, usually prefixed
by some metadata on the left.  (And the reason for leaving the period
off the subject line is thus to save space.)

Think of the commit message is an introduction to the change.  A
reviewer will read the commit message right before reading the diff
itself, so the commit message's purpose is to put the reader in the
right frame of mind to understand the code change.  The level of
detail and specificity used in the message depends on the change.  If
you're unsure, take a look at the repository's history, comparing
commit messages with the corresponding diffs, and use that as a guide.

**Mention related issues:** If the commit is related to one or more
issue tickets, please mention each ticket number in the commit
message, like this: "issue #123".

## Indentation and Whitespace

Please uses spaces not tabs for indentation, and avoid trailing
whitespace.  If a language has a standard indentation amount, use that
amount.  E.g., indent Javascript code by 2 spaces per level, Python
code by 4 spaces, etc.

This repository uses [EditorConfig](https://editorconfig.org/) to help
manage whitespace.  Your IDE probably has an [EditorConfig plugin](https://editorconfig.org/#download).

## Keep Unrelated Changes Separate

Please put logically distinct changes into separate commits.  For
example, this commit message -- although correctly formatted -- should
never happen:

```
  Fix issue #51 latency bug; also, fix formatting
  
  Stop syncing to permanent storage on every write.  That was causing
  latency problems for clients who sent many files in a single request.
  
  Also, fix some longstanding formatting issues (trailing whitespace,
  inconsistent indentation, etc) in the larger surrounding code block.
```

Even though one contiguous region of code was affected, there were
really two logically unrelated changes made to it.  The better way
would be to first commit the formatting fixes by themselves, with a
commit message like this (a subject line is enough for a change like
this -- the commit message does not need a body):

```
  Formatting fixes only; no substantive change
```

...and _then_ make the real change as a separate commit:

```
  Fix issue #51 latency bug
  
  Stop syncing to permanent storage on every write.  That was causing
  latency problems for clients who sent many files in a single request.
```

Having the substantive change in its own commit means that now someone
reviewing the change can see exactly the relevant diff, without being
distracted by mere whitespace changes that don't affect the code's
behavior.  This same reasoning applies even when the "other" change
isn't a whitespace-only change: it's much easier for a reviewer to
comprehend one change at a time than to try to comprehend multiple
changes mixed together.

## Using Repository Templates

Put standard repository templates in the root of the project, not in a
forge-specific subdirectory (e.g., not in a `.github/` subdirectory).

By "standard repository templates", we mean files typically used in
most project repositories, such as `LICENSE.md`, `CONTRIBUTING.md`,
`PULL_REQUEST_TEMPLATE.md` or the `PULL_REQUEST_TEMPLATE/` directory,
and `ISSUE_TEMPLATE.md` or the `ISSUE_TEMPLATE/` directory.  GitHub
originated such templates (see
[here](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors),
[here](https://github.blog/2016-02-17-issue-and-pull-request-templates/),
[here](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates),
[here](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository),
and
[here](https://docs.github.com/en/github/managing-your-work-on-github/about-automation-for-issues-and-pull-requests-with-query-parameters)),
but they are used elsewhere as well now.  Whether a project is
hosted on GitHub or somewhere else, these files should work fine from
the root of the project.

(GitHub's documentation mentions that certain templates offer optional
additional behaviors if-and-only-if they are located in the `.github/`
subdirectory.  OTS projects do not rely on those additional
behaviors.)

Having these files in the top directory of the project offers two main
advantages:

1. New developers will see them more easily there.  This is especially
   important for `LICENSE.md` and `CONTRIBUTING.md`.

2. The repository won't be forge-specific.  This means that later if
   the project decides to host elsewhere, less adjustment would be
   needed.  Possibly some of the templates wouldn't be useful at other
   hosting forges, but that's okay; they can be ignored or used
   manually.  However, note that other forges
   [tend to implement](https://github.com/go-gitea/gitea/issues/5488#issuecomment-449572640) 
   GitHub's functionality anyway, since GitHub has been a
   standard-setter in this area.

## Change Documentation and Tests Together With Code

Whenever possible, please include related documentation updates and
test-suite updates directly in your change, i.e., in the same pull
request as the core source code change.

## Fixing Security Vulnerabilities

If your change fixes a security vulnerability that is not already
public knowledge, please report it privately (in whatever manner is
appropriate for that project) before discussing the change in any
public forums or posting a PR.

### Expunging Branches Once They Are Merged

In OTS repositories, once a branch has been merged to mainline we
usually delete the branch.  This can be done via the web interface, or
from the command line:

    # Make sure you're not on the branch you want to delete.
    $ git branch | grep '^\* '
    * main

    # No output from this == up-to-date, nothing to fetch.
    $ git fetch --dry-run

    # Delete the branch locally, if necessary.
    $ git branch -d some-now-fully-merged-branch

    # Delete it upstream.
    $ git push origin --delete some-now-fully-merged-branch

This only applies to branches in the repositories we manage, of
course.  For your own repositories (including the ones that are cloned
from ours), you decide your own deletion policy, of course.

## Handling license and copyright notices

When creating a new project, put its open source license in a
top-level file called `LICENSE.md`.  (Markdown versions of all the
usual open source licenses are easy to find on the Internet.)

There is no need to put a copyright notice or license header in each
individual source file; the top-level `LICENSE.md` is sufficient for
our purposes.  However, when we are modifying existing third-party
code -- e.g., when our project is based on a forked version of some
other project -- individual source files may already have copyright
notice lines and license headers blurbs.  In those cases, leave the
existing copyright information intact, and just add a new copyright
line in each file that we change.  For example:

```
  Copyright (C) 1999  Upstream Third Party, Inc
  Copyright (C) 2021  Open Tech Strategies, LLC
```

For third-party code that we redistribute together with our project,
leave all of its license information intact, of course, including any
`LICENSE` or `COPYING` or similar file.  When we redistribute
third-party code, we are not changing its license nor claiming
copyright ownership of it; we are just another downstream user of the
code under whatever license it already has.

## Effort Estimation

We sometimes use **TSML** labels to estimate issue size:

* **Tiny**: Trivial.  E.g. fixing a typo in a string, or a color or a
    minor layout issue on a page, or something like that.  Typically
    one does several tiny things in an hour, and it's the testing,
    writing commit messages, etc, that takes the time.

* **Small**: Like tiny but a bit bigger and a bit more complicated.
    Might take up to half a day to fully wrap up, but at every step
    it's obvious what to do.  There is no need for much research, no
    likelihood of non-trivial unexpected complications, etc.

* **Medium**: Takes somewhere from half a day to two days.  Might need
    some investigation, and there is a possibility of unexpected
    complications arising.

* **Large**: Requires some thinking/research/investigation up front,
    has a substantial chance of dragging in unexpected complications,
    and takes multiple days or possibly a week or so.

If something's really large, then we try to break it down into
subtasks, since it's hard to think of a thing that takes multiple
weeks as one single task -- although sometimes that can happen, for
example with a single unified change across an entire code base, like
changing from raw DB queries to using an ORM or something like that.

## Strolls

At OTS, we do "**strolls**".  These are similar to what are called
"[sprints](https://en.wikipedia.org/wiki/Scrum_sprint)" in Agile/Scrum
terminology, but at the end of a stroll we feel energized and
refreshed, not tired and sweaty.

Not all strolls necessarily have the same duration, even within the
same project.  We choose a stroll's length based on the work to be
done, how everyone is feeling after immediately preceding strolls, and
what people's schedules look like during the upcoming stroll.

(This section doesn't really belong here in `CONTRIBUTING.md`, of
course, since it's really about working style within OTS, not about
how to contribute to one of the open source projects we're responsible
for.  We just put it here because we don't yet have another place for
it, and we wanted to be able to refer to it easily.)
