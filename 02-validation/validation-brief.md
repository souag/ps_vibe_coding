# Validation Brief

**Scenario:** Scenario 1 - Custom Application Tracking System (ATS) for recruiters

## 1 · Hypothesis
Recruiters can complete their core daily workflow (sourcing → screening → scheduling → offer status tracking) inside a single interface without swiveling between separate SaaS tools, and this reduces friction and task time versus their current multi-tool workflow.

## 2 · Risk type & kill switch
- **Risk type:** Usability
- **Build to test it:** Clickable mockup
- **Kill switch:** If fewer than 60% of recruiters complete the core task flow without confusion, backtracking, or requesting a second tool — do not proceed to build.

## 3 · Success criteria
- Task completion rate on core flows (target: >60%)
- Time-on-task vs. recruiter's self-reported current workflow time
- SUS (System Usability Scale) score or equivalent
- Number of times a recruiter says "I'd need to check [other tool] for this"

- Not testable via mockup — defer to post-launch measurement:
- Recruiter CSAT +10% (Yr1) / +15% (Yr2)
- Time to Hire −10%
- "Agentic experience" — undefined term; needs a concrete definition (e.g., does this mean AI-suggested next actions, auto-drafted outreach, automated scheduling?) before it can be a success criterion at all


## 4 · Problem Framework
1. Goal — Reduce time to hire; improve recruiter experience
2. Problem — Recruiter workflow information is fragmented across multiple SaaS systems (ATS, CRM, scheduling, sourcing tools — name them if known), forcing manual context-switching
3. Context — Prototype tests whether a consolidated view improves task efficiency and surfaces the right insights at the point of decision
4. Constraints — Underlying systems are third-party SaaS, not owned; integration depends on API/data access availability from each vendor — this is a build-phase risk, not just a constraint, and should probably get its own validation step before committing engineering resources
5. Success (post-launch, not mockup) — CSAT +10% Yr1 / +15% Yr2; Time to Hire −10%
6. Explore — Rapid prototyping to test feasibility and usability before committing to build

---
_Module 2 · Vibe Coding Certification · frame before you build._
