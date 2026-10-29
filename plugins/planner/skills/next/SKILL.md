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

## 2. Bring the plan up to date

The plan may be behind git. Before the recap, check `git status`, `git log --oneline` on the
current branch and the default branch, and `gh pr list --state all` when `gh` is available.

- A step whose commit title is in the log becomes `[x]` with its short sha.
- A PR that is open or merged gets `open #N` or `merged #N <date>`.
- Move the **Resume** line to the first step that is not `[x]`.
- If the code no longer matches what the plan assumes (files moved, the step already done some
  other way), say so in the recap and offer `/planner:plan <slug>` to revise.

Write these updates to the files without asking; they record facts, not decisions.

## 3. Recap and wait

Unless the arguments say `go` (or the user said to start in their own words), show at most 8
lines and wait:

- **PR N, step M:** commit title
- **What it does:** one or two lines on the result, not the method
- **Where:** files it will touch
- **Check:** how we will know it works
- **Branch:** the branch, and whether it has to be created
- Anything from `handoff.md` or step 2 the user should know

Then ask with AskUserQuestion: **Start** (recommended), **Revise the plan first**
(`/planner:plan <slug>`), **Pick another step**. With `go`, give the recap in one line and start.

## 4. Build the step

- Follow the repo's own rules (`CLAUDE.md`, `.claude/rules/`, contributing docs). They win over
  this skill.
- Do this step only. If it turns out bigger than planned, or the plan is wrong, stop and ask.
  Small fixes to open steps can go straight into the PR file, with a line under Changes in
  `index.md`. Anything that changes the shape of the plan goes through `/planner:plan`.
- Never rewrite a pushed commit or touch a merged PR. A change to an open PR is a new commit
  on it; if the change reworks its core, stop and offer `/planner:plan` to close and replace it.
- Run the step's **Check**. If it fails, report the output; don't mark the step done.

## 5. Record and stop

1. Mark the step `[~]` and update **Resume**.
2. Tell the user in at most 5 lines what changed and how it was checked.
3. Ask with AskUserQuestion:
   - **Commit as `<title>`**: choosing it is the user asking to commit. Use the repo's commit
     skill or convention, then mark the step `[x] <sha>`.
   - **Next step**: back to section 3 for the following step.
   - **Stop here.**

Never commit, push or open a PR unless the user picks or asks for it. When the last step of a PR
is committed, offer to push and open the PR, and mark it `open #N` once it exists.

## Keeping context small

- One PR per session is a good limit. After a PR is done, or when the conversation has grown
  long, suggest `/clear` and `/planner:next` instead of carrying on.
- When stopping in the middle of a step, write `handoff.md` in the plan folder: what is done, by
  file; what is left; what was found on the way; and changes in the tree that are not part of
  this step. Delete it when the step is done.
- Send wide searches to an Explore agent and keep only what it concludes.
