# standards/ index

The universal layer. Nothing here mentions a framework, a vendor, or one of your projects,
so every file applies to every repo you own.

| Path | What it is | How to load it |
|---|---|---|
| **[Clean-Code/](./Clean-Code/README.md)** | 41 lesson standards, one rule per file. The mandatory craft baseline | Open the single `NN-*.md` that matches the task. Never the whole folder |
| **[Principles.md](./Principles.md)** | SOLID, Clean Architecture, testing, concurrency, security, and the surrounding philosophy | Section-load. It is long, and loading all of it wastes the budget you need for code |

## Which one answers your question

| Question | Go to |
|---|---|
| Is this name any good? Should this be a comment? Is this function doing too much? | `Clean-Code/` |
| Should this be an interface? How do I test this? Is this dependency pointing the right way? | `Principles.md` |

Craft belongs to `Clean-Code/`. Architecture and testing belong to `Principles.md`. Where
the two overlap, `Clean-Code/` wins and `Principles.md` gets corrected toward it.

## Authority order

1. Project **`AGENTS.md`**, and only for an exception it names explicitly
2. **`Clean-Code/`**
3. **`../skills/engineering/uncle-bob/SKILL.md`**
4. Other `../skills/*` and **`Principles.md`**

That order exists so an agent facing two pieces of contradictory advice has a rule to
follow rather than a preference to invent.
