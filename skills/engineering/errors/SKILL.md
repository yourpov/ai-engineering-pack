---
name: errors
description: >
  Use this skill for anything about failure, both how code should fail and how
  failures should read to a human. Part A covers Robert C. Martin's ("Uncle Bob")
  error-handling principles from Clean Code: exceptions over return codes,
  try-catch-finally first, providing context, defining exceptions by the caller's
  needs, the Special Case pattern, and never returning or passing null. Part B
  covers user-facing error messages: the three laws (what / why / action),
  severity levels, copy rules, and accessibility. Trigger on "handle this error",
  "this exception is useless", "should I return null", "improve this error
  message", "write an error state", or any review of failure code or copy.
  Language- and framework-agnostic.
---

# Errors, Handling in Code, Messages to Users

Failure has two audiences. **The next programmer** needs code that fails loudly,
with context, in one obvious place. That is Uncle Bob's domain (Part A). **The
person using the product** needs to know what happened, why, and what to do.
That is the three laws (Part B). A complete error story does both: catch cleanly
in code, then translate that into something a human can act on. Both parts are
language- and framework-agnostic.

## Authority (read first)

**Part A craft rules are defined by** `standards/Clean-Code/`:

| Topic | Canonical lesson |
|---|---|
| Exceptions over error codes | `03-exceptions-over-error-codes.md` |
| Contract / try first | `12-define-contract-before-logic.md` |
| No null | `13-stop-returning-null.md` |
| Catch is not if; no swallow | `34-catch-is-not-if.md` |
| Happy path breathes | `37-algorithm-breathes.md` |
| Wrap third parties | `31-wrap-third-party-apis.md` |

If this skill and those lessons disagree, **the Clean-Code lesson wins**. This file
adds examples and **Part B (user-facing copy)**. Do not invent a second error policy.

**Examples** use Java-like syntax as illustration only. Translate to the repo’s
language. Prefer exceptions wherever the language has them (Clean Code default).

---

## Part A, Error handling in code (Clean Code, ch. 7)

Robert C. Martin's rule of thumb: **error handling is a separate concern.** You
should be able to read the main logic without it being shredded by failure
checks, and you should be able to read the failure handling on its own. The
principles below all serve that goal, clean *and* robust, not one at the cost of
the other.

### A1. Use exceptions, not return codes

Canonical: Clean-Code **03**.

Return codes force the caller to check immediately and clutter the happy path.
Exceptions separate the algorithm from its error handling.

```java
// ✗ return codes, error checks drown the logic
if (deletePage(page) == E_OK) {
  if (registry.deleteRef(page.name) == E_OK) {
  if (configKeys.delete(page.key) == E_OK) { logger.log("page deleted"); }
  else { logger.log("delete failed"); return E_ERROR; }
 } …
}

// ✓ exceptions, the logic and the handling are each readable alone
try {
  deletePage(page);
  registry.deleteRef(page.name);
  configKeys.delete(page.key);
} catch (Exception e) {
  logger.log(e.getMessage());
}
```

### A2. Write your try-catch-finally first

Canonical: Clean-Code **12** (boundary owns the contract; algorithm inside stays flat — **37**).

When code can throw, **write the `try-catch-finally` before the body** at the function
that owns the public failure type. It defines the transaction boundary. Write a test
that forces the exception first when practical. Catch translates/rethrows or reports —
it does not implement normal business branches (**34**).

### A3. Prefer unchecked exceptions

Checked exceptions violate the Open/Closed Principle: a low-level method adding a
`throws` clause forces a signature change, and a recompile, on every method
above it in the call stack, breaking encapsulation. Reserve checked exceptions
for critical libraries where the caller genuinely must handle them; default to
unchecked. (Languages without checked exceptions get this for free.)

### A4. Provide context with every exception

An exception's job is to tell you **what operation failed and why.** A bare
`throw new RuntimeException()` is useless at 3 a.m. Include the operation, the
relevant identifiers/inputs, and the kind of failure, and pass enough to your
logs.

```java
// ✗
throw new IllegalStateException();

// ✓, names the operation, the subject, and the cause
throw new StorageException(
  "failed to persist order " + orderId + ": connection pool exhausted", cause);
```

A good exception message is itself a *what + why* (the same idea as Part B's three
laws, aimed at a developer instead of an end user).

### A5. Define exception classes by the caller's needs

