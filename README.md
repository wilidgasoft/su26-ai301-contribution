# Contribution 1: Dialog withCloseButton documentation incorrect

**Contribution Number:** 1
**Student:** Wilman Garcia
**Issue:** https://github.com/mantinedev/mantine/issues/8951
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it involves TypeScript and React, technologies I use
actively in my own SaaS projects. The bug is clearly described and the author
already points to the exact file and line where the problem exists, which makes
it a great starting point for my first open source contribution.

I am also interested in learning how a large, well-maintained component library
like Mantine handles default props and JSDoc documentation internally. Fixing
this issue will help me understand those patterns, which I can apply to my own
projects.

---

## Understanding the Issue

### Problem Description

The `Dialog` component has a `withCloseButton` prop. The JSDoc comment says its
default value is `true`, but the component never actually sets that default.
This means if you don't pass the prop explicitly, the close button never renders,
even though the documentation says it should appear by default.

### Expected Behavior

When a user renders `<Dialog>` without passing `withCloseButton`, the close
button should appear by default, as the JSDoc comment states `@default true`.

### Current Behavior

The close button never renders unless `withCloseButton={true}` is passed
explicitly, because the default value is missing in the component's defaultProps
or default parameters.

### Affected Components

- `packages/@mantine/core/src/components/Dialog/Dialog.tsx` — line 34

---

## Reproduction Process

### Environment Setup

[To be completed in Phase II]

### Steps to Reproduce

1. Create a basic Mantine app with Vite
2. Render `<Dialog opened={true}>Content</Dialog>` without passing `withCloseButton`
3. Observe that the close button does not appear

### Reproduction Evidence

- **Commit showing reproduction:** [To be added in Phase II]
- **Screenshots/logs:** [To be added in Phase II]
- **My findings:** [To be added in Phase II]

---

## Solution Approach

### Analysis

[To be completed in Phase II]

### Proposed Solution

[To be completed in Phase II]

### Implementation Plan

[To be completed in Phase II]

---

## Testing Strategy

[To be completed in Phase III]

---

## Implementation Notes

[To be completed in Phase III]

---

## Pull Request

**PR Link:** [To be submitted in Phase IV]

---

## Learnings & Reflections

[To be completed at the end of the program]

---

## Resources Used

- https://mantine.dev/core/dialog/
- https://github.com/mantinedev/mantine/issues/8949
