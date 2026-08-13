# Tauri Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Pair with the matching `languages/*` file and project `AGENTS.md`. Reviews: `skills/review/audit/SKILL.md`. Craft: `skills/engineering/craft/SKILL.md`. Prefer extending an existing repo over scaffolding a parallel tree.

Clean structure for cross-platform desktop apps with **Tauri v2**, a React + TypeScript
webview frontend over a Rust backend, talking through typed IPC. Small binaries, native
OS access, web UI. Layered so the Rust core stays pure and testable and the UI stays a
thin view.

---

## House style vs minimal

This document describes a **reference house style** used on larger Tauri apps: Clean Architecture layers on the Rust side, React shell, design tokens, full plugin set.

- **Greenfield small tool** (few commands, no complex domain): use Tauri v2 + typed IPC + CSP + secrets in env only. Skip full `domain/application/infrastructure` until complexity demands it.
- **Greenfield product / complex domain**: the layered tree below is appropriate.
- **Existing repo**: match what is already there. Do not re-scaffold to match this tree unless asked.
- **Versions** in the tables are snapshots, not eternal pins. Prefer the project's lockfile over this doc when they disagree.

---

## Tech Stack

A solid default stack for a larger Tauri app. Swap pieces per project. Treat versions as guidance; the project's lockfiles win.

### Frontend (`package.json`)

| Package | Version | Role |
|---|---|---|
| `react` / `react-dom` | ^19 | UI |
| `@tauri-apps/api` | ^2 | IPC (`invoke`, events, window, app) |
| `@tauri-apps/plugin-opener` | ^2 | open external URLs/files |
| `framer-motion` | ^12 | dialog / toast animation |
| `recharts` | ^3 | charts (add only if you graph data) |
| `lucide-react` | latest | icon set |
| `clsx` | ^2 | conditional classNames |
| `tailwindcss` + `@tailwindcss/vite` | ^4 | styling |
| `vite` | ^7 | dev server / bundler |

### Backend (`src-tauri/Cargo.toml`)

| Crate | Role |
|---|---|
| `tauri` v2 | app runtime, IPC, windowing |
| `tauri-plugin-log` | logging to stdout + log dir |
| `tauri-plugin-opener` | open external targets |
| `tokio` v1 (rt-multi-thread, macros, time, sync) | async tasks |
| `serde` / `serde_json` | (de)serialization across IPC |
| `thiserror` | typed error enum |
| `async-trait` | async traits for ports |
| `reqwest` (rustls, json) | outbound HTTP, **from Rust, never the webview** |

Keep the crate as a `*_lib` library plus a thin `main.rs` binary. Size-optimize the
release profile: `opt-level = "s"`, `lto = true`, `panic = "abort"`, `strip = true`.

---

## Project Structure

```
my-app/
├─ index.html                  app entry (#root)
├─ package.json                scripts + deps
├─ vite.config.ts              dev server :1420, manual vendor chunks
├─ src/                        React webview
│  ├─ main.tsx                 React root (StrictMode)
│  ├─ App.tsx                  shell: ErrorBoundary → Titlebar/Sidebar/Workspace → Toasts
│  ├─ index.css                Tailwind theme tokens, fonts, scrollbars, reduced-motion
│  ├─ types.ts                 shared TS shapes mirroring Rust DTOs
│  ├─ api/                     invoke/listen wrappers (one file per domain)
│  ├─ hooks/                   data + UI hooks
│  ├─ utils/                   pure helpers
│  ├─ pages/                   one component per tab/route
│  └─ components/
│     ├─ shared/               Titlebar, Sidebar, GlassPanel, Button, Toasts, …
│     └─ <feature>/            feature-specific components
└─ src-tauri/                  Rust backend
   ├─ Cargo.toml               crate <name>_lib + deps
   ├─ tauri.conf.json          window, CSP, bundle
   ├─ icons/                   app/installer icons
   └─ src/
      ├─ main.rs               binary → <name>_lib::run()
      ├─ lib.rs                Tauri builder: plugins, setup, invoke_handler
      ├─ state.rs              AppState wiring all adapters
      ├─ dto.rs                IPC view structs (Serialize, camelCase)
      ├─ error.rs              AppError enum
      ├─ domain/               pure logic, no I/O
      ├─ application/          use-cases + ports (traits)
      ├─ infrastructure/       adapters implementing the ports
      └─ commands/             thin #[command] handlers over AppState
```