Classify exceptions by **how they'll be caught**, not by their source or type.
The most common need is "something failed in this operation", so wrap noisy
third-party APIs behind one class and handle it in one place. This shrinks
dependencies on the external library and makes the calling code clean.

```java
// ✗ caller must know every exception the device API can throw
try { port.open(); }
catch (DeviceResponseException e) { reportError(e); }
catch (ATM1212UnlockedException e) { reportError(e); }
catch (GMXError e) { reportError(e); }

// ✓ wrap the API; the caller handles one meaningful type
public class LocalPort { // wrapper
  public void open() {
  try { innerPort.open(); }
  catch (DeviceResponseException | ATM1212UnlockedException | GMXError e) {
  throw new PortDeviceFailure(e);
 }
 }
}
```

### A6. Define the normal flow, the Special Case pattern

Canonical: Clean-Code **34** + **13**.

Don't make the *caller* handle ordinary alternate cases with try/catch. Instead of
throwing and catching to select a normal path, return a Special Case / default object
that does the right thing.

```java
// ✗ business logic interrupted by an exception for an ordinary case
try { total += getMeals(id).getTotal(); }
catch (MealExpensesNotFound e) { total += getMealPerDiem(); }

// ✓ getMeals always returns a MealExpenses; the special case handles itself
total += getMeals(id).getTotal(); // PerDiemMealExpenses.getTotal() returns the per-diem
```

### A7. Don't return null

Canonical: Clean-Code **13**.

Returning null invites missed checks and `NullPointerException`s, and litters
callers with `if (x != null)`. Return an **empty collection**, a **Special Case
object**, or **throw** — never null. Never hide a failed call as empty success.

```java
// ✗
List<Employee> employees = getEmployees();
if (employees != null) { for (Employee e : employees) total += e.pay(); }

// ✓ getEmployees() returns Collections.emptyList() when there are none
for (Employee e : getEmployees()) total += e.pay();
```

### A8. Don't pass null

Passing null into a method is worse than returning it, the bug surfaces deep
inside the callee. Avoid it by policy; when a null *would* arrive, fail fast at
the boundary with a clear message rather than letting it propagate.

```java
public double xProjection(Point a, Point b) {
  if (a == null || b == null)
  throw new InvalidArgumentException("xProjection received a null Point");
  return (b.x - a.x) * 1.5;
}
```

### A9. Mapping to languages without exceptions

Canonical policy is still Clean-Code **03** (exceptions first). When the language
has no exceptions:

- **Go** — use `error` with context (`fmt.Errorf("persist order %d: %w", id, err)`). Keep the happy path free of nested status pyramids. Never return a half-built value with `err == nil`. "Don't return null" → zero value + non-nil error on failure.
- **Rust / Swift / FP** — `Result`/`Either` for failure; `Option`/`Maybe` or Special Case for absence — not raw null. Same readability bar as exceptions: algorithm must breathe (**37**).
- **TypeScript / Java / C# / etc.** — use **exceptions**, not hand-rolled `{ ok, error }` bags.
- **Never fine anywhere:** silent `null`, empty catch, failure returned as successful empty, generic messages without context.

### Part A checklist

- [ ] Aligns with Clean-Code 03 / 12 / 13 / 31 / 34 / 37?
- [ ] Failure uses exceptions (or the language’s single idiomatic error channel) — not status bags in exception languages?
- [ ] Is try/catch only at the contract boundary, not as an if for normal paths?
- [ ] Does every failure carry **context** (operation, identifiers, cause)?
- [ ] Are third-party APIs wrapped so callers catch *your* type?
- [ ] Ordinary alternate cases use Special Case / defaults — not throw+catch?
- [ ] Nothing **returns null**; nothing **passes null** (reject at boundary)?
- [ ] No empty catch; no failure disguised as empty success?
- [ ] Can the main logic be read without the error handling, and vice versa?

---

## Part B, Error messages to users (the three laws)

Once the code has caught the failure cleanly, a human still has to read about it.
**Never show them the exception.** Translate it.

### B1. The three laws

A good user-facing error does exactly three things, in order:

1. **What happened**, the fact, plainly, no jargon.
2. **Why it happened**, a cause the user can understand.
3. **Clear action**, the single next step.

