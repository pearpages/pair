---
name: implement
description: Make the agreed failing tests pass with the smallest change that follows the repo's conventions, in a pair programming session. Use after pair:write-tests, when asked to implement, make the tests pass, "go green", build it now, or on /pair:implement. Never edits a test to make it pass; if a test looks wrong it stops and says why. Builds in small steps along the plan's layers, then runs tests, typecheck and lint once each and reports. Does not commit.
---

# Implement

Goal: turn the failing tests into passing ones with the least code that does it, written the way this project's own developers would write it. Someone is watching, so say in plain English what you're about to change and why before each edit.

## Rules

- **The tests are fixed.** Don't edit, delete, skip (`.skip`, `.only`, `xit`) or loosen any test, its fixtures or its snapshots. If a test looks wrong (it contradicts the spec, another test or the glossary, or can't pass without breaking the design), stop, quote the line as a `file:line` link, say why, and ask. The person decides; only then does a test change.
- **Smallest change that passes.** No features, options, abstractions or refactors the tests don't ask for. If you see something worth doing beyond that, name it in the report as a next step instead of doing it.
- **The plan's design.** Follow the layers agreed in `pair:plan`: pure logic apart from data access and UI. Put each piece in the module the plan named. If the code wants a different shape, stop and say so instead of quietly changing the design.
- **The repo's conventions.** Mirror the nearest existing file for naming, file layout, imports, error handling and styling. If `PAIR_PROGRAMMING.md` exists, its "Conventions found" section wins. Name things with the words in its "Glossary". Styles go in the project's own styling approach, never inline styles or style objects unless that is what the repo already does.
- **No new dependencies.** If something seems to need one, say so and ask.
- **Don't commit.** Stop when the checks are run; committing comes later.

## Steps

1. **Read the failures.** Run only the new tests once, in single-run mode, never watch (`npx vitest --run <file>`, `CI=true npx jest <file>`, or the script plus `-- --run`). Say in one line each what's missing.

2. **Inside out.** Implement the pure logic first and run its tests; then the data access; then the UI wiring. One layer at a time, a small edit at a time, each preceded by a sentence on what it does. After each layer, run that layer's tests again.

3. **Replace the stubs.** Any `not implemented` stub or empty component left by `pair:write-tests` gets its real body. None remains at the end.

4. **When a test won't pass,** read the failure before changing anything. If the cause is in your code, fix the code. If it's in the test, apply the first rule: stop and ask.

5. **Run the checks once each**, using the commands in `PAIR_PROGRAMMING.md` (or `package.json`): the full test suite in single-run mode, the typecheck, and the lint. Don't auto-fix lint across files you didn't touch. If a check fails on code you didn't change, say it was already failing and leave it.

## Report

In chat, at most 10 lines: the files changed, one line per layer on what was added, the result of tests, typecheck and lint (pass or fail, with the count), any test you stopped on, and anything you noticed but left out of scope. Then stop. Next comes `pair:smoke-test`.