Organize both sides by **domain**, not by type, once a feature grows. Keep it flat until
that hurts.

---

## Principles

**The webview is a view, the Rust core is the app**
All real work, I/O, system access, heavy compute, lives in Rust. The frontend renders
state and sends intent. Never put business rules in React that Rust should own.

**Hexagonal backend (ports & adapters)**
`application/ports.rs` defines traits; `infrastructure/` implements them; `domain/` stays
pure. Dependencies point **inward** (`../standards/Principles.md` §8.2). Everything crosses
boundaries as `Arc<dyn Trait>`, so use-cases are unit-testable with fakes.

**Thin commands**
`#[command]` functions are glue: parse args, call a use-case on `AppState`, map the error.
No logic in command handlers.

**Typed IPC contract**
Every payload is a `Serialize` DTO in `dto.rs` with a matching TS type in `types.ts`. The
two files are the contract; keep them in lockstep.

**Single source of state**
The backend owns the truth. The frontend subscribes to events and keeps a local view; it
never invents state the backend doesn't know about.

---

## Backend (Rust, `src-tauri/src`)

### Entry & wiring

- **`main.rs`**, `#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]`
  (no console window in release); calls `<name>_lib::run()`.
- **`lib.rs`**, `run()` builds the Tauri app: registers plugins (`opener`, `log` → stdout
  + log dir), constructs `AppState` in `setup`, calls `app.manage(state)`, and registers
  the `invoke_handler` with every command.
- **`state.rs`**, `AppState` holds each use-case behind an `Arc`. Built once at startup;
  injected into commands via `tauri::State`.

### Layers

- **`domain/`**, pure, dependency-free logic and value types. No I/O, no Tauri, no
  `async`. The most heavily unit-tested layer.
- **`application/`**, use-cases that orchestrate the domain through ports. Owns
  long-running tasks and in-memory caches.
- **`application/ports.rs`**, the trait boundary: `trait Store`, `trait HttpClient`,
  `trait EventSink`, etc., plus the request/response value types they exchange.
- **`infrastructure/`**, concrete adapters implementing the ports (a file store, an HTTP
  client via `reqwest`, a Tauri event emitter). The only layer that knows about the
  outside world.
- **`commands/`**, Tauri `#[command]` functions; thin handlers calling `AppState`.

### Ports & adapters example

```rust
// application/ports.rs, the boundary
#[async_trait::async_trait]
pub trait Store: Send + Sync {
    async fn load(&self, key: &str) -> Result<Option<String>, AppError>;
    async fn save(&self, key: &str, value: &str) -> Result<(), AppError>;
}

// infrastructure/json_store.rs, one adapter
pub struct JsonStore { path: PathBuf }

#[async_trait::async_trait]
impl Store for JsonStore {
    async fn save(&self, key: &str, value: &str) -> Result<(), AppError> {
        // write to `${path}.tmp` then atomic rename, never a half-written file
        atomic_write(&self.path, key, value).await
    }
    // ...
}
```

### Async tasks

Long-running work (watchers, pollers, pipelines) runs on `tokio` tasks spawned from a
use-case, communicating over bounded channels and pushing results to the frontend via an
`EventSink` port. Hold a `StopToken` (an `Arc<AtomicBool>`) so the command layer can stop
them cleanly.

---

## IPC

Two directions: the frontend **calls** commands; the backend **emits** events.

### Commands (frontend → backend)

```rust
#[tauri::command]
async fn save_note(
    state: tauri::State<'_, AppState>,
    id: String,
    body: String,
) -> Result<(), String> {
    state.notes.save(&id, &body).await.map_err(|e| e.to_string())
}
```

