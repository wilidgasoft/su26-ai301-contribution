# Contribution 1: Dialog withCloseButton documentation incorrect

**Contribution Number:** 1
**Student:** Wilman Garcia
**Issue:** https://github.com/rubyevents/rubyevents/issues/1788
**Status:** Phase I Complete

---
## Why I Chose This Issue

I chose this issue because it is a well-scoped, clearly described bug labeled
"good first issue" by the maintainer. The fix involves changing a sort order in
a database query — a concept I understand well from my SQL background — even
though Ruby on Rails is a new framework for me.

I also found this issue through the official CodePath resource for good first
issues (github.com/issues?q=is:open+is:issue+label:"good+first+issue"), which
confirms it is an appropriate contribution target. The rubyevents project is
an active open source community with 550+ stars and a welcoming maintainer.

---

## Understanding the Issue

### Problem Description
On a user's profile page, both past and future events are displayed. Past events
are correctly sorted in descending order (most recent first). However, future
events are also sorted descending — meaning the furthest upcoming event appears
first instead of the soonest one. Future events should be sorted ascending so
the next upcoming event appears at the top of the list.

### Expected Behavior
Future events on the profile page should be sorted by date ascending — the
soonest upcoming event appears first.

### Current Behavior
Future events are sorted descending, same as past events, causing the furthest
future event to appear first.

### Affected Area
- The query or scope that fetches future events for the profile page, likely
  in a Rails model or controller — exact file to be confirmed during reproduction.

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
