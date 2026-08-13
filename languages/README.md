# languages/ index

Pick **one** file per project, the one you are actually writing. These sit on top of
`../standards/`, they do not replace it: the standards say what good code is, these say
what good code looks like in this language.

| File | Use it for |
|---|---|
| **[TypeScript.md](./TypeScript.md)** | Web apps, desktop frontends, CLI tooling. The default for most work here |
| **[Bun-Node.md](./Bun-Node.md)** | Server-side JS/TS: routes, services, database layer, middleware, HTTP clients |
| **[Go.md](./Go.md)** | Concurrent services and daemons. Packages, interfaces, goroutine discipline |
| **[Python.md](./Python.md)** | Data work, ML, scripting. Modules, type hints, packaging |
| **[Cpp.md](./Cpp.md)** | Native and performance work. Memory and resource management, threading, hot reload |
| **[Luau.md](./Luau.md)** | Roblox only |

## What is in every file

Project Structure, Principles, Error Handling, Testing, Configuration, a Summary, and a
**Project Prompt** with Setup, Deliverables, Validation, and Pre-Delivery steps you can
hand straight to an agent.

Each one opens with an **Agent load** blockquote naming the sections to read first. That
line is the point of the whole folder. A language guide is thousands of tokens, and an
agent that swallows all of it has less room left for your actual code.

## Pairing

- Building a desktop app or a React frontend? Add one file from [`../frameworks/`](../frameworks/README.md).
- Reviewing rather than writing? Load [`../skills/review/audit/SKILL.md`](../skills/review/audit/SKILL.md) first.
- Unsure what to build with at all? [`../STACK.md`](../STACK.md) has a decision walkthrough.

## Adding a language

1. Copy the closest existing file and keep the section order.
2. Keep the required sections listed above, including the Project Prompt.
3. Copy the **Agent load** blockquote from any sibling and edit it to match.
4. Discover verify commands from the project. Never hard-code a toolchain that only exists
   on your machine.
5. Add a row to the table above.
