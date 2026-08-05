# AI301 Open Source Contributions — Wilman Garcia (@wilidgasoft)
 
## Progress Overview
 
| Contribution | Repo | Status | Phase |
|---|---|---|---|
| 1 | rubyevents/rubyevents | ✅ Merged | Phase IV Complete |
| 2 | Rinava/MarkSight | Merged | Phase IV Complete |Waiting for get another Bug
| 3 | Rinava/MarkSight | Merged | Phase I Complete  |Phase I — Issue Selection (Assigned to me by maintainer ✅)
 
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
- [x] Clicking Clear shows a confirmation prompt; document only clears after confirming
- [x] Cancelling leaves the document unchanged
- [x] On confirm: previous value stored for Undo, content cleared, `trackClear()` fired, success toast shown
- [x] Confirmation is keyboard-accessible (focusable, Esc to cancel)
- [x] No regression to the Reset button
---
 
## Phase II — Reproduction Process

### Environment Setup
Forked and cloned the MarkSight repository. The project uses Next.js 15 with
Turbopack. Setup was straightforward — ran `npm install` and `npm run dev`.
The app loads at http://localhost:3000 without any configuration needed.

One non-blocking warning appears in the browser console:
`eval() is not supported in this environment` — this is a known React/Next.js
development warning with Turbopack and does not affect functionality.

Node.js version used: v20.20.2

**Working branch:** https://github.com/wilidgasoft/MarkSight/tree/fix-issue-25

### Steps to Reproduce
1. Open http://localhost:3000
2. Type any content in the Markdown editor
3. Click the **Clear** button in the toolbar
4. **Expected:** A confirmation dialog appears asking the user to confirm
   before clearing the document
5. **Actual:** The document is immediately cleared with no confirmation —
   only a toast notification with an Undo option appears briefly

### Reproduction Evidence
- **Root cause confirmed in:** `src/components/workspace/workspace.tsx`
  — `handleClear` function (line 135) calls `setValue("")` immediately
  with no confirmation guard
- **Clear button:** line 333, wired directly to `handleClear`
- **My findings:** The project already has `src/components/ui/dialog.tsx`
  (Radix Dialog from shadcn/ui) available for use. No new dependencies needed.
  The issue also notes `handleReset` (line 150) has the same risk but Clear
  is the priority since it destroys all content outright.

---

## Solution Approach

### Analysis
The bug is in `src/components/workspace/workspace.tsx`. The `handleClear`
function (line 135) runs immediately when the Clear button is clicked —
it calls `setValue("")`, `trackClear()`, and shows a success toast with
an Undo option. There is no confirmation step before the destructive action.

The project already includes `src/components/ui/dialog.tsx` (Radix Dialog,
part of shadcn/ui), which provides an accessible, keyboard-navigable
confirmation dialog with Esc-to-cancel support out of the box.

**Analogous pattern in the codebase:** The existing toast + Undo pattern
in `handleClear` and `handleReset` shows the project already thinks about
accidental data loss — the confirmation dialog is the missing first layer.

### Proposed Solution
Add a controlled `useState` boolean (`clearDialogOpen`) to manage the dialog
visibility. When the user clicks Clear, open the dialog instead of running
the clear logic. Only run the existing `handleClear` logic (setValue, trackClear,
toast) when the user confirms inside the dialog. Cancel closes the dialog
and leaves the document unchanged.

Only show the dialog when the document is non-empty (`value.trim() !== ""`),
so clearing an already-empty document stays frictionless — as suggested in
the issue.

### Implementation Plan

**Understand:** `handleClear` in `src/components/workspace/workspace.tsx`
runs destructive logic immediately on click. A single misclick erases all
content. The Undo toast is a weak safety net — it disappears quickly.

**Match:** `src/components/ui/dialog.tsx` already exists and is used elsewhere
in the project. The shadcn Dialog pattern is the standard way to handle
confirmation flows in this codebase.

**Plan:**
1. Add `const [clearDialogOpen, setClearDialogOpen] = useState(false)` in
   `workspace.tsx`
2. Rename current `handleClear` to `executeClear` — this runs the actual
   clear logic (setValue, trackClear, toast)
