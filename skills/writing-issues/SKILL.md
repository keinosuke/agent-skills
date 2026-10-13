---
name: writing-issues
description: |
  Write or revise a GitHub issue so the assignee can start and finish from the issue alone — without asking anyone or re-investigating. Covers task issues (features, changes, chores) as well as bug reports, and reviewing existing issues for missing information before they are assigned. Checks the draft against the code (what already exists, real screen labels, the state of linked issues and PRs), separates what is decided from what is still open, and states start conditions and completion criteria. Triggers on "write an issue", "file an issue", "draft an issue", "review these issues", "is this issue ready", "hand this issue to". 日本語でも起動する: 「Issue を書いて」「Issue を起票して」「Issue にして」「Issue を見直して」「この Issue で着手できる?」「担当に渡す Issue」。Body shape and posting rules follow the menu-recipe skill when it is available.
---

# Writing issues the assignee can act on

One test for every issue: **could the assignee start, and know when they are done, from this
issue alone?** If they would have to ask someone or re-read the code to find out, the issue is
not finished.

Issues fail that test in the same few ways. In one review of 18 open issues, 13 needed changes
before work could start:

- the approach was still undecided, but the issue was already assigned
- work that was already half implemented was described as all still to do
- there was no way to tell when the work was done
- the issue depended on another PR, but did not say "start after it lands"
- real data (customer names, addresses, record contents) had been pasted in

This skill prevents those. The two-layer shape (a plain-language part for the person who
prioritises, a technical note for the implementer) and the posting rules come from the
menu-recipe skill. Use it alongside this one when it is available.

## 1. Before writing: check against the code

Do these before drafting, and record what you checked against (branch and revision).

- **Is any of it already built?** Search the code and the commit log. If part exists, the issue
  must say what is done (with the commit) and what remains, or someone will build it twice.
- **Use the real labels.** Quote screen titles, field names and messages as they appear in the
  source, not from memory.
- **Check every linked issue and PR now.** Open, closed or merged, as of today. Do not cite a
  closed issue as the reason something is blocked.
- **Search for an existing issue** that already covers it.
- **Find in-flight work on the same place.** Open PRs that touch the same screen, endpoint or
  file decide the order of work.

## 2. Shape of a task issue

For bug reports, use menu-recipe's issue shape and add **Start conditions** and **Completion
criteria** from below. For everything else:

````markdown
## Purpose
<What should be possible afterwards, and why — for the person prioritising. No identifiers.>

## Current state
<What exists today. If partly implemented: what is done (commit) and what remains.>

## Decided
- <Each decision, with its date.>

## Open questions
- <Question> — decided by <@handle or role>. <Say whether work can start before it is answered.>

## Start conditions
- <What must land first (PR / issue) and why — e.g. "both change the same endpoint".>

## Completion criteria
- <An action and what you should then see. Mark which ones are fixed by automated tests.>
- <What must keep working as before.>

## Out of scope
- <What this issue does not cover, and where it goes instead.>

<details><summary>Technical note (for agents)</summary>

```
- Checked against: <branch> @ <revision>
- <path>:<line>  <what is there>
- Related: <issues / PRs and their state>
```
</details>
````

Omit a section only when it would be empty. Even then, keep **Open questions** with "none", so
the reader knows the question was asked.

## 3. Rules

1. **Never mix decided and open.** An open point inside "Decided" ("needs checking whether it
   fits the limit") reads as settled, and the assignee builds on it. Move it to "Open
   questions", and name who decides. If an open question blocks the work, write "do not start
   until this is decided" and do not present the issue as ready.
2. **Completion criteria must be checkable.** Each criterion is an action plus what you should
   see ("register with the box ticked → the new user has the permission when reopened").
   "Works correctly" is not a criterion. Include the unchanged path (what happens when the new
   option is not used), and server-side rules that the UI alone cannot guarantee.
3. **Start conditions name the blocker and the reason.** Write "start after #N is merged,
   because both change X", not "related: #N". When you promise to notify the assignee
   ("I will comment here when it is merged"), you then have to keep that promise.
4. **One issue, one deliverable.** A verification report is not a work issue. Close it, and move
   anything left over into its own issue with an owner.
5. **A question to a person is not a work issue.** If the issue exists to get a decision, say
   who must answer it. Close it once it is answered, recording the answer.
6. **No real data.** Leave out personal names (use handles such as @id), customer and company
   names, addresses, real record contents (listing titles, external-service IDs), production
   counts and user counts, and credentials. Use descriptions instead ("the test listing created
   on 2026-01-15"). This applies to private repositories too: issues get quoted, copied and
   outlive their access lists.
7. **Can't write a concrete scenario? Then it is a question, not an issue** (menu-recipe rule 1).
   Ask first, or file it clearly marked as a question.

## 4. Reviewing existing issues

When asked to check issues before they are assigned, produce a report, not edits.

For each issue:

- **Verdict:** ready / needs changes / can be closed
- **What you checked:** revision, and which linked issues and PRs you looked at
- **Problems,** each tagged with the check it fails:
  - (a) who does what
  - (b) scope and out of scope
  - (c) start conditions
  - (d) completion criteria
  - (e) decided vs open; stale references
  - (f) current state matches the code
  - (g) real data
- **Replacement text:** "now → proposed" for small fixes, or the full section for larger ones
- **Closing comment draft,** if the verdict is "can be closed"

Order the report by what would block the assignee first, not by issue number.

## 5. Japanese headings

| English | 日本語 |
|---------|--------|
| Purpose | 目的 |
| Current state | 今の状態 |
| Decided | 決めたこと |
| Open questions | まだ決まっていないこと |
| Start conditions | 着手の条件 |
| Completion criteria | 完了の条件 |
| Out of scope | やらないこと |
| Technical note (for agents) | 技術メモ（エージェント向け） |

## 6. Posting

Do not create, edit or comment on an issue on your own initiative. Follow menu-recipe's
"Where the output goes": present the full text, ask whether to post it, and treat any change to
the text as cancelling the approval. If menu-recipe is not available, apply the same rule
yourself. Immediately before posting, re-fetch the issue and confirm that the part you are
replacing has not changed since you read it.
