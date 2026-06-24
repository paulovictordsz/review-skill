# review-skill

A [Claude Code](https://claude.com/claude-code) skill for **reliably verifying frontend/UI changes in a running app** — the kind driven by a hot-reload dev server and a mock API (e.g. MSW), checked via browser automation (Puppeteer/Playwright).

## What it solves

When a UI change "doesn't show up" or your automated check passes/fails in a way that contradicts the code, the problem is often **not the code** — it's the verification setup lying to you:

- **Stale bundle:** a compile error makes the watch dev-server keep serving the *last good* bundle. The app still returns `200`, so it "looks up" — but your change isn't there. **A 200 is not proof your code compiled.**
- **Out-of-zone automation:** synthetic events inside `page.evaluate` mutate the DOM but don't trigger the framework's change detection. Drive with **trusted** events (real click/type/fill); use `evaluate` only to *read* state.
- **Mock-API signals:** a `200` only proves interception, not that your scenario works; in-memory DBs reset on reload — seed deterministically.
- **Angular OnPush form gotchas:** whitespace passes `required`; `markAllAsTouched()` doesn't emit `statusChanges`; `setErrors()` does.

## Install

Copy the skill folder into your personal Claude Code skills directory:

```bash
git clone https://github.com/paulovictordsz/review-skill.git
cp -r review-skill/verifying-ui-in-running-app ~/.claude/skills/
```

The skill loads automatically when the triggering conditions match.

## How it was built

Authored with the TDD-for-skills process (RED → GREEN): a baseline agent *without* the skill buried the stale-bundle trap; *with* the skill it leads with "run the build first."
