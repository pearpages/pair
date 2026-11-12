---
name: onboard
description: Analyze a whole unfamiliar project at the start of a pair programming session and write a local PAIR_PROGRAMMING.md the session works from, without touching any tracked file. Use when asked to onboard, analyze, get to know, or set up for a project, at the start of an interview or pairing session, or on /pair:onboard. Finds the stack, the entry point, the commands, the test runner, the styling approach, the folder layout and the commit style, flags anything odd or inconsistent, runs the tests once, and separates what the repo really does (with evidence) from everything that is only an assumption.
---

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
   - **Anything odd.** While you look, note what doesn't add up: two conventions that contradict each other (some components use CSS Modules, others Tailwind), config for a tool that isn't installed, or a tool installed with no config, a script that points at a missing file, tests that are skipped, a disabled lint rule, a dependency pinned to a very old version, a folder that breaks the layout everyone else follows. Each needs the paths you saw. Odd doesn't mean wrong: say what you saw and why it might matter, not that it's a mistake.

4. **Run the tests once.** Use a single-run command, never watch mode: `npx vitest --run`, `CI=true npx jest`, or the script plus the right flag (`npm test -- --run` for vitest). Note the result: how many passed, failed or were skipped, and how long they took. If the tests fail or cannot run, do not fix anything. Write it down as a finding and tell the person in chat.

5. **Write the files and hide them from git.**
   - Write `PAIR_PROGRAMMING.md` at the repo root using the template below.
   - `CLAUDE.local.md` at the repo root: if it does not exist, create it with the single line `@PAIR_PROGRAMMING.md`. If it exists and lacks that line, append it. If `git ls-files --error-unmatch CLAUDE.local.md` says it is tracked, do not touch it: stop and ask the person.
   - Add both names to git's local ignore list, which lives only on this machine and is never committed. Find it with `git rev-parse --git-path info/exclude` and append each name only if it is not already there. If this is not a git repo, skip this and say so.

6. **Report in chat**, in plain English, at most 12 lines: what the project is, the entry point (`file:line`), how to run tests, the result of the test run, the conventions that matter most for the work ahead, one line each, the odd things you found, and the questions for the interviewer from the "Not project conventions" section. Keep the conventions in mind for the rest of the session; the file is loaded through `CLAUDE.local.md`, so later steps can follow it.

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
- Before anything listed under "Not project conventions", say so out loud and ask.
- Do not add dependencies without asking.
- Run tests in single-run mode, never watch mode.
- Explain in plain English; define jargon the first time.
```
