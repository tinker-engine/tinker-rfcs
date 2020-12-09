---
RFC: 0000
Author: Roni Choudhury
Created: 2020-12-09
Last-Modified: 2020-12-09
---

# RFC 0000: Adopt Django-esque process for creating RFCs

This RFC describes a process for creating RFCs that adapts Django's methodology
for creating "DEPs" (Django Enhancement Proposals). [Their
process](https://github.com/django/deps/blob/master/final/0001-dep-process.rst)
is detailed via its own DEP.

Because this RFC is being adapted directly from another project at Kitware, it
is the one and only "self-executing" RFC, bypassing the procedures listed herein
and moving straight to `final` status.

## Motivation

It turns out that our RFC documents have multiple paths towards resolution: some
of them are accepted as intended, and become useful information, or actionable
plans. Others are modified in flight and settle in a different form than
intended, while others are rejected. In order to track these paths, as well as
provide a distinction between "informational" and "actionable", we intend to
adopt a slightly richer process adapted from Django's use of "DEPs" (which in
turn is adapted from Python's famous "PEPs").

The main additions to our process are:
- a directory structure between which proposals move as their status changes
- a template that is to be copied into new draft RFCs to avoid the difficulty of
  getting started
- different resolved states, including `accepted` for proposals that are accepted;
  `final` for ones that have been accepted *and* implemented; `superseded` for
  ones that have been made obsolote by newer RFCs; `rejected` for those are
  rejected (but kept for informational/historical value); `withdrawn` for those
  the author wishes to abandon; and `draft` as a place for in-flight RFCs can be
  kept

## Proposals

### New Process for RFC Creation/Review/Acceptance/Rejection

1. The author clones this repository, creates a new directory in the `drafts`
   directory with `X`s in place of the RFC number, (e.g.,
   `drafts/XXXX-new_proposal/`), and copies the root-level `template.md` file
   there as `README.md`.
2. The author modifies the new `README.md` file to contain their newly proposed
   RFC.
3. If supplemental materials are needed (this should be generally rare), they
   can go into the new directory alongside the `README.md` file.
4. The author opens a pull request. The title of the pull request should be the
   RFC title preceded by a prefix of `RFC:`. A PR description is optional, and
   can generally be omitted if the PR is simply introducing a new RFC.
5. Reviewers provide feedback, engage in discussion, etc., just as in a normal
   GitHub-based code review.
6. The RFC becomes "settled". Settlement involves one of several paths of
   action, detailed next.

### Settling an RFC

RFCs can be settled in several ways.

**Acceptance.** An RFC can be accepted as written (which includes mutually
agreed-upon edits).

1. Two reviewers provide a review approval.
2. The author provides their "final cut" (that is, their intention to make no
   further updates to the document).
3. A reviewer or the author determines the RFC number (by first ensuring that
   the PR branch is up-to-date, merging from master if necessary, then examining
   the top level `next_rfc_number.txt` file).
4. That person renames the draft directory using the four-digit number in place
   of the `XXXX` draft prefix.
5. The person then commits an incrementation of the number in
   `next_rfc_number.txt`.
6. The person moves the draft directory to the `accepted` top level directory,
   and merges the pull request.

**Rejection.** The RFC can be rejected if it does not fit into the larger aims
of the project.

1. Discussion ends with a determination that the RFC should be rejected.
2. A reviewer or the author determines the RFC number (consulting
   `next_rfc_number.txt`).
3. That person renames the draft directory using the four-digit number in place
   of the `XXXX` draft prefix.
4. The person commits an incrementation of the number in `next_rfc_number.txt`.
5. The person moves the draft directory to the `rejected` top level directory,
   and merges the pull request.

Note that despite **rejecting** the RFC, this procedure **accepts** the pull
request. This is necessary to preserve the history of ideas and information that
flowed through the project. The RFC number must still be assigned to make
reference to this RFC easier in the future.

If rejection leads to an obvious successor RFC candidate, that new RFC must be
worked on in a new branch and pull request thread.

**Withdrawal.** The RFC can be withdrawn by the author.

1. The author determines the appropriate RFC number and renames their draft
   directory as in the cases above.
2. The author moves the draft directory to the `withdrawn` top level directory
   and informs the pull request thread of their intention to withdraw.
3. A reviewer or the author merges the pull request.

### Revisiting Settled RFCs

Settled RFCs can be revisited in several ways.

**Revival.** Anyone who wants to bring a `withdrawn` RFC back for
reconsideration can do so by opening a pull request that moves that RFC (with
its original RFC number) into the `drafts` folder. The same procedures as above
resume from the beginning, leading to settlement of that RFC once more.

**Supersession.** An accepted RFC can be *superseded* (that is, replaced) by a
new RFC. This can be done because the team has changed its collective mind on
something, or because new facts demand an update of old information or
procedures.

1. A new RFC is formulated for review using the usual procedure as detailed
   above.
2. The RFC must indicate which RFC it supersedes in its metadata.
3. The pull request should move the superseded RFC directory to the `superseded`
   directory.
4. Review proceeds as usual.

**Modification.** An accepted or final RFC can be modified after the fact by
opening a new PR that makes edits or additions. The edits should include an
update to the `Last-Modified` metadata field. The PR should have a title
describing the nature of the modification, and the description should describe
the motivation for doing so. As with any other RFC PR, discussion followed by
two approvals can lead to a merge.