```typescript
// src/api/notes.ts
import { invoke } from '@tauri-apps/api/core';

export function saveNote(id: string, body: string): Promise<void> {
  return invoke('save_note', { id, body });
}
```

### Events (backend → frontend)

Namespace channels `domain://action`. The backend emits via an `EventSink` adapter; the
frontend subscribes through a typed wrapper.

```typescript
// src/api/events.ts
import { listen } from '@tauri-apps/api/event';
import type { NoteView } from '../types';

export const onNoteSaved = (cb: (note: NoteView) => void) =>
  listen<NoteView>('notes://saved', (e) => cb(e.payload));
```

### Shared shapes

`dto.rs` (`#[derive(Serialize)]`, `#[serde(rename_all = "camelCase")]`) ↔ `types.ts`.
These mirror each other field-for-field; change them together.

---

## Frontend (React, `src/`)

### Shell

```
<ErrorBoundary>
  <div h-screen flex-col>
    <Titlebar/>                       ← frameless drag region + window controls
    <div flex flex-1>
      <Sidebar tab onSelect/>
      <main p-6><Workspace tab/></main>
    </div>
  </div>
  <ToastContainer/>                    ← fixed bottom-right
</ErrorBoundary>
```

`App.tsx` holds the active `tab` and composes the data layer once, passing it down.
`Workspace` switches on a `Tab` string union, no router library for a single-window app.

### State management

No Redux/Context-heavy store. Use **local hooks + module-level external stores** via
`useSyncExternalStore` for cross-cutting state (e.g. toasts). A top-level hook composes
the app's data layer:

```typescript
function useApp() {
  // subscribe to backend events, keep a local view, expose actions
  const notes = useNotes();      // listens to notes://* , exposes save/delete
  return { notes };
}
```

- One hook per backend domain; each owns its events and the `invoke` calls.
- Persist small UI preferences in `localStorage` (namespaced keys like `app.selection`).
- API wrappers in `src/api/` are the only place that calls `invoke` / `listen`.

---

## Design System

A reusable dark, glassmorphic system. Tokens live in `index.css` (`@theme`); the accent is
violet by default, swap `--color-primary` for any brand and the rest follows.

### Window shell (`tauri.conf.json`)

Frameless (`decorations: false`), e.g. `1180×760`, min `960×640`, centered, resizable,
opaque background `#080810`. A custom `Titlebar` provides the drag region (`data-tauri-drag-region`)
and Minimize/Close controls via `getCurrentWindow()`.

### Color tokens (`index.css` `@theme`)

| Token | Value | Use |
|---|---|---|
| `--color-bg` | `#080810` | app background |
| `--color-surface` | `rgba(10,10,20,0.7)` | panels |
| `--color-surface-alt` | `#0e0e1a` | menus, dialogs, tooltips |
| `--color-border` / `-subtle` | white 8% / 5% | borders |
| `--color-primary` / `-bright` | `#7c3aed` / `#8b5cf6` | accent |
| `--color-accent-light` | `#a78bfa` | highlights, group labels |
| `--color-danger/success/warning/info` | red/emerald/amber/blue | status |
| `--shadow-sm/md/lg/glow` |, | elevation + accent glow |

Text tints via white opacity: primary `~95%`, secondary `70%`, muted `45%`.

### Typography

A geometric sans (e.g. **Plus Jakarta Sans**, weights 300-700) as `--font-sans`. Recurring
label micro-style: `text-[10px] font-medium uppercase tracking-wider text-white/40`.

### Surfaces & shape

Glassmorphism: translucent fills (`bg-white/2`), hairline borders (`border-white/5-10`),
`backdrop-blur-xl`. Radii, panels `rounded-2xl`, cards/inputs `rounded-lg`/`rounded-xl`,
pills `rounded-full`. A `GlassPanel` primitive (`rounded-2xl border-white/8 bg-white/2
backdrop-blur-xl`) is the base surface.

