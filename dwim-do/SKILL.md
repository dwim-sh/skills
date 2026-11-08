---
name: dwim-do
description: Run a dwim plan step by step until it is done: hand each step's task file to dwim, the local coding agent, watch the run, review the change as a pull request, fix it, verify it, commit it with Generated-by dwim, and tick it off in .dwim/plans/<name>/PLAN.md, then go on to the next. Use this whenever the user says to run, continue, do or redo a dwim step or plan, asks dwim to implement the next step, or asks where a dwim plan stands, including bare "/dwim-do", "next step", "keep going with dwim" or "status". Takes an optional plan name, then a step number, "next", "redo" or "status".
---

# dwim-do

A plan written by `/dwim-plan` is a directory `.dwim/plans/<name>/` with
a `PLAN.md` whose `## Steps` checklist is the state, and one task file
`NN-<slug>.md` per step. This skill runs the plan's steps one after
another, each end to end: dwim types,
you watch, review, fix, verify and commit. dwim's work is not done until
you have read it and the checks pass under your own hands; the commit is
yours.

## The arguments

- none (or `all`): every unticked step in order, in a loop, until one
  fails review or none remain, with the report for each as it lands.
- `next`: only the first unticked step, then stop.
- `N`: only step N, whatever its state, then stop.
- `redo`: the first unticked step again, after the user edited its task
  file; throw away what the last run left in the tree first
  (`git checkout -- .` and `git clean -fd` for the files that run made,
  never for the plan directory).
- `status`: the checklist with each step's subject, ticked or not, and
  stop.

A plan's name may come first, as in `/dwim-do skills` or
`/dwim-do skills next`, to pick the plan under `.dwim/plans/` when there
are several; alone, it runs all of that plan's unticked steps. With
nothing unticked, print the status and stop.

## 1. Find the plan and get ready

The plan is the directory under `.dwim/plans/` whose `PLAN.md` has an
unticked step; with several, ask which. Read `PLAN.md` for the checks and
the step's subject, and the task file in full: you are about to review
against it.

Before running, check that the tree is clean (`git status`), that you are
on a branch and not `main` (if on `main`, branch as the plan is named),
that the build is green, and that dwim is at hand: `./target/debug/dwim`
when the repository is dwim's own, so that it tests itself, and otherwise
`dwim` on the path. A missing binary is the user's to fix: say so and
stop.

## 2. Run dwim

A step takes minutes to tens of minutes, so run it in the background,
with the reply, the tool calls and the transcript in files under the
plan's `runs/` directory:

```sh
mkdir -p .dwim/plans/<name>/runs
./target/debug/dwim --transcript .dwim/plans/<name>/runs/01.jsonl \
    "$(cat .dwim/plans/<name>/01-<slug>.md)" \
    > .dwim/plans/<name>/runs/01.out 2> .dwim/plans/<name>/runs/01.err
```

dwim's reply goes to `.out`; each tool call, as `edit(path at line 12)`
or `bash(cargo test)`, with its output, goes to `.err`. Given a prompt
dwim runs in auto mode: it goes on by itself until it says it is done.

Don't edit the repository while it runs: dwim is editing it. Check on it
now and then with the last calls:

```sh
grep -E '^(bash|read|edit|write|grep|glob)\(' .dwim/plans/<name>/runs/01.err | tail -n 15
```

It is lost when it reads the same file over and over, when twenty minutes
pass without an `edit(` or `write(`, or when it edits files the step does
not name. Then kill it (`pkill -f 'dwim --transcript'`), look at what it
did with `git status` and `git diff`, and decide: keep what is good and
rerun with the task file sharpened (usually the file or the API it could
not find, spelled out), or revert and rerun. Tell the user when this
happens.

## 3. Review as a pull request

When it finishes, read its reply in `.out`, then review the change as
you would someone else's:

1. **Scope.** `git status` and `git diff`. Is it the step, the whole step
   and nothing else? Revert files the step did not name. A test deleted
   or weakened to pass is a defect, not a change.
2. **Checks.** Run every command under `## Checks` in `PLAN.md` yourself.
   dwim's "tests pass" is a claim; the run is the evidence.
3. **Code.** Read the diff against the task file. Fix small things by
   hand: a doc comment, a name, a missed edge case, a format.
4. **Corrections.** For a bigger miss, give dwim one correction as a new
   prompt of its own: name the file, say what is wrong and exactly what
   to change, and run it the same way to `runs/01-fix.out`. One round. If
   it still is not right, finish by hand and say so in the report: a
   second round costs more than typing.
5. **Exercise.** If the change touches the agent itself, rebuild and ask
   it something that uses the change. Tests miss replies that end with
   nothing to show.

## 4. Commit and tick

Commit with the step's subject from `PLAN.md`, the reasoning in the body
in the project's style, and the trailer `Generated-by: dwim` when dwim
wrote the change, as dwim's own `CONTRIBUTING.md` asks. Fixes of your own
are part of the commit; name them in the body if they changed what dwim
did. Then tick the step in `PLAN.md` with the commit's short hash:

```
- [x] 01 skills: Find the skills in the project and the user's directories (3fa26cf)
```

The plan directory is outside git, so the tick is not part of the commit.

## 5. Report

In a few lines: what dwim did, how long and how many calls (count the
`(`-lines in `.err`), what you fixed, the commit hash, and what is next.

## 6. Loop

Unless the argument was `next`, `N` or `redo`, go back to step 1 for the
next unticked step, without waiting for the user. Stop when none remain
and print the status, or at the first step whose review fails, a run
that is lost twice, or a check you cannot make pass, and say why.
