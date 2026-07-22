---
description: "Use when writing CSS, creating or editing CSS Module files, or applying styles to React components. Covers CSS Modules usage, scoping, and forbidden patterns."
applyTo: "**/*.{css,module.css,scss,module.scss}"
---

# CSS / Styling Rules

## Scoping

- Use **CSS Modules** for all component styles. File naming: `ComponentName.module.css`.
- Never write global class names that could collide across components. Exception: `:global()` syntax is allowed specifically to override styles of third-party library components (e.g. `antd`, `react-select`, `mui`) when there is no other way to target their internals.
- Alternatives (styled-components, Tailwind) are acceptable if already established in the project — do not mix approaches within the same project.

## Naming

- Use `camelCase` for all CSS Module class names: `.submitButton`, `.errorMessage`, `.isLoading`.
- Names must reflect **meaning and purpose**, not visual appearance. Use `.loading`, not `.bigYellowSpinnyThing`. Use `.errorText`, not `.redBoldText`.

## Media Queries

- Place `@media` rules **immediately after the base styles of the same selector**, nested inside it. Do not collect all media queries at the bottom of the file.

  ```css
  .card {
    padding: 1rem;

    @media (max-width: 768px) {
      padding: 0.5rem;
    }
  }
  ```

  This keeps all states of an element (base + responsive) visible in one screen without scrolling.

## Forbidden Patterns

- Never use `!important`. If specificity is a problem, restructure the selectors.

## Layout

- Prefer relative units (`rem`, `em`, `%`) over fixed `px` widths where possible.
