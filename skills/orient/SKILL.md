---
name: orient
description: Answer a question about one specific part of the code, in plain English, during a pair programming session. Use when asked how a specific feature, file, module, flow or function works, where something happens, what calls what, or "walk me through X", or on /pair:orient. Traces the path through the code with file:line links and says what to watch out for. Writes no files. For the whole project at the start, use onboard instead.
---

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
