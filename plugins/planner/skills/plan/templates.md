# Plan templates

Leave out a section that would be empty. Keep the status legend in every `index.md`, so any
session reading the plan knows what the marks mean.

Length limits. The plan is read at the start of every `/planner:next` session, so every line
costs time twice: once to write and again each time it is read.

- Intro: at most 4 sentences. Goal and Out of scope: one line each.
- Where things are today: at most 6 bullets, one or two lines each. Name files and functions;
  don't retell the research.
- Decisions: one line each. Follow-ups and Things to watch: one or two lines each.
- PR files: Why, at most 3 sentences. Each step, at most 2 lines plus a one-line Check.
- A PR file over about 40 lines usually means the PR should be split.

## index.md

```markdown
# <Goal in a few words>

<Two to four sentences: what is wrong or missing today, and what changes for the user or
the project once this is done.>

**Goal.** <What done looks like, checkable.>
**Out of scope.** <What this plan will not do.>

## Where things are today

Checked on <YYYY-MM-DD>. Re-check before relying on it.

- **<Area>.** <How it works now, with exact file and function names.>

## Decisions

- **<Topic>.** <What was chosen, and why in one clause.> (<YYYY-MM-DD>)

## Open topics

- <Topic still to decide, while planning is in progress.>

## PRs

| # | PR | Depends on | Status |
|---|---|---|---|
| 1 | [<title>](01-<slug>.md) | | todo |
| 2 | [<title>](02-<slug>.md) | 1 | todo |

**Resume:** PR 1, step 1.

Status: `todo`, `in progress`, `open #N`, `merged #N <YYYY-MM-DD>`,
`closed #N, replaced by PR <M>`, `dropped`.
Steps: `[ ]` todo, `[~]` implemented, not committed, `[x]` committed `<sha>`.
Merged PRs are frozen. Pushed commits are never rewritten. An open PR changes only by new
commits, or is closed and replaced.

## Follow-ups

- **<Idea>.** <Why it came up, why it waits.>

## Changes

- <YYYY-MM-DD>: <what changed in the plan, and why>

## Sources

- <Title>: <link> (read <YYYY-MM-DD>)
```

## NN-pr-slug.md

```markdown
# PR <N>: <title>

**Why.** <The problem this PR solves, in one to three sentences.>
**Goal.** <What works after it is merged, checkable.>
**Where.** <Files and modules it touches.>
**Branch.** `<type>/<slug>`

## Steps

1. [ ] `<suggested commit title>`
   <What changes and where, in one to three lines.>
   Check: <how to see it works: a test, a command, what to look at.>
2. [ ] `<suggested commit title>`
   ...

## Things to watch

- <Risk, edge case or trap, and what to do about it.>
```

## handoff.md

Written by `/planner:next` only when a session stops in the middle of a step. Delete it once the
step is done.

```markdown
# Handoff: PR <N>, step <M>

<YYYY-MM-DD>. Read this before the PR file.

- **Done so far.** <What is in the working tree, by file.>
- **Left to do.** <The rest of the step.>
- **Found on the way.** <Facts the next session would otherwise have to find again.>
- **Not mine.** <Changes in the tree that belong to something else. Don't stage them.>
```
