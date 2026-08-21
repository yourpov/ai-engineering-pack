# C++ Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Principles, structure, and conventions for C++17 / MSVC / Windows projects.

---

## Project Structure

```
project/
 include/            # All headers  one per module
    config.h        # Shared state namespace (cfg::), enums, constexpr tables
    ...             # One header per class or namespace module
 src/
    main.cpp        # Entry point, orchestration, message routing
    core/           # Primary pipeline / domain logic
    input/          # User input handling
    net/            # Network, APIs, IPC
    ui/             # Console / visual output
 web/                # Frontend (if applicable)
    index.html
    css/
    js/
        app.js      # Main UI logic
        modules/    # ES module per concern
 tools/              # Standalone utilities / scripts
 build/              # Build artifacts  gitignored
 build.bat           # Single-file MSVC build script
 app.manifest        # UAC manifest (if admin required)
```

Organize `src/` by **domain**, not by type. Group related files into folders like `core/`, `input/`, `net/`, `ui/`.

---

## Principles

### One class, one job

Each class handles exactly one concern. Classes never reach into each other  `main.cpp` wires them together.

```cpp
class Capture  { /* grabs frames    */ };
class Analyzer { /* checks data     */ };
class Sender   { /* dispatches I/O  */ };
```

### Namespaces for stateful subsystems

Subsystems that own threads, sockets, or persistent connections expose a namespace with `start()` / `stop()` / query functions. All internal state is file-static in the `.cpp`. The caller never sees internals.

```cpp
// net.h  entire public API
namespace net {
    void start(int port);
    void broadcast(const std::string& msg);
    void stop();
    void setHandler(std::function<void(const std::string&)> handler);
}
```

### Classes for data-bound logic

When behavior is tied to an owned resource (a handle, a buffer, an engine), use a class with RAII cleanup. Constructed where needed, destroyed automatically.

```cpp
class FrameBuffer {
    HANDLE handle;
    int width, height;
public:
    FrameBuffer(int w, int h);
    ~FrameBuffer();  // releases handle
    bool capture(BYTE* out);
};
```

### Shared config via `inline` globals

All tunable settings live in a `cfg::` namespace with `inline` variables. Any translation unit can read them. Only `main.cpp` writes them (via message handlers or user input). Persistence is separate.

```cpp
// config.h
namespace cfg {
    inline int width  = 40;
    inline int height = 40;
    inline int tolerance = 10;
    inline DWORD activationKey = VK_XBUTTON2;
    inline bool safeMode = true;
}
```

### `constexpr` for compile-time data tables

Lookup tables, thresholds, and static data are `constexpr`  zero runtime allocation.

```cpp
constexpr Timing timings[MODE_COUNT] = {
    { 85, 110},  { 75, 100},  {170, 210},
};

constexpr Color palette[] = { {254,69,69}, {255,100,95}, {230,50,50} };
```

### Header-only for small utilities

Tiny parsers, config I/O, data types, and simple utilities are header-only with `inline` functions. No `.cpp` needed.

```cpp
// random.h  entire file
class RandomGenerator {
    std::mt19937 engine{std::random_device{}()};
public:
    int range(int lo, int hi) {
        return std::uniform_int_distribution<int>(lo, hi)(engine);
    }
};
```

---

## File Organization

### Headers (`include/`)

One header per module. Every header uses `#pragma once`. Headers declare the public interface only  no implementation unless `inline` or `constexpr`.

Include order: **own header  project headers  system headers**.

```cpp
// analyzer.h
#pragma once
#include <windows.h>
#include "config.h"

class Analyzer {
    const Color* palette;
    size_t count;
    int threshold;
public:
    Analyzer(const Color* colors, size_t count, int tolerance);
    bool check(const BYTE* data, int width, int height);
};
```

### Source (`src/`)

Organized by domain:

| Folder     | Purpose              |
|------------|----------------------|
| `core/`    | Primary pipeline     |
| `input/`   | User input handling  |
| `net/`     | Network & APIs       |
| `ui/`      | Console / display    |
| (root)     | Orchestration        |

Implementation files include their own header first.

