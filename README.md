# Contribution 1: Dialog withCloseButton documentation incorrect

**Contribution Number:** 1
**Student:** Wilman Garcia
**Issue:** https://github.com/rubyevents/rubyevents/issues/1788
**Status:** Phase II Complete

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


## Phase II
## Reproduction Process

### Environment Setup
Forked and cloned the rubyevents repository. The project requires Ruby 4.0.1
and Node.js 22.15.1. I had to upgrade Ruby from 2.6 to 4.0.1 using rbenv and
install Node 22.15.1 using nvm. Dependencies were installed with `bundle install`
and `yarn install`. The app was running locally with `bin/setup` and `bin/dev`
at http://localhost:3000 within about 30 minutes.

No major blockers — the CONTRIBUTING.md had clear requirements and the setup
scripts worked as expected on macOS M1.

### Steps to Reproduce
1. Go to https://www.rubyevents.org/profiles/marcoroth/events (production)
2. Scroll to the **Future Events** section
3. Observe that the furthest upcoming event appears first (e.g. November before July)
4. **Expected:** Soonest upcoming event appears first (ascending order)
5. **Actual:** Furthest future event appears first (descending order)
6. Note that Past Events are correctly sorted descending — this should stay unchanged

### Reproduction Evidence
- **Branch:** https://github.com/wilidgasoft/rubyevents/tree/fix-issue-1788
- **Root cause confirmed in:** `app/views/profiles/_events.html.erb` line 1
- **My findings:** Both future and past events were using `.sort_by(&:sort_date).reverse`,
  which forces descending order on both. Removing `.reverse` from the future_events
  line fixes the sort order. Verified in Rails console — query returns
  `ORDER BY events.start_date ASC` after the fix.

---

## Solution Approach

### Analysis
The bug is in `app/views/profiles/_events.html.erb`. The first two lines are:

```ruby
<% future_events = events.upcoming.sort_by(&:sort_date).reverse %>
<% past_events = (events.to_a - future_events).sort_by(&:sort_date).reverse %>
```

Both use `.reverse`, forcing descending order on future events when they should
be ascending. The `Event` model already has the correct scopes in
`app/models/event.rb` (`scope :upcoming` uses `order(start_date: :asc)`), but
the view overrides this with `.reverse`.

### Proposed Solution
Remove `.reverse` from the `future_events` line only. The `past_events` line
keeps `.reverse` to maintain correct descending order for past events.

### Implementation Plan
1. In `app/views/profiles/_events.html.erb`, change line 1 from:
```ruby
   <% future_events = events.upcoming.sort_by(&:sort_date).reverse %>
```
   to:
```ruby
   <% future_events = events.upcoming.sort_by(&:sort_date) %>
```
2. Leave `past_events` line unchanged
3. Verify in Rails console that future events return ascending order
4. Verify visually in the browser on a profile with future events
5. Open PR referencing issue #1788
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
