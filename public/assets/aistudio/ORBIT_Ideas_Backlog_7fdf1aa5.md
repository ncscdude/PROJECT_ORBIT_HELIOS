# ORBIT IDEAS BACKLOG
Companion to `ORBIT_AI_Studio_Prompt_v4.md`. This file is the single place new dashboard experiences are proposed, approved, and tracked. The builder (AI Studio) reads it and builds only what is approved.

## HOW TO USE (human steps, one at a time)
1. Copy the **Entry Template** at the bottom. Fill every field. Set `Status: Proposed`.
2. Paste it under **PROPOSED**. Do not edit the core prompt.
3. When you want it built, change `Status` to `Approved`.
4. In AI Studio, attach the v4 prompt and this file. Say: `Build every Approved entry not yet marked Live, one at a time, using ADD-EXPERIENCE.`
5. After it builds, change `Status` to `Live` and add the build date.
6. To add a single idea to an app that already exists: paste only that entry and say `Add this experience using the ADD-EXPERIENCE procedure.`

## STATUS KEY
| Status | Meaning | Builder action |
|---|---|---|
| Proposed | Idea captured, not yet reviewed | Show in Experience Catalog only |
| Approved | Ready to build | Build |
| Building | In progress | Continue |
| Live | Built and registered | None |
| Parked | Not now | Show in Experience Catalog only |

## BUILDER INTAKE RULES (for AI Studio)
- Build only `Approved` entries. Process in `Priority` order (1 first), one at a time.
- If an entry is missing **Question, Persona, Metrics, View, or Done when**, build nothing for it. List what is missing at the end of your reply.
- Follow the ADD-EXPERIENCE procedure and Definition of Done in the core prompt.
- Extend types additively. Do not change the behavior of existing views except to show an extra element the entry names.
- Honor the firewall (core rule 5), fictional data, human-in-the-loop, and AI badge rules. If an entry conflicts with a core rule, the core rule wins; build the safe part and say what you skipped.
- Every metric an entry names must exist in `registry/metrics.ts`. If an entry introduces a metric not in the core registry, add it with a pure `compute` function and a tooltip.
- Update `CHANGELOG.md` and the Experience Catalog after each build.

---

# APPROVED

### IDEA-001 Pipeline Health and Stage Gate
- Status: Approved · Priority: 1 · Date built: n/a
- Persona: BMO Lead, Board Recorder
- Question: Is the board population moving from ingested to finalized fast enough to lock on time?
- Source: Metrics Catalog sections 1 and 2
- Metrics: M-101 to M-106, M-201 to M-205
- Data: `flow` on every candidate already in the model. Ensure seed has 2 records stuck over 24h, 1 failed ingest, 1 validated record with 3 errors, spread across all five stages.
- View: extends `VIEW-HeliosCommand` sub-tab A. Five-stage funnel (Ingested, Validated, Locked, Scoring, Finalized) with counts and an error badge on Validated. KPI strip for M-101 to M-105 with amber and red states. 7-day throughput chart with slope arrow. Stuck and Failed lists. **Retry** button on failed ingests (BMO Lead, logs `PIPELINE_RETRY`, simulated outcome).
- Guardrails: counts must equal the Queue's candidate counts for the selected board.
- Done when: funnel counts sum to pool size; stuck and failed items are listed and clickable to the candidate; retry writes a ledger entry.

### IDEA-002 Source Freshness Monitor
- Status: Approved · Priority: 2 · Date built: n/a
- Persona: BMO Lead, Panelist (badges only)
- Question: Which authoritative sources are stale right now, and which candidates does that affect?
- Source: Metrics Catalog section 3
- Metrics: M-301 to M-304
- Data: `SourcePipeline` for ARMS, MilPDS, MyVector, SURF; per-candidate `freshness` includes `milpds` and `myvector`.
- View: extends `VIEW-HeliosCommand` sub-tab B. One row per source: authoritative source name, last sync, age, threshold bar (amber and red markers), status pill with icon. Click a source to list affected candidates (those whose record is older than the amber threshold). Panel Workspace freshness badges use the same thresholds.
- Guardrails: thresholds come from the registry only.
- Done when: seeded amber and red cases both appear; affected-candidate list matches the badges in `VIEW-PanelReview`.