```cpp
// analyzer.cpp
#include "analyzer.h"
#include "config.h"
```

### Static helpers stay in the `.cpp`

Functions only used within one file are `static`  never exposed in headers.

```cpp
static const char* statusLabel(int code) { ... }
static std::string formatMessage(const std::string& m) { ... }
static void broadcastState() { ... }
```

---

## Module Patterns

### Namespace modules (subsystems)

For subsystems that own threads or global resources. Internal state uses file-static variables.

```cpp
// api.h  public API
namespace api {
    void start(std::function<void()> onUpdate);
    void stop();
    bool connected();
    std::string status();
    void refresh();
}

// api.cpp  all state is file-static
static std::mutex apiMutex;
static std::thread pollThread;
static std::atomic<bool> running{false};
static std::string currentStatus;
```

### Class modules (owned resources)

For logic bound to a resource lifetime. Constructed on the stack or via `unique_ptr`.

```cpp
auto buffer = std::make_unique<FrameBuffer>(width, height);
auto pixels = std::make_unique<BYTE[]>(width * height * 4);
RandomGenerator random;
```

### Config module (shared state)

One `config.h` for all tunable values. Read from anywhere. Written only in `main.cpp`, always followed by persist + broadcast.

```cpp
// Any module can read
if (cfg::safeMode) Sleep(random.range(5, 20));

// Only main.cpp writes
cfg::width = value; saveCfg(); broadcastState();
```

---

## Hot-Reload Pattern

Long-running loops should detect config changes and rebuild their objects mid-loop without restarting.

```cpp
static void loop() {
    int activeWidth = cfg::width;
    auto buffer = std::make_unique<FrameBuffer>(activeWidth, activeHeight);

    while (running) {
        // Hot-reload on config change
        if (cfg::width != activeWidth) {
            activeWidth = cfg::width;
            buffer = std::make_unique<FrameBuffer>(activeWidth, activeHeight);
        }
        // ... work ...
        Sleep(cfg::tick);
    }
}
```

---

## Message Routing

`main.cpp` is the single message router. Incoming messages (e.g. from a socket or IPC) are plain strings parsed with prefix matching. Handlers are grouped by domain and chained via early return.

```cpp
static void handleCmd(const std::string& msg) {
    if (msg == "state") { broadcastState(); return; }
    if (msg == "keys")  { broadcastKeys();  return; }
    if (handleCore(msg))    return;
    if (handleApi(msg))     return;
    handleExternal(msg);
}
```

Each handler follows: **parse  validate  mutate `cfg::`  persist  broadcast**.

```cpp
static bool handleCore(const std::string& msg) {
    if (msg == "start") {
        if (!looping) { looping = true; std::thread(loop).detach(); }
        return true;
    }
    if (msg == "stop") { looping = false; return true; }

    if (msg.rfind("mode:", 0) == 0) {
        int idx = atoi(msg.c_str() + 5);
        if (idx >= 0 && idx < MODE_COUNT) {
            cfg::mode = (Mode)idx;
            saveCfg(); broadcastState();
        }
        return true;
    }
    return false;
}
```

Message format conventions:
- **Commands**: `"start"`, `"stop"`, `"refresh"`, `"toggle_safe"`
- **Key:value**: `"mode:3"`, `"width:60"`, `"tolerance:15"`
- **Domain-prefixed**: `"api_refresh"`, `"ext_enable:true"`
- **State broadcast**: Single JSON blob after every mutation

---

## Naming

### Full words, no abbreviations

```cpp
// No
int tol; DWORD vk; bool lk; std::mutex mu; int ht;

// Yes
int tolerance; DWORD virtualKey; bool locked; std::mutex dataMutex; int height;
```

Exceptions: universally known abbreviations (`rpc`, `dc`, `dx`, `dy`, `rgb`, `lo`/`hi` for ranges, `idx` for loop index).

### Classes  PascalCase nouns

```cpp
FrameBuffer, Analyzer, Sender, Toggle, RandomGenerator
```

### Namespaces  lowercase

```cpp
cfg::, net::, api::, ui::, keys::, json::
```

### Functions  camelCase verbs

