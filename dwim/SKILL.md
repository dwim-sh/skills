---
name: dwim
description: Delegate coding to dwim, the local coding agent. "/dwim plan <request>" investigates the code, decides the design, and writes one self-contained task file per commit under .dwim/plans/<name>/; it runs nothing. "/dwim do" runs that plan step by step until it is done: hands each task file to dwim, watches the run, reviews the change as a pull request, fixes and verifies it, commits it with Generated-by dwim, and ticks it off in PLAN.md. Use this whenever the user wants something implemented, built or fixed "with dwim" or "using dwim", asks for a plan dwim will carry out, says to run, continue, do or redo a dwim step or plan, or asks where a dwim plan stands, including bare "/dwim", "next step", "keep going with dwim" or "status".
---

# dwim

dwim is a coding agent that runs a small model locally. You plan and
review; dwim types. The work goes through a plan on disk:

```
.dwim/plans/<name>/
  PLAN.md          goal, design, checks, and the steps as a checklist
  01-<slug>.md     the task for step 1: a complete prompt for dwim
  02-<slug>.md
  runs/            each run's reply, tool calls and transcript
```

`PLAN.md`'s `## Steps` checklist is the state: `- [ ] 01 <subject>` until
the step is committed, then `- [x] 01 <subject> (<short hash>)`.

## Commands

- `/dwim plan <request>`: write a plan for the request. Read
  [plan.md](plan.md) and follow it. It ends with the plan on disk and a
  short report; never go on to run a step, however obvious the next move
  looks. The user reads the plan first.
- `/dwim do [<plan>] [next | N | redo | all | status]`: run the plan.
  Read [do.md](do.md) and follow it. With no argument it runs every
  unticked step in a loop until one fails review or none remain.
- `/dwim status`: the same as `/dwim do status`.
- `/dwim` alone: if a plan under `.dwim/plans/` has unticked steps, say
  which and run `/dwim do`. Otherwise ask what to plan.
- Anything else after `/dwim`, such as `/dwim add a --json flag`, is a
  request to plan: treat it as `/dwim plan` with that request.

Read only the file for the command at hand.
