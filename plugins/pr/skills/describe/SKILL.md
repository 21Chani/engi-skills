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
- **House style:** `.github/pull_request_template.md` if it exists (fill it, lightly), and the
  titles of a few recent merged PRs (`gh pr list --state merged --limit 5`) for tone and
  language. Write in the language the repo's PRs use.
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
- the user's notes contradict the code.

Ask in one go, at most three short questions in plain text, then wait. If only one detail is
uncertain, draft anyway and mark it so the user can confirm, rather than blocking on it.

## 4. Draft

Default shape (follow the repo template instead if there is one):

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
- No headings beyond the first, no bold labels like **Checked:**, no bullet list for two facts.

## 5. Show, then post

Show the title and description to the user and end the turn. Say: "`post` to put it on the PR,
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
