---
name: menu-recipe
description: |
  Write anything that two audiences must read — a person who decides and an agent who builds — as a "menu and recipe" (献立とレシピ): a menu a non-engineer can decide from, plus a recipe with file paths, line numbers, repro steps, and the fix that a coding agent can execute from. Use for code review, pull request review, security audit, design review, spec or instruction-document review, research write-ups, pull request bodies, and issue bodies. Triggers on "review this", "review the PR", "audit this", "check this code", "summarize the findings", "write up the review", "look over this spec", "write the PR description", "open a PR", "create a pull request", "file an issue", "open an issue", "write it as menu and recipe". 日本語でも起動する: 「レビューして」「PR を見て」「監査して」「指摘をまとめて」「レビュー結果を書いて」「指示書を確認して」「献立とレシピで書いて」「PR を作って」「PR の本文を書いて」「Issue を起票して」「Issue にして」。Produces output that can be pasted directly into a GitHub PR comment, PR body, or issue body.
---

# Menu and recipe (献立とレシピ): two-layer reports, PRs, and issues

A review, a pull request, and an issue all have two audiences: the **person deciding what to do**
(often not an engineer) and the **agent that will do the work**. Write only for one and you get
a text the human cannot evaluate, or one the agent cannot act on.

So every piece of content is written in two layers, and this skill names them:

- **Menu (献立)** — the layer for the person who decides. Like a menu, it says what will be
  served and what it costs, in words anyone can read, so they can choose. Symptoms, impact,
  decisions, what was and was not checked. **No identifiers, file names, type names, or API
  names.**
- **Recipe (レシピ)** — the layer for the agent who builds. Like a recipe, it is exact enough to
  reproduce the dish without asking: file paths with line numbers, identifiers and values, the
  repro, the fix, the tests and commands that were run.

The principle: **reading only the menu is enough to decide, and reading only the recipe is
enough to build.** Neither layer may lean on the other to be understood.

**"Menu" and "recipe" are this skill's working vocabulary, not labels for the output.** Do not
print them as headings or labels in a report, PR body, or issue body — a reader who has not
seen this skill does not know what a "recipe" is. In the output, the menu has no label of its
own (it is simply the body), and the recipe is labelled **"Technical note (for agents)"** —
in Japanese, **「技術メモ（エージェント向け）」**.

## Scope

Code review, pull request review, security audit, design review, spec/instruction-document
review, research reports — anything whose deliverable is a list of findings. Also **pull request
bodies** and **issue bodies** (see "Shape of a PR body" and "Shape of an issue body").

This skill is agent-neutral: it assumes only a shell, `git`, and — for GitHub targets — the
`gh` CLI. It does not depend on any one agent's tool names or settings. Everything it requires
of the operator is stated here, so it does not silently rely on the reader's personal global
rules being configured a particular way.

## Non-negotiable rules

1. **Never report a guess.** If you cannot write a concrete failure scenario — which input or
   state, and what breaks — it is not a finding. Move it to "Open questions" and phrase it as a
   question instead.
2. **Separate what you verified from what you only read.** List the commands you actually ran,
   with their results, at the top. If you ran nothing, say "static reading only". Never imply
   you ran something you did not.
3. **Phrase findings as requests, not commands.** "Could you ... ?" / "I think ..." rather than
   "Fix this" / "You must". Review comments on someone else's work read as colder than intended;
   keep the content terse but the tone warm.
4. **Write the report in the language the user is using**, in that language's neutral written
   register — **section headings included** (see the heading table below). The report is a
   deliverable that other people will read — do not carry over the conversational persona,
   tone, or verbal tics you use in chat.
5. **Do not rubber-stamp anyone.** This applies to subagents, other tools, **and the author's
   own replies**. Mark which findings are corroborated and which are single-source, and correct
   any conclusion that is wrong or incomplete rather than passing it through.
6. **Retract your own findings out loud.** If verification disproves something you already told
   the user or posted, say plainly that it was wrong and why, then remove it. Never delete a
   finding silently — someone may have already read it and started acting on it.
7. **Check severity against reality, not against the code.** Before calling anything must-fix,
   confirm the harm can actually occur: is the system in production, does the affected data
   exist, is the path reachable? A defect in a feature nobody has run yet is an open question,
   not a must-fix. Ask if you do not know.

## Before you write: verify

Reading the diff is where findings come from. Running the code is what turns them into findings
you can defend. Do both, in that order.

**Write a throwaway script that calls the code under review.** This is the highest-value step
and the one most often skipped. Check out the exact revision into a scratch working copy (a
`git worktree` keeps the main checkout untouched and is safe when other people or sessions share
it), call the functions the finding is about with the states you claim are dangerous, and print
what actually comes out. Delete the script and remove the worktree when you are done.