### Iconography

`lucide-react`, sized consistently (16-20px), tinted to the text opacity scale. Window and
installer icons live in `src-tauri/icons/`.

### Interaction polish (`index.css`)

- Global `user-select: none` for app feel; re-enable on inputs / `textarea` /
  `contenteditable`.
- `:focus-visible` → accent ring (`0 0 0 2px rgba(139,92,246,0.5)`); suppressed for mouse
  focus.
- Custom thin scrollbars (white 10%, 20% on hover); `img` drag disabled.

### Motion

| Where | Tech | Behavior |
|---|---|---|
| Dialogs | framer-motion | `scale 0.96→1` + `opacity 0→1` over a dimmed backdrop |
| Toasts | framer-motion | slide-up fade + `layout` reflow of the stack |
| Charts | recharts | area draws over ~400ms linear |
| Hover/nav/buttons | Tailwind | `transition-colors duration-150` |

**Reduced motion is non-negotiable.** A global `@media (prefers-reduced-motion: reduce)`
rule forces near-zero durations, and any `requestAnimationFrame` canvas effect must check
`matchMedia('(prefers-reduced-motion: reduce)')` and bail.

---

## Error Handling

Three layers, each translating to the next (`../standards/Principles.md` §5, and `skills/engineering/errors/SKILL.md`
for user copy):

**Rust, one typed error enum**

```rust
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("validation: {0}")] Validation(String),
    #[error("storage: {0}")]    Storage(String),
    #[error("network: {0}")]    Network(String),
}
```

Domain and use-cases return `Result<T, AppError>` with context. Commands map it at the
boundary: `.map_err(|e| e.to_string())`, the frontend receives a string, never a panic. A
panic crashes the webview, so **never `unwrap()` in a command path**.

**Frontend, render vs async errors**

- **Render errors** → an `ErrorBoundary` (class component) shows fallback UI.
- **Async / IPC errors** → boundaries don't catch these. `try/catch` the `invoke`, put the
  message in state, and surface it as a toast or inline, translated for the user, not the
  raw `AppError` string.

```typescript
try {
  await saveNote(id, body);
} catch {
  toast('Couldn’t save your note. Try again in a moment.', 'error');
}
```

---

## Testing

**Rust, unit-test the pure layers**

`domain/` has no I/O, so it tests directly. Use-cases test against **fake adapters**
implementing the ports, no real files, network, or Tauri.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct FakeStore;
    #[async_trait::async_trait]
    impl Store for FakeStore {
        async fn load(&self, _: &str) -> Result<Option<String>, AppError> { Ok(None) }
        async fn save(&self, _: &str, _: &str) -> Result<(), AppError> { Ok(()) }
    }

    #[tokio::test]
    async fn saves_a_valid_note() {
        let notes = Notes::new(Arc::new(FakeStore));
        assert!(notes.save("1", "hello").await.is_ok());
    }
}
```

Run with `cargo test`. Keep tests **fast and self-validating** (`../standards/Principles.md` §9.2).

**Frontend, logic, not chrome**

Test hooks and pure utils with Vitest; **stub the Tauri API** so nothing hits IPC:

```typescript
vi.mock('@tauri-apps/api/core', () => ({ invoke: vi.fn() }));
```

Leave the thin `api/` wrappers and visual chrome to manual checks in `tauri dev`.

---

## Configuration

Config splits across `tauri.conf.json`, the Rust backend, and per-user data, keep them
distinct.

### `tauri.conf.json`

- **Window**, size, min size, `decorations: false`, background color.
- **CSP**, lock it down. `default-src 'self'`; allow `data:`/`asset:` for images, your
  font host for styles, and `connect-src 'self' ipc: http://ipc.localhost` for IPC. The
  webview should talk **only IPC**, make outbound HTTP from Rust (`reqwest`), which isn't
  subject to the webview CSP.
- **Bundle**, set a reverse-DNS `identifier` (e.g. `dev.you.app`) and targets (msi, nsis,
  deb, appimage, dmg).

