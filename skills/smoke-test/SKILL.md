---
name: smoke-test
description: Check by hand, in the real browser, that a finished change meets its spec, in a pair programming session. Use after the tests pass, when asked to smoke test, try it in the browser, check it manually, "see if it actually works", or on /pair:smoke-test. Starts the dev server, drives the page with the Claude in Chrome tools, checks each spec, does one keyboard-only pass and reads the console, then reports pass or fail with evidence. Fixes nothing.
---

# Smoke test

Goal: in about 3 minutes, show that the feature works for a real user in a real browser, not only in tests. Tests run in a fake DOM; this catches what they can't: styling that hides things, wiring that tests mocked out, console errors, focus that gets lost. Narrate each check in plain English so the person can follow along.

## Rules

- **Report only.** Don't edit any file. If something fails, describe it; the person decides what to fix.
- **Local only.** Use the local dev server. Don't log in to real accounts or enter real personal data; use the project's seed or test data, or ask.
- **Chrome first.** Use the Claude in Chrome tools (`mcp__claude-in-chrome__*`). If they're deferred, load the ones you need in one `ToolSearch` call: `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `computer`, `read_page`, `find`, `form_input`, `read_console_messages`, `read_network_requests`. If Chrome isn't connected, say so and use the built-in browser if there is one, or stop and ask.

## Steps

1. **Get the specs.** Use the test list in the "Current plan" section of `PAIR_PROGRAMMING.md`, or the one agreed in chat, or the task as the interviewer gave it. Write the checks as a numbered list in chat, one user-visible action and expected result each. Add the ⚠ edge cases you can reach from the UI.

2. **Start the app.** Take the dev command from `PAIR_PROGRAMMING.md` (or `package.json` `dev`/`start`). If a server is already running, reuse it. Otherwise start it in the background, never in the foreground, and read its output for the local URL and any build errors. A build error is a failed smoke test: report it and stop.

3. **Open the page** in a new tab and go to the screen the change is on. Take one screenshot so the person sees the starting state.

4. **Check each spec in order.** Do what a user would do: click, type, submit. After each, confirm the result with `read_page` or `find` (the text and roles on screen) rather than guessing from a screenshot. Take a screenshot only when it shows something words can't.

5. **One keyboard-only pass** over the new UI: Tab through it, use Enter and Space, Escape on anything that opens. Check that every control is reachable, the focus outline is visible, and focus lands somewhere sensible after an action (a dialog closing returns focus to what opened it).

6. **Read the console and network.** `read_console_messages` with errors only, and `read_network_requests` for failed requests (4xx, 5xx) during your checks. Ignore noise that was there before the change (browser extensions, dev-tool warnings), but mention it in one line.

7. **Optional, if time allows:** a narrow window (375 px wide) to check nothing overflows or overlaps.

## Report

In chat:

```
| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | <action → expected> | ✅ Pass / ❌ Fail | <what you saw, one line> |
| … | Keyboard only | … | … |
| … | Console / network | … | <N errors, or "clean"> |

<Failures, one or two plain sentences each: what you did, what happened, what was expected.>
```

Leave the dev server running and say so, so the person can try it themselves. Do not fix anything until they say which failures to act on.
