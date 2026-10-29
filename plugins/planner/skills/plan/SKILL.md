---
name: plan
description: Research a goal and write it down as a plan of small PRs in markdown, or revise an existing plan. Plans only; never implements.
disable-model-invocation: true
argument-hint: "[goal or plan name]"
---

# Plan

Turn a goal into a plan of small PRs, written to markdown so a fresh session can build it with
`/planner:next`. This skill plans. It never edits source code, runs migrations or commits, even
when the fix looks obvious. The only files it writes are the plan documents and
`.claude/planner.md`.

Goal or plan name: $ARGUMENTS

## 1. Settings

Read `.claude/planner.md` in the repo root.

If it is missing, ask with one AskUserQuestion call, then write the file:

- **Where plans live:** `.claude/plans/` (default), `docs/plan/`, or a path the user types.
- **How git treats them:**
  - `commit`: plans are tracked like any other file.
  - `gitignore`: add the plans folder and `.claude/planner.md` to `.gitignore`.
  - `exclude`: add both to `.git/info/exclude`. Local only; teammates never see a change.

```markdown
# Planner settings

- Plans: .claude/plans
- Git: exclude
```

Change these settings only when the user asks to. Never because a plan seems to fit elsewhere.

## 2. New plan or revision

List the folders in the plans folder. If `$ARGUMENTS` names one of them, or clearly describes
the same goal, this is a revision: go to section 7. If it is empty, ask which plan to revise or
what to plan.

## 3. Understand the goal

Before researching, be sure of the result the user wants, not only the feature name. Ask with
AskUserQuestion when any of these is unclear: the outcome, what done looks like, hard
constraints (deadline, platform, budget, must not break X), and what is out of scope. Skip
questions the code or the prompt already answers.

## 4. Research

Keep the main context for decisions. Send the reading to subagents and keep only what they
conclude.

- **Code:** one or more Explore agents map what exists: files, entry points, current behaviour,
  tests, conventions, and earlier attempts.
- **Web:** search whenever the goal touches anything outside the repo: a third-party API, a
  library, a protocol, a platform rule, or a practice where the industry has a known answer
  (performance, security, data modelling, UX patterns). Do this even when the user did not ask
  and the repo already has code for it. Existing code shows what was done, not what is current
  or correct. Use a general-purpose agent with WebSearch and WebFetch; prefer official docs,
  changelogs and primary sources; ask it for versions, dates, limits and links.
- Run independent agents in parallel. Ask each for a short report: findings, options with
  trade-offs, open risks, sources.

## 5. Decide with the user

Count what the user has to read. If presenting the findings would take more than about 15
lines, or there are more than two real decisions, go topic by topic:

1. Order topics by dependency first, then priority: a decision others build on comes first.
2. For each topic write a short story, at most 8 lines: what the problem is, what was found, the
   options, and which one you recommend and why.
3. Ask with AskUserQuestion, recommended option first. Use plain text questions only when the
   answer is free-form.
4. Record the decision in `index.md` right away, under Decisions, and remove the topic from Open
   topics. A long planning session can then stop and pick up again without losing anything.

Otherwise give everything in one short message and ask once.

If a decision raises a question the research didn't cover, research again before asking.

## 6. Write the plan

Write `<plans>/<slug>/index.md` and one `NN-<pr-slug>.md` per PR, following
[templates.md](templates.md). Create the folder at the start of section 5, so decisions have
somewhere to go.

Break the work into PRs a reviewer can follow:

- One PR, one purpose. A reviewer should understand it in one sitting; aim for under about 400
  changed lines. Split refactors and moves away from behaviour changes.
- Each PR leaves the project building and working, even if the feature is not visible yet.
- Steps inside a PR are commits. Each step gets a suggested commit title in the repo's
  convention (check `git log --oneline -20`).
- Order PRs by dependency. Say which PR waits for which.
- Put risks and things to watch on the PR they belong to, not in a general list.
- Note follow-ups that came up but are not part of the goal, so they are not lost.

When the plan is written, tell the user in at most 5 lines: the plan's path, the PRs in one line
each, and how to start: `/clear`, then `/planner:next`.

## 7. Revise a plan

Plans change. What has shipped does not.

Read `index.md` and the PR files first. Then check git (`git log`, `git status`, `git branch -r`,
`gh pr list --state all` when `gh` is available) and update each status before changing
anything.

- **Merged PRs are frozen.** A change to merged work becomes a new PR.
- **Pushed commits are never rewritten**, wherever they are. No amend, rebase or force-push
  in the plan.
- **An open PR can still change, going forward.** Pick the way that fits, and let the user
  decide with AskUserQuestion:
  - **Add commits** (recommended when the change fits): a few new steps on the same PR, when
    they don't take much work and the PR still matches the steps after it.
  - **Close and replace:** when the change reworks the PR's core or no longer fits the later
    steps. Mark the old PR `closed #N, replaced by PR <M>` and plan a new one from scratch.
- **Not pushed yet:** steps implemented but not committed, and commits that exist only
  locally. These may be reworded, merged, split, reordered or dropped. If a dropped step has
  uncommitted changes in the working tree, say which files, and leave them alone.

Use the same flow as a new plan: research what is new, decide topic by topic if it is large,
then update the files. Add a line under Changes in `index.md` for each revision.

## Writing

- Plain words, short sentences. The reader is an engineer in the middle of other work. Prefer
  "slow" to "suboptimal", "use" to "leverage".
- Say each thing once. No summaries of what you just said.
- Name files, functions and commands exactly, so the next session can find them.
- Facts that may go stale (versions, line counts, API limits) get a date.