- "Something went wrong." → **0 / 3**
- "Something went wrong. Please try again." → **1 / 3** (action only)
- "Your photo couldn't upload because it's 48 MB (the limit is 25 MB). Compress it or choose a smaller file." → **3 / 3**

### B2. The template

```text
WHAT: "[Subject] couldn't [verb]."
WHY: "[The reason, in terms the user can understand or control]."
ACTION: "[Do X] to [outcome]."
```

The **subject is always the user's thing**, *their* photo, *their* message, never
"the system." "The request could not be completed" hides the subject; "Your
message didn't send" names it.

### B3. Wrong vs. right

| ✗ Wrong | ✓ Right |
| --- | --- |
| `Error 403` | "You don't have permission to view this file. Ask the owner to share it with your account, then try again." |
| `Upload failed.` | "Your photo couldn't upload because it's 48 MB, the limit is 25 MB. Compress it or choose a smaller file." |
| `Network error.` | "Your message couldn't send because the connection dropped. Reconnect, then tap Retry." |
| `Invalid input.` | "Your password is too short. Passwords need at least 8 characters. Add more, then try again." |
| `NullPointerException` | "Couldn't load your profile because the data was incomplete. Refresh the page, if it persists, contact support with code PROFILE_NULL." |

### B4. Copy rules

- **What**, plain past tense: "Sign-in failed," not "Authentication failure."
- **Why**, actionable: "The server is temporarily unavailable," not "Server returned 503." If the cause is unknown, say so honestly.
- **Action**, one imperative phrase: "Try again in a few minutes." If two are needed, one primary button + one escape-hatch link.
- **Avoid:** "Oops"/"Whoops" (trivializing), "Sorry" as a substitute for an action, "Unexpected error" (filler), error codes in the main sentence (append `code: …` for support), exclamation marks, and passive voice.

### B5. Severity levels, pick the least intrusive surface that works

- **Inline validation**, next to the field, on blur; never blocks. `role="alert"`, `aria-live="polite"`, `aria-invalid` on the input.
- **Toast / banner**, transient, auto-dismiss; for recoverable errors. Offer the action (Retry) before it disappears. `role="alert"`.
- **Alert / confirm dialog**, blocks because a decision is required. Two buttons max; safe action is the default. `role="alertdialog"` + `aria-modal="true"`; move focus in.
- **Full-screen / empty state**, a whole view failed. Icon + what + why + one action, plus an escape hatch. **Never a dead end.** `role="status"`.
- **Non-UI**, CLI: one line of what/why/fix, exit non-zero. API: stable `code` + human `message` + `resolution`, no stack traces leaked. Logs/exceptions: see Part A, context is the what/why for the next developer.

### B6. Accessibility, non-negotiable

Errors must be perceivable **without color.** Pair the danger color with an icon
*and* text, wire up the ARIA role + live region for the level, and move focus to
the error (or the offending control). Red borders alone fail.

### B7. Visual treatment, adapt, don't prescribe

This skill doesn't own your aesthetic. Use *your* design system's danger color,
type scale, elevation, and motion, and respect `prefers-reduced-motion`. With no
system: danger color + icon + bold short title + muted body + one prominent
action. In dark mode, confirm the red still reads.

### Part B checklist

- [ ] Title says **what**, plain language, no code or system term.
- [ ] Body says **why**, a cause the user can act on.
- [ ] Exactly one **clear action** that resolves or escapes the error.
- [ ] Subject is the **user's** thing, never "the system."
- [ ] Free of "Oops", "Sorry", "Unexpected", passive voice, exclamation marks.
- [ ] Severity level fits, inline / toast / dialog / full-screen / non-UI.
- [ ] Perceivable **without color**; correct ARIA role + live region; focus handled.
- [ ] A **way out** exists, no dead ends. Internal codes kept out of the body copy.

---

## The bridge, one failure, both jobs

The same failure flows through both parts. Keep the developer's truth and the
user's message separate:

```java
catch (StorageException e) {
  log.error("persist order {} failed: pool exhausted", orderId, e); // Part A: full context, for devs/logs
  showError( // Part B: translated, for the user
  "Your order couldn't be placed because our system is busy. " +
  "Wait a moment and try again, you weren't charged.");
}
```

The exception text (Part A) is precise and technical and goes to the logs. The
user message (Part B) is the three laws and never exposes the exception, the stack
trace, or the internal code. Get both right and a failure becomes debuggable *and*
recoverable.
