---
name: verifying-ui-in-running-app
description: Use when verifying a frontend/UI change in a running hot-reload dev-server app (often with a mock API like MSW) via browser automation (Puppeteer/Playwright), and the change "doesn't show up", or your automated check passes/fails in a way that doesn't match the code. Covers stale bundles, change-detection zones, and mock-API signals.
---

# Verifying UI Changes in a Running App

## Overview

When you verify a UI change in a running app, **two layers can lie to you**: the dev server (may serve a *stale* bundle) and your automation harness (may not drive the framework's reactivity). Don't trust "it looks broken" or "it looks fixed" until you've ruled both out.

**Core principle:** Confirm the build, drive with *trusted* events, and read *framework state* — not just the DOM.

## When to use

- You changed template/form/component code and "it doesn't appear" in the running app.
- Your Puppeteer/Playwright check gives a result that contradicts the code.
- App uses a watch/HMR dev server and/or a mock API (MSW, mirage, etc.).

## The traps (the meat)

| Trap | Symptom | Fix |
|---|---|---|
| **Stale bundle from a compile error** | App returns 200 and "looks up", but your change isn't there; tests reflect old behavior | A TS/template error makes the watch server keep serving the **last good bundle**. **Run the real build** (`nx build <proj>`, `tsc --noEmit`) to surface the error. **"Server responds 200" is NOT proof your code compiled.** Fix, rebuild, then hard-reload. |
| **`evaluate` runs OUTSIDE the framework's change-detection / zone** | Synthetic `el.value=…; dispatchEvent(new Event('input'))` or `el.click()` change the DOM but nothing re-renders | Synthetic events in `page.evaluate` don't tick CD or update the form model. **Drive with trusted events**: real `page.click()` / `page.type()` / `fill`. Use `evaluate` only to **read state** or `fetch`. |
| **Asserting on the DOM only** | Control is invalid but no error shows; value "didn't change" | Read **framework state**: in Angular dev mode `window.ng.getComponent(el)` → `control.value/valid/errors/touched`, signals. The DOM can lag a CD tick. |
| **Mock-API signals mislead** | `fetch(endpoint)` → 200, so "it works" | 200 only proves the worker intercepts — not that *your scenario* passes. The in-memory DB usually **resets on full reload** (records created via API vanish). Use a deterministic seed (`faker.seed(n)`) so data is stable across reloads. |

## Angular OnPush form-error gotchas

- `Validators.required` treats whitespace `'  '` as **valid** → use a **trimming** required validator (e.g. a DS `DmValidators.required`) for true "required".
- A form-field renders errors off the control's **`statusChanges`**. `markAllAsTouched()` does **not** emit `statusChanges`, so under OnPush the error won't render on submit. Use a util that also calls `updateValueAndValidity()` (emits the event). `setErrors()` *does* emit — use it for server/duplicate errors so the field renders.
- A conditionally-projected hint (`ContentChild`) may not render under OnPush → render it as a plain **sibling** element instead.

## Red flags — stop and check

- "The dev server is up (200)" → **did you actually build?**
- "My puppeteer click did nothing" → trusted event vs `evaluate`.
- "Control is invalid but no error shows" → `statusChanges` / `markAllAsTouched` / build.
- "Created a record but it's gone after reload" → mock DB reset; seed it.

## Reliable verification loop

1. **Build** the affected project — catch compile errors that would freeze the bundle.
2. Wait for the dev server to recompile; **hard-reload** the page.
3. Drive the UI with **real** clicks/typing (trusted events).
4. Assert on **framework state** (`window.ng`) and the rendered DOM/screenshot.
5. For mock APIs, rely on **seeded, deterministic** data — not on freshly created records surviving a reload.