```cpp
void broadcastState();
bool startProcess();
void saveConfig();
std::string formatJson();
```

### Variables  camelCase nouns

```cpp
int currentWidth;
ULONGLONG startTick;
std::string connectionState;
HANDLE childProcess;
```

### Constants  camelCase or UPPER_CASE

```cpp
constexpr int MODE_COUNT = (int)Mode::COUNT;
constexpr int defaultTick = 1;
```

### Enums  PascalCase type, UPPER_CASE values

```cpp
enum class Mode { STANDARD, ADVANCED, CUSTOM, COUNT };
```

### Structs  PascalCase nouns, descriptive members

```cpp
struct Color    { BYTE r, g, b; };
struct Timing   { int lo, hi; };
struct KeyEntry { DWORD virtualKey; const char* label; };
struct Config   { bool enabled; std::string name; int threshold; };
```

---

## Memory & Resource Management

### RAII for system handles

Handles, DCs, sockets, pipes  acquired in constructor, released in destructor.

```cpp
FrameBuffer::FrameBuffer(int w, int h) {
    handle = CreateResource(w, h);
}

FrameBuffer::~FrameBuffer() {
    if (handle) ReleaseResource(handle);
}
```

### `std::unique_ptr` for heap objects

```cpp
auto buffer = std::make_unique<FrameBuffer>(width, height);
auto data   = std::make_unique<BYTE[]>(width * height * 4);
```

### No raw `new`/`delete`

All dynamic allocation uses `make_unique` or containers (`std::vector`, `std::string`).

---

## Error Handling

### Return `bool` for can-fail operations

```cpp
bool FrameBuffer::capture(BYTE* out);
bool Sender::dispatch();
bool startProcess();
```

### Fail silently in non-critical paths

Network failures, file I/O, and optional features just return or skip.

```cpp
inline void saveConfig() {
    std::ofstream file("config.json");
    if (file) file << buffer;  // if it fails, nothing breaks
}
```

### Guard against invalid state

```cpp
if (!handle || !IsValid(handle)) return false;
if (name.empty()) return false;
if (idx < 0 || idx >= MODE_COUNT) return;
```

### No exceptions

Use `/EHsc` for system compatibility but never throw. Error handling is all early-return.

---

## Threading

### `std::thread` + `std::atomic` for lifecycle

```cpp
static std::atomic<bool> running{true};   // process lifetime
static std::atomic<bool> looping{false};  // work loop on/off

// Start
looping = true;
std::thread(loop).detach();

// Stop
looping = false;
```

### `std::mutex` + `std::lock_guard` for shared data

```cpp
static std::mutex dataMutex;

static bool startProcess() {
    std::lock_guard<std::mutex> lock(dataMutex);
    if (isRunning()) return true;
    // ...
}
```

### Detached threads for fire-and-forget

Server accept loops, poll threads, work loops, and one-off async calls all run as detached threads.

```cpp
std::thread([id]() {
    net::broadcast("{\"result\":" + api::fetchData(id) + "}");
}).detach();
```

---

## JSON

### No third-party library

Minimal hand-rolled parser:

```cpp
namespace json {
    int getInt(const std::string& raw, const char* key, int fallback = 0);
    bool getBool(const std::string& raw, const char* key, bool fallback = false);
    std::string getString(const std::string& raw, const char* key);
    double getDouble(const std::string& raw, const char* key, double fallback = 0);
    std::string escape(const std::string& source);
}
```

### Output JSON via `snprintf`

No builder, no extra allocations  just a stack buffer.

```cpp
char buffer[1024];
snprintf(buffer, sizeof(buffer),
    "{\"key\": %lu, \"enabled\": %s, \"mode\": %d}",
    cfg::activationKey, cfg::safeMode ? "true" : "false", (int)cfg::mode);
```

### Composite state via string concatenation

Multiple JSON fragments are composed by inserting into a base JSON object.

```cpp
static void broadcastState() {
    std::string state = core::stateJson();
    std::string extra = api::stateJson();
    state.insert(state.size() - 1, "," + extra);
    net::broadcast(state);
}
```

---

## Persistence

Config files are managed by save/load pairs. Each file is independent.

