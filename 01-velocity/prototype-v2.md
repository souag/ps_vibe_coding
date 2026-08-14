# Prototype v2: Two Upgrade Passes (Lab 2)

> Module 1 · Velocity. Same prototype, two surgical upgrade passes, closing the gap from toy to tool.

## Weakest section (from v1)

Where does it look like a toy? Where would a VP of Design refuse to open it? Where does the logic feel fake? Pick the one spot that costs you the most credibility.

v1 was a single static screen for one hardcoded requisition (TM-88213). There was no navigation, nothing was clickable, and the "stage tracker" showing progress was just decorative HTML — it couldn't advance, and there was no second requisition to compare it against. A VP of Design would open it, click around for five seconds, find nothing responds, and conclude it's a screenshot with a <div> wrapper, not a tool. The credibility gap: no workflow, no state, no proof the underlying logic (gating, multi-role candidates, approval) was ever actually modeled — it just looked plausible in one frozen moment.

_____

## Upgrade paths run (pick two)

- [ ] Design Match
- [X] Add Interactivity
- [X] Surgical Refinement
- [ ] Existing Product Track

## v2 build

- v2 shareable link: None. Still a downloadable HTML artifact from this conversation, not deployed to a public URL — no link exists to put here.
Before → after: v1: one static screen, one hardcoded req, zero interactivity, decorative stage tracker. v4 (the "v2" checkpoint for this lab): a role-switchable app (Hiring Manager / Recruiter) with a real status-gated workflow — reqs move through Draft → Pending Approval → Approved/Rejected via working Approve/Reject buttons, and only Approved reqs become visible to the Recruiter role, enforced in the data model rather than the UI. Four approved reqs, each with 10+ real candidate applications (one candidate can hold multiple applications across different reqs). A working "+ Add Candidate" flow, a searchable/sortable/filterable Candidates view grouped by requisition, and a Reports tab with computed pipeline-health flags and auto-generated performance insights. The "New Requisition" flow is agentic: it calls the live Claude API to parse a hiring manager's free-text description into structured requisition fields before the human reviews and submits.
What each pass changed:
- Add Interactivity pass: Turned the static mockup into a real state machine — role switching, functional Approve/Reject actions that move a req's visibility live, a working Add Candidate modal, and search/sort/stage-filter controls on the Candidates tab. This is what makes it clickable rather than a picture of an app.
- Surgical Refinement pass: Fixed the specific credibility gaps a reviewer would poke at first — replaced the single hand-picked requisition with four reqs across different pipeline stages (including one genuinely empty pipeline and one at Offer), padded thin candidate pools to a realistic 10+ per req instead of 2–4, and replaced hardcoded "insight" text in Reports with logic actually computed from the data (funnel counts, conversion rates, health flags) — while explicitly labeling the health thresholds as arbitrary defaults, not validated business rules, so the refinement doesn't overstate its own credibility.

## Show & Swap read, round 2

_A NEW partner, a blind read. What landed differently from v1?_

_____
