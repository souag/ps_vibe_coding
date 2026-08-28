# Integration Plan — T-Mobile Unified Recruiter Workspace

## 1. Current status

| Area | State |
| --- | --- |
| Frontend | TanStack Start v1 + React 19 + Tailwind v4. Features grouped by stage: `review`, `screen`, `schedule-interview`, `offer`, `hire`, `pipeline`, `candidate-workspace`. |
| Data | `requisitions`, `candidates`, `candidate_pipeline_state`, `invites` exist in the backend and are seeded (3 reqs, 21 candidates). |
| Reads | `src/lib/pipeline.functions.ts` fetches reqs + candidates server-side via the **admin** client (RLS bypassed). `src/lib/invites.functions.ts` reads/writes invites via admin too. |
| Writes | Only invites persist. All stage progression, notes, scorecards, offers and hires live in React `useState` and vanish on refresh. |
| Auth | None. No sign-in route, no `_authenticated/` subtree, no user identity. Header shows a literal "Sourabh A.". |
| RLS | Enabled on `invites` (sender-scoped) and `candidate_pipeline_state` (user-scoped); `candidates` is authenticated-read-only; `requisitions` is public-read. Policies are effectively unused because the server uses the admin key. |

## 2. Data audit — hardcoded values

| Value | Location | Should come from |
| --- | --- | --- |
| `BASELINE_TIME_TO_HIRE = 55` | `features/pipeline/data/stages.ts`, used in all five stage screens | `org_metrics` row per org/req |
| Fallback `days_to_hire` in Hire screen | `features/hire/HireScreen.tsx` | computed from `req.opened_at` → `hire.hired_at` |
| Recruiter identity "Sourabh A. · In-house Recruiting" | `features/candidate-workspace/components/TalentWorkspace.tsx` | `profiles` row for `auth.uid()` |
| Stage counts in the stepper / pipeline rail | local `useState` | aggregate query over `candidate_pipeline_state` |
| Panel members, interview slots | `data/reqs.ts` JSON blobs | `requisitions.panel` / `requisitions.slots` (already in DB, still read from fixtures in places) |
| Scorecard, must-have checks, offer comp, approvals | component state | new persisted tables (below) |
| AI suggestion copy and flags | static strings in candidates fixture | `candidates.ai_flag` + a `suggestions` table later |

## 3. Schema to add

```
profiles            id (auth user), full_name, title, org, avatar_url
user_roles          user_id, role app_role('recruiter','hiring_manager','coordinator','admin')
org_metrics         org, metric_key, metric_value, effective_from   -- baseline TTH lives here
scorecards          candidate_id, user_id, must_have_checks bool[], note, rating, submitted_at
interviews          candidate_id, req_id, slot_start, slot_end, panel jsonb, status
offers              candidate_id, comp, equity, start_date, status('draft','pending','approved','sent','accepted','declined'), approvals jsonb
hires               candidate_id, req_id, hired_at, days_to_hire, handoff_complete
activity_log        user_id, candidate_id, action, payload jsonb, created_at
```

`candidate_pipeline_state` stays as the single source of truth for the current stage; the new tables carry per-stage artifacts. Every table gets `created_at`/`updated_at` + the shared update trigger, plus GRANTs for `authenticated` and `service_role`.

## 4. RLS rules

- `profiles` — read for any authenticated user; write only own row.
- `user_roles` — read own rows; writes only `service_role`. Role checks go through a `has_role(uuid, app_role)` security-definer function (never read roles from the client to decide access).
- `scorecards`, `interviews`, `offers`, `hires`, `activity_log` — insert/update where `user_id = auth.uid()`; select where `user_id = auth.uid()` **or** `has_role(auth.uid(),'hiring_manager')` for the reqs they're on.
- `offers` — comp fields readable only by recruiter owner + `hiring_manager` + `admin`; coordinators get interviews only.
- `candidates` — keep authenticated-read; drop the admin-client read path and query as the signed-in user via `requireSupabaseAuth`.
- `requisitions` — keep public read (no PII).
- `invites` — unchanged, sender-scoped.

## 5. Three sequential build prompts

**Prompt 1 — Schema & persistence**
> Add `profiles`, `org_metrics`, `scorecards`, `interviews`, `offers`, `hires`, `activity_log` with timestamps, update triggers and GRANTs. Seed `org_metrics` with the T-Mobile 55-day baseline. Replace every hardcoded metric in the stage screens with values read from the database, and persist stage progression, notes, scorecards, slot selection, offer and hire actions to their tables instead of local state.

**Prompt 2 — Auth & RLS**
> Add Google sign-in and an `/auth` route, move the workspace under `_authenticated/`, create `user_roles` + `has_role()`, and enable RLS on every new table with owner- and role-scoped policies. Replace the admin-client reads in `pipeline.functions.ts` and `invites.functions.ts` with `requireSupabaseAuth` server functions so RLS actually applies. Gate compensation, background-check and approval UI behind role checks, and replace the hardcoded header identity with the signed-in profile.

**Prompt 3 — Edge cases & resilience**
> Add `pendingComponent`, `errorComponent` and `notFoundComponent` to every route, skeleton states for the pipeline rail and candidate panel, empty states for zero reqs / zero candidates / over-filtered lists, optimistic mutations with rollback and toast retry, offline detection, and removal of all non-null assertions on query data.

## 6. Edge cases to handle

- Zero requisitions, zero candidates in a req, filters matching nothing.
- Deep link `/candidate/:id` to a missing or out-of-req candidate → notFound, not a crash.
- Two recruiters acting on the same candidate → last-write-wins with an `updated_at` conflict warning.
- Stage regression (Offer → Screen) and re-entry into a completed stage.
- Rejected / withdrawn candidates removed from active counts but kept in history.
- Interview slot already taken, or panel member unavailable.
- Offer declined after approval; hire without an accepted offer (block it).
- Invite send failure — the UI must not mark the stage complete.
- Session expiry mid-flow → redirect to `/auth`, restore the intended candidate URL.
- Missing optional fields (no `booked_slot`, no `offer_comp`, no `days_to_hire`).

## 7. Stress tests to run

1. **Load** — seed 2,000 candidates across 20 reqs; measure pipeline rail render and filter latency (<200 ms interaction).
2. **Slow network** — Chrome 3G throttle + 3 s artificial server-fn delay; confirm skeletons appear and nothing freezes.
3. **Offline** — go offline mid-offer; confirm the mutation queues or fails loudly with retry, never silently.
4. **RLS negative tests** — sign in as recruiter B and attempt to read recruiter A's scorecards, offers and invites directly through the Data API; every one must return zero rows.
5. **Role matrix** — walk the full flow as coordinator, recruiter and hiring manager; verify comp/background fields hide correctly.
6. **Concurrency** — two sessions advance the same candidate simultaneously; state must converge, not duplicate.
7. **Refresh at each stage** — reload at Review, Screen, Schedule, Offer, Hire; state must survive.
8. **Kill-switch rehearsal** — 5 recruiters complete Review → Hire unaided; target ≥60% completion with no reference to LinkedIn Recruiter or Workday.
