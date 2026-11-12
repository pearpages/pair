---
name: plan
description: Turn the interviewer's spec into an agreed plan before any code is written, in a pair programming session. Use when given a task or spec to build, asked to plan, design, break down or "think about how we'd do this", or on /pair:plan. Asks at most three clarifying questions for the person to read to the interviewer, then proposes a modular, layered design (pure logic apart from UI and data access), the list of tests to write with edge cases that could break the app, and what is out of scope. Writes no files and no code; waits for agreement.
---

# Plan

Goal: in about 5 minutes, agree with the person (and through them, the interviewer) on **what** to build, **how it is split up**, and **which tests prove it works**, before writing anything. The test list is the contract for the next steps. Plain English: no unexplained jargon, and define a term in a few words the first time.

## Rules

- **The interviewer holds the requirements.** Don't invent them. When the spec is silent, ask; don't decide.
- **Write nothing.** No files, no code, no tests. The output is a plan in chat.
- **Follow the project.** If `PAIR_PROGRAMMING.md` exists, read it first. File names, folders, test style and naming in the plan come from its "Conventions found" section. Anything that isn't there is flagged as a choice.
- **Speak the domain language.** Name modules, functions, props and tests with the words in the "Glossary" section of `PAIR_PROGRAMMING.md`, not synonyms. If the spec brings a new concept, propose a name for it and list it under **New terms** so the interviewer can agree it; if the spec's words clash with the code's, ask which one wins.
- **Fit the time.** This is a 45-minute interview. Plan the smallest thing that meets the spec; list the rest as out of scope.

## Steps

1. **Restate the spec** in two or three plain sentences, so the person can check you understood it.

2. **Ask at most three questions**, in one round, only if the answers change the design or the tests. Number them so the person can read them aloud. Good questions are about behavior ("What should happen when the list is empty?"), not technology. If the spec is clear, say "No questions" and go on. Then **stop and wait** for the answers.

3. **Find where it fits.** Look at the code the change touches (start from the entry point in `PAIR_PROGRAMMING.md`). Find the closest existing feature and how it is split, so the new code can mirror it. Read only what you need.

4. **Propose the design as layers**, each with one job and a small public interface:
   - **Logic**: pure functions with no React, no I/O and no globals. Input in, output out. This is where most rules and edge cases live, and it's the easiest to test.
   - **Data access**: anything that talks to the outside world (fetch, storage, time, randomness), behind one small function or module so tests can replace it.
   - **UI**: components that call the logic and show the result. As thin as possible.
   Skip a layer the task doesn't need. For each module give the file path (in the repo's naming) and its public function signature or props. No new abstraction unless two places need it.

5. **List the tests**, one line each, grouped by module, written as behavior ("returns an empty list when no items match", "shows an error message when saving fails"), not as implementation. Each line says which layer it tests. Mark with ⚠ the **edge cases** that could break the app. Check each of these and include the ones that apply:
   - empty, missing or `null` input; one item; very many items
   - invalid input: wrong type, whitespace only, too long, special characters
   - boundaries: zero, negative, the exact limit and one past it
   - async: loading state, failure, slow response, a second request finishing before the first
   - user actions: double submit, clicking while disabled, keyboard only, screen reader names
   - state: what happens after an error, after a reset, on a second use

6. **Say what's out of scope** and any assumption you made, each as one line.

## Output format

At most one screen, in chat.

```
**Spec as I understand it:** <2–3 sentences>

**Design**
- Logic — `<path>`: `<signature>` — <one line>
- Data — `<path>`: `<signature>` — <one line>
- UI — `<path>`: `<props>` — <one line>

**Tests** (<runner>, <where they go, from the conventions>)
- Logic: <behavior>
- Logic: ⚠ <edge case>
- UI: <behavior>
- UI: ⚠ <edge case>

**New terms:** `<Term>` — <meaning> (or "none")
**Out of scope:** <…>
**Assumptions:** <… or "none">
```

Once agreed, add any new terms to the "Glossary" table in `PAIR_PROGRAMMING.md`.

End by asking for agreement. Do not start writing tests until the person says go; then `pair:write-tests` takes the test list as it is.