Pattern: `snprintf`  `std::ofstream`. Load via `std::ifstream`  `json::getInt` / `json::getBool` / etc. with validation + fallbacks.

```cpp
static void saveSettings() {
    char buffer[512];
    snprintf(buffer, sizeof(buffer),
        "{\n  \"enabled\": %s,\n  \"name\": \"%s\",\n"
        "  \"threshold\": %.2f,\n  \"interval\": %d\n}",
        settings.enabled ? "true" : "false", settings.name.c_str(),
        settings.threshold, settings.interval);
    std::ofstream file("settings.json");
    if (file) file << buffer;
}

static void loadSettings() {
    std::ifstream file("settings.json");
    if (!file) return;
    std::string raw((std::istreambuf_iterator<char>(file)), {});
    settings.enabled   = json::getBool(raw, "enabled");
    settings.name      = json::getString(raw, "name");
    settings.threshold = json::getDouble(raw, "threshold", 0.55);
    if (settings.threshold <= 0) settings.threshold = 0.55;  // fallback
}
```

Always validate loaded values and provide sensible fallbacks.

---

## Frontend (Web)

### Architecture

Single-page app served by the C++ backend over WebSocket. ES modules split by concern:

| File              | Purpose                                        |
|-------------------|-------------------------------------------------|
| `sock.js`         | WebSocket client with auto-reconnect            |
| `state.js`        | Reactive state store  `update()`, `get()`, `watch()` |
| `app.js`          | All page setup, DOM creation, paint loop        |

### State flow

```
C++ backend                    JS frontend

net::broadcast(json)    sock.js onmessage
                             state.update(data)
                             watchers fire
                             paint(state) updates DOM

user clicks button      sock.send("mode:3")
                               handleCmd("mode:3")
                             cfg::mode = 3
                             saveCfg(); broadcastState()
                               ... cycle repeats
```

### Naming (JS)

Same principles as C++: full words, no abbreviations.

```js
// No
const btn = ...; let ts = ...; let lk = ...;

// Yes
const button = ...; let timestamp = ...; let keyList = ...;
```

Functions: camelCase verbs (`setupNav`, `renderCard`, `showProfile`, `formatDuration`).
Variables: camelCase nouns (`colorProfile`, `notificationId`, `connectionStatus`).
Constants/module-level: camelCase or UPPER_CASE (`maxRetries`, `categories`).

---

## Console Apps

Applies only to tools that live entirely in a terminal (loaders, scanners, CLIs). Skip this
section for windowed apps. Two layers on top of everything above: tagged logging and shared
UX helpers.

### Tag-based logging

All output goes through log helpers, never bare `printf`. Every line gets an aligned tag,
a color, and a plain-English message.

```cpp
// log.h  entire public API
void logLine(const char* tag, const char* msg);   // generic tagged line
void logInfo(const char* msg);                    // [INFO]
void logAction(const char* tag, const char* msg); // [OK] and friends
void logDebug(const char* msg);                   // [DBG], gated on cfg::debugMode
```

- Tags are SCREAMING single words in brackets: `INFO`, `OK`, `ERROR`, `WARN`, `DBG`
- Colors come from macros in `theme.h` (`WHITE`, `RESET`, ...) wrapping ANSI escapes
- `logDebug` checks `cfg::debugMode` itself so call sites stay one-liners
- Messages are written for the end user, not the developer: "That's not a .dll file.
  Drop a DLL onto the loader and try again." beats "invalid extension"

```cpp
logInfo("Checking aim.dll");
logAction("OK", "Injected");
logLine("ERROR", "No DLL was found. Drop a .dll onto the loader and try again.");
logDebug(("dllPath: " + dllPath).c_str());
```

### Shared UX helpers

Small free functions every interactive console tool reuses instead of reimplementing:

```cpp
void pressEnter(const char* prompt);   // "\n" + prompt + drain stdin
bool fileExists(const char* path);
std::string dllPathFromCurrentDirectory(const std::string& name);
```

Patterns:

- **Every error path ends the same way**: `logLine("ERROR", ...)` then
  `pressEnter("Press Enter to go back...")`, then early return