### IDEA-003 Portfolio Readiness and Staffing
- Status: Approved · Priority: 3 · Date built: n/a
- Persona: HQ Strategic Viewer, BMO Lead
- Question: Will each upcoming cycle be ready, with people assigned, before its start date?
- Source: Metrics Catalog section 6 (Dashboard Board Execution, Dashboard Readiness ILE-SLE)
- Metrics: M-501 to M-505, M-504 also feeds the Command Center
- Data: `StaffSlot` (Board President, 14 Panelist slots, Scribes) per active cycle; `CommanderAction` list with 2 overdue.
- View: extends `VIEW-HeliosCommand` sub-tab C. Cycle cards with start date, M-501 and M-502 gauges, Packet Completeness vs the 95% lock gate, staffing block (`Board President: Assigned`, `Panelists 12/14`, Scribes), Commander Actions list with overdue in amber. **Lock population** button (BMO Lead): disabled with a stated reason unless M-504 >= 95%; logs `POPULATION_LOCK`.
- Guardrails: HQ role is read-only.
- Done when: lock is blocked below 95% and allowed at or above; overdue commander actions turn the card amber.

### IDEA-004 Endorser and Stratification Insights
- Status: Approved · Priority: 4 · Date built: n/a
- Persona: Panelist, BMO Lead
- Question: Does an endorser's pattern of top-3 placements or stratification change how much weight their words deserve?
- Source: Metrics Catalog section 5 (202605 2 dash)
- Metrics: M-404, M-405
- Data: `Endorser`, `endorserId`, `endorserRank` on candidates. Seed one endorser who places many of their unit in the top 3.
- View: new section in `VIEW-CrossAnalytics`. Endorsee Scale donut (Top 3 vs Other) per endorser with selector; Stratification Overview stacked bar (DP, P, Non-Strat) for the candidate's unit; shown as context beside the candidate in `VIEW-PanelReview` Narrative lens.
- Guardrails: context only. Never changes a score or OoM. Label "Context, not a score". Seed shares must sum to 100% (see Open Question 1).
- Done when: selecting a candidate shows their endorser's Top 3 share and unit stratification mix.

### IDEA-005 Career Planner N-Box
- Status: Approved · Priority: 5 · Date built: n/a
- Persona: CFM / Career Planner, Assignments Officer
- Question: Who is key talent, who might leave, and what does it cost us if they do?
- Source: Metrics Catalog section 7 (Impact Loss)
- Metrics: M-601 to M-605
- Data: `talent` on candidates whose board is Finalized (seed for at least 20).
- View: new `VIEW-CareerPlanner` (group Talent). 3x3 N-Box grid (default Performance by Potential, configurable labels, cells like "Key Talent") with counts and click to list. Risk of Loss by Impact of Loss 3x3 heat map. Advancement Readiness timeline (Ready now, 1-2 yrs, 3+ yrs). Candidate drawer with all five values. Optional overlay chip in `VIEW-MatchLab` for Assignments Officer, off by default.
- Guardrails: firewall rule 5 applies in full. Risk of Loss and Talent Score carry the purple AI badge and "Advisory only". Not used in scoring, OoM, or default match weights. Hidden from Panelist role and from any candidate whose board is not Finalized.
- Done when: a Panelist cannot reach the view or the overlay; Career Planner sees only Finalized candidates; N-Box counts equal candidate counts.

---

# PROPOSED (roadmap only, not built until Approved)

### IDEA-006 Panelist Workload Balancer
- Status: Proposed · Priority: 6
- Persona: BMO Lead, Board Recorder
- Question: Are records spread fairly across panelists, and will scoring finish inside the execution window?
- Metrics: new: records per panelist, pace per hour, projected finish; K-06 Scoring Throughput
- View: per-panelist progress bars, pace vs plan, a **Rebalance preview** that proposes moving pending records (human approves, logged).
- Done when: projected finish updates live as scores are submitted.

