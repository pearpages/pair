# pair

Skills for a 45–50 minute AI-assisted pair programming interview built around day-to-day tasks, packaged as a local Claude Code plugin called `pair`.

The idea: reuse existing skills where they already fit, and build only the gaps.

| Skill | What it does |
|---|---|
| `pair:onboard` | Analyzes the whole project, finds the entry point, builds a glossary of its domain terms, flags anything odd, runs the tests once, and writes a local `PAIR_PROGRAMMING.md` without touching any tracked file. |
| `pair:orient` | Answers a question about one part of the code in chat, with `file:line` links. Writes nothing. |
| `pair:plan` | Asks at most three questions for the interviewer, then proposes a layered design (pure logic, data access, UI), the test list with edge cases, and what's out of scope. Writes no code; once agreed, saves the plan to the local `PAIR_PROGRAMMING.md`. |
| `pair:write-tests` | Turns the agreed test list into failing tests in the repo's own pattern, testing public interfaces only, with Testing Library queries in priority order. No implementation. |
| `pair:implement` | Makes the failing tests pass with the smallest change in the repo's conventions, layer by layer. Never edits a test; stops and asks if one looks wrong. Runs tests, typecheck and lint once. |
| `pair:smoke-test` | Starts the dev server and checks each spec by hand in Chrome, plus a keyboard-only pass and the console. Reports pass or fail; fixes nothing. |
| `pair:review-diff` | A fresh sub-agent reviews `git diff` and returns at most five issues, ranked by severity. |
| `pair:react-component` | Builds a React component test-first, in the repo's own conventions, with accessibility built in. |

All of them explain things in plain English: no unexplained jargon, and a short definition the first time a term appears.

## Install

You don't need to publish anything. This repo is its own local marketplace (`.claude-plugin/marketplace.json`).

```bash
claude plugin marketplace add ~/Projects/pair
```

```bash
claude plugin install pair@pair
```

Or, inside a session, `/plugin marketplace add ~/Projects/pair`, then install `pair` from `/plugin`. The skills load on the next session start, namespaced as `pair:…`, so `pair:onboard` doesn't clash with `xray:onboard`. After you edit a skill here, restart the session to pick up the change. If it doesn't show up, run `claude plugin marketplace update pair`.

**Fallback (no plugin system available):** copy `skills/*` into `<their-repo>/.claude/skills/` and add `.claude/skills/` to `<their-repo>/.git/info/exclude` so it stays out of their git status. The skills then lose the `pair:` prefix.

### Also used, from other plugins

- `mattpocock-skills:grilling` (formerly `grill-me`): clarifying a task. It's capped, see below.
- `mattpocock-skills:tdd`: test-first features.
- `mattpocock-skills:diagnosing-bugs`: bugs.
- `craft:commit-message`: commits in the repo's own style, shown before committing, no co-author trailer.

### Not used, and why

- `git-guardrails-claude-code`: its files ship with mattpocock-skills (`skills/misc/`), but the plugin doesn't offer it as a skill, and the permission mode does the job more visibly (see below).
- PRD, issue and triage skills: don't fit in 45 minutes.
- `zoom-out`: removed upstream; `pair:onboard` covers it.
- xray: not ready, so don't rely on it.
- Other installed plugins (craft, xray) stay enabled. If one triggers during the interview (`craft:implement`, `xray:onboard`…), decline it. They read knowledge files and file issues, which doesn't fit someone else's repo.

## Permission mode: default, not auto

Run the interview in Claude Code's **default permission mode**: every file edit and command asks for approval. Don't use auto mode or "accept edits".

Why: each approval prompt is visible on the shared screen, so the interviewer can see that you read and approve what the AI does. That is a stronger signal than a guardrail hook that blocks things silently. It also replaces `git-guardrails`: nothing like `git push` or `git reset --hard` runs without you saying yes.

## Clarifying is capped

`pair:plan` asks its own clarifying questions: one round, three at most, numbered so you can read them to the interviewer. You only need `grilling` (below) when you want to stress-test an idea outside that flow.

### If you use grilling, cap it

`mattpocock-skills:grilling` is built to question you relentlessly, round after round, until a whole plan is settled. In a timed interview that's the wrong shape, and the **interviewer** holds the requirements, not you.

So cap it: **one round, three questions at most**. Then **you read the questions to the interviewer** and pass their answers back. Clarifying requirements is your job in front of them; the skill only drafts good questions.

How to call it:

```
/mattpocock-skills:grilling one round only, at most 3 questions: <the task as the interviewer gave it>
```

## Why the reviewer is a lightweight sub-agent

`mattpocock-skills:code-review` already exists. It reviews on two axes, Standards and Spec, in parallel sub-agents, against a fixed base and an originating issue or spec. In an interview there is no issue, there is a lot less time, and two agents take longer.

