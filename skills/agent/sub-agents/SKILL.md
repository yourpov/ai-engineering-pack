---
name: sub-agents
description: >
  Use when delegating work to sub-agents, or deciding whether to delegate at all.
  Covers the model tier table (which class of model gets which class of work), the
  rule that every delegated call sets its model explicitly, cheapest-first
  escalation, and the cases where delegation costs more than it saves. Also covers
  how to write a brief an agent can act on, who owns commits, and verifying delegated
  work before accepting it. Trigger on "spawn an agent", "delegate this", "run these
  in parallel", "use a subagent", "fan this out", "which model should this run on",
  "brief a subagent", "should the agent commit", or any plan that hands work to more
  than one agent. Harness-agnostic.
---

# Sub-Agents, Delegation and Model Tiers

## The frame: delegation does not reduce tokens

Every sub-agent starts cold and re-derives context the parent already holds, so a
fan-out almost always spends **more** raw tokens than doing the work inline. What
delegation actually buys is parallelism, independence from a polluted context, and
the option to run cheap work on a cheap model.

Usage limits are weighted by model cost, so the saving comes from **routing**, not
from delegating. Anyone who reports large savings from a delegation policy is
reporting the effect of moving bulk work down a tier. If you delegate rarely, this
file saves you nothing.

## Never spawn unprompted

Sub-agents cost real money and real wall-clock time, and their output lands without
the human having seen the plan. Do not fan out because a task "looks parallel".

Delegate when the human asked for it. Otherwise propose it in one or two sentences
(the shape, and roughly what it costs) and wait for a yes. A task with several parts,
or one described as "thorough", is not a delegation trigger. Handle it inline.

## Set the model on every call

Every delegated call sets its model parameter **explicitly**. Omitting it silently
inherits the parent session model, so one expensive session quietly spawns a fleet
of expensive agents. That single omission is the most common source of runaway cost
in agent work, and it is invisible until the bill arrives.

## Tier table

| Tier | Work it should get |
|---|---|
| Cheapest (e.g. Haiku) | Mechanical bulk: renames, boilerplate, format conversion, log triage, gathering file lists |
| Mid (e.g. Sonnet) | The default. Well-specified implementation with clear acceptance criteria |
| Top (e.g. Opus) | Genuinely hard: concurrency, subtle algorithms, gnarly debugging, adversarial verification |
| Independent (e.g. Fable) | Rare. Only when independence from the parent's context is the point, such as reviewing the parent's own plan or a large diff. Ask first, every time |

Model names differ per harness and change over time. Map the harness to these tiers
rather than hard-coding a name this pack cannot guarantee still exists.

**When unsure between two tiers, pick the cheaper one and escalate on failure.** A
failed cheap attempt costs less than a needless expensive one, and the failure
usually sharpens the brief for the retry.

## Brief quality beats tier

A vague brief on an expensive model is worse value than a sharp brief on a cheap
one. Before reaching for a higher tier, check whether the task is genuinely hard or
merely under-specified. Acceptance criteria, exact file paths, and a stated
definition of done move work **down** a tier more reliably than any model upgrade.

## When not to delegate at all

- You already hold the context. Re-deriving it costs more than finishing the work.
- The change is small. Roughly 30 lines across one or two files stays inline.
- The task is sequential. Sub-agents cannot share intermediate state cheaply.
- The output needs your judgement anyway, so you will re-read every line of it.
- **Writing the brief would cost more than doing the task.** That is not an
  inconvenience, it is the signal that delegation is wrong here.
- You cannot write the acceptance criteria. If you cannot say what "done" looks
  like, a sub-agent cannot either.

## Brief like the agent has never met you

It has not. A sub-agent sees none of the conversation, none of the decisions you
already made, and none of the constraints the human stated an hour ago. Every brief
is self-contained or the work comes back wrong:

- Exact file paths, not descriptions of where the code probably lives
- The specific change, not the goal it serves
- Acceptance criteria you could check yourself
- The verify command to run, taken from the project rather than invented
- The commit policy, stated explicitly

A sharp brief is also what moves work **down** a tier. Most tasks that seem to need
an expensive model are just under-specified.

## Commit authority

Default: **the sub-agent does not commit.** The parent session owns staging and
commits, because the parent is the only one that can see whether the change is
actually finished.