Three things this catches, all of which occurred in practice:

- A claim you were confident about is simply wrong (library behaves differently across major
  versions, the branch is unreachable, another layer already guards it). Retract it per rule 6.
- A claim is right but the blast radius is different from what you assumed.
- A finding that reads as one case is actually two, and the author's rebuttal only covers one
  of them. See "Re-review" below.

**Distinguish your environment's failures from the target's.** If a test fails because of your
scratch setup (a shared `node_modules`, a missing native binary, absent credentials), say so
explicitly in the same line where you report the count. A bare "1 failing" attributed to the
change under review is a false finding, and it is the kind that spreads.

**Changing a constant: grep for the old value, not the new name.** When a change alters a
limit, count, or threshold, the dangerous call sites are the ones that never referenced the
constant — a hardcoded `[0, 1, 2, 3]`, `slice(0, 4)`, or `length: 4`. Searching for the
constant's name finds only the places that were already correct, so the search that matters is
for the old literal. Check the writer and the reader as a pair: a form that renders N rows and
a handler that collects N rows are two halves of one invariant, and changing only one silently
drops the extra data with no error anywhere. This applies when you are the one making the
change, not only when reviewing someone else's.

**Record the exact revision you reviewed.** Everything below is anchored to it.

## Overall structure

```
# <target> review

Target: <branch / files / document> @ <commit SHA or revision> (size: N files, +X -Y)
Verified: <commands run and their results — or "static reading only">

Verdict: **N must-fix (before merge) / M open questions**

---

## Must-fix 1. <one sentence a non-engineer understands>
(repeat per must-fix)

---

## Open questions
### A. <heading>
(A, B, C, ...)

### Minor (no action proposed)
- <bullet>

---

## What is good here
```

The `Target:` line **must carry the revision** (commit SHA, tag, or document version). A review
without one cannot be checked for staleness later, by you or by anyone else.

Headings must state the **symptom**, not the identifier. Not "`cancelRemovesListing` is
inconsistent" — instead "Re-submitting right after a delete publishes the same property twice".
Readers triage on the heading alone. This applies to open questions too: the heading states what
is observable, and the body asks the question.

**When there are no must-fix items** — a common and perfectly good outcome — keep the verdict
line with `0`, drop the must-fix section entirely, and add one sentence after the verdict saying
the change looks safe to merge as-is. Do not manufacture a must-fix to justify the review, and
do not pad the open questions.

### Heading names by report language

Headings follow the language of the report body (rule 4). Japanese set:

| English | 日本語 |
|---------|--------|
| Target / Verified / Verdict | 対象 / 検証 / 判定 |
| Must-fix N. | 要対応 N. |
| What happens / Impact / Worth noting / Suggested direction | 何が起きるか / 影響 / 補足 / お願いしたいこと |
| Open questions | 確認したい点 |
| Minor (no action proposed) | 細かい点（対応不要と判断したもの） |
| What is good here | 良かった点（次に触る人が壊さないでほしいところ） |
| Technical note (for agents) | 技術メモ（エージェント向け）。中身は英語のままでも可 |

The Menu/Recipe split applies inside each finding: everything above the recipe block is menu,
the block is the recipe.

## Shape of one must-fix

````markdown
## Must-fix N. <symptom in one sentence>

**What happens**

<Explanation a non-engineer can follow. Start from the domain fact that makes it possible
("The portal has no delete button, so removing a listing means marking it as withdrawn...").
No identifiers, type names, or API names in this prose. 3-6 sentences.>

**Impact**

<Who is hurt and how, in business terms: cost, contract terms, what a customer sees,
data exposure, support load. 1-3 sentences.>

**Worth noting**   <- only when the obvious fix does not actually work

<Why the naive fix fails. State the trade-off explicitly. This is the section that stops a
reviewer's suggestion from being applied blindly.>

**Suggested direction**

<What to do, phrased as a request. A rough size ("probably a few lines") helps triage.>

```
Technical note (for agents)
- <path>:<line>  <identifier or value>
- <related function / variable and the branch condition>
- Repro: <the order of operations or state that triggers it>
- Proposed fix: <what changes, and to what>
```
````

Put the recipe in a plain code fence (no language tag) so humans skip it and agents can
lift it verbatim. **Always include file path and line number** — an identifier alone is not
enough to locate the target. Prefer a repro you actually executed, pasted as the calls you made
and the values that came back.

## Shape of an open question

Use it for things that may be deliberate, are low-impact, or concern intent and consistency.
Label them A, B, C so the discussion can reference them. Two to four sentences, ending in an
actual question. Attach the same recipe block.