- **Numbered pickers** for choosing between N items: print `[1] name`, read a line,
  `atoi`, bounds-check before use
- **Inline prompts**, no newline: `printf(WHITE "Select a DLL: " RESET); fflush(stdout);`
- **Read input with `std::getline`**, never `>>` which leaves the newline behind
- **Banner first**: `banner::show()` at startup, blank line before the first log line

```cpp
for (size_t i = 0; i < dlls.size(); i++)
    printf(WHITE "[%d]" RESET " %s\n", (int)(i + 1), dlls[i].c_str());
printf(WHITE "Select a DLL: " RESET);
fflush(stdout);
std::string line;
std::getline(std::cin, line);
int index = atoi(line.c_str());
if (index < 1 || index > (int)dlls.size()) { /* error path */ }
```

---

## Build

Single `build.bat`. No CMake, no Makefile. MSVC `cl.exe` with all sources listed explicitly.

```bat
cl /std:c++17 /O2 /EHsc /utf-8 /I"include" ^
    src\main.cpp src\ui\display.cpp ^
    src\core\capture.cpp src\core\analyzer.cpp ^
    src\input\toggle.cpp ^
    src\net\server.cpp src\net\api.cpp ^
    /Fo"build\\" /Fe:"build\app.exe" ^
    /link user32.lib gdi32.lib ws2_32.lib advapi32.lib winhttp.lib ^
    /MANIFEST:EMBED /MANIFESTINPUT:app.manifest /MANIFESTUAC:NO
```

| Flag          | Purpose                                                    |
|---------------|------------------------------------------------------------|
| `/std:c++17`  | Structured bindings, `inline` variables, `if constexpr`    |
| `/O2`         | Full optimization                                          |
| `/EHsc`       | C++ exceptions enabled (for system compat, never used)     |
| `/utf-8`      | Source and execution charset                               |

Link only what you use. Common Win32 libraries:

| Library         | Purpose            |
|-----------------|--------------------|
| `user32.lib`    | Input, windows     |
| `gdi32.lib`     | Screen capture     |
| `ws2_32.lib`    | Winsock            |
| `advapi32.lib`  | Crypto, registry   |
| `winhttp.lib`   | HTTP requests      |

---

## Testing

No test framework. A header-only `check()` macro and a separate `tools/test.cpp` keep
tests dependency-free, same as the rest of the project.

```cpp
// tools/check.h  entire harness
#pragma once
#include <cstdio>

inline int testFailures = 0;

#define check(cond) \
    do { if (!(cond)) { \
        printf("FAIL %s:%d  %s\n", __FILE__, __LINE__, #cond); \
        ++testFailures; \
    } } while (0)
```

Test pure logic, parsers, validation, `json::` helpers, range math. Anything that
owns a socket, thread, or Win32 handle gets wrapped behind a namespace so the test
links the parsing free of the I/O.

```cpp
// tools/test.cpp
#include "check.h"
#include "../include/json.h"

int main() {
    check(json::getInt("{\"width\": 40}", "width") == 40);
    check(json::getInt("{}", "width", 99) == 99);          // fallback
    check(json::getBool("{\"safe\": true}", "safe") == true);

    printf(testFailures ? "%d failed\n" : "all passed\n", testFailures);
    return testFailures ? 1 : 0;
}
```

Compile and run as its own executable; non-zero exit fails the build.

```bat
cl /std:c++17 /EHsc /utf-8 /I"include" tools\test.cpp /Fe:"build\test.exe"
build\test.exe
```

Keep tests **fast and self-validating** (see `../standards/Principles.md` §9.2). Validate behavior
(inputs, outputs, fallbacks), not the shape of the parser internals.

---

## Configuration

Configuration lives in three tiers, no `.env`, no config library, just code + JSON.

| Tier | Where | Mutable? | Use for |
|------|-------|----------|---------|
| **Compile-time** | `constexpr` tables in headers | No | Lookup tables, timings, palettes, limits |
| **Runtime** | `cfg::` `inline` globals in `config.h` | Yes (only `main.cpp` writes) | Tunable settings the user changes while running |
| **Persisted** | JSON files via `saveCfg` / `loadCfg` | Yes | Settings that survive a restart |

