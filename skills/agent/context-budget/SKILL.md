---
name: context-budget
description: >
  Where an agent session's tokens actually go, and which levers move the number.
  Covers the startup floor (everything always-loaded is billed on every single
  call), prompt caching as a prefix match and what silently invalidates it, why
  switching models mid-session is the most expensive habit there is, and why
  output is the costliest token you can spend. Use when a session feels
  expensive, when usage limits keep getting hit, when cache hit rate is low, or
  when deciding what belongs in always-loaded context. Trigger on "reduce token
  usage", "why is this so expensive", "hitting my limits", "prompt caching",
  "cache hit rate", "context is full", "should I compact", "trim CLAUDE.md".
  Harness-agnostic.
---

# Context Budget

Most advice about agent cost is about doing less work. This is about paying less
for the same work.

Three levers move the number, roughly in order of how much they matter:

1. **The startup floor.** Everything always-loaded is billed on every single call.
2. **The cache.** A cache read costs about a tenth of a fresh read. Small habits
   throw it away without any visible signal.
3. **Output.** The most expensive token you can spend, and it becomes input on
   the next turn and every turn after that.

## 1. The startup floor

Every request re-sends the whole conversation. Your system prompt, your
`CLAUDE.md`, every installed skill's description, every MCP server's tool
schemas, every hook's output. That block is the floor, and you pay it on turn one
and turn eighty alike.

Caching softens the floor but cannot remove it, because a cache read is still
billed. The only way to shrink a floor is to put less on it:

- Keep `CLAUDE.md` short. Push depth into linked files the agent opens on demand.
  See [`../claude-folder/SKILL.md`](../claude-folder/SKILL.md).
- Uninstall skills and MCP servers you do not use. Every one of them ships a
  description into every request whether it fires or not.
- Check what your hooks print. A chatty `SessionStart` hook is a permanent tax.

This is also why the pack's core rule exists: load one file at a time. A doc the
agent opens for one task costs once. A doc in always-loaded context costs forever.

## 2. The cache is a prefix match

**Any byte that changes anywhere in the prefix invalidates everything after it.**
That single sentence explains almost every surprising cache miss.

The prompt renders in a fixed order: **tools, then system, then messages**. So
anything volatile placed early poisons everything downstream of it.

| Operation | Cost, relative to an uncached read |
|---|---|
| Cache write, 5-minute TTL | 1.25x |
| Cache write, 1-hour TTL | 2x |
| Cache read | 0.1x |

A 5-minute cache pays for itself on the second request (1.25 + 0.1 against 2.0
uncached). The 1-hour TTL costs double to write, so it needs a third request
before it wins. Use it for gaps in bursty traffic, not by default.

### Not everything invalidates everything

This is the part people get wrong. There are three cache tiers, and a change only
invalidates its own tier and below:

| What changed | Tools | System | Messages |
|---|---|---|---|
| Tool definitions added, removed, or reordered | lost | lost | lost |
| **Model switch** | lost | lost | lost |
| System prompt content | kept | lost | lost |
| `tool_choice`, images, thinking toggled | kept | kept | lost |
| Message content | kept | kept | lost |

So per-turn changes are cheap and structural changes are not. **Switching models
mid-session is the single most expensive habit available to you**, because it is
a full rebuild with no escape hatch: caches are model-scoped. If a cheap sub-task
wants a cheap model, delegate it to a sub-agent and leave the main loop where it
is. See [`../sub-agents/SKILL.md`](../sub-agents/SKILL.md).

Editing `CLAUDE.md` mid-session is safe, because it does not take effect until
the session reloads.

### Silent invalidators

Nothing errors. The cache just never hits. Grep your prompt-building path for:

- `datetime.now()`, `Date.now()`, or any timestamp in the system prompt
- UUIDs or request IDs generated per call and placed early
- JSON serialized without sorted keys, or anything iterating a set
- A user ID or session ID interpolated into the system prompt, which gives every
  user their own private prefix that nobody else can share
- Conditional system sections, where every flag combination is a separate prefix
- A tool list built per user, which lands at position zero and caches for nobody

The fix is the same in every case: make it deterministic, or move it after the
last cache breakpoint. A fact injected at turn five invalidates nothing before
turn five.

### Two gotchas worth knowing

**Minimum cacheable prefix.** Below it, nothing caches and nothing tells you.
The threshold is model-dependent and it is **not monotonic across generations**,
so a prompt that caches on one model silently will not on another. Check the
current number for the model you are on rather than assuming.

**The 20-block lookback.** A cache breakpoint searches backward at most 20
content blocks for a prior entry. Agentic loops blow through that easily, since
every tool call adds two blocks. A turn with 15 tool calls has already pushed the
previous breakpoint out of reach, and the next request silently starts cold.

### Verify instead of assuming

The response usage object tells you the truth:

- `cache_read_input_tokens` served at 0.1x
- `cache_creation_input_tokens` written at 1.25x or 2x
- `input_tokens` **only the uncached remainder**, not the total

That last one trips people up. Total prompt size is all three added together. A
long session reporting a small `input_tokens` is a session with a working cache,
not a small session.

If `cache_read_input_tokens` is zero across repeated requests with what should be
an identical prefix, you have a silent invalidator. Diff the rendered bytes of
two consecutive requests and you will find it.

## 3. Output is the most expensive token

Output costs several times what input costs, and then it gets re-read as input on
every subsequent turn. You pay for it once at the high rate and then forever at
the low one.

- **Keep evidence on disk, not in context.** Write the full log, the whole test
  output, the complete diff to a file and read back the part that matters. A
  60,000-token log dumped into the conversation is not a one-time cost.
- **Cap sub-agent reports.** An uncapped agent returns everything it saw, and the
  parent then re-reads that report on every turn that follows. Ask for findings,
  not transcripts.
- **Do not paste whole files when you need a function.**

## 4. Measure, do not guess

Every number above is a mechanism, not a prediction. What is actually expensive
in your sessions is an empirical question, and the answer differs per project.

Two rules keep the measurement honest:

**Count tokens per accepted result, not per response.** An optimization that
halves token spend and doubles rework is not an optimization. Cheap output you
throw away costs more than expensive output you keep.

**Change one thing at a time.** Trim `CLAUDE.md` and drop three MCP servers in
the same session and you have learned nothing about either.

[token-shield](https://github.com/khalilmaaouni/token-shield) is a Claude Code
plugin that reads real usage counters out of your local session transcripts and
reports cache hit ratios, first-request cost, and how much of your output came
from sub-agents. It measures rather than estimates, and prints `NO DATA` instead
of guessing when it cannot measure something. It is the fastest way to find out
which of the three levers above is actually costing you.

---

## Drop this in your CLAUDE.md

```markdown
## Context budget

Do not switch models mid-session. A model switch discards the entire prompt cache
and re-bills the whole prefix. If cheap bulk work needs a cheap model, delegate it
to a sub-agent and leave this session where it is.

Keep evidence on disk and out of context. Write full logs, test output, and large
diffs to a file, then read back only the part that matters. Never paste a whole
file when a function will do.

Cap what sub-agents report. Ask for findings, not transcripts. Their output
becomes my input on every turn after this one.
```

## One honest caveat

People report large drops in usage after adopting a policy like this, and the
mechanism is real, but it is not magic. The saving comes from paying less for
work you were doing anyway: a warm cache instead of a cold one, bulk work routed
to a cheap model, evidence on disk instead of in the transcript. None of it makes
a session that does more work cost less.