`pair:review-diff` is deliberately smaller:

- **One** sub-agent with a **clean context**. The session that wrote the code has already convinced itself the code is right. The reviewer sees only the diff, the one-line task and `PAIR_PROGRAMMING.md`.
- **At most five issues**, most severe first (Must fix / Should fix / Minor), each with `file:line` and a one-line suggested fix. Five is enough to discuss and few enough to act on.
- About **60 seconds**, run once per task before committing.
- It **reports only**. You decide what to fix.

## What `pair:onboard` writes, and why it doesn't pollute their repo

- `PAIR_PROGRAMMING.md` at the repo root (unless one exists). It has the commands, the conventions found **with the evidence path for each**, a **glossary** of the project's domain terms (marking which meanings were inferred), and a separate, verbose **"Not project conventions"** section, and a **"Current plan"** section that `pair:plan` fills in for each task, so the agreed plan survives a long conversation. Every assumption, default or gap gets its own entry: what was assumed, why, what it changes, and a question for the interviewer.
- `CLAUDE.local.md` with one line, `@PAIR_PROGRAMMING.md`. Claude Code loads this file automatically next to `CLAUDE.md`, so their `CLAUDE.md` is never touched.
- Both names go in `.git/info/exclude`, git's local-only ignore list. They never appear in `git status`, the diff or your commits, and their `.gitignore` stays unchanged.

When you're done, delete both files and their two lines in `.git/info/exclude`.

## Interview flow

1. **Explore (0–5 min):** `pair:onboard`. Read the summary aloud: entry point, the conventions that matter, anything odd, and the questions it raised. Confirm the tests run.
2. **Each task:**
   1. Plan: `pair:plan` with the spec. Read its questions to the interviewer, pass the answers back, and agree on the design and the test list. It saves the agreed plan in the "Current plan" section of `PAIR_PROGRAMMING.md`, which the next steps read.
   2. Write tests: `pair:write-tests`. They should all fail, for the right reason. (For a single new component, `pair:react-component` does steps 2 and 3 in one go; for a bug, `diagnosing-bugs`.)
   3. Implement: `pair:implement`. The tests are fixed: if one looks wrong, it stops and says why. Ends with tests, typecheck and lint run once.
   4. Smoke test: `pair:smoke-test`.
   5. Review: `pair:review-diff`, then decide aloud which findings to act on.
   6. Commit: `craft:commit-message`.
3. **Last 5 min:** say the trade-offs you made and the next steps, out loud. No skill needed.

Throughout: narrate what you're doing and why, and ask the interviewer whenever a requirement isn't clear.

## Which skill for which situation

| Situation | Use | Notes |
|---|---|---|
| Session starts, project is new to you | `pair:onboard` | Once, first thing. |
| "How does X work?" or "where does Y happen?" about one part of the code | `pair:orient` | Chat answer only; good to read aloud. |
| You're given a task or spec | `pair:plan` | Three questions at most, read aloud; then design and test list. Saved to `PAIR_PROGRAMMING.md` once agreed. |
| Plan agreed, time to write tests | `pair:write-tests` | All tests first, all failing for the right reason. |
| Tests written and failing, time to build | `pair:implement` | Smallest change that passes; never edits a test. |
| Add a small feature or change one behavior, no plan needed | `mattpocock-skills:tdd` | Red → green → refactor, one test at a time. |
| Build a new React component | `pair:react-component` | Already test-first, so no need to also call `tdd`. |
| Something is broken, throwing, failing or slow | `mattpocock-skills:diagnosing-bugs` | Reproduce first, then a failing test, then the fix. |
| Tests pass, does it really work? | `pair:smoke-test` | In Chrome; pass/fail table, fixes nothing. |
| Change is done, before committing | `pair:review-diff` | You choose which findings to fix. |
| Commit | `craft:commit-message` | Matches the repo's commit style; you confirm the message. |
| Need to sketch a UI or state model quickly before committing to it | `mattpocock-skills:prototype` | Only if time allows; throw it away after. |
| Final 5 minutes | No skill | Summarize trade-offs and next steps yourself. |
| A craft or xray skill triggers on its own | Decline it | Built for your own repos, not an interview repo. |

## Before the interview

- [ ] Confirm in writing that Claude Code with custom skills and plugins is allowed.
- [ ] Install this plugin (see above) and check that `pair:onboard` is listed in a new session.
- [ ] Make sure your global `~/.claude/CLAUDE.md` doesn't push you towards wrap-ups or filing issues in repos you don't own.
- [ ] Set the permission mode to default.
- [ ] Check Claude in Chrome is connected, for `pair:smoke-test`.