The flow on every change is fixed: **mutate `cfg::`  persist to JSON  broadcast new
state** (see [Message Routing](#message-routing) and [Persistence](#persistence)).

```cpp
// config.h  the single source of runtime config
namespace cfg {
    inline int  width        = 40;
    inline int  tolerance    = 10;
    inline DWORD activationKey = VK_XBUTTON2;
    inline bool safeMode     = true;
}
```

Rules:

- **Never hard-code a tunable value** at its use site, add it to `cfg::` or a
  `constexpr` table.
- **Only `main.cpp` writes `cfg::`.** Every other module reads. This keeps the write
  path (validate  persist  broadcast) in one place.
- **Always validate persisted values on load** and fall back to a safe default, a
  corrupt or hand-edited JSON file must never leave the app in a bad state.
- **Secrets** (API keys, tokens) never go in committed headers. Load them from a local
  file that's gitignored, or the Windows registry / credential store.

---

## Summary

- **One header, one module.** Public interface in `include/`, implementation in `src/`.
- **Namespaces for subsystems**, classes for owned resources.
- **Shared config** in `cfg::` namespace with `inline` globals. Written only in `main.cpp`.
- **RAII everywhere.** No raw `new`/`delete`. `unique_ptr` for heap. Destructors release handles.
- **Full words.** No abbreviations unless universally known.
- **Early return.** Validate inputs, fail fast, never throw.
- **Message router in `main.cpp`.** Parse  validate  mutate  persist  broadcast.
- **No dependencies.** Win32 APIs only. Hand-rolled JSON, sockets, HTTP.
- **Persistence** via `snprintf` + `ofstream` / `ifstream` + `json::` parsing with fallbacks.
- **Hot-reload.** Long-running loops rebuild objects when config changes mid-run.
- **Threading.** `std::atomic` for lifecycle, `std::mutex` for shared data, detached threads.
- **Frontend** is reactive: `state.watch()` triggers `paint()` on every backend message.
- **Console apps** add tag-based logging (`logLine` / `logAction`) and shared UX helpers
  (`pressEnter`, numbered pickers) on top of the base rules.

---

## Project Prompt

Write C++ against the structure and rules above. Where they disagree with your defaults,
this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Architecture**
- Follow the exact structure defined above
- Headers for interfaces, source files for implementation
- Namespaces for subsystems, classes for owned resources
- Shared config via `cfg::` namespace

**Memory Management**
- RAII everywhere, no raw `new`/`delete`
- `std::unique_ptr` for heap objects
- Clear ownership semantics
- NULL checks after all allocations

**Error Handling**
- Return `bool` for can-fail operations
- Guard against invalid state with early returns
- No exceptions, all error handling via early-return
- Validate loaded values with sensible fallbacks

**Build**
- Single `build.bat` with MSVC `cl.exe`
- No CMake, no Makefile
- Debug and release configurations

### Setup

```bat
:: From a "Developer Command Prompt for VS" (so cl.exe is on PATH)
mkdir include src src\core src\input src\net src\ui tools build
:: create build.bat (see Build section), config.h, and main.cpp
build.bat
```

No package manager and no third-party dependencies, the project compiles against the
Win32 SDK only.

### Deliverables

1. Complete project following architecture structure above
2. Header files with clear interfaces (`#pragma once`)
3. Implementation files with encapsulation
4. Memory-safe operations throughout
5. Error handling on all functions
6. build.bat for building
7. README with build instructions

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] RAII for all resources (no raw new/delete)
- [ ] All pointers checked before dereferencing
- [ ] `#pragma once` in all headers
- [ ] No memory leaks
- [ ] All errors handled
- [ ] No globals (except `cfg::` namespace)
- [ ] Natural, human variable names
- [ ] Clean compilation (no warnings)

### Pre-Delivery

```bat
build.bat                          :: compiles clean, no warnings
cl /std:c++17 /EHsc /utf-8 /I"include" tools\test.cpp /Fe:"build\test.exe"
build\test.exe                     :: all tests pass, exit 0
build\app.exe                      :: smoke-runs and exits cleanly
```
