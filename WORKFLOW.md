# How I work with Claude Code in this session

I use a small set of skills (saved, reusable prompts) so every step follows the same routine. Each one follows **this repo's own conventions**, explains things in plain English, and works in Claude Code's default permission mode, so **every file edit and command waits for my approval** on screen.

You hold the requirements. When something isn't clear, I ask you; the AI doesn't guess.

## The flow

| # | Phase | What I type | What it does | What it won't do |
|---|---|---|---|---|
| 1 | Explore | `/pair:onboard` | Reads the config and a few typical files, then summarizes the conventions: stack, commands, tests, styling, folders, commit style. Finds the **entry point** and flags **anything odd**. Runs the tests once. Every claim points to a file as evidence, and guesses are labeled as guesses. | Touch any of your files. Its notes go in a local `PAIR_PROGRAMMING.md` that git ignores. |
| 2 | Plan | `/pair:plan <your spec>` | Asks **at most 3 questions**, which I read to you. Then it proposes a **modular, layered** design: pure logic, data access and UI kept apart. It also lists **the tests we'll write**, including the edge cases that could break the app, and what's out of scope. | Write any code. It waits until we agree. |
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