One exception. An agent that is the sole writer in its own worktree, working through
several sequential tasks, can be told to commit atomically as it goes. Never grant
that to parallel agents sharing a worktree. Two agents staging into one index produce
commits that contain each other's half-finished work.

Fan out only when the work partitions cleanly by file, with no overlapping writes. If
two agents need to touch the same file, that is one task, not two.

## Verify before you accept

Never take a sub-agent's self-report as done. Agents report success on work that does
not compile, and they report it confidently.

Read the actual diff. Run the project's verify gate. Check the acceptance criteria you
wrote. Only then is the work accepted. If you granted commit authority, review
`git log -p` across its range and run the gate before treating any of it as finished.

## If you are the expensive tier

When you are the most costly model in the session, your comparative advantage is
decomposition, judgement, verification, and integration. Spend your tokens there and
route execution down.

Keep for yourself: planning, architecture, judgement calls, resolving ambiguity with
the human, and final acceptance. A context-free agent on your own tier doing those
costs the same and decides worse.

This is a default with criteria, not an absolute. Understanding a codebase, answering
a question, or making a small edit is still faster done directly.

## Reporting back

A sub-agent's report is not shown to the human. Relay what matters, in your own
words, including anything the agent failed to do. Never present a pending agent's
results as finished, and never invent what a running agent is likely to find.

**Cap what comes back.** Ask for findings, not transcripts. An uncapped agent
returns everything it read, that report lands in the parent's context, and the
parent then re-reads it on every turn for the rest of the session. Output is the
most expensive token there is, and a sub-agent's output becomes the parent's input
forever. Tell it where to leave the full evidence on disk and what to summarize.
See [`../context-budget/SKILL.md`](../context-budget/SKILL.md).

## Orchestrated workflows

Some harnesses ship a workflow or pipeline tool that runs several agents as one
scripted unit, with stages, fan-out, and judge panels. Check what yours actually has
before writing a policy for it. The rules below apply wherever that exists.

**Reach for it** when a task splits into three or more genuinely independent subtasks,
or when the work wants a pipeline shape: generate, then critique, then revise.

**Opt in first.** An orchestration is a bigger commitment than a single sub-agent and
the human has seen none of it. Unless they already turned orchestration on for the
session, propose the shape and the rough cost in a sentence or two and wait. Their yes
is the opt-in.

**Every agent call inside the script sets its model.** Same rule as everywhere else,
and it matters more here, because one omission multiplies across every stage.

**Keep the independent tier out of the script.** An agent chosen for independence from
your context is answering a question about your judgement, not executing a stage. If
that review is warranted, run it after the workflow finishes, as its own call, with the
human's agreement. Burying it in a pipeline gets you the cost without the independence.

---

## Drop this in your CLAUDE.md

The whole file is the reasoning. This is the part worth pasting into `~/.claude/CLAUDE.md`
so it applies to every session without being loaded on demand.

```markdown
## Delegating to sub-agents

Set the `model` parameter explicitly on every delegated call. Omitting it inherits the
session model, so an expensive session quietly spawns a fleet of expensive agents.

- `haiku`   mechanical bulk: renames, boilerplate, format conversion, log triage
- `sonnet`  default for well-specified implementation with clear acceptance criteria
- `opus`    genuinely hard: concurrency, subtle algorithms, gnarly debugging, adversarial review
- `fable`   rare. Only when independence from your context is the point. Ask me first, every time

When unsure between two tiers, pick the cheaper one and escalate on failure.

Never spawn unprompted. A task with several parts, or one I called "thorough", is not a
delegation trigger. Propose the shape and rough cost in two sentences, then wait for my yes.

Act directly when the change is under ~30 lines across one or two files, when you already
hold the context, or when writing the brief would cost more than doing the work.

Every brief is self-contained: exact paths, the specific change, acceptance criteria, and
the verify command from this project. State the commit policy in the brief. Default is
"do not commit, I own commits." Never grant commit authority to parallel agents sharing a
worktree.

Never accept a sub-agent's self-report. Read the diff and run the verify gate before you
call it done.
```

Two things to keep honest about it:

**Model names age.** Those are current Claude Code tiers. When the lineup changes, the tier
mapping survives and the names do not. Fix the names, keep the shape.

**Know why it works.** People report large drops in usage after adopting a policy like this,
and the mechanism is routing bulk work onto cheap models, not delegating more. Usage limits
are weighted by model cost. If you paste this and then delegate more often than you did
before, you will spend more, not less.
