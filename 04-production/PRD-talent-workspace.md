# PRD: Talent Workspace — Unified Recruiter Interface

*Extracted from the clickable prototype (`recruiter-workspace-prototype.html`),
its README, and the validation brief. Status: hypothesis under test, not yet
validated — see note below.*

---

## Problem

In-house professional recruiters at T-Mobile operate across at least six
disconnected tools — LinkedIn Recruiter, Workday Recruiting ATS,
HiredScore, Paradox, Accurate, and Power BI — to move a single candidate
from review to hire. This fragmentation is the subject of the project's
**working hypothesis**: that consolidating the core workflow into one
interface would reduce context-switching friction and, over time, Time to
Hire (current baseline: **55 days, confirmed company-wide**, not
recruiter-segment-specific).

Two pieces of real recruiter feedback anchor this problem:

> "Search and filtering feels like it was designed by someone who's never
> actually sourced a candidate."

> "They say it's impossible to use without workarounds."

**Important caveat:** this hypothesis has *not* been validated. The
project's own kill switch (≥60% of test recruiters completing the full
flow unaided) has not been run against real recruiters — only an internal
role-play simulation exists so far. This PRD describes what has been
*built to test* the hypothesis, not a confirmed solution.

## Users & jobs

**Primary user:** In-house professional recruiter, managing multiple open
requisitions concurrently, responsible for the full candidate lifecycle
from sourcing through hire.

**Job to be done:** "When I need to move a candidate through hiring, I
want to see everything relevant to that decision in one place, so I don't
have to reconstruct context by checking multiple separate systems."

Secondary jobs surfaced during role-play feedback, not yet built for:
- "When I triage a new batch of applicants, I want to act on several
  candidates at once, not one at a time."
- "When I check in on my workload, I want to know which REQs need
  attention today, not just descriptive counts."

## Scope

**In scope (built in the prototype):**
- REQ queue dashboard with search and sort
- REQ detail view with candidate grid
- Cross-REQ candidate list with search and sort
- Single-candidate workflow: Review → Screen → Schedule → Offer → Hire
- AI-assisted suggestions (match confidence, screening questions,
  interview slots, offer band) requiring explicit recruiter confirmation
- Recruiter Insights (aggregate stage/source/priority breakdowns)
- Previous/Next navigation between candidates

**Out of scope (explicitly, per the original validation brief and
subsequent build decisions):**
- Sourcing and reporting screens
- Full offer-approval-chain logic (comp bands, manager sign-off,
  counter-offers) — Offer/Hire are summary/decision states only
