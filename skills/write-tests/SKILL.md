---
name: write-tests
description: Write the agreed test list as failing tests before any implementation, following the repo's own test patterns, in a pair programming session. Use after a plan is agreed, when asked to write the tests first, turn the plan into tests, or on /pair:write-tests. Tests only public interfaces so they survive implementation changes, covers accessibility with role and label queries in Testing Library's priority order, runs them once and shows each fails for the right reason. Writes test files only, never the implementation.
---

# Write tests

Goal: the plan, written down as tests that fail now and will pass once the feature works. Someone is watching, so explain each step in plain English as you go.

## Rules

- **Tests only.** Write or edit test files, plus the smallest stub needed for the test to load (an exported function that throws `not implemented`, or an empty component). No real implementation.
- **Public interfaces only.** Test what callers and users can see: a function's return value, what's on screen, what the user can do, which callback was called with what. Never internal state, private helpers, hooks' insides, class names or call counts on things the user can't see. A rewrite of the insides should break no test.
- **The repo's pattern.** Mirror one existing test file: location (co-located or a `tests` folder), naming (`.test.ts` vs `.spec.ts`), `describe`/`it` vs `test`, how renders and mocks are set up, any shared helpers. If `PAIR_PROGRAMMING.md` exists, its "Conventions found" section wins.
- **No new dependencies.** If Testing Library or user-event is missing, say so and ask.

## Steps

1. **Get the test list.** Use the one agreed in `pair:plan`. If there isn't one, list the behaviors in chat in short bullets and ask the person to confirm before writing.

2. **Pick a model test.** Find the existing test most like the one you're writing and say which one, as a `file` link. Copy its setup, not its assertions.

3. **Write logic tests first.** For pure functions: plain unit tests, one `it` per behavior, input in and expected output out. No mocks needed; if you need one, the function isn't pure, so say so.

4. **Then UI tests**, with Testing Library. Query the way a user or a screen reader finds things, in this order, and take the first one that works:
   1. `getByRole` with `name` — best for accessibility and meaning.
   2. `getByLabelText` — form controls.
   3. `getByText` — visible text that isn't a control.
   4. `getByTestId` — last resort, only when nothing above can find it. Say why when you use it.

   Use `findBy…` for anything that shows up after an async step, and `queryBy…` only to check something is absent. Drive interactions with `@testing-library/user-event` (`await user.click`, `user.type`, `user.keyboard`) if the repo has it, otherwise `fireEvent`.

5. **Cover accessibility as behavior**, not as a separate checklist:
   - every control is found by role and accessible name (if `getByRole` can't find it, that's a real a11y bug, not a reason to use a test id)
   - at least one keyboard-only test per interaction: Tab to it, Enter or Space to use it
   - errors and status messages are announced: `getByRole('alert')` or `getByRole('status')`
   - disabled and invalid states are visible to assistive tech (`toBeDisabled`, `aria-invalid`)

6. **Fake only the outside world.** Replace network, time, storage and randomness at the data-access boundary from the plan (`vi.fn`/`jest.fn`, MSW if the repo uses it, fake timers). Don't mock the module under test or its own pure helpers.

7. **Run once and read the failures.** Single-run mode, never watch (`npx vitest --run`, `CI=true npx jest`, or the script plus `-- --run`). Every new test must fail, and for the **right reason**: an assertion on missing behavior, not a syntax error, a wrong import or a typo in a query. Fix any test that fails for the wrong reason. A test that already passes is testing nothing new; say so.

## Report

In chat, at most 10 lines: the files written, the model test you followed, one line per test grouped by module (⚠ on edge cases), and the run result showing they fail for the right reason. Then stop. The person says when to implement; at that point the tests are fixed and the implementation must not change them.
