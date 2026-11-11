---
name: review-diff
description: Review the current uncommitted change with a fresh pair of eyes and return at most five issues, ranked by severity, in plain English. Use before committing in a pair programming session, when asked to review the diff, check the change, or "look over this before I commit", or on /pair:review-diff. Runs one sub-agent with a clean context that reads git diff itself, so it is not biased by the conversation. Reports only; fixes nothing.
---

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