**The menu is not optional here.** An open question gets the same treatment as a
must-fix: open from the domain fact, not the identifier. A reader who does not know the codebase
should understand what breaks and why it matters before any symbol name appears. File paths,
function names and commit hashes belong in the recipe. Brevity is not a licence to fall
back on symbol names — a short paragraph of plain prose is shorter than a long one full of
identifiers.

**Ask only when the author plausibly made a choice.** Evidence of a choice looks like: a comment
explaining the trade-off, a test pinning the behaviour, a line in the spec, or a reply defending
it. Where that evidence exists, "Is that intentional?" is a real question.

**Where nothing suggests the case was considered, do not ask whether it is acceptable.** Say
plainly that the case looks unhandled, then ask which direction they want — or propose one and
ask whether it is feasible, leaving room for them to offer a better option from their knowledge
of the code. "Is this acceptable?" invites a rubber stamp and hands the thinking back to the
author for no reason.

Never ask a question whose answer you can already predict. If any answer but one would surprise
you, you are not asking — you are softening a finding. State it, and propose the fix.

**Use your own vocabulary for severity, not the author's.** If they label their fixes with terms
your review never used, do not adopt those terms to describe your own findings — check what you
actually wrote last time and stay consistent, or the two of you end up counting different things.

## "What is good here"

Always include it — not as flattery, but to mark **what the next person must not break**.
Be specific: not "good design" but "the reason for this branch is traceable from the comment
next to it".

## Severity

| Bucket | Test |
|--------|------|
| Must-fix | You can write a repro for real harm **that can occur today**: data loss, an irreversible external effect, information disclosure, or a state the user cannot recover from |
| Open question | No harm, but intent, consistency, missing tests, or maintainability is at stake |
| Minor | Pre-existing pattern, fixed by a reload, stale comments, things you checked and decided need no action. Collect as a bullet list at the end of the open questions |

Use CRITICAL / HIGH / MEDIUM / LOW labels **only when explicitly asked for them**. For readers
who are not engineers, "must-fix / open question" communicates better.

State the minor items you deliberately dismissed, with the reason. A reader who spots the same
thing later needs to know it was considered, not overlooked.

## Re-review: reporting on a second pass

Reviewing the author's response to a previous review is the common case, and it needs its own
shape. Never reuse the first report's structure as if nothing had happened.

1. **Re-verify what they fixed.** Run the checks again against the new revision. Say which of
   their claims you independently confirmed and which you took on trust. Their "all tests pass"
   is their evidence, not yours.
2. **Check that each rebuttal covers the whole finding.** When the author declines a finding
   with a reason, test the reason against *every* case the finding covered. A rebuttal that is
   correct for one input and silently wrong for another is the single most common way a real
   defect survives review — and it survives with a comment in the code that now asserts it is
   fine. Say so specifically: "this explains case X, but case Y still behaves as originally
   described", with the repro.
3. **Close out what is resolved, briefly.** List the resolved items in one short block. Do not
   re-explain them.
4. **Report discrepancies in their evidence neutrally.** If their stated counts or results differ
   from yours, note it once, without insinuation, and move on.

Anything still open keeps its original letter/number where practical, so the discussion thread
stays followable.

## Merging multiple reviewers

When combining subagent or multi-tool results, order them:

1. **Unanimous** — highest confidence, goes first.
2. **Partial agreement** — note who raised it.
3. **Single-source** — always attach your own verification: is it real, and is the severity right?

Where another reviewer's conclusion is incomplete, say so plainly: "Both agents suggest flipping
the flag, but that leaves failure mode Y, so it is not sufficient."

## Shape of a PR body

A pull request body is read by the person who approves the merge and by the agent (or human)
who reviews or later continues the work. Write the menu first, in prose a non-engineer can
follow, then the recipe folded away.

````markdown
## What changes
<What the user or operator will notice after this merges. 1-3 sentences. No identifiers.>

## What was wrong
<The problem this solves, from the user's side: what happened, who was affected, what it cost.
Say explicitly whether money, external effects, or data were involved — or that they were not.>

## Decisions
<Choices made in this PR that someone might question later, each with the reason and, where a
person decided it, the date. Omit the section if there were none.>

## Verified / not verified
- Verified: <what was actually run or reproduced, and what came out>
- Not verified: <what was not checked, and why>
- <Which environments were and were not touched — production, staging, external services>

## Not fixed in this PR
<Known problems left out of scope, and where they will be handled. Omit if none.>

## User-facing explanation
<Changes to user documentation, help pages, or announcements. Omit if none.>