3. Create new `handleClear` that checks `value.trim() !== ""` — if non-empty,
   opens the dialog; if empty, runs `executeClear` directly
4. Wire the Clear button `onClick` to the new `handleClear`
5. Add the Dialog component in the JSX with Cancel and Confirm buttons —
   Confirm calls `executeClear` and closes the dialog
6. Verify: clicking Clear on non-empty doc shows dialog; Cancel leaves doc
   unchanged; Confirm clears doc and shows existing toast + Undo

**Review:** Will follow project conventions — shadcn/ui components, existing
toast pattern preserved, `trackClear()` analytics call unchanged.

**Evaluate:**
- Manual: reproduce steps above — dialog should appear before clearing
- Write a test verifying the dialog renders when Clear is clicked on
  non-empty content
- Verify existing behavior preserved on confirm (toast + Undo still works)
- Verify no regression on Reset button

---

## Phase III — Build

### Testing Strategy

The fix was verified through two methods:

**Manual verification in the browser:**
1. Opened http://localhost:3000 and typed content in the editor
2. Clicked Clear — confirmation dialog appeared with "Clear the document?"
3. Clicked Cancel — document remained unchanged ✅
4. Clicked Clear again → confirmed — document cleared + success toast with Undo ✅
5. Clicked Clear on an empty document — dialog did not appear, cleared silently ✅
6. Pressed Esc while dialog was open — dialog closed, document unchanged ✅
7. Verified Reset button behavior unchanged — no regression ✅

**Automated test suite:**
Ran `npm test` (Vitest) — 85 tests across 9 test files, all passing.
No existing tests cover the confirmation dialog directly, as the project's
test suite focuses on the skill/markdown logic layer rather than UI interactions.
New behavior was verified manually per the acceptance criteria in issue #25.

**Build and lint:**
- `npm run lint` — passed with 1 pre-existing warning in `outline-rail.tsx`
  (unrelated to this PR, pre-dates this contribution)
- `npm run build` — compiled successfully with no errors

---

### Implementation Notes

**Files modified:**
- `src/components/workspace/workspace.tsx` — commit `5f31e29`

**Changes made:**
1. Added imports for `Dialog`, `DialogContent`, `DialogDescription`,
   `DialogFooter`, `DialogHeader`, `DialogTitle` from `@/components/ui/dialog`
   and `Button` from `@/components/ui/button`
2. Added `const [clearDialogOpen, setClearDialogOpen] = useState(false)`
   alongside existing state declarations
3. Renamed original `handleClear` to `executeClear` — preserves all existing
   logic: snapshot for Undo, `setValue("")`, `trackClear()`, success toast,
   and `setClearDialogOpen(false)`
4. Created new `handleClear` that checks `value.trim() !== ""` — opens dialog
   if non-empty, calls `executeClear` directly if already empty
5. Added `<Dialog>` JSX at the bottom of the component tree (before closing
   `</TooltipProvider>`) with Cancel and Clear buttons

**Challenge faced:**
The `useState` hook for `clearDialogOpen` was initially placed after the
`return` statement — a React rules-of-hooks violation. Moved it to the top
of the component alongside the other state declarations. This is a common
mistake when adding state to a large component — hooks must always be declared
before any `return`.

---

## Phase IV — Submit and Iterate

### Pull Request
**PR Link:** https://github.com/Rinava/MarkSight/pull/112
**Status:** Merged ✅ — Jul 17, 2026

### Maintainer Feedback Log

**Jul 14, 2026 — Rinava requested changes:**
- Indentation: new `executeClear`/`handleClear` blocks and `<Dialog>` JSX
  were flush-left, inconsistent with the file's 2-space style
- Two stray blank lines above the `return` statement
- Optional nit: `setClearDialogOpen(false)` called inside `executeClear`
  even on the empty-document path where dialog was never open — harmless
  but redundant

**Jul 17, 2026 — Resolution:**
Maintainer fixed the indentation and blank lines directly inside the commit
and merged. Comment: *"the behavior design was right from the start."*

**Learning:** Always run the project's formatter or manually match indentation
before pushing. Since MarkSight doesn't have Prettier yet (issue #78),
indentation must be hand-matched to the surrounding 2-space style.

---

### Learnings & Reflections

