# engi-skills

Claude Code plugins by 21Chani.

## Install

```bash
claude plugin marketplace add 21Chani/engi-skills
claude plugin install planner@engi-skills
claude plugin install pr@engi-skills
```

## planner

Plan a feature as small PRs, then build it one commit at a time across sessions. Both skills
run only when you type them.

| Command | Does |
|---|---|
| `/planner:plan <goal>` | Researches the code and the web, settles decisions with you topic by topic, and writes the plan as markdown. Run it on an existing plan to revise it. Never writes code. |
| `/planner:next [go]` | Loads the plan, syncs it with git, recaps the next step and waits for your go. Builds one step, then stops. |

The first `/planner:plan` in a repo asks where plans live and whether git tracks, ignores or
locally excludes them. The answer is saved in `.claude/planner.md`.

A plan is a folder: `index.md` holds the goal, decisions, PR table and where to resume; each
PR has its own file with its steps. Merged PRs are frozen and pushed commits are never
rewritten. An open PR gets new commits, or is closed and replaced when the change reworks its
core; you choose. The rest of the plan can change freely.

## pr

| Command | Does |
|---|---|
| `/pr:describe [PR or branch] [notes]` | Reads the diff, commits and conversation, and writes a short PR description: what it does and why, in a line or two. Asks only when the answer changes the text. Shows the draft and posts it only when you say `post`. |

Claude can also use it on its own when opening a PR. Text in `[brackets]` in your notes is
for Claude, not for the PR.