<details><summary>Technical note (for agents)</summary>

```
Closes #<issue>
- <path>:<line>  <what changed>
- Repro (before the change, @ <revision>): <how the old behaviour was reproduced>
- Tests: <files and counts>; <what was removed to confirm each test fails without the change>
- Commands: <test / type-check commands and results>
```
</details>
````

- **Headings name outcomes, not code.** "What changes" is answered with what a user sees, not
  with the functions that were edited. Identifiers, file names, and commit hashes go in the
  recipe only.
- **On GitHub, fold the recipe** in a `<details>` block so the page opens on the menu. Keep the
  recipe inside a plain code fence within it, as for findings.
- **"Verified / not verified" follows rule 2.** Never let the menu imply more testing than the
  recipe records.
- Keep the `🤖 Generated with ...` or equivalent attribution line if your environment requires
  one; it goes after the recipe.

## Shape of an issue body

An issue is read by the person who prioritises it and by whoever picks it up. The menu must let
the prioritiser decide without opening the code; the recipe must let the implementer start
without re-investigating.

````markdown
## What is happening
<The symptom, as a user or operator would observe it. Use a concrete example. No identifiers.>

## Impact
<Who is hurt and how, in business terms. Say how often it happens — or that frequency was not
measured.>

## Cause
<Why it happens, in plain words ("the whole fetch is marked complete even when one lot fails").
The code locations go in the recipe.>

## What was checked
<What was actually run or reproduced, what was only read, and what was not checked (rule 2).>

## Proposed fixes (in order of preference)
1. <Option, with a rough size and what it depends on>
2. ...

## Decisions needed
<Questions only a person can answer — policy, cost, spec. Each phrased as an actual question.
Omit if none.>

<details><summary>Technical note (for agents)</summary>

```
- <path>:<line>  <identifier and the branch condition>
- Repro: <calls made and values that came back, @ <revision>>
- Related: <issues / PRs>
```
</details>
````

- Rule 1 applies: an issue for which you cannot write a concrete repro or failure scenario is
  a question, not an issue. File it as a question, or ask first.
- Mark each proposed fix that needs a policy decision (money, external effects, spec changes)
  so the prioritiser can see which ones they must decide on before anyone starts.

### Japanese headings for PR and issue bodies

| English | 日本語 |
|---------|--------|
| What changes | 何が変わるか |
| What was wrong | 困っていたこと |
| Decisions | 決めたこと |
| Verified / not verified | 確かめたこと・確かめていないこと |
| Not fixed in this PR | この PR では直していないこと |
| User-facing explanation | 利用者向けの説明 |
| What is happening | 何が起きているか |
| Impact | 影響 |
| Cause | 原因 |
| What was checked | 確認したこと |
| Proposed fixes (in order of preference) | 直し方の案（推奨順） |
| Decisions needed | 決めてほしいこと |
| Technical note (for agents) | 技術メモ（エージェント向け） |

## Where the output goes

Default to writing the report into the conversation, formatted so it can be pasted straight into
a PR comment. If it should be saved to a file, propose a path and wait.

**Never post to GitHub or send anywhere external on your own initiative.** This covers review
comments **and creating a pull request or an issue** (and editing an existing PR body, issue
body, or comment). Posting is gated by three checks, all of which must pass immediately before
the call that publishes:

1. **Explicit approval for this exact text.** Present the full body and ask whether to post.
   **Approval expires the moment the body changes.** Edits, reorderings, tone changes, and
   dropped findings all invalidate it — present the revised body and ask again. Instructions
   like "soften this" or "drop that item" are edits, not approvals.
2. **Freshness of the target.** Re-fetch and confirm the target has not moved since you reviewed
   it. For a GitHub PR:

   ```
   git fetch <remote>
   gh pr view <N> --json headRefOid,updatedAt,comments
   ```

   When creating a pull request, confirm the branch you are proposing is the one the body
   describes (its head revision matches the recipe) and that the base has not moved in a way
   that changes what the PR does. When filing an issue, confirm it is still reproducible at the
   current revision and search for an existing issue that already covers it.

   If the head revision differs from the one on your `Target:` line, or comments have arrived
   that you have not read, **do not post**. Re-review at the new revision and rebuild the report;
   much of it may already be resolved. Posting a report written against an older revision tells
   the author that finished work is still outstanding, and it is not recoverable by editing
   afterwards.
3. **Freshness of your reading of their side.** If you are responding to the author's comments,
   confirm you have their latest ones. A reply that argues against a position they already
   abandoned costs more trust than saying nothing.

After posting, report the resulting URL. If you stopped at check 2 or 3, say explicitly that
nothing was posted and why.
