# AI301 Open Source Contributions — Wilman Garcia (@wilidgasoft)
 
## Progress Overview
 
| Contribution | Repo | Status | Phase |
|---|---|---|---|
| 1 | rubyevents/rubyevents | ✅ Merged | Phase IV Complete |
| 2 | Rinava/MarkSight | 🔄 In Progress | Phase I — Issue Selection |
 
---
 
# Contribution 1: Dialog withCloseButton documentation incorrect
 
**Contribution Number:** 1
**Student:** Wilman Garcia
**Issue:** https://github.com/rubyevents/rubyevents/issues/1788
**Status:** Phase IV Complete — Merged ✅
 
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
 
The fix was verified through two methods:
 
**Manual verification in production:**
Compared the live site at https://www.rubyevents.org/profiles/marcoroth/events
before and after the fix. Future events were displaying in descending order
(November before July), confirming the bug. The expected behavior is ascending
order (soonest upcoming event first).
 
**Rails console verification locally:**
```ruby
user = User.joins(event_participations: :event)
  .where("events.start_date >= ?", Date.current)
  .first
events = user.participated_events.includes(:series)
future_events = events.upcoming.sort_by(&:sort_date)
future_events.map { |e| "#{e.name} - #{e.start_date}" }
# => ["RubyConf 2026 - 2026-07-14"]
```
The query confirms `ORDER BY events.start_date ASC` after the fix.
Past events sort order was verified unchanged — still descending.
 
The project uses minitest. No existing tests cover the profile events sort
order directly. The fix is a one-line view change with clear before/after
behavior verified visually and through the Rails console.
 
---
 
## Implementation Notes
 
The fix required changing a single line in `app/views/profiles/_events.html.erb`:
 
```ruby
# Before
<% future_events = events.upcoming.sort_by(&:sort_date).reverse %>
 
# After
<% future_events = events.upcoming.sort_by(&:sort_date) %>
```
 
The root cause was that both future and past events used `.reverse`, forcing
descending order on both sections. The `Event` model already had the correct
scope defined (`scope :upcoming` uses `order(start_date: :asc)`), but the
view was overriding it with `.reverse`.
 
Removing `.reverse` from the `future_events` line aligns the view with the
model's intended sort order. The `past_events` line retains `.reverse` to
keep correct descending order for past events.
 
The linter (`bin/lint`) passed with no errors — ERB, Ruby, JS, and YAML
all clean before opening the PR.
 
---
 
## Pull Request
**PR Link:** https://github.com/rubyevents/rubyevents/pull/1816
**Status:** Merged ✅
 
---
 
## Learnings & Reflections
 
This contribution taught me more than I expected from a one-line fix.
 
The most valuable lesson was learning how to navigate an unfamiliar codebase
methodically — using grep to trace the code path, reading the model scopes,
and finding where the view was overriding the intended behavior. The bug was
not where I initially looked (the controller), but in the view layer.
 
Setting up a Ruby on Rails environment from scratch on M1 was also a new
experience for me. I had never worked with Ruby before, but the concepts
translated directly from my PHP and Python background. Having to upgrade
from Ruby 2.6 to 4.0.1 using rbenv and configure Node 22.15.1 with nvm
taught me how to manage multiple runtime versions on the same machine.
 
I also learned the importance of running the linter before opening a PR
(`bin/lint`) and keeping the branch up to date with upstream via rebase
before submitting. These are small habits that make a real difference in
a collaborative open source project.
 
The process of finding the right issue took longer than the fix itself —
I evaluated several projects (Mantine, Kanboard, Flarum, InvoiceShelf,
laravel/vs-code-extension) before landing on rubyevents. That search
process taught me how to read a project's health signals: activity level,
number of core contributors, issue response time, and whether bugs are
being picked up by external contributors or only by the core team.
 
---
 
## Resources Used
 
- https://github.com/rubyevents/rubyevents/issues/1788 — Original issue
- https://github.com/rubyevents/rubyevents/pull/1816 — Merged PR
- https://github.com/rubyevents/rubyevents/blob/main/CONTRIBUTING.md — Contribution guidelines
- https://github.com/rbenv/rbenv — Ruby version management
- https://github.com/nvm-sh/nvm — Node version management
---
---
 
# Contribution 2: Add a confirmation before clearing the document
 
**Contribution Number:** 2
**Student:** Wilman Garcia
**Issue:** https://github.com/Rinava/MarkSight/issues/25
**Status:** Phase I — Issue Selection (Confirmed, awaiting maintainer go-ahead)
 
---
 
## Fork
 
**Fork link:** https://github.com/wilidgasoft/MarkSight
 
---
 
## Issue Eligibility Verification
 
