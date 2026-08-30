---
name: review-report-format
description: |
  Format review, audit, and investigation findings as a two-layer report that a non-engineer can act on and a coding agent can execute from. Use for code review, pull request review, security audit, design review, spec or instruction-document review, and research write-ups. Triggers on "review this", "review the PR", "audit this", "check this code", "summarize the findings", "write up the review", "look over this spec". 日本語でも起動する: 「レビューして」「PR を見て」「監査して」「指摘をまとめて」「レビュー結果を書いて」「指示書を確認して」。Produces output that can be pasted directly into a GitHub PR comment.
---

# Two-layer review report format

A review has two audiences: the **person deciding what to do** (often not an engineer) and the
**agent that will make the fix**. Write only for one and you get a report the human cannot
evaluate, or one the agent cannot act on.

So every finding is written twice: a plain-language explanation, followed by a technical note
block with the exact anchors.

## Scope

Code review, pull request review, security audit, design review, spec/instruction-document
review, research reports. Anything whose deliverable is a list of findings.

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
   register. The report is a deliverable that other people will read — do not carry over the
   conversational persona, tone, or verbal tics you use in chat.
5. **Do not rubber-stamp other reviewers.** When merging results from subagents or other tools,
   mark which findings are corroborated and which are single-source, and correct any conclusion
   that is wrong or incomplete rather than passing it through.

## Overall structure

```
# <target> review

Target: <branch / files / document> (size: N files, +X -Y)
Verified: <commands run and their results — or "static reading only">

Verdict: **N must-fix (before merge) / M open questions**

---

## Must-fix 1. <one sentence a non-engineer understands>
(repeat per must-fix)

---

## Open questions
### A. <heading>
(A, B, C, ...)

---

## What is good here
```

Headings must state the **symptom**, not the identifier. Not "`cancelRemovesListing` is
inconsistent" — instead "Re-submitting right after a delete publishes the same property twice".
Readers triage on the heading alone.

## Shape of one must-fix

```markdown
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
```

Put the technical note in a plain code fence (no language tag) so humans skip it and agents can
lift it verbatim. **Always include file path and line number** — an identifier alone is not
enough to locate the target.

## Shape of an open question

Use it for things that may be deliberate, are low-impact, or concern consistency of approach.
Label them A, B, C so the discussion can reference them. Two to four sentences, ending in an
actual question ("Is that intentional?"). Attach the same technical note block.

## "What is good here"

Always include it — not as flattery, but to mark **what the next person must not break**.
Be specific: not "good design" but "the reason for this branch is traceable from the comment
next to it".

## Severity

| Bucket | Test |
|--------|------|
| Must-fix | You can write a repro for real harm: data loss, an irreversible external effect, information disclosure, or a state the user cannot recover from |
| Open question | No harm, but intent, consistency, missing tests, or maintainability is at stake |
| Minor | Pre-existing pattern, fixed by a reload, stale comments. Collect as a bullet list at the end of the open questions |

Use CRITICAL / HIGH / MEDIUM / LOW labels **only when explicitly asked for them**. For readers
who are not engineers, "must-fix / open question" communicates better.

## Merging multiple reviewers

When combining subagent or multi-tool results, order them:

1. **Unanimous** — highest confidence, goes first.
2. **Partial agreement** — note who raised it.
3. **Single-source** — always attach your own verification: is it real, and is the severity right?

Where another reviewer's conclusion is incomplete, say so plainly: "Both agents suggest flipping
the flag, but that leaves failure mode Y, so it is not sufficient."

## Where the output goes

Default to writing it into the conversation, formatted so it can be pasted straight into a PR
comment. **Always confirm before posting to GitHub or sending anywhere external** — never post
on your own initiative. If it should be saved to a file, propose a path and wait.
