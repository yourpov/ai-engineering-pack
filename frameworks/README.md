# frameworks/ index

Framework conventions layered on top of a language file. Always load a
[`../languages/*`](../languages/README.md) file as well, since these assume the language
rules are already in play.

| File | Use it for | Pair with |
|---|---|---|
| **[React-Tailwind.md](./React-Tailwind.md)** | Web UI: components, custom hooks, state, Tailwind patterns | `TypeScript.md` |
| **[Tauri.md](./Tauri.md)** | Desktop apps with a Rust backend and a web frontend. Small binaries, real system access | `TypeScript.md` |
| **[Electron.md](./Electron.md)** | Desktop apps where you want the whole Chromium runtime. Main process, preload bridge, IPC | `TypeScript.md` |
| **[Valkyrie.md](./Valkyrie.md)** | Desktop apps on the system WebView. Roughly 2 MB binaries instead of Electron's 150 to 300 MB | `TypeScript.md` |

## Choosing a desktop framework

| You want | Take |
|---|---|
| Native performance, small binary, Rust backend | Tauri |
| Node APIs in the main process and a mature plugin ecosystem | Electron |
| The smallest possible binary and a CLI-driven build | Valkyrie |

`Tauri.md` calls out **house style vs minimal** near the top. Read that section before you
start, because it decides how opinionated the generated UI will be.

## After you ship

`../prompts/reviews/Tauri-QC.md` is the one framework-specific review prompt in the pack.
Everything else under `../prompts/reviews/` is stack-neutral and works here too.

## Adding a framework

1. Copy the closest existing file.
2. State which language file it assumes in the first paragraph.
3. Keep the **Agent load** blockquote at the top.
4. Cover what the framework changes, not what the language already covers. Duplication
   between layers is how the two drift apart.
5. Add a row to the table above.