Verified as of July 5, 2026: the issue is **open**, **unassigned**, has **no
linked pull request**, and carries **no blocking labels** (e.g. no "in
progress" or "claimed" tag). It is a live, claimable issue.
 
---
 
## Why I Chose This Issue
 
I picked this issue because it matches skills I already use daily — I work in
React, TypeScript, and shadcn/ui on my own product (AgentDesk), so I can move
past environment setup quickly and spend more of my time on the actual
contribution and on responding well to review feedback, which is the harder
skill this course is trying to teach.
 
My learning goal for this cycle is different from my first contribution.
Contribution 1 taught me how to trace a bug through an unfamiliar codebase.
For this one, my goal is to practice designing a small piece of UI/UX
behavior (a confirmation flow) the way a maintainer would want it — matching
existing patterns (shadcn/ui's Dialog primitive) instead of inventing my own,
and covering accessibility (keyboard focus, Esc to cancel) as a first-class
requirement rather than an afterthought.
 
I also understand exactly what's being asked: right now, clicking "Clear" in
MarkSight wipes the entire document instantly, with no way to double-check
the action first. That matters because there's no server-side backup — if a
user misclicks, their only safety net is noticing and clicking "Undo" in a
toast notification before it disappears, which is easy to miss. The fix is to
add a confirmation dialog in front of the existing clear logic, so an
accidental click no longer costs someone their work.
 
**Feasibility checklist applied:**
- [x] Labeled "good first issue" and `enhancement` by the maintainer
- [x] Repo has recent commit/PR activity (57 commits, 3 open PRs at time of review, live production app)
- [x] CONTRIBUTING.md and CODE_OF_CONDUCT.md exist with clear setup steps
- [x] Issue has a clear, verifiable expected vs. actual behavior, with exact file/line references and acceptance criteria written by the maintainer
- [x] Scoped small: a single confirmation-gate added in front of an existing, working `handleClear` function
**Candidates evaluated before landing here:**
- `CESARBR/knot-gateway-webui#8` — rejected: issue open 6+ years with zero activity, repo has only Dependabot commits, stack is legacy AngularJS (not my current stack)
- `Rinava/MarkSight#25` — **selected**: fresh issue (opened Jun 30, 2026), active single-maintainer repo cultivating outside contributions, stack matches my daily work
**Repo health signals:**
- Live production app: marksight.laramateo.com
- Stack: Next.js 15 (App Router), React 19, TypeScript, Tailwind v4, shadcn/ui, CodeMirror, vitest
- Maintainer (Lara Mateo / @Rinava) actively labels and writes detailed "good first issue" tickets, and explicitly invites new contributors in the README
- 3 pull requests open at time of review — active contributor traffic, not a dead repo
---
 
## Understanding the Issue
 
### Problem Summary (in my own words)
MarkSight's editor has a "Clear" button that wipes out everything you've
written the instant you click it — there's no "are you sure?" step. The only
safety net is a toast notification with an "Undo" link that disappears after
a few seconds, so a single misclick (easy to do, since Clear sits right next
to other toolbar buttons) can permanently cost someone their work. This
matters because MarkSight stores documents only in the browser's
`localStorage` with no server-side backup, so there's nothing else protecting
the user's content. The fix the maintainer proposed is straightforward: gate
the existing clear logic behind a confirmation dialog, reusing a UI component
the project already has, so nothing breaks — it just adds one extra,
reversible step before anything is deleted.
 
### Expected Behavior
Clicking Clear should show a confirmation dialog ("Clear the document? This
removes all content.") with Cancel / Clear actions. The document should only
be cleared if the user confirms.
 
### Current Behavior
Clicking Clear runs `handleClear` immediately with no confirmation, clearing
the document unconditionally.
 
### Affected Area
- `src/app/page.tsx`: the Clear `Button` (lines 225–232) and its handler
  `handleClear` (lines 170–181)
- Existing UI primitive available: `src/components/ui/dialog.tsx` (Radix Dialog
  via shadcn/ui) — to be used for the confirmation, per maintainer's
  implementation notes
- Existing behavior to preserve on confirm: store previous value for Undo,
  `setValue("")`, `trackClear()` analytics call, existing success toast
### Acceptance Criteria (from issue)
- [ ] Clicking Clear shows a confirmation prompt; document only clears after confirming
- [ ] Cancelling leaves the document unchanged
- [ ] On confirm: previous value stored for Undo, content cleared, `trackClear()` fired, success toast shown
- [ ] Confirmation is keyboard-accessible (focusable, Esc to cancel)
- [ ] No regression to the Reset button
---
 
## Phase II — Reproduction Process
 
_(Not yet started — next steps below.)_
 
---
 
## Phase III — Build
 
_(Not started. Will begin after Phase I is confirmed and the local environment
is reproducing the bug.)_
 
---
 
## Phase IV — Submit and Iterate
 
_(Not started. Will open PR after implementation and testing are complete.)_
 
---
 
## Next Steps
 
1. Comment on the issue to request assignment (see draft comment below)
2. Fork and clone `Rinava/MarkSight`
3. Run `npm install` and `npm run dev` to stand up the local environment
4. Reproduce current behavior: click Clear, confirm it wipes the document with no prompt
5. Implement the confirmation dialog using the existing `src/components/ui/dialog.tsx`
6. Verify all acceptance criteria, including keyboard accessibility
7. Run `npm run lint` and existing vitest suite before opening the PR
8. Update this README with reproduction evidence and implementation notes
---
 
## Resources Used
 
- https://github.com/Rinava/MarkSight/issues/25 — Original issue
- https://github.com/Rinava/MarkSight — Repository
- https://github.com/Rinava/MarkSight/blob/main/CONTRIBUTING.md — Contribution guidelines
- https://github.com/Rinava — Maintainer profile (Lara Mateo), reviewed to confirm active involvement and responsiveness to contributors
- https://github.com/Rinava/MarkSight/pulls — Open PRs at time of selection, reviewed as evidence of active contributor traffic
- CodePath good-first-issue search: `github.com/issues?q=is:open+is:issue+label:"good+first+issue"`
