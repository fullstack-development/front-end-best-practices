---
description: "Use when writing, editing, or reviewing React components. Covers component structure, useState, useEffect, props declaration, children, and JSX patterns."
applyTo: "**/*.tsx"
---

# React Component Rules

## useState

- Name setter with `set` + same name as state variable: `const [isOpen, setIsOpen] = useState(false)`.
- Never: `const [isOpen, openModal] = useState(false)`.

## useEffect

- Use **one `useEffect` per concern**. Never combine unrelated side effects into a single hook.
- Place all `useEffect` calls at the **bottom of the component body**, just before `return`. Exception: a render-function that depends on complex conditional logic may appear after `useEffect`, directly before `return`.

## JSX

- Do not assign JSX to variables before `return`. Write expressions directly inside the `return` statement.
  Exception: extract to a variable only when the condition is deeply nested or the inline expression significantly hurts readability. In that case, use a named function (not a variable) and call it inside `return`.
- When rendering lists, always provide an explicit `key` prop. Never use array index as `key` for dynamic/reorderable lists.

## Constants and Helpers

- Never declare constants, lookup objects, or pure helper functions inside the component body. Move them outside the component, into a `constants.ts`, `helpers.ts`, or `utils.ts` file co-located with the component, or to `src/shared/` (place depends on project structure) if reused elsewhere.

## Props

- Declare the `Props` type in the **same file**, directly above the component function.
- Name it exactly `Props` (not `MyComponentProps`).
- Do not export `Props` unless it is consumed by another module. If you must export it, rename on export: `export { type Props as MyComponentProps }`.
- Optional props must have default values.

### Props Order (in the type declaration)

1. Required component/React-node props
2. Optional component/React-node props
3. Required value props
4. Optional value props
5. Required function props
6. Optional function props

### `children` prop

- If `children` is required, declare it explicitly in `Props` as required.
- If `children` is optional, use `PropsWithChildren<Props>` from React.

### Event handler props

- Name event handler props with `on` prefix: `onClick`, `onSubmit`, `onInputChange`.
- Use method syntax in the type: `onClick(): void` rather than `onClick: () => void`.

## HTML and Accessibility

- Use semantic HTML elements in JSX: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`, `<button>`, `<form>`, etc. Do not use `<div>` or `<span>` where a semantic element exists.
- Placeholder links (no real URL yet) must use `href="#mock-address"`.
- Links to external sites must have `target="_blank"`.
- Every link with `target="_blank"` must also have `rel="noreferrer"` to prevent the opened page from accessing `window.opener` and to avoid sending the referrer header.

## Exports

- Use **named exports** for components. Avoid default exports.
- Every folder with multiple component files must have an `index.ts` that re-exports the public API.
