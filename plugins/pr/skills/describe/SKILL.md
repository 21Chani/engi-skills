---
name: describe
description: Write or rewrite a pull request description that reads like a short, friendly explanation of what the change does and why. Use when opening a PR, when the user asks for a PR description, or to shorten one that is too long.
argument-hint: "[PR number or branch] [notes]"
---

# PR description

Write the description a teammate would give out loud: what this PR does for the project and
why, in a few plain lines. A reader should get it from the first one or two lines. It is not
a code tour, a changelog of files, or a lesson.

Arguments: $ARGUMENTS

## 1. Gather context

Collect what the change is, quietly, before asking anything:

- **The change:** if a PR number or URL is given, `gh pr view <n> --json title,body,files,commits`
  and `gh pr diff <n>`. Otherwise the current branch against the default branch:
  `git log --oneline <base>..HEAD` and `git diff <base>...HEAD --stat`, then the diff itself.
- **The why:** the conversation so far, commit messages, linked issues, and a plan from
  `/planner:plan` if `.claude/planner.md` points to one that covers this branch.
- **Template:** look for a PR template before drafting. GitHub accepts
  `pull_request_template.md` in `.github/`, the repo root or `docs/`, any capitalization,
  and several templates in a `PULL_REQUEST_TEMPLATE/` folder in any of those places
  (GitLab: `.gitlab/merge_request_templates/`). One quick search covers it:
  `git ls-files | grep -iE '(pull_request|merge_request)_template'`. With several templates,
  pick the one that matches the change by name; ask only if none clearly fits. If there is a
  template, section 4 follows it.
- **House style:** the titles and bodies of a few recent merged PRs
  (`gh pr list --state merged --limit 5 --json title,body`) for tone, language and how
  people actually fill the template. Write in the language the repo's PRs use.
- **Notes from the user:** anything in the arguments. Text in `[brackets]` is a note for you,
  not text for the PR.

An existing long description is a source of facts, not a draft to trim line by line.

## 2. Decide what goes in

The skill's job is to guess what the user would say about this change, and leave the rest out.
Sort what you found:

- **Always:** what the PR does, as a result someone notices (behavior, not classes). Why it
  was needed, if it isn't obvious from the what.
- **Usually:** defaults and how to change them; what this sets up for later, when the PR is
  clearly a first step.
- **Only if it's worth it:** a "why this way" note, when a reviewer would otherwise ask or
  undo it (a non-obvious workaround, a constraint that bites, a value that must never change).
  One sentence each, at most two of them.
- **Only if a reviewer must act:** what wasn't tested, or a known gap. One short line.
- **Leave out:** file paths, class and method names (unless the reader needs them to find or
  use the thing), lifecycle hooks, how it was implemented, "both commits compile", and
  anything the diff already says plainly.

Size follows the change, not the effort. A two-file feature gets two to four lines. A big PR
may get a short list, but each item stays one line.

## 3. Ask only when it changes the text

Don't ask about what you can read or reasonably guess. Ask when the answer changes what the
description says, for example:

- the reason for the change is nowhere in the code, commits or conversation;
- something could go either way and matters to readers (is this user-facing yet? is the
  untested part worth flagging?);
- the user's notes contradict the code;
- the template has a section only the author can fill: how it was tested, evidence,
  screenshots, recordings, reproduction steps (see section 4).

Ask in one go, at most three short questions in plain text, then wait. If only one detail is
uncertain, draft anyway and mark it so the user can confirm, rather than blocking on it.

## 4. Draft

### With a repo template

Keep the template's headings, in its order; reviewers scan for them. Then go section by
section, by what kind of section it is, not by its exact name:

- **Instructions to the author** ("Describe your change here", "Remove what doesn't apply",
  HTML comments): follow them, then remove them.