### IDEA-007 Board Calendar and Conflict Detector
- Status: Proposed · Priority: 7
- Persona: BMO Lead, HQ Strategic Viewer
- Question: Do overlapping cycles collide on people, dates, or data refreshes?
- Metrics: new: overlap count, double-booked panelists, lock dates inside a source refresh window
- View: 12-cycle calendar with phase bands, conflict markers, and a what-if drag of a lock date showing downstream effect on M-504 and forecast.
- Done when: dragging a date recomputes the forecast and flags conflicts.

### IDEA-008 Succession Bench by Billet
- Status: Proposed · Priority: 8
- Persona: Assignments Officer, CFM / Career Planner
- Question: For each critical billet, who is ready now, ready in 1 to 2 years, and ready in 3 or more?
- Metrics: new: bench depth per billet; uses M-605
- View: billet rows with three readiness columns of candidate chips; thin-bench flags.
- Guardrails: firewall rule 5; informational only.

### IDEA-009 Data Conflict Finder
- Status: Proposed · Priority: 9
- Persona: BMO Lead, Board Recorder
- Question: Where do two sources disagree about the same person (for example TIG in ARMS vs MilPDS)?
- Metrics: new: conflict count per source pair, conflicts resolved
- View: conflict queue with side-by-side values, source freshness, and a **Resolve** action requiring a note (logged).
- Done when: resolving a conflict updates the candidate's Eligibility lens.

### IDEA-010 Activation and Declination Cascade Tracker
- Status: Proposed · Priority: 10
- Persona: Assignments Officer
- Question: When a candidate declines or defers, what happens downstream, and how long are cases open?
- Metrics: K-09 Activation Resolution Timeliness; new: open case age
- View: case list with aging timers, cascade preview reusing Match Lab "if X declines", and resolution actions (logged).

### IDEA-011 Executive Brief Generator
- Status: Proposed · Priority: 11
- Persona: HQ Strategic Viewer, BMO Lead
- Question: What does leadership need to know this week, in one page?
- Metrics: reads Command Center metrics only
- View: one-page brief drafted from live metrics with purple AI badge, editable, **Approve** required before Copy or Print (logged).
- Guardrails: no AI text leaves the app without a human sign-off.

### IDEA-012 Lessons Learned Library
- Status: Proposed · Priority: 12
- Persona: BMO Lead, Board Recorder
- Question: What did earlier cycles learn that applies to the cycle we are about to run?
- Metrics: K-11 Improvement Action Completion Rate; new: lessons reused
- View: searchable library of Phase 8 lessons tagged by phase and cause, with a "suggested for this cycle" list.

### IDEA-013 Panelist Rehearsal Mode
- Status: Proposed · Priority: 13
- Persona: Panelist, BMO Lead
- Question: Can a new panelist practice scoring on sample records and see how they compare before the real board?
- Metrics: new: practice agreement vs reference scores
- View: sandbox queue of fictional practice records; results never touch real scores or the official ledger (practice ledger only).
- Guardrails: clearly labeled PRACTICE on every screen.

---

# ENTRY TEMPLATE (copy, fill every field)

### IDEA-0NN Short Name
- Status: Proposed · Priority: N · Date built: n/a
- Persona: (role names from the core prompt)
- Question: (one sentence decision question this answers)
- Source: (where the idea came from: catalog section, screenshot, meeting)
- Metrics: (existing M-ids and K-ids, plus any new metric with its formula and target)
- Data: (new fields or records needed; what the seed must include, including one amber and one red case)
- View: (what the user sees and does; which view it extends or the new `VIEW-` name; ledger types for any action)
- Guardrails: (firewall, AI badge, role limits, anything that must not happen)
- Done when: (2 to 3 checks a reviewer can verify in the demo)
