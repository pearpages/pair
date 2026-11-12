# How I work with Claude Code in this session

I use a small set of skills (saved, reusable prompts) so every step follows the same routine. Each one follows **this repo's own conventions**, explains things in plain English, and works in Claude Code's default permission mode, so **every file edit and command waits for my approval** on screen.

You hold the requirements. When something isn't clear, I ask you; the AI doesn't guess.

The table below is the short version. To see exactly what each skill tells the AI, go to [The skills in full](#the-skills-in-full).

## The flow

| # | Phase | What I type | What it does | What it won't do |
|---|---|---|---|---|
| 1 | Explore | `/pair:onboard` | Reads the config and a few typical files, then summarizes the conventions: stack, commands, tests, styling, folders, commit style. Finds the **entry point**, builds a **glossary** of the project's own domain words (its ubiquitous language) so code and tests use the same names, and flags **anything odd**. Runs the tests once. Every claim points to a file as evidence, and guesses are labeled as guesses. | Touch any of your files. Its notes go in a local `PAIR_PROGRAMMING.md` that git ignores. |
| 2 | Plan | `/pair:plan <your spec>` | Asks **at most 3 questions**, which I read to you. Then it proposes a **modular, layered** design, named with the glossary's words: pure logic, data access and UI kept apart. It also lists **the tests we'll write**, including the edge cases that could break the app, and what's out of scope. | Write any code. It waits until we agree. |
| 3 | Write tests | `/pair:write-tests` | Turns the agreed list into tests, following this repo's existing test patterns. Tests only the **public interface**, so they survive a rewrite. Covers **accessibility**. Runs them and shows each one **fails for the right reason**. | Write the implementation. |
| 4 | Implement | *"Make the tests pass following the conventions. Don't edit the tests."* | Writes the smallest code that passes, then runs tests, typecheck and lint. | Change a test to make it pass. If a test looks wrong, it stops and says why. |
| 5 | Smoke test | `/pair:smoke-test` | Opens the app in a real browser (Chrome) and checks each spec by hand. Also does a **keyboard-only** pass and reads the console and network for errors. Reports pass/fail with evidence. | Fix anything. I decide what to act on. |
| 6 | Review | `/pair:review-diff` | A **separate agent**, which didn't write the code, reviews the change. It reports **at most 5 issues**, most severe first, in plain English. | Fix anything. I decide what to act on. |
| 7 | Commit | `/craft:commit-message` | Drafts a message in this repo's commit style and shows it before committing. | Push. |

Two more for situations along the way:

- `/pair:orient <question>`: "how does X work?" for one part of the code. It answers in chat with links to the exact lines and writes nothing.
- `/pair:react-component`: for a single new component, it does phases 3 and 4 in one test-first pass.

## How the tests find things on screen

Tests look for things the way a user or a screen reader would, in this order:

1. `getByRole` (with its name): best for accessibility and meaning
2. `getByLabelText`: form controls
3. `getByText`: visible text
4. `getByTestId`: last resort only

If a test can't find a control by its role and name, that's treated as an accessibility bug to fix, not a reason to add a test id.

## Why it's set up this way

- **Conventions first:** the code should look like your team wrote it, not like the AI's defaults.
- **Plan and tests before code:** we agree on what "done" means before anything is built.
- **A fresh reviewer:** the agent that wrote the code is biased towards it, so a clean one reviews it.
- **I stay in control:** every edit and command needs my approval, and nothing is fixed, committed or pushed without my say-so.

## The skills in full

Below is the exact prompt each skill gives the AI, word for word from this plugin. Open one to see it. Phase 4 (Implement) has no skill: the prompt in the table above is the whole instruction. `craft:commit-message` comes from another plugin, so it isn't included here.

### `pair:onboard` — 1 · Explore

*When it triggers:* Analyze a whole unfamiliar project at the start of a pair programming session and write a local PAIR_PROGRAMMING.md the session works from, without touching any tracked file. Use when asked to onboard, analyze, get to know, or set up for a project, at the start of an interview or pairing session, or on /pair:onboard. Finds the stack, the entry point, the commands, the test runner, the styling approach, the folder layout, the commit style and a glossary of the project's domain terms (its ubiquitous language), flags anything odd or inconsistent, runs the tests once, and separates what the repo really does (with evidence) from everything that is only an assumption.

<details>
<summary>Show the full prompt</summary>

````markdown
# Onboard

Goal: in about 3 minutes, learn how this project works and write it down, so that every later step follows the project's own conventions instead of your defaults. Someone is watching, so talk in plain English: no unexplained jargon, and define a term in a few words the first time you use it.

## Rules

- **Do not touch tracked files.** You write only `PAIR_PROGRAMMING.md` and `CLAUDE.local.md`, and you hide both from git (step 5). Never edit `CLAUDE.md`, `AGENTS.md`, `.gitignore` or any source file here.
- **Read config, not source.** Open config files and two or three representative files. Do not read the whole codebase; that takes too long.
- **Evidence or label.** Every statement in the file is either backed by a path you saw, or written up as an assumption (see the template). Never present a guess as a convention.

## Steps

1. **Check what already exists.** Look for `PAIR_PROGRAMMING.md`, `CLAUDE.md`, `AGENTS.md`, `CLAUDE.local.md` and `README*`. Read the ones that exist. If `PAIR_PROGRAMMING.md` already exists, do not overwrite it: read it, check that it still matches the repo, report any differences in chat, and skip to step 6.

2. **Find the stack and the commands.**
   - Package manager: the lockfile (`package-lock.json` → npm, `pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `bun.lockb` → bun). Non-JS projects: the equivalent manifest.
   - `package.json` scripts: test, lint, typecheck, format, dev, build. Note the exact command for each. Note the ones that are **missing**.
   - Language and strictness: `tsconfig.json` (`strict`, path aliases).
   - Framework: React, Next, Vite, Remix and so on, from dependencies and config files.
   - **Entry point**: where the app starts. Look at `index.html` (`<script src>`), `main`/`module`/`bin` in `package.json`, `src/main.*`, `src/index.*`, `app/` or `pages/` for Next, the server file for a backend. Then find where routing or the top-level component is set up. Write both down as `file:line`. A library or CLI may have several entry points; list each one.

3. **Find the conventions.**
   - Tests: runner (vitest, jest, playwright…), config file, where tests live (`find . -name "*.test.*" -not -path "*/node_modules/*" | head`), co-located or in a `__tests__`/`tests` folder, Testing Library present or not, how one existing test is written.
   - Styling: CSS Modules (`*.module.css|scss`), plain CSS/SCSS with class naming (BEM: `block__element--modifier`), Tailwind (`tailwind.config.*`), CSS-in-JS (styled-components, emotion). Note which one an existing component actually uses.
   - Components: folder layout, file naming (`Button.tsx` vs `button.tsx`, `index.ts` barrels), default vs named exports, how props are typed.
   - Lint and format: ESLint, Prettier, Biome config files.
   - Folder layout: top two levels, leaving out `node_modules`, build output and dot folders.
   - Commit style: `git log --oneline -15`. Conventional Commits or not, tense, scopes.
   - **Domain language** (the project's ubiquitous language: the words the team and the code use for the things the app is about, like `Order`, `Basket`, `Checkout`, not technical words like `hook` or `reducer`). First look for one that already exists: `GLOSSARY.md`, `CONTEXT.md`, `docs/`, the README. If there isn't one, collect up to about 10 terms from where the domain shows: type and interface names (`grep -rhoE "(type|interface) [A-Z][A-Za-z]+" src | sort | uniq -c | sort -rn | head -20`), folder and feature names, API routes, and the words on screen in one or two components. For each term write what it means in one plain sentence and where you saw it. A meaning you read from a comment or doc is evidence; a meaning you guessed from the name is an assumption, so mark it with "(inferred)". Note **synonyms** (two words for one thing, like `user` and `customer`) and **one word used for two things**; those go under "Looks odd" too, and are good questions for the interviewer.
   - **Anything odd.** While you look, note what doesn't add up: two conventions that contradict each other (some components use CSS Modules, others Tailwind), config for a tool that isn't installed, or a tool installed with no config, a script that points at a missing file, tests that are skipped, a disabled lint rule, a dependency pinned to a very old version, a folder that breaks the layout everyone else follows. Each needs the paths you saw. Odd doesn't mean wrong: say what you saw and why it might matter, not that it's a mistake.

4. **Run the tests once.** Use a single-run command, never watch mode: `npx vitest --run`, `CI=true npx jest`, or the script plus the right flag (`npm test -- --run` for vitest). Note the result: how many passed, failed or were skipped, and how long they took. If the tests fail or cannot run, do not fix anything. Write it down as a finding and tell the person in chat.

5. **Write the files and hide them from git.**
   - Write `PAIR_PROGRAMMING.md` at the repo root using the template below.
   - `CLAUDE.local.md` at the repo root: if it does not exist, create it with the single line `@PAIR_PROGRAMMING.md`. If it exists and lacks that line, append it. If `git ls-files --error-unmatch CLAUDE.local.md` says it is tracked, do not touch it: stop and ask the person.
   - Add both names to git's local ignore list, which lives only on this machine and is never committed. Find it with `git rev-parse --git-path info/exclude` and append each name only if it is not already there. If this is not a git repo, skip this and say so.

6. **Report in chat**, in plain English, at most 12 lines: what the project is, the entry point (`file:line`), how to run tests, the result of the test run, the conventions that matter most for the work ahead, one line each, the three to five domain terms that matter most, the odd things you found, and the questions for the interviewer from the "Not project conventions" section. Keep the conventions in mind for the rest of the session; the file is loaded through `CLAUDE.local.md`, so later steps can follow it.

## `PAIR_PROGRAMMING.md` template

```markdown
# Pair programming notes

Local notes for this session. Not committed (listed in .git/info/exclude).

## What this project is
<two or three plain-English sentences>

## Entry point
- App starts at: `<path:line>` — <one sentence: what it mounts or runs>
- Routing / top-level component: `<path:line>`

## Commands
| What | Command | Evidence |
|---|---|---|
| Install | ... | package-lock.json |
| Test (single run) | ... | package.json "test" |
| Lint | ... | ... |
| Typecheck | ... | ... |

Test run at onboarding: <N passed, N failed, N skipped, time>.

## Conventions found in the repo
Each one is something the repo really does. Follow these exactly.

- **Tests**: <runner, location, style> — evidence: `<path>`
- **Styling**: <approach and class naming> — evidence: `<path>`
- **Components**: <naming, exports, props typing> — evidence: `<path>`
- **Lint/format**: ... — evidence: `<path>`
- **Commits**: <style> — evidence: `git log`
- **Folder layout**: <short tree or description>

## Glossary
The project's own words for the things it is about. Use these words, spelled this way, in code, tests, names and when talking. Don't introduce a synonym.
Source: <existing glossary file, or "collected at onboarding">

| Term | Means | Evidence |
|---|---|---|
| `<Term>` | <one plain sentence> | `<path>` |
| `<Term>` | <one plain sentence> (inferred) | `<path>` |

Synonyms or clashes: <e.g. "`user` and `customer` both mean the buyer" — or "none found">

## Looks odd
Things in the repo that don't add up. Not necessarily wrong; worth knowing before changing nearby code.

- <what you saw> — evidence: `<path>`, `<path>` — <why it might matter for the work>

<or "Nothing odd found.">

## Not project conventions
Everything below was NOT found in the repo. Each entry is a default, a guess or a gap.
Do not present any of these as "how this project does it".

### <short title, e.g. "No lint script">
- **What was assumed or is missing:** <be specific>
- **Why:** <what was looked for and not found, or why this default was picked>
- **What it changes for the work:** <e.g. "lint will not catch unused imports; check by eye">
- **Ask the interviewer:** <one concrete question>

<one entry per assumption or gap; be verbose here>

## Working rules for this session
- Follow "Conventions found" over any personal or default preference.
- Name things with the words in "Glossary". A new concept gets a name agreed with the interviewer, then added there.
- Before anything listed under "Not project conventions", say so out loud and ask.
- Do not add dependencies without asking.
- Run tests in single-run mode, never watch mode.
- Explain in plain English; define jargon the first time.
```
````

</details>

### `pair:plan` — 2 · Plan

*When it triggers:* Turn the interviewer's spec into an agreed plan before any code is written, in a pair programming session. Use when given a task or spec to build, asked to plan, design, break down or "think about how we'd do this", or on /pair:plan. Asks at most three clarifying questions for the person to read to the interviewer, then proposes a modular, layered design (pure logic apart from UI and data access), the list of tests to write with edge cases that could break the app, and what is out of scope. Writes no files and no code; waits for agreement.

<details>
<summary>Show the full prompt</summary>

````markdown
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
````

</details>

### `pair:write-tests` — 3 · Write tests

*When it triggers:* Write the agreed test list as failing tests before any implementation, following the repo's own test patterns, in a pair programming session. Use after a plan is agreed, when asked to write the tests first, turn the plan into tests, or on /pair:write-tests. Tests only public interfaces so they survive implementation changes, covers accessibility with role and label queries in Testing Library's priority order, runs them once and shows each fails for the right reason. Writes test files only, never the implementation.

<details>
<summary>Show the full prompt</summary>

````markdown
# Write tests

Goal: the plan, written down as tests that fail now and will pass once the feature works. Someone is watching, so explain each step in plain English as you go.

## Rules

- **Tests only.** Write or edit test files, plus the smallest stub needed for the test to load (an exported function that throws `not implemented`, or an empty component). No real implementation.
- **Public interfaces only.** Test what callers and users can see: a function's return value, what's on screen, what the user can do, which callback was called with what. Never internal state, private helpers, hooks' insides, class names or call counts on things the user can't see. A rewrite of the insides should break no test.
- **The repo's pattern.** Mirror one existing test file: location (co-located or a `tests` folder), naming (`.test.ts` vs `.spec.ts`), `describe`/`it` vs `test`, how renders and mocks are set up, any shared helpers. If `PAIR_PROGRAMMING.md` exists, its "Conventions found" section wins.
- **The domain's words.** Test names and `describe` blocks use the terms in the "Glossary" section of `PAIR_PROGRAMMING.md`, so a test reads like a sentence about the business: "applies the discount to the basket total", not "returns correct value".
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
````

</details>

### `pair:smoke-test` — 5 · Smoke test

*When it triggers:* Check by hand, in the real browser, that a finished change meets its spec, in a pair programming session. Use after the tests pass, when asked to smoke test, try it in the browser, check it manually, "see if it actually works", or on /pair:smoke-test. Starts the dev server, drives the page with the Claude in Chrome tools, checks each spec, does one keyboard-only pass and reads the console, then reports pass or fail with evidence. Fixes nothing.

<details>
<summary>Show the full prompt</summary>

````markdown
# Smoke test

Goal: in about 3 minutes, show that the feature works for a real user in a real browser, not only in tests. Tests run in a fake DOM; this catches what they can't: styling that hides things, wiring that tests mocked out, console errors, focus that gets lost. Narrate each check in plain English so the person can follow along.

## Rules

- **Report only.** Don't edit any file. If something fails, describe it; the person decides what to fix.
- **Local only.** Use the local dev server. Don't log in to real accounts or enter real personal data; use the project's seed or test data, or ask.
- **Chrome first.** Use the Claude in Chrome tools (`mcp__claude-in-chrome__*`). If they're deferred, load the ones you need in one `ToolSearch` call: `tabs_context_mcp`, `tabs_create_mcp`, `navigate`, `computer`, `read_page`, `find`, `form_input`, `read_console_messages`, `read_network_requests`. If Chrome isn't connected, say so and use the built-in browser if there is one, or stop and ask.

## Steps

1. **Get the specs.** Use the agreed test list from `pair:plan`, or the task as the interviewer gave it. Write the checks as a numbered list in chat, one user-visible action and expected result each. Add the ⚠ edge cases you can reach from the UI.

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
````

</details>

### `pair:review-diff` — 6 · Review

*When it triggers:* Review the current uncommitted change with a fresh pair of eyes and return at most five issues, ranked by severity, in plain English. Use before committing in a pair programming session, when asked to review the diff, check the change, or "look over this before I commit", or on /pair:review-diff. Runs one sub-agent with a clean context that reads git diff itself, so it is not biased by the conversation. Reports only; fixes nothing.

<details>
<summary>Show the full prompt</summary>

````markdown
# Review diff

Goal: a quick, honest second opinion on the change, in about a minute, before it is committed. The person decides what to act on.

## Why a sub-agent

The session that wrote the code has already convinced itself the code is right. A sub-agent starts with a clean context: it sees only the code, the task and the project notes, not the reasoning that produced them.

## Steps

1. Check there is something to review: `git status --porcelain`. If nothing changed, say so and stop.
2. Write the task in one or two sentences, from what the person or interviewer asked for. This is the only conversation context the reviewer gets. Do not include your opinion of the code.
3. Start **one** sub-agent (Agent tool, general-purpose) with this prompt, filling in the task:

   > You are reviewing an uncommitted change in this repository. The task was: <task>.
   >
   > 1. Run `git diff HEAD` to see changed tracked files, and `git status --porcelain` to find new untracked files; read those in full.
   > 2. If `PAIR_PROGRAMMING.md` exists at the repo root, read it: its "Conventions found" section is the standard to review against.
   > 3. Run the project's single-run test command (from `PAIR_PROGRAMMING.md`, or `package.json`; never watch mode).
   >
   > Report **at most five** issues, most severe first. Look for, in this order: bugs and broken behavior; the change not doing what the task asked, or doing more than it asked; missing or weak tests for the new behavior; accessibility problems in UI code; breaking the project's conventions. Skip style nitpicks a formatter would fix.
   >
   > For each issue, one block:
   > **<n>. <Must fix | Should fix | Minor> — <short title>** — `file:line`
   > What is wrong, in one or two plain-English sentences, and what would go wrong for a user or developer.
   > Suggested fix: one sentence.
   >
   > Then one line with the test result. If you find nothing worth reporting, say "No issues found" and the test result. Do not edit any file.

4. Relay the reviewer's report to the person **as it is**. Do not soften it, defend the code, or add issues of your own. You may add one line afterwards if a finding looks wrong to you, clearly marked as your opinion.
5. Do not fix anything until the person says which findings to act on.
````

</details>

### `pair:orient` — Situational

*When it triggers:* Answer a question about one specific part of the code, in plain English, during a pair programming session. Use when asked how a specific feature, file, module, flow or function works, where something happens, what calls what, or "walk me through X", or on /pair:orient. Traces the path through the code with file:line links and says what to watch out for. Writes no files. For the whole project at the start, use onboard instead.

<details>
<summary>Show the full prompt</summary>

````markdown
# Orient

Goal: give a clear, short answer about one part of the code that the person can repeat out loud to the interviewer. Plain English: no unexplained jargon, and define a term in a few words the first time.

## Steps

1. If `PAIR_PROGRAMMING.md` exists at the repo root, read it first so you use the project's own words and layout.
2. Find the entry point for the question: search for the route, component, function, event name or text on screen (`grep -rn`). Start from where the user or caller enters, not from the deepest helper.
3. Follow the path step by step. Read only the files on that path.
4. Stop when you can answer. If the question is too broad ("how does the app work?"), say so and suggest `pair:onboard` or a narrower question.

## Answer format

At most about 15 lines, in chat. Write no files.

```
**What it does:** <2–3 plain sentences>

**The path:**
1. <step> — [File.tsx:12](path/File.tsx:12)
2. <step> — [api.ts:40](path/api.ts:40)
3. ...

**Watch out for:** <1–3 things: surprising behavior, missing tests, a shared piece other features use>

**Not sure about:** <anything you could not confirm from the code, or "nothing">
```

Separate what you read in the code from what you are inferring. If something is an inference, say "probably" and why.
````

</details>

### `pair:react-component` — Situational

*When it triggers:* Build a new React component test-first, following the repo's own conventions, with accessibility built in. Use when asked to create, add or build a React component, widget, form, modal, list or other piece of UI, or on /pair:react-component. Writes a co-located Testing Library test first using role and label queries, then a typed, semantic, keyboard-accessible component, then the styles in whatever approach the repo already uses. Never uses inline styles or snapshot tests.

<details>
<summary>Show the full prompt</summary>

````markdown
# React component

Goal: a small, working, accessible component that looks like it was written by someone on the team. Explain each step in plain English as you go.

## 1. Learn the conventions (1 minute)

- If `PAIR_PROGRAMMING.md` exists, read its "Conventions found" and "Not project conventions" sections and follow them.
- Either way, find the **existing component most similar** to the one being built and mirror it: folder and file naming, default or named export, how props are typed (`type` vs `interface`, `Props` naming), barrel `index.ts` or not, where the test lives and how it is written, and the styling approach.
- If something you need is not in the repo (for example Testing Library is not installed, or there is no styling approach), stop and say so in plain words. Do not add a dependency or pick an approach without asking.

## 2. Pin down the behavior

Before writing code, list in chat in 3–6 short bullets what the component must do: its props, what the user sees, what they can click or type, and the states that matter (empty, loading, error, disabled). Ask about anything the task does not say. Keep it to what was asked; do not invent features.

## 3. Write the test first (red)

- Put it next to the component, or wherever the repo puts tests, with the repo's naming (`Foo.test.tsx`, `Foo.spec.tsx`…).
- Query the way a user or a screen reader finds things, in this order, and take the first one that works:
  1. `getByRole` with `name` — best for accessibility and meaning.
  2. `getByLabelText` — form controls.
  3. `getByText` — visible text that isn't a control.
  4. `getByTestId` — last resort, only when nothing above can find it. Say why when you use it.
  Use `findBy…` for anything that appears after an async step, and `queryBy…` only to check something is absent.
- Drive it with `@testing-library/user-event` if the repo has it, otherwise `fireEvent`.
- One test per behavior from step 2. Include at least one keyboard check for anything interactive (Tab to it, Enter/Space to use it).
- Test the public interface only: props in, what the user sees and can do out, and callbacks called with the right arguments. A rewrite of the component's insides should not break any test.
- No snapshot tests. No checking internal state, hooks or class names.
- Run it once (single-run mode, never watch) and show that it fails for the right reason.

## 4. Write the component (green)

- Use native elements first: `<button>`, `<a href>`, `<label>` + `<input>`, `<ul>/<li>`, headings in order. Add ARIA only when no native element does the job.
- Every interactive element has an accessible name. Every input has a label. Images have `alt` (empty `alt=""` if decorative).
- Keyboard: everything clickable is reachable and usable from the keyboard; do not remove the focus outline without replacing it.
- Typed props with no `any`. Export the props type if the repo does.
- Keep it small: no new abstraction, hook or context unless the task needs it.
- Do not change a test to make it pass. If a test looks wrong, stop and say why.
- Run the test again and show it passing.

## 5. Style it

- Use the repo's approach exactly: CSS Modules, plain CSS/SCSS with its class naming (for example BEM), Tailwind, or whatever the similar component uses.
- **Never** use inline `style={{…}}`, style objects or CSS-in-JS unless that is the repo's established approach.
- If the repo has no styling approach at all, say so and ask; if told to choose, use a CSS Module next to the component.

## 6. Check

Run the repo's typecheck and lint commands if they exist, plus the test once more. Then summarize in plain English: what was built, the files touched, and anything you assumed.
````

</details>