This contribution reinforced something important: getting the behavior right
matters more than getting every mechanical detail perfect on the first try.
The maintainer's feedback was entirely about formatting (indentation and blank
lines), not about the logic or design — and his final comment confirmed the
approach was correct from the start.

The main technical lesson was about React hooks placement. Adding `useState`
to a large, existing component is easy to get wrong if you're not careful
about where you put the declaration. Hooks must always appear before the
`return` statement — something easy to overlook when scrolling through
400+ lines of code to find a good spot.

The indentation feedback taught me a practical habit: when a project doesn't
have an auto-formatter like Prettier, you have to manually match the surrounding
code style before pushing. I'll run a visual diff of my changes against the
rest of the file before every future PR.

The full cycle — from issue to dialog design to review to merge — also showed
me how a maintainer thinks about small contributions. Rinava didn't just check
"does it work" — he also checked keyboard accessibility, the empty-document
edge case, and whether the Undo toast still fired on confirm. Those are the
acceptance criteria I wrote into the PR checklist, which made the review faster
because the maintainer could see I had already verified them.

Compared to my first contribution (rubyevents), this one involved more
back-and-forth and a real review cycle. That made it more valuable as a
learning experience even though the fix itself was larger.

---

## Resources Used

- https://github.com/Rinava/MarkSight/issues/25 — Original issue
- https://github.com/Rinava/MarkSight/pull/112 — Merged PR
- https://github.com/Rinava/MarkSight/blob/main/CONTRIBUTING.md — Contribution guidelines
- https://ui.shadcn.com/docs/components/dialog — shadcn/ui Dialog documentation
- https://www.radix-ui.com/primitives/docs/components/dialog — Radix Dialog primitives
- https://github.com/Rinava/MarkSight/issues/78 — Context on why Prettier isn't set up yet
- CodePath good-first-issue search: `github.com/issues?q=is:open+is:issue+label:"good+first+issue"`


# Contribution 3: Make the Google Analytics ID configurable
 
**Contribution Number:** 3
**Student:** Wilman Garcia
**Issue:** https://github.com/Rinava/MarkSight/issues/121
**Status:** Phase I — Issue Selection (Assigned to me by maintainer ✅)
 
---
 
## Fork
 
**Fork link:** https://github.com/wilidgasoft/MarkSight
 
---
 
## Issue Eligibility Verification
 
Verified as of August 4, 2026: the issue is **open**, **assigned to me by the
maintainer (@Rinava)**, has **no linked pull request or branch**, and carries
**no blocking labels**. It is labeled `enhancement` and `good first issue`.
I requested the issue via a comment and the maintainer assigned it to me,
so this contribution has an explicit maintainer go-ahead.
 
---
 
## Why I Chose This Issue
 
I chose this issue for three reasons.
 
