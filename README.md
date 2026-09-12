# agent-skills

Reusable skills for coding agents (Claude Code, Codex, Gemini CLI, Cursor, …).

## Skills

| Skill | What it does |
|-------|--------------|
| [review-report-format](skills/review-report-format/SKILL.md) | Turns review, audit, and investigation findings into a two-layer report: a plain-language explanation the decision-maker can act on, plus file paths, line numbers, repro conditions, and a proposed fix the agent can execute from — verified by running the code, not just reading it |

### Why two layers

A review has two audiences. The person deciding what to do is often not an engineer and cannot
evaluate a finding written as `cancelRemovesListing is inconsistent`. The agent making the fix
cannot act on a finding written as "the delete flow has a problem". Most reviews pick one
audience and lose the other.

So each finding is written twice — the symptom and its business impact in prose, then a fenced
block with the exact anchors:

```
Technical note (for agents)
- app/src/adapters/suumo/module.ts:50  cancelRemovesListing: true
- Repro: cancel succeeds -> re-submit before the portal's scheduled deletion
- Proposed fix: block re-submission while canceledListingRef is set
```

Around that shape, the skill enforces the habits that separate a review people trust from one
they have to double-check: never report a guess, phrase findings as requests rather than
commands, state plainly which commands you actually ran — and retract your own findings out
loud when verification disproves them.

### What it makes you check

Three of the rules exist because reviews fail the same three ways, and each one caught a real
error while the skill was being written.

**Run the code, do not just read it.** Findings come from reading the diff; findings you can
defend come from calling the functions with the states you claim are dangerous and printing
what comes out. In practice this splits confident-sounding findings into "reproduced",
"real but smaller than I thought", and "simply wrong — the library behaves differently in this
major version". Failures from your own scratch setup get labelled as such, so a missing native
binary never gets reported as a regression.

**Check that a rebuttal covers the whole finding.** When an author declines a finding with a
reason, test that reason against every case the finding covered. A rebuttal that is right for
one input and quietly wrong for another is the most common way a real defect survives review —
and it survives with a comment in the code now asserting it is fine.

**Confirm freshness immediately before posting.** Approval expires the moment the body changes,
and a report written against an older revision tells the author that finished work is still
outstanding. The skill re-fetches the head revision and refuses to post if the target moved,
which is not recoverable by editing the comment afterwards.

## Install

### Claude Code (plugin)

```
/plugin marketplace add keinosuke/agent-skills
/plugin install agent-skills@agent-skills
```

### Any agent (symlink)

`SKILL.md` is ordinary Markdown with YAML frontmatter, so agents without a skill mechanism can
read it as a plain document.

```bash
git clone git@github.com:keinosuke/agent-skills.git ~/develop/agent-skills

# Claude Code
ln -sfn ~/develop/agent-skills/skills/review-report-format ~/.claude/skills/review-report-format

# agent-neutral location
ln -sfn ~/develop/agent-skills/skills/review-report-format ~/.agents/skills/review-report-format
```

## Layout

```
.claude-plugin/
  plugin.json         # plugin manifest
  marketplace.json    # marketplace manifest (this repo is its own marketplace)
skills/
  <skill-name>/
    SKILL.md          # the skill (frontmatter: name, description)
```

One directory per skill.

## Adding a skill

- Create `skills/<name>/SKILL.md` with `name` and `description` in the frontmatter.
- List concrete **trigger phrases** in `description` — vague descriptions never fire. Include
  phrasings in every language you expect to work in.
- Add a row to the Skills table above.

## Language

Skill instructions are written in English so they work regardless of the user's language. The
report itself is produced in whatever language the user is writing in.

## License

MIT — see [LICENSE](LICENSE).
