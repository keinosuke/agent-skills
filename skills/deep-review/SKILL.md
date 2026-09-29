---
name: deep-review
description: |
  Multi-lane code review engine. Runs several parallel review lanes with deliberately different prompt designs (perspective enumeration for depth, simplification for noise reduction, mechanical checklist for coverage), verifies the merged findings, then hands the result to the menu-recipe skill for the final two-layer report. Use when asked for a thorough / high-accuracy review of a diff, branch, or PR. Triggers on "deep review", "thorough review", "multi-agent review", "レビュー精度を上げて", "徹底レビュー", "多角レビュー", "しっかりレビューして". Output formatting is NOT defined here — the final report always follows the menu-recipe skill.
---

# Deep review: parallel finding lanes + menu-recipe output

This skill generates findings. It never formats the final report — that is the menu-recipe
skill's job, and every rule there (verify before you write, never report a guess, posting gates)
applies downstream. If menu-recipe is not available in the session, stop and say so instead of
improvising a format.

Design rationale (from a controlled comparison of five review skills on identical PRs, published
by Tecotec's engineering blog: https://tec.tecotec.co.jp/entry/2026/05/18/090000 — the skills
themselves are their internal work and are not reproduced here; this skill independently
implements the publicly described design patterns): prompt design is the dominant factor in
review output (>2x differences), domain-specific perspective enumeration produces the
highest-quality findings, parallel agents produce volume, ultra-detailed checklists work well on
cheaper models, and giving reviewers whole files instead of bare hunks changed one lane's yield
by +375%. This skill encodes those results.

## Stage 0 — scope

- Identify the revision and record it (menu-recipe's `Target:` line needs it).
- Build the review corpus: the diff plus the full current text of every touched file. Never send
  a lane bare hunks — context starvation is the cheapest accuracy loss to fix.
- If a project checklist exists (look for `docs/review-checklist.md`,
  `.claude/review-checklist.md`, or a path named in the project's `CLAUDE.md`), load it for
  Lane C. No checklist → Lane C still runs with the generic list below.

## Stage 1 — parallel lanes

Run the lanes as parallel subagents where the environment supports it; otherwise run them
sequentially with a fresh context each (one lane must not see another lane's findings —
independence is what makes agreement in Stage 2 meaningful).

Every lane returns raw material in this shape, one block per finding, nothing else:

```
- file: <path>:<line>
  claim: <one sentence>
  failure: <concrete input/state -> what breaks>   # no failure scenario = do not report
  severity-proposal: must-fix | question | minor
  confidence: saw-it-run | traced-the-code | pattern-match
```

Lane wording never reaches the final report; the menu layer is rewritten from scratch later.

### Lane A — perspective enumeration (depth)

Strongest model available. Review the corpus against explicitly enumerated perspectives, one pass
each, and say under which perspective each finding was made:

1. runtime failures — nil/None, bounds, division, overflow;
2. concurrency and ordering — races, await gaps, reentrancy, idempotency;
3. resource and lifecycle — leaks, unclosed handles, retain cycles, missing cleanup on the error
   path;
4. error handling — swallowed errors, partial failure leaving inconsistent state;
5. security — injection, authz gaps, secrets, unsafe deserialization;
6. data integrity — writer/reader pairs, changed constants (grep the OLD literal, per
   menu-recipe), migrations.

Extend the list with the project's domain (payments, iOS UI, scraping etiquette, ...) when
`CLAUDE.md` declares one.

### Lane B — simplification (noise + design)

Strongest model available. Ignore bugs. Report only: dead or unreachable code introduced by the
change, needless abstraction, duplicated logic that already exists elsewhere in the repo (search
before claiming), misleading names/comments, API surface wider than its use. For each finding,
the `failure` field states the maintenance cost, not a crash.

### Lane C — mechanical checklist (coverage)

Cheapest capable model. Walk the checklist from Stage 0 item by item over every touched file and
report pass/fail per item, with `file:line` for each fail. Generic fallback list: TODO/FIXME left
in the diff; debug prints/logs; commented-out code; hardcoded URLs, paths, credentials; missing
tests for new branches; changed public API without doc update; lint suppressions added.
Checklist lanes must not editorialize — a fail is a fail even if it looks intentional (Stage 2
decides).

### Optional Lane X — external reviewers

If an external reviewer already commented on the PR (CodeRabbit, Copilot review, a human), fetch
those comments and normalize them into the same block shape, marked `confidence: external`. They
enter Stage 2 as one more lane, and menu-recipe's rule 5 (no rubber-stamping other tools) applies
to them.

## Stage 2 — merge and verify

Merge per menu-recipe's "Merging multiple reviewers": unanimous first, then partial agreement
(name the lanes), then single-source.

Then verify — this is where accuracy is actually bought:

- Every finding proposed as must-fix, and every single-source finding regardless of severity,
  gets the menu-recipe verification treatment: a throwaway script in a scratch worktree that runs
  the claimed failure, or an explicit downgrade to open question with the reason verification was
  not possible.
- `confidence: pattern-match` findings that fail verification are dropped here, silently — they
  were never reported to anyone. (Retraction rules apply only after something has been shown to
  the user.)
- Checklist fails that the diff's context shows to be deliberate (e.g. a suppression with a
  linked issue) move to minor with the reason recorded.

## Stage 3 — report

Invoke the menu-recipe skill and write the report exactly as it prescribes: two layers, symptom
headings, severity buckets, "What is good here", re-review shape on second passes, and its
posting gates for anything that leaves the session. Findings enter the report in Stage 2's merge
order. The `Verified:` line lists what Stage 2 actually ran — lane output alone is "static
reading only" and must be labelled as such.