- **Summary or description:** the short explanation from section 2, in the voice below.
- **Classification** (kind of change, area, scope, breaking or not): decide it yourself from
  the diff and commits; a `feat:` commit is a feature, a `deps` bump is dependencies. Do what
  the template says with the options it lists: tick `[x]` the ones that apply, and delete
  the rest if it asks you to.
- **Author checklist** (code reviewed by its author, docs updated, no new warnings, tests
  cover the change): these are the author's promises, so leave them unticked for the author.
  Tick one only when the diff itself proves it, like "added tests" when the diff adds tests.
- **Testing, evidence, screenshots, how to reproduce, QA notes:** this is the author's to
  say, and often a place for images or a recording. Use what the user told you, in their
  words. Otherwise leave the section empty under its heading for them to fill, and say so
  when you show the draft. Never invent a test run, and never write commands as the test
  story ("ran `go test ./...` and `make test`"); nobody fills it that way. When the change
  needs no testing (docs, a typo, a comment, config nobody runs), a short honest line is
  right: "Not needed, docs only." or "Not tested."
- **Sections that don't apply** (no linked issue, no new dependencies): "None." on one line,
  or drop the section if the template marks it optional.

The voice rules below still apply inside each section. Don't add sections the template
doesn't have, except a one-line heads-up from section 2 when a reviewer must act.

### Without a template

Default shape:

```markdown
# This PR will:

<One or two sentences: what it does and why.>

<Optional: defaults, how to change them, what it sets up next.>

<Optional: one "why this way" or "heads up" line.>
```

Voice:

- Talk to the reader. "Without it the game runs as fast as it can and burns GPU for nothing."
- Short sentences, everyday words. Name the effect, not the mechanism.
- Code formatting only for things the reader types or looks for (a setting name, a command).
- No headings beyond the first (or the template's own), no bold labels like **Checked:**, no
  bullet list for two facts.

## 5. Show, then post

Show the title and description to the user and end the turn. If you left template sections
empty for them (testing, evidence), list them in one line. Say: "`post` to put it on the PR,
or tell me what to change." Posting is visible to others; never post without that go.

- `post` on an existing PR: `gh pr edit <n> --body-file <file>` (and `--title` if it changed).
- No PR yet: `gh pr create --title ... --body-file <file>` only if the user asked to open one;
  otherwise give them the text.
- Write the body to a file in the scratchpad first so quotes and backticks survive.

## Example

A PR that caps the frame rate, two files. The original description walked through the static
class, its load hooks, the enum ordering, the focus event and a test report. What the user
actually wanted to say:

```markdown
# This PR will:

Cap the frame rate at 60 FPS. Without a cap the game renders as many frames as it can
(around 300 in the Editor) and keeps the CPU and GPU busy for nothing. When the window loses
focus it drops to 15 FPS.

There are three modes (capped, VSync, uncapped), ready to plug into user settings later.

Heads up: the 15 FPS drop when unfocused couldn't be tested yet.
```

What stayed: the effect, the reason, the default, what it sets up, and the one thing a
reviewer should try by hand. What went: file paths, `BeforeSceneLoad`, VSync internals, enum
ordering, "compiles with no warnings". Any of those would come back only if a reviewer would
otherwise get it wrong.

## Example with a template

The same PR in a repo whose template has a summary, a list of change kinds ("keep only the
ones that apply"), a pre-merge checklist and a "Verification" section. The user didn't say
how they checked it:

```markdown
## Summary

Cap the frame rate at 60 FPS. Without a cap the game renders as many frames as it can and
keeps the CPU and GPU busy for nothing. When the window loses focus it drops to 15 FPS.

## Kind of change

- [x] New feature

## Before merging

- [ ] Reviewed my own diff
- [ ] Docs updated where needed
- [ ] No new warnings
- [ ] Tests cover the change

## Verification

```

The kind was picked from the diff, the instruction lines and unused options are gone, the
checklist is left for the author, and Verification waits for the user's own words or
screenshots.