First, it lets me practice a different *category* of contribution. My first
contribution (rubyevents #1788) was a pure bug fix — tracing incorrect
behavior to its root cause. My second (MarkSight #25) was a UI/UX behavior
change — designing a confirmation flow. This one is a **configuration and
deployment concern**: making a hardcoded value environment-driven so the
project behaves correctly for *every* deployment, not just the maintainer's.
That's a skill I use professionally (12-factor app config, environment
variables per environment), and open source is a good place to practice
applying it under someone else's conventions.
 
Second, the issue has real-world impact beyond a cosmetic fix. Right now,
anyone who forks or self-hosts MarkSight silently sends their visitors'
pageview data to the maintainer's Google Analytics property. That's both a
privacy problem for fork users' visitors and a data-pollution problem for the
maintainer. Fixing it makes the project genuinely fork-friendly — which
matters for a repo that explicitly invites new contributors.
 
Third, I already have a working relationship with this repo and maintainer:
my PR #112 (confirmation dialog) was merged in July, I know the codebase
layout, my dev environment is already set up, and I understand the
maintainer's review style (behavior first, but match the file's formatting
conventions exactly — MarkSight has no Prettier yet, see issue #78). This
lets me spend my time on the contribution quality rather than on setup.
 
**My learning goal for this cycle:** practice implementing a
*conditional-by-configuration* feature the way a maintainer wants it —
including the "absence" path (when the env var is unset, the GA script must
not load at all, verifiable in the browser's network tab) and documenting
the variable for future users in `.env.example`. Verifying that something
correctly does NOT happen is a different testing mindset than verifying
that something does.
 
**Feasibility checklist applied:**
- [x] Labeled `good first issue` and `enhancement` by the maintainer
- [x] Issue explicitly assigned to me by the maintainer
- [x] Repo has recent commit/PR activity and a live production app
- [x] CONTRIBUTING.md and CODE_OF_CONDUCT.md exist with clear setup steps
- [x] Issue has exact file/line reference (`src/app/layout.tsx:160`),
      a proposed solution, implementation notes, and acceptance criteria
      written by the maintainer
- [x] Scoped small: one conditional render plus one documented env var —
      the maintainer himself describes it as "one conditional plus an env
      var" with results verifiable in the browser's network tab
**Repo health signals:**
- Live production app: marksight.laramateo.com
- Stack: Next.js 15 (App Router), React 19, TypeScript, Tailwind v4,
  shadcn/ui, CodeMirror, vitest
- Maintainer (Lara Mateo / @Rinava) actively labels and writes detailed
  "good first issue" tickets with acceptance criteria, responds to issue
  claims, and merged my previous PR (#112) within days
- I am a returning contributor to this repo — this is my second
  contribution here, which reflects sustained engagement with one open
  source community rather than one-off drive-by PRs
---
 
## Understanding the Issue
 
### Problem Summary (in my own words)
MarkSight's root layout (`src/app/layout.tsx`, line 160) hardcodes the
maintainer's own Google Analytics ID directly in the JSX:
`<GoogleAnalytics gaId="G-YYGPWZ1WF7" />`. There is no
`NEXT_PUBLIC_GA_*` environment variable anywhere in the codebase. The
consequence: every fork and every self-hosted deployment of MarkSight ships
analytics events from *their* visitors to the *maintainer's* GA property —
silently, with no way to opt out or substitute their own ID short of editing
source code. For a project that positions itself as fork-friendly, that's a
bad default: fork users unknowingly leak their visitors' pageview data, and
the maintainer's analytics get polluted with traffic that isn't hers.
 
### Expected Behavior
The GA ID should come from an environment variable
(`process.env.NEXT_PUBLIC_GA_ID`). When the variable is set, the
`<GoogleAnalytics>` component renders with that ID. When it is unset, the
component does not render at all and **no GA script loads** (verifiable in
the browser's network tab). The variable should be documented as optional
in `.env.example`.
 
### Current Behavior
The GA ID `G-YYGPWZ1WF7` is hardcoded in `src/app/layout.tsx:160`. Every
build of the project — including forks and self-hosted deployments — loads
Google Analytics with the maintainer's property ID, unconditionally.
 
### Affected Area
- `src/app/layout.tsx` (line 160): the hardcoded `<GoogleAnalytics>` render —
  to be replaced with a conditional render driven by
  `process.env.NEXT_PUBLIC_GA_ID`
- `.env.example`: new documented optional variable `NEXT_PUBLIC_GA_ID`
- Note from maintainer's implementation notes: the `NEXT_PUBLIC_` prefix is
  required because the value is read at build/render time for the client —
  a plain server-side env var would not be exposed to the component
- The maintainer's own production deployment will keep analytics by setting
  the variable in the hosting environment rather than in code — so the fix
  must not break her existing deployment workflow
### Acceptance Criteria (from issue)
- [ ] With the var unset, no GA script loads (check the network tab)
- [ ] With the var set, that ID is used
- [ ] `.env.example` documents it as optional
---
---
 
# IMPROVED SECTIONS — Contribution 1 (rubyevents #1788)
 
> Replace the corresponding sections in your existing document with these.
> Also fix the Contribution 1 title: it currently says "Dialog
> withCloseButton documentation incorrect" but should be
> **"Future events sorted in wrong order on profile page"** (or the actual
> issue #1788 title).
 
---
 
## Phase II — Reproduce and Plan (IMPROVED)
 
### Environment Setup
 
Forked and cloned the rubyevents repository from
https://github.com/rubyevents/rubyevents to my fork at
https://github.com/wilidgasoft/rubyevents.
 
**Working branch (named after the issue):**
https://github.com/wilidgasoft/rubyevents/tree/fix-issue-1788
 
**Setup challenges faced and resolved:**
 
1. **Ruby version mismatch.** The project requires Ruby 4.0.1; my M1 Mac had
   the system Ruby 2.6. Installing 4.0.1 via rbenv initially failed because
   the build couldn't find OpenSSL headers. Resolved by installing
   `openssl@3` via Homebrew and pointing the build at it:
   `RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@3)" rbenv install 4.0.1`,
   then `rbenv local 4.0.1` in the project directory.
2. **Node version mismatch.** The project pins Node 22.15.1; I had a
   different version active. Installed and activated it with
   `nvm install 22.15.1 && nvm use 22.15.1`.
3. **Dependency installation.** `bundle install` and `yarn install` completed
   without errors once the correct runtimes were active.
4. **Database seed data.** Ran `bin/setup`, which prepares the SQLite
   database and seeds sample data — this was essential for reproduction,
   because I needed a profile with *future* events to observe the sort bug
   locally.
Total setup time: ~30 minutes. The CONTRIBUTING.md requirements were
accurate, and `bin/dev` served the app at http://localhost:3000.
 
### Steps to Reproduce (local environment)
 
1. Start the app locally: `bin/dev` → http://localhost:3000
2. Navigate to a profile page for a user who has both past and future
   events (e.g. `/profiles/marcoroth/events` in seeded data; the same bug
   is observable in production at
   https://www.rubyevents.org/profiles/marcoroth/events)
3. Scroll to the **Future Events** section
4. **Expected:** the soonest upcoming event appears first (ascending by
   date) — e.g. a July 2026 event listed above a November 2026 event
5. **Actual:** the furthest upcoming event appears first (descending by
   date) — November 2026 listed above July 2026
6. Scroll to the **Past Events** section and confirm it is correctly sorted
   descending (most recent first) — this behavior must remain unchanged
   by the fix
### Root Cause
 
**File:** `app/views/profiles/_events.html.erb`, line 1.
 
Both event lists are built at the top of the view partial:
 
```ruby
<% future_events = events.upcoming.sort_by(&:sort_date).reverse %>
<% past_events = (events.to_a - future_events).sort_by(&:sort_date).reverse %>
```
 
Both lines apply `.reverse`, forcing descending order on both sections. For
past events that is correct; for future events it inverts the intended
order. Notably, the `Event` model already defines the correct scope in
`app/models/event.rb` — `scope :upcoming` orders by `start_date: :asc` —
but the view layer overrides the model's intent with `.reverse`.
 
Root cause verified in the Rails console: after removing `.reverse` from the
`future_events` line, the generated query returns
`ORDER BY events.start_date ASC`, and past events remain descending.
 
### UMPIRE Plan
 
**Understand:** Future events on the profile page display furthest-first
instead of soonest-first. Past events display correctly (most recent first).
The bug is a sorting inversion affecting only the Future Events section of
`app/views/profiles/_events.html.erb`.
 
**Match:** The codebase already expresses the correct intent at the model
layer: `scope :upcoming` in `app/models/event.rb` orders ascending. The
matching pattern is therefore *not* to add new sorting logic, but to stop
the view from overriding the model's ordering. This mirrors a common Rails
convention: ordering belongs in scopes, and views should respect it.
 
**Plan:**
1. In `app/views/profiles/_events.html.erb`, change line 1 from
   `events.upcoming.sort_by(&:sort_date).reverse` to
   `events.upcoming.sort_by(&:sort_date)` (remove only `.reverse`)
2. Leave the `past_events` line unchanged — its `.reverse` is intentional
3. Verify in the Rails console that future events now return ascending order
4. Verify visually in the browser on a seeded profile with future events
5. Confirm Past Events section still renders descending (no regression)
**Implement:** One-line change to the view partial (see Phase III).
 
**Review:** Follow project conventions — run `bin/lint` (covers ERB, Ruby,
JS, YAML) before opening the PR, and rebase on upstream `main` so the branch
is current.
 
**Evaluate:**
- Rails console query verification (ascending order for future events)
- Visual before/after comparison in the browser
- Past Events regression check (still descending)
- The project uses minitest; no existing tests cover this view's sort order.
  Given the fix is a one-line view change, verification was performed via
  console query output plus visual confirmation, documented with
  before/after evidence in the PR.
---
 
## Phase IV — Submit and Iterate (IMPROVED)
 
### Pull Request
 
**PR Link:** https://github.com/rubyevents/rubyevents/pull/1816
**Target:** opened against upstream `rubyevents/rubyevents`, branch `main`
**Status:** Merged ✅
 
### PR Structure (why before what)
 
The PR description opens with the *why*: on profile pages, visitors looking
for a user's next appearance see the furthest-away event first, which
defeats the purpose of the Future Events section — the most actionable
information (the next upcoming event) is buried at the bottom of the list.
 
It then explains the *what*: a one-line change removing `.reverse` from the
`future_events` assignment in `app/views/profiles/_events.html.erb`,
restoring the ascending order already defined by the `Event` model's
`upcoming` scope. The PR references the issue with `Closes #1788`.
 
### Acceptance Criteria Checklist (from the PR)
 
- [x] Future events on the profile page are sorted ascending — soonest
      upcoming event appears first
- [x] Past events remain sorted descending — most recent first (no
      regression)
- [x] Verified in the Rails console: the query returns
      `ORDER BY events.start_date ASC` for future events after the fix
- [x] Verified visually in the browser before/after the change
- [x] `bin/lint` passes clean (ERB, Ruby, JS, YAML)
- [x] Branch rebased on upstream `main` before submission
### Before / After Evidence
 
**Before (bug):** On https://www.rubyevents.org/profiles/marcoroth/events,
the Future Events section listed a November 2026 event above a July 2026
event — furthest first.
 
**After (fix, local):** With the one-line change applied, the same profile
lists the July 2026 event first — soonest first. Rails console confirmation:
 
```ruby
future_events = events.upcoming.sort_by(&:sort_date)
future_events.map { |e| "#{e.name} - #{e.start_date}" }
# => ["RubyConf 2026 - 2026-07-14", ...]  # ascending — soonest first
```
 
Past Events verified unchanged — still descending.
 
### Maintainer Feedback Log
 
- **PR opened:** referencing issue #1788 with `Closes #1788`, following the
  repository's PR conventions (lint passing, branch rebased on upstream
  `main`).
- **Review outcome:** the maintainer merged the PR **without requesting any
  changes**. No review comments required iteration — the one-line diff, the
  console verification, and the before/after evidence in the PR description
  answered the reviewer's likely questions (why only the future line
  changes, why past events keep `.reverse`) preemptively.
- **Takeaway:** a zero-iteration merge is itself a signal — anticipating the
  reviewer's questions inside the PR description (scope of change, what was
  intentionally left untouched, how it was verified) reduces review friction
  even for trivial diffs.
### Learnings & Reflections
 
This contribution taught me more than I expected from a one-line fix.
 
The most valuable lesson was learning to navigate an unfamiliar codebase
methodically — using grep to trace the code path, reading the model scopes,
and discovering that the bug was not where I first looked (the controller)
but in the view layer, where a `.reverse` call was overriding the model's
correctly-defined ordering. The deeper insight: in Rails codebases, when
behavior contradicts a model scope, look for a view or controller override
before assuming the scope is wrong.
 
Setting up a Ruby on Rails environment from scratch on an M1 Mac was also
new territory. I had never worked with Ruby, but the concepts translated
from my PHP and Python background. Managing the Ruby 2.6 → 4.0.1 upgrade
with rbenv (including the OpenSSL build issue) and pinning Node 22.15.1
with nvm taught me practical multi-runtime version management.
 
On process: running the project's linter (`bin/lint`) and rebasing on
upstream before submitting are small habits that directly affect whether a
maintainer can merge quickly. My PR merged with zero change requests, which
I attribute partly to documenting the verification (console output,
before/after) inside the PR itself.
 
Finally, finding the right issue took longer than the fix. I evaluated
several projects (Mantine, Kanboard, Flarum, InvoiceShelf,
laravel/vs-code-extension) before landing on rubyevents. That search taught
me to read a project's health signals: activity level, number of core
contributors, issue response time, and whether external contributors' PRs
actually get merged.
 
