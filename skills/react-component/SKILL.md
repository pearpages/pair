---
name: react-component
description: Build a new React component test-first, following the repo's own conventions, with accessibility built in. Use when asked to create, add or build a React component, widget, form, modal, list or other piece of UI, or on /pair:react-component. Writes a co-located Testing Library test first using role and label queries, then a typed, semantic, keyboard-accessible component, then the styles in whatever approach the repo already uses. Never uses inline styles or snapshot tests.
---

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