- Bulk candidate actions
- Any real integration with LinkedIn Recruiter, Workday, or the other
  four tools in the stack — this prototype does not read from or write
  to any of them. **Confirmed direction: this sits on top of Workday**
  as a layer, not a replacement (see Open Questions #1 — LinkedIn
  Recruiter's relationship to this interface is still unconfirmed).
- Prescriptive/predictive insights (e.g. flagging at-risk REQs) —
  Insights is descriptive only

## Requirements

| Requirement | Priority | Acceptance criteria |
|---|---|---|
| Recruiter can view all open REQs with search and sort | P0 | Dashboard lists REQ ID, title, manager, candidate count, days open, priority; search filters by ID/title/manager; clicking a column header sorts ascending/descending |
| Recruiter can view REQ detail without leaving the interface | P0 | Clicking a REQ shows description, must-haves, manager, days open, priority, REQ avg TTH, and a candidate grid |
| REQ detail candidate list handles zero-data state | P0 | If a REQ has no candidates, the grid shows "No records found" and the "View all" button is hidden |
| Recruiter can view all candidates across REQs with search and sort | P0 | Candidates tab lists name, REQ, stage, source, days in stage; same search/sort behavior as Dashboard |
| Recruiter can complete a candidate's workflow in one interface | P0 | Review → Screen → Schedule → Offer → Hire all accessible as tabs without navigating away |
| AI suggestions never act without confirmation | P0 | Every AI-assisted action (match, questions, slots, offer band) requires a recruiter click before state changes |
| Hire completion is measurable and strict | P0 | Completion badge appears only when Review, Screen, Schedule, and Offer are all confirmed — not merely visited |
| **Hire action is gated by prior stage completion** | **P0 — not yet met (confirmed direction: hard block)** | Currently the "Confirm hire" button can be clicked regardless of prior stage status. **Confirmed: hard-block the button** — disable "Confirm hire" until Review, Screen, Schedule, and Offer are all confirmed. |
| AI agent overlay within the single pane of glass (hybrid AI-assisted + agentic) | P2 — future phase, not in current build | Defined by stakeholder as: "a hybrid approach where an AI agent is overlaid within the SPoG." **Confirmed: still requires confirmation for every action — no autonomous actions, even within the agent overlay.** No acceptance criteria beyond that constraint defined yet — needs its own scoping pass before this becomes buildable. |
| Recruiter can move between candidates without returning to the list | P1 | Previous/Next buttons in the candidate workspace step through the last-viewed Candidates list order; disabled at list boundaries |
| Recruiter can see aggregate pipeline stats | P1 | Insights screen shows open REQ count, candidates in pipeline, avg days open, avg TTH, and breakdowns by stage/source/priority, computed live from current data |
| Context bar is scoped to the workspace only | P2 | Candidate name/stats/stage appear only on the Candidate Workspace screen, hidden on Dashboard, Candidates, REQ Detail, Insights |
| Insights are prescriptive, not just descriptive | P2 — not built | Flags at-risk REQs (e.g. trending past typical TTH) or stalled candidates, not just counts |
| Bulk candidate actions | P2 — not built | Recruiter can apply an action (e.g. send screen) to multiple selected candidates from the Candidates tab |

## Data & events

**Current state — all mocked, none real:**
- 12 REQs and 24 candidates, hand-authored as illustrative sample data
  in the prototype's JavaScript, not sourced from Workday or LinkedIn
  Recruiter
- Comp figures, requirement lists, and REQ descriptions are invented
- No data persistence: refreshing the browser resets all state (stage
  completions, selected candidate, everything)
- No analytics/telemetry events are actually emitted — there is no
  event logging in this prototype

**What a real build would need (not implemented here):**
- **Confirmed: two separate integration paths, not one uniform approach.**
  - **Workday** — a staging database will be built, syncing **in real
    time**. This is the system of record for REQ and candidate data.
  - **LinkedIn Recruiter** — connected via **API** directly, not through
    the staging database. Sync cadence (real-time vs. polling), scope of
    fields pulled, and auth/rate-limit handling are not yet specified
    (see Open Questions #10).
  - **Neither integration path currently covers** the other four tools
    in the stack (HiredScore, Paradox, Accurate, Power BI) — see Open
    Questions #11. If left unaddressed, recruiters may still need to
    leave the interface for data sourced from those systems.
- Event instrumentation for at least: `candidate_opened`,
  `stage_confirmed` (per stage), `ai_suggestion_shown`,
  `ai_suggestion_confirmed`, `hire_confirmed`, `hire_blocked` (new —
  needed now that the hire gate is a hard block, to measure how often
  recruiters attempt it early), `search_performed`, `sort_applied` —
  needed to actually measure the kill switch (unaided task completion)
  and later, real Time to Hire impact
- If Workday and LinkedIn data have different sync guarantees
  (real-time vs. API-dependent), consider whether the UI needs to
  surface data freshness per field/source, so a recruiter isn't misled
  into treating stale API-sourced data as equally current to real-time
  Workday data

## Open questions

### Resolved

1. ~~Is this meant to replace LinkedIn Recruiter/Workday, or sit on top
   of them?~~ **Resolved: sits on top of Workday.** LinkedIn Recruiter
   will be connected via API (see #9) — full "replace vs. sit on top of"
   framing wasn't specified for LinkedIn specifically, but the
   integration mechanism is now confirmed.
2. ~~Is the underlying SaaS data even accessible?~~ **Resolved in
   principle: yes, via a staging database** for Workday, syncing in
   **real time** (see #8). LinkedIn Recruiter connects via API rather
   than the staging database (see #9).
3. ~~What should happen when a recruiter clicks Hire early?~~
   **Confirmed: hard block.** The "Confirm hire" button will be
   disabled until Review, Screen, Schedule, and Offer are all
   confirmed. Reflected in Requirements table above.
4. ~~What does "agentic experience" mean beyond AI-assisted?~~
   **Resolved: a hybrid approach, with an AI agent overlaid within the
   single pane of glass** — distinct from, and going beyond, the
   AI-assisted (suggest + confirm) pattern already built. Added to
   Requirements as a future-phase item.
5. ~~What counts as "completion" for the kill switch when navigation is
   non-linear?~~ **Confirmed: "Hire reached with all prior stages
   confirmed" is the correct definition.**
6. ~~Is 55 days the right baseline, and is it recruiter-specific or
   company-wide?~~ **Confirmed: company-wide.**
7. ~~How does the AI agent overlay interact with the existing "confirm
   before acting" requirement?~~ **Confirmed: AI suggestions never act
   without confirmation — no exception for the agent overlay.** The
   hybrid agentic experience (#4) does not introduce autonomous action;
   it remains suggest + confirm. This removes the earlier conflict risk
   between the two requirements.
8. ~~What is the staging database's scope and sync model?~~
   **Resolved: Workday, real-time sync.** Scope is Workday specifically
   — the staging database is not confirmed to cover the other SaaS
   tools (HiredScore, Paradox, Accurate, Power BI). LinkedIn Recruiter
   is handled separately via API (#9), not through this staging
   database.
9. ~~What is LinkedIn Recruiter's relationship to this interface?~~
   **Resolved: connected via API**, distinct from the Workday staging
   database approach. Sync model (real-time vs. polling), scope of data
   pulled, and auth/rate-limit considerations are not yet specified —
   see new open question below.

### Still open

10. **(New) What is the LinkedIn Recruiter API's sync model and scope?**
    Workday syncs in real time via a staging database; LinkedIn
    Recruiter connects via API instead, but real-time vs. polling
    cadence, which data fields are pulled, and authentication/rate-limit
    handling are unspecified. Two different integration approaches
    for the two source systems is a legitimate design choice, but it
    means the data-freshness guarantee may differ between Workday-backed
    and LinkedIn-backed fields on screen — worth deciding whether that
    difference should be visible to the recruiter (e.g. a "last synced"
    timestamp per data source) or abstracted away entirely.
11. **(New) Does the staging database or LinkedIn API cover the other
    four tools in the stack** (HiredScore, Paradox, Accurate, Power BI)?
    Neither integration decision so far mentions them. If they remain
    outside both, recruiters may still need to leave the interface for
    data sourced from those systems — which would undercut the "single
    pane of glass" premise for exactly the friction points not yet
    addressed.
