# Frontend Guidelines

This document contains common coding guidelines and best practices.


## TypeScript

- Use `.ts` for files without JSX. Use `.tsx` only when the file contains JSX.
- Do not use `any`. If a type is unknown, use `unknown` and narrow it.
- Do not use non-null assertion (`!`) without a comment explaining why it is safe.

---

## Naming

- Variable and function names must be meaningful and self-descriptive. No single-letter names outside of loop counters.
- Boolean variables and props: use `is`, `has`, or `can` prefix — `isLoading`, `hasError`, `canSubmit`.
- Event handler functions: prefix with `handle` — `handleClick`, `handleSubmit`, `handleInputChange`.
- Event handler props: prefix with `on` — `onClick`, `onSubmit`, `onInputChange`.
- Use `camelCase` for variables and functions, `PascalCase` for components and types.

---

## Functions and Methods

- Extract complex multi-condition checks into named variables or functions.
- If a function takes more than 3 parameters, group them into an options object and destructure.

---

## Comments

- Avoid obvious comments that describe what code does.
- Use comments to explain complex logic or algorithms.
- Use comments to document non-obvious workarounds.
- Use comments for TODO/FIXME notes.
- Use comments for important warnings.
- Prefer self-explanatory code over commented code.

---

## Components

- Keep components modular and reusable.
- Follow the project's component organization structure.

---

## Immutability

- Never mutate objects or arrays directly. Use spread, `Array.map`, `Array.filter`, `Array.reduce`, etc.
- Never mutate props or state objects.

---

## Exports

- Prefer **named exports** over default exports.
- Every folder with multiple files should have an `index.ts` re-export file.

---

## Environment Variables

- All `.env` usage must go through a centralized `environment.ts` (file's place depends on project structure).
- Never access `process.env` directly in components or business logic.
- The project must include an `.env.example` file listing all required variables (without values).

---

## After Making Code Changes

Run the project's lint command before marking any task as complete. Check `package.json` scripts to find the correct command (commonly `npm run lint`, `pnpm run lint:fix`, or similar). Do not skip this step.
