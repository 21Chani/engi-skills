---
name: open
description: Open a pull request safely. Checks whether the branch already has a PR, works out the right base (default branch or stacked), fits the title to the repo's title checks, and writes the description. Use whenever the user asks to open, create or raise a PR.
argument-hint: "[base branch | stacked | draft] [notes]"
---

# Open a PR

Open the PR the user means, once, with a title that passes the repo's checks on the first try.
"Open a PR" is short for several different wishes; most of this skill is about telling them
apart before anything is pushed.

Arguments: $ARGUMENTS

## 1. Baseline

Collect these quietly, in one or two shell calls, before deciding anything:

- **Default branch:** `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
- **Current branch** and its upstream: `git branch --show-current`, `git status -sb`.
- **Working tree:** uncommitted or untracked changes (`git status --porcelain`).
- **Commits:** what the branch has over the default branch (`git log --oneline <default>..HEAD`)
  and what is not pushed yet (`git log --oneline @{u}..HEAD` when there is an upstream).
- **PRs for this branch, any state:**
  `gh pr list --head <branch> --state all --json number,state,title,url,baseRefName,headRefOid`.
- **Where the branch came from**, when it isn't the default branch: whether its merge base sits
  on another open PR's branch (`gh pr list --state open --json number,headRefName,url`, then
  `git merge-base --is-ancestor origin/<that-branch> HEAD`). That is how a stacked branch shows.

## 2. Work out what the user wants

Match the baseline to one of these cases.

| Situation | Most likely meaning |
|---|---|
| Feature branch, no PR, commits ahead of its base | Open a PR for this branch. |
| Branch already has an **open** PR, nothing new | The user missed it. Don't open another one: give the link. |
| Open PR, plus new local commits or changes | Either push them to that PR, or start a new PR on top of it (stacked) or off the default branch. |
| Branch's PR is **merged or closed**, new commits after it | A new PR for the new work, usually off the default branch. |
| On the default branch with local commits or changes | A new branch for them, then a PR into the default branch. |
| Branch based on another open PR's branch | A stacked PR, with that branch as the base. |

**Decide alone only when you are over 90% sure.** Sure means the baseline points one way and
nothing in the request or the conversation points another. Things that settle it: the user's
own words ("stack it", "from main", "new PR for this"), the arguments, the plan from
`/planner:plan` if `.claude/planner.md` points to one that covers this work, and an
unambiguous row above (no PR, clean branch, one obvious base).

Otherwise ask with one AskUserQuestion call, recommended option first, and say in the question
what you found ("`feat/login` already has open PR #42, and you have 2 new commits"). Typical
options:

- **Use the existing PR #N:** push the new commits there and stop. No new PR.
- **Stacked PR on #N:** a new branch from here, PR base = `<current branch>`.
- **New PR off `<default>`:** a new branch from `<default>`, with only the new work.
- **New branch from `<default>`** (when on the default branch): ask for or suggest a name.

Ask about the branch name only if the repo's branch names follow a pattern you can't infer
(check `git branch -r` for one).

### Uncommitted changes

Never commit unless the user asked. If there are uncommitted changes, list them in one line and
ask whether they belong in this PR, in the same AskUserQuestion call as above when there is one.
Commit only what they say, in the repo's commit convention.

### Moving work to a new base

For "new PR off the default branch" when the commits sit on top of other unmerged work: create
the new branch from `origin/<default>` and cherry-pick only the commits that belong to the new
PR. Show which commits you will take before doing it. Never rebase, reset or force-push the
existing branch; it is pushed and may have reviewers.

## 3. Title: check the repo's rules first

PR title checks fail often and quietly block merges. Before writing the title, find out what the
repo enforces, in this order:

1. **CI on PR titles:** `grep -rliE 'pull_request|pr-title|semantic|commitlint|conventional' .github/workflows`,
   then read the matches. Common ones: `amannn/action-semantic-pull-request` (reads `types`,
   `scopes`, `requireScope`, `subjectPattern`, `headerPattern`, `validateSingleCommit`),
   `deepakputhraya/action-pr-title` (`regex`, `prefixes`, `min_length`, `max_length`),
   commitlint run on the title, ticket-key checks (`[A-Z]+-\d+`), and Danger or custom
   scripts. GitLab: the MR title rules in `.gitlab-ci.yml`.
2. **Lint config the check reads:** `commitlint.config.*`, `.commitlintrc*`, the `commitlint`
   key in `package.json`, `.github/semantic.yml`. Note allowed types, allowed scopes, case
   rules (`subject-case`), and max header length.
3. **Written rules:** `CONTRIBUTING.md`, the PR template, the repo's `CLAUDE.md`.
4. **House style:** `gh pr list --state merged --limit 10 --json title`. This shows scope
   names actually in use and whether ticket keys are expected.

Then write the title to fit every rule found. When a check gives a regex, test the title
against it in the shell before opening the PR. Pick the type and scope from the commits and
the diff, not from the branch name alone. If a rule needs something you don't have (a ticket
key, a required scope with no clear match), ask for it with the other questions in section 2.

With no rules at all, follow the recent merged titles; if those have no pattern either, use
Conventional Commits. With squash merges the title becomes the commit on the default branch,
so it should read like one.

## 4. Description

If the `pr:describe` skill is available, use it to write the body: it reads the diff, fills the
repo's template and keeps the text short. Opening the PR is the go to post, so don't stop for
its `post` confirmation; questions it needs answered still come first.

If it isn't available, write the body the usual way: follow the repo's PR template if there is
one, otherwise a few plain lines on what the PR does and why.

For a stacked PR, the first line says what it builds on: "Stacked on #42; merge that first."

## 5. Push and open

1. Push the branch: `git push -u origin <branch>`. Plain push only; never force.
2. Write the body to a file in the scratchpad so quotes and backticks survive.
3. `gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>`, with
   `--draft` if the user asked for a draft.
4. If the repo has a title check, look at it once: `gh pr checks <n>` right after creation may
   still be pending; don't wait on it. If it already failed, fix with `gh pr edit <n> --title`.

Reply with the PR link, the title, the base, and one line for anything left to the user (empty
template sections, a check still pending). Nothing else.

For "use the existing PR", push and reply with that PR's link and how many commits went up.