### Secrets & env

API keys and tokens live in the **Rust** process, read from the environment at run time, **never bundled into the webview**, where any user can read them. Expose only results over
IPC. Commit a `.env.example`; gitignore the real `.env`.

### Per-user data

Persist under `app_data_dir()` (via `tauri::Manager::path`), never next to the binary
(which may be read-only). Write atomically (`.tmp` + rename) and ignore malformed files
with a warning rather than crashing. Small UI prefs can stay in the webview's
`localStorage`.

---

## Summary

- **Rust owns the app; the webview renders it.** No business logic in React.
- **Hexagonal backend**, pure `domain/`, use-cases + ports in `application/`, adapters in
  `infrastructure/`, thin `commands/`. Inject `Arc<dyn Trait>` for testability.
- **Typed IPC**, `dto.rs` ↔ `types.ts`, kept in lockstep; namespaced `domain://` events.
- **One `AppError` enum** mapped to `String` at the command boundary; never `unwrap()` in a
  command.
- **Design system** in `@theme` tokens, dark glassmorphism, swappable accent, mandatory
  reduced-motion support.
- **Lock the CSP**, keep secrets and HTTP in Rust, persist under `app_data_dir()` with
  atomic writes.

---

## Project Prompt

Build a Tauri v2 desktop app: React and TypeScript in the webview, Rust in the backend.
Follow the structure and rules above. Where they disagree with your defaults, this file
wins.

Read `../standards/Principles.md` alongside this file before starting.

**Architecture**
- Hexagonal Rust backend: pure `domain/`, use-cases + ports in `application/`, adapters in
  `infrastructure/`, thin `commands/`
- Dependencies point inward; cross boundaries as `Arc<dyn Trait>`
- Webview is a thin view, all real work in Rust

**IPC**
- Typed DTOs in `dto.rs` mirrored by `types.ts`
- Thin `#[command]` handlers; namespaced `domain://` events
- All `invoke`/`listen` calls isolated in `src/api/`

**Error Handling**
- One `AppError` enum (`thiserror`) → `String` at the command boundary
- Never `unwrap()` on a command path
- `ErrorBoundary` for render errors; state + toasts for async/IPC errors

**Security**
- Strict CSP; webview talks only IPC
- Outbound HTTP and secrets stay in Rust, never the webview
- Per-user data under `app_data_dir()`, written atomically

### Setup

```bash
npm create tauri-app@latest          # React + TypeScript template
cd my-app
npm install
npm run tauri dev                    # Vite (:1420) + Rust app, hot-reload
```

### Deliverables

1. Complete project following the structure above
2. Hexagonal Rust backend (domain / application+ports / infrastructure / commands)
3. `AppState` wiring adapters at startup
4. Typed IPC: `dto.rs` ↔ `types.ts`, namespaced events
5. React shell (Titlebar, Sidebar, Workspace) with hook-based state
6. Design-system tokens in `index.css` + a `GlassPanel` primitive
7. `AppError` enum + `ErrorBoundary` + toast feedback
8. Locked-down `tauri.conf.json` (window, CSP, bundle)
9. README with setup and build instructions
10. Rust unit tests for domain + use-cases; frontend hook tests

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] No business logic in the webview or in command handlers
- [ ] No `unwrap()`/`expect()` on a command path
- [ ] Every IPC payload is a typed DTO mirrored in `types.ts`
- [ ] CSP locked down; no secrets in the webview
- [ ] Per-user data under `app_data_dir()`, written atomically
- [ ] `prefers-reduced-motion` respected
- [ ] Domain + use-cases unit-tested with fake adapters
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] `cargo clippy` and `tsc` clean

### Pre-Delivery

```bash
cargo fmt --check && cargo clippy -- -D warnings   # Rust clean
cargo test                                         # backend tests pass
npm run build                                      # tsc + vite build, no errors
npm test                                           # frontend tests pass
npm run tauri build                                # production bundles
```
