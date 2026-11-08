# dwim skills

Agent skills for working with [dwim](https://dwim.sh), the local coding
agent. Each directory is one skill, a `SKILL.md` with a name, a
description and instructions, in the format the open agent skills
ecosystem shares.

```sh
npx skills add dwim-sh/skills
```

| Skill | What it is for |
|---|---|
| [`dwim-plan`](dwim-plan/) | Plan a change for dwim to implement: investigate, decide the design, and write one self-contained task file per commit under `.dwim/plans/<name>/`. |
| [`dwim-do`](dwim-do/) | Run a plan step by step until it is done: hand each task to dwim, review the change, verify it, commit it with `Generated-by: dwim`, tick it off, and go on to the next. `next` or a step number runs just one. |

The division of labour: the agent you talk to plans, decides, reviews
and commits; dwim gets one bounded task at a time, with everything it
needs in the prompt, and types. A plan is a directory in the repository,
kept out of git, so it survives the session and the user can edit a
task before it runs:

```
.dwim/plans/skills/
  PLAN.md               goal, design, checks, and the steps as a checklist
  01-skills-module.md   a complete prompt for step 1
  02-list-in-prompt.md
  runs/                 each run's reply, tool calls and transcript
```

## Installing by hand

Into a directory your agent reads skills from, such as `.agents/skills/`
or `.claude/skills/` in a project, or the same under your home directory:

```sh
git clone https://github.com/dwim-sh/skills ~/src/dwim-sh/skills
ln -s ~/src/dwim-sh/skills/dwim-plan ~/.claude/skills/dwim-plan
ln -s ~/src/dwim-sh/skills/dwim-do ~/.claude/skills/dwim-do
```
