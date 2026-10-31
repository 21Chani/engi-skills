---
name: next
description: Pick up a plan written by /planner:plan, recap the next step, and build it once the user says go. One step at a time.
disable-model-invocation: true
argument-hint: "[plan name] [PR N step M] [go]"
---

# Next step

Build the next step of a plan from `/planner:plan`. A step is one commit. The user stays in
control: recap first, build only after they say go, then stop.

Arguments: $ARGUMENTS

## 1. Find the plan

1. Read `.claude/planner.md` for the plans folder. If it is missing, there are no plans here:
   say so and point to `/planner:plan`.
2. A plan is active while any PR in its `index.md` is not `merged`, `closed` or `dropped`. Use the one
   named in the arguments; else the only active one; else ask which with AskUserQuestion.
3. Read `index.md`, `handoff.md` if it exists, and the PR file named on the **Resume** line.
   Read other PR files only when the step depends on them.

## 2. Check the plan against git

The plan may be behind git. Before the recap, check `git status`, `git log --oneline` on the
current branch and the default branch, and `gh pr list --state all` when `gh` is available.

- A step whose commit title is in the log is done, with its short sha.
- A PR that is open or merged is `open #N` or `merged #N <date>`.
- The next step is the first one that is not done.
- If the code no longer matches what the plan assumes (files moved, the step already done some
  other way), say so in the recap and offer `/planner:plan <slug>` to revise.

Keep these facts in mind; they are written to the plan in section 5, not now.

## 3. Recap and wait

Unless the arguments say `go` (or the user said to start in their own words), reply with a
recap of at most 8 lines and end the turn:

- **PR N, step M:** commit title
- **What it does:** one or two lines on the result, not the method
- **Where:** files it will touch
- **Check:** how we will know it works
- **Branch:** the branch, and whether it has to be created
- Anything from `handoff.md` or section 2 the user should know

End with one line: "Say `go` to start, or tell me what to change." Don't use AskUserQuestion
here: the recap is for reading, and a plain reply is faster than a dialog. With `go`, give the
recap in one line and start.

## 4. Build the step

- Follow the repo's own rules (`CLAUDE.md`, `.claude/rules/`, contributing docs). They win over
  this skill.
- Do this step only. If it turns out bigger than planned, or the plan is wrong, stop and ask.
  Small fixes to open steps are kept for section 6 and written then. Anything that changes
  the shape of the plan goes through `/planner:plan`.
- Never rewrite a pushed commit or touch a merged PR. A change to an open PR is a new commit
  on it; if the change reworks its core, stop and offer `/planner:plan` to close and replace it.
- Run the step's **Check**. If it fails, report the output; don't mark the step done.

## 5. Report and stop

Tell the user in at most 5 lines what changed and how it was checked. End with one line:
"`commit` to commit as `<title>`, `next` for the following step, or `stop`." Don't use
AskUserQuestion. Then end the turn.

- `commit` is the user asking to commit. Use the repo's commit skill or convention.
- `next` goes back to section 3 for the following step. Don't update the plan files yet.
- Never commit, push or open a PR unless the user asks. When the last step of a PR is committed,
  offer to push and open the PR.

## 6. Update the plan once

Don't touch the plan files while working. Update them in one pass when the work for this
session ends: the user says `stop`, a PR is finished, you suggest `/clear`, or the session is
about to end mid-step.

In that pass, rewrite each changed file with one Write call, all calls in one message:

- PR file: each step `[x] <sha>` if committed, `[~]` if built but not committed, `[ ]` otherwise.
- `index.md`: PR statuses (`in progress`, `open #N`, `merged #N <date>`), the **Resume** line,
  and a line under Changes for any fix made to open steps.
- `handoff.md`: write it if stopping mid-step, delete it if that step is now done.

Include the git facts from section 2, even when no step was built.

## Keeping context small

- One PR per session is a good limit. After a PR is done, or when the conversation has grown
  long, suggest `/clear` and `/planner:next` instead of carrying on.
- `handoff.md` (written in section 6 when stopping mid-step) says what is done, by file; what
  is left; what was found on the way; and changes in the tree that are not part of this step.
- Send wide searches to an Explore agent and keep only what it concludes.
