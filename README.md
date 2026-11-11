# pair

Skills for a 45–50 minute AI-assisted pair programming interview built around day-to-day tasks, packaged as a local Claude Code plugin called `pair`.

The idea: reuse existing skills where they already fit, and build only the gaps.

| Skill | What it does |
|---|---|
| `pair:onboard` | Analyzes the whole project, runs the tests once, and writes a local `PAIR_PROGRAMMING.md` without touching any tracked file. |
| `pair:orient` | Answers a question about one part of the code in chat, with `file:line` links. Writes nothing. |
| `pair:review-diff` | A fresh sub-agent reviews `git diff` and returns at most five issues, ranked by severity. |
| `pair:react-component` | Builds a React component test-first, in the repo's own conventions, with accessibility built in. |

All four explain things in plain English: no unexplained jargon, and a short definition the first time a term appears.

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

## Grilling is capped

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

- `PAIR_PROGRAMMING.md` at the repo root (unless one exists). It has the commands, the conventions found **with the evidence path for each**, and a separate, verbose **"Not project conventions"** section. Every assumption, default or gap gets its own entry: what was assumed, why, what it changes, and a question for the interviewer.
- `CLAUDE.local.md` with one line, `@PAIR_PROGRAMMING.md`. Claude Code loads this file automatically next to `CLAUDE.md`, so their `CLAUDE.md` is never touched.
- Both names go in `.git/info/exclude`, git's local-only ignore list. They never appear in `git status`, the diff or your commits, and their `.gitignore` stays unchanged.

When you're done, delete both files and their two lines in `.git/info/exclude`.

## Interview flow

1. **Orientation (0–5 min):** `pair:onboard`. Read the summary aloud, share the questions it raised, and confirm the tests run.
2. **Each task:**
   1. Clarify: ask the interviewer. Use capped `grilling` if the task is fuzzy.
   2. Test first: `tdd` for a feature, `diagnosing-bugs` for a bug, `pair:react-component` for a new component.
   3. Implement until green.
   4. Review: `pair:review-diff`, then decide aloud which findings to act on.
   5. Commit: `craft:commit-message`.
3. **Last 5 min:** say the trade-offs you made and the next steps, out loud. No skill needed.

Throughout: narrate what you're doing and why, and ask the interviewer whenever a requirement isn't clear.

## Which skill for which situation

| Situation | Use | Notes |
|---|---|---|
| Session starts, project is new to you | `pair:onboard` | Once, first thing. |
| "How does X work?" or "where does Y happen?" about one part of the code | `pair:orient` | Chat answer only; good to read aloud. |
| The task is vague or has several readings | Ask the interviewer, or capped `mattpocock-skills:grilling` | One round, three questions or fewer, read aloud. |
| Add a feature or change behavior | `mattpocock-skills:tdd` | Red → green → refactor. |
| Build a new React component | `pair:react-component` | Already test-first, so no need to also call `tdd`. |
| Something is broken, throwing, failing or slow | `mattpocock-skills:diagnosing-bugs` | Reproduce first, then a failing test, then the fix. |
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
