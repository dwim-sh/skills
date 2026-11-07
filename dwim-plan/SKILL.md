---
name: dwim-plan
description: Plan a coding change for dwim, the local coding agent, to implement step by step: investigate the code, decide the design, split the work into commits, and write one self-contained task file per commit under .dwim/plans/ for /dwim-do to run. Use this whenever the user wants something implemented, built or fixed "with dwim" or "using dwim", asks for a plan dwim will carry out, or asks to break work down for dwim, even if they do not say "plan". Writes the plan only; runs nothing.
---

# dwim-plan

`dwim` runs a 27B model locally. It does well on one clear, bounded task
with everything it needs in the prompt, and badly on anything vague,
open-ended or spread over several steps: its context is small, it cannot
ask anyone, and a task that needs a design decision ends with the wrong
one. So the planning is yours. You read the code, make the decisions and
write each step out as a prompt dwim can carry out on its own. `/dwim-do`
then runs the steps one at a time.

This skill ends with the plan on disk and a short report. It runs no
step: the user reads the plan, edits what they want, and starts
`/dwim-do` themselves.

## 1. Investigate

Read the code until you can name the files, types and functions each step
touches, with your own tools or with subagents for wide reading. Read the
project's instructions (`AGENTS.md`, `CONTRIBUTING.md`) for the style and
the check commands: dwim gets `AGENTS.md` in its system prompt, and your
task files will restate the rules that matter. Where the request points
at prior art (a `third_party/` directory, a spec, another project's
implementation), study it and note what to borrow.

Ask the user only where a choice is genuinely theirs and the readings
lead to materially different work. Everything else you decide.

## 2. Decide and split

Settle the design before splitting: what is new, where it lives, how the
pieces talk to each other. Then cut the work into commits. A step is the
right size for dwim when it is one thing, such as one module with its
tests, one tool, one command, one documentation pass, and when you could
write its API down completely: type and function names, signatures,
error strings, constants with their values. If you would have to leave a
decision to dwim, the step is too big or the design is not done.

Steps build on each other through APIs the task files spell out. A later
step gets the earlier step's signatures written into it; it is never
told to go and look.

## 3. Write the plan

The plan is a directory in the repository, `.dwim/plans/<name>/`, where
`<name>` is a short kebab-case word for the change, such as `skills` or
`prompt-file`. Keep it out of git by adding `.dwim/` to
`.git/info/exclude` (not `.gitignore`: that would change the repository).
It holds:

```
.dwim/plans/<name>/
  PLAN.md               the goal, the design, the steps as a checklist
  01-<slug>.md          the task for step 1: a complete prompt for dwim
  02-<slug>.md
  …
```

### PLAN.md

```markdown
# <Title>

## Goal
What the user asked for, in a paragraph.

## Design
The decisions, in a few paragraphs: what is added, where, how the pieces
fit, what was borrowed from where. Enough that someone reading only this
could review the steps.

## Checks
The commands every step must pass, as the project runs them, such as
`cargo test`, `cargo clippy --all-targets -- -D warnings`, `cargo fmt --check`.
How to exercise the result by hand, if there is a way.

## Steps
- [ ] 01 <area>: <commit subject in the imperative>
- [ ] 02 <area>: <commit subject>
```

The checklist is the state `/dwim-do` reads and ticks. Commit subjects
follow the project's log: look at `git log --oneline` for the area
prefixes and the mood.

### Task files

Each `NN-<slug>.md` is the whole prompt for its step, forty to eighty
lines. Shorter leaves dwim guessing; longer it stops following. It
contains, in this order:

1. **The task in one sentence**, then the files to create or change, by
   path.
2. **The API, exactly.** Signatures dwim can copy, not descriptions it has
   to interpret. The wording of every error and message a test checks.
   Constants with their values. A doc comment's first sentence where the
   project cares about them.
3. **What each function does**, in enough detail to write it, with the
   edge cases named.
4. **The tests**: where they go, what each one sets up and asserts.
5. **The style rules that matter for this step**, restated from the
   project's instructions in a few lines.
6. **What to run**: the test filter, the linter, the formatter, "and fix
   what they report". Then: **Do not commit.**

A task file never mentions the plan, another step, or an alternative. It
does not say "read PLAN.md": that fills dwim's context with every step
and invites it to do the ones it should not. Where the step needs
something an earlier step made, write that API into the file, or name
the one file to read.

## 4. Report and stop

Tell the user, in a few lines: the design in a sentence or two, the steps
by subject, where the plan is, and that `/dwim-do` runs the first step.
Do not start a step.

## Example task file

```markdown
Add a `skills` module to the library: create skills/mod.rs and add
`pub mod skills;` to lib.rs.

The module holds:
- `pub struct Skill { pub name: String, pub description: String, pub dir: PathBuf }`:
  a skill, its name, what it is for, and the directory its SKILL.md is in.
- `pub fn parse(text: &str, dir: &Path) -> Result<Skill, String>`: parses a
  SKILL.md. The frontmatter is the lines between a first line of `---` and
  the next; without it the error is "no frontmatter". A line `key: value`
  sets a key; a line starting with whitespace continues the previous value,
  joined with a space. Only `name` and `description` are read. `description`
  is required: the error is "no description". `name` defaults to the
  directory's file name and must be 1 to 64 characters of lowercase ASCII
  letters, digits and dashes; otherwise the error is
  `name "X" is not lowercase letters, digits and dashes`.

Tests, in `mod tests` at the bottom of skills/mod.rs: a name defaulting to
the directory, a quoted value losing its quotes, a folded description, and
each of the three errors with its exact text.

Style: no pub(crate); one `use` per crate with nested paths, grouped std,
external, crate, separated by blank lines; a doc comment on every item in
plain sentences; a module doc comment saying what a skill is and the format.

Run `cargo test skills`, `cargo clippy --all-targets -- -D warnings` and
`cargo fmt`, and fix what they report. Do not commit.
```
