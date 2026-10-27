# ROLE
You are a senior front-end engineer and UX designer. Build **Project ORBIT (Operational Repository for Boards & Talent)**, a single-page concept demo for a USSF Enterprise Talent Management Office / Board Management Office (ETMO / BMO). ORBIT unifies four perspectives that today live apart:

- **BMO perspective:** is the board pipeline moving, ready, and risk-managed?
- **HQ strategic perspective:** is the portfolio of upcoming cycles ready to start?
- **Panel and assignments perspective:** who is strong, who fits which billet, and what changes if assumptions change?
- **CFM / career planner perspective (post-board):** who is key talent, who might leave, and what does it cost if they do?

The product's value is the link between them: every operational KPI drills down to candidates, and every candidate decision rolls up to operational KPIs. All data is fictional. No backend.

# FILES YOU WILL RECEIVE
1. This prompt (core spec).
2. `ORBIT_Ideas_Backlog.md` (dashboard experience backlog and intake rules).

Precedence: this prompt wins on any conflict. Build every backlog entry marked `Status: Approved`. Entries marked `Proposed` or `Parked` are roadmap only: list them in the Experience Catalog, do not build them. If the backlog is not attached, build Stages 1 to 3 and ask for it before Stage 4.

# DELIVERY STAGES (stop after each stage, give a 3-line summary, wait for "continue")
1. Foundation: types, seed, metrics registry, experience registry, DELTA ledger, shell, roles. Then `VIEW-CommandCenter`, `VIEW-CandidateQueue`, `VIEW-PanelReview`.
2. `VIEW-HeliosCommand` (core), `VIEW-ScoringAudit`, `VIEW-OpsRegisters`, `VIEW-DeltaLedger`.
3. `VIEW-MatchLab`, `VIEW-CrossAnalytics`.
4. Approved backlog experiences, one at a time, using the ADD-EXPERIENCE procedure below.
5. Polish, accessibility, empty states, Experience Catalog drawer, Demo Guide.

# STACK
React 18 + TypeScript + Tailwind CSS + Lucide Icons + Recharts. Tab state only (no router). `useReducer` + Context. Persist to `localStorage` in try/catch. "Reset Demo" in footer. Heavy computation (matching, sensitivity) in `useMemo` or a Web Worker so the UI never freezes.

# DATA SAFETY (mandatory)
- Every person, DoD ID, and PAS code is **invented**. PAS format `XX#XX#X#` (8 characters). Units are generic (`USSPACECOM J3`, `SpOC Delta 9`, `SSC`, `HQSF S3`). Use the field shapes from the metrics catalog (for example name, grade, DAFSC, TIG `4y 2m`, TAFCSD `14y 7m`) but never real names or real PAS codes.
- Footer: `FICTIONAL DATA · NOT FOR OPERATIONAL USE`. Banner (11px mono): `CAPABILITY CONCEPT BRIEF · USSF S1/ETM/TMEB · ORBIT / HELIOS · HUMAN-IN-THE-LOOP ENFORCED`.
- Matching and scoring use **job-relevant attributes only** (skills, performance, experience, education, SOI preferences, eligibility). Never protected characteristics. State this in the "How matching works" popover.

# DESIGN
| Token | Hex | Use |
|---|---|---|
| bg | #05080f | page |
| surface | #0a1020 | cards |
| border | #1e2d45 | 1px |
| cyan | #00d4ff | primary, active tab |
| green | #00ff88 | on track, PASS |
| amber | #ffb800 | at risk, WARN |
| red | #ff4757 | delayed, FAIL, outlier |
| purple | #a855f7 | AI-generated only |
| phase-blue | #2B6CB0 | workflow phases 1 to 4 |
| phase-purple | #5B4282 | workflow phases 5 to 8 |

Inter for UI, JetBrains Mono for IDs (`PDO-*`, `BRD-*`, `BIL-*`, `DELTA-*`, `PIPE-*`, `AGENT-*`, `CHAT-SNAPSHOT-*`, `EXP-*`, `M-*`), PAS, DAFSC, timestamps. Dense, no decorative animation. Body text 13px min. Color never the only signal. Visible focus rings, full keyboard access, WCAG AA. Target 1280px, usable at 1024px. Every AI-generated element gets a purple **AI** badge plus "Advisory only".

# PERSPECTIVES AND ROLES
Top-right chip with dropdown. Actions outside a role are disabled with a tooltip naming who can do them. Actor name and role are stamped on every ledger entry.

| Role | Perspective | Default landing | Can edit |
|---|---|---|---|
| Panelist | Panel | `VIEW-CandidateQueue` | own scores, flags, snapshot sign-off |
| Board Recorder | BMO | `VIEW-OpsRegisters` | milestones, risks, actions |
| BMO Lead | BMO | `VIEW-CommandCenter` | KPI cells, rollback, routing commit, population lock |
| Assignments Officer | Assignments | `VIEW-MatchLab` | matches, overrides, scenario promotion |
| HQ Strategic Viewer | HQ | `VIEW-CommandCenter` | read-only, print |
| CFM / Career Planner | Post-board | `VIEW-CareerPlanner` (when built) | talent notes |

# HUMAN-IN-THE-LOOP RULES (enforce in code)
1. **Anti-anchoring:** score buttons start unselected. Model score and panel average on the scoring dock stay hidden until the panelist submits.
2. No AI output is final. Snapshots, machine matches, PSDMs, and AI-drafted briefs need an explicit human click, logged.
3. Overrides need a justification (min 20 characters), logged.
4. Match Lab scenario work is **sandboxed**. Nothing changes official state until "Promote scenario to proposal" (Assignments Officer, logged).
5. **Firewall:** post-board talent data (Risk of Loss, Impact of Loss, Talent Score, N-Box cell, Advancement Readiness) is never visible to the Panelist role, never visible for a candidate before that board reaches Finalized, and is excluded from OoM and from default match weights. It appears only as an informational overlay for Assignments Officer and CFM / Career Planner.

# EXTENSIBLE ARCHITECTURE (mandatory: this is how new dashboards get added later)
**Nothing is hardcoded into navigation or tiles.** Build two registries and read everything from them.

```ts
type Role = 'Panelist'|'Board Recorder'|'BMO Lead'|'Assignments Officer'|'HQ Strategic Viewer'|'CFM / Career Planner';
interface ExperienceDef {
  id: string;                 // EXP-001
  ideaId: string|null;        // IDEA-001 from the backlog, null for core views
  viewId: string;             // VIEW-CommandCenter, etc.
  title: string;
  group: 'Leadership'|'Board'|'Assignments'|'Talent'|'Audit';
  roles: Role[];              // who can open it
  question: string;           // one-line decision question it answers
  metricIds: string[];        // ids from the metrics registry
  status: 'Live'|'Roadmap';
  component: React.LazyExoticComponent<React.ComponentType>;
  demoStep?: string;          // one line for the Demo Guide
}
interface MetricDef {
  id: string;                 // M-101
  name: string; unit: string;
  formula: string;            // human-readable
  target?: string;
  thresholds?: { amber: number; red: number; direction: 'above'|'below' };
  owner: string; sourceRef: string;
  compute: (s: AppState) => MetricValue;   // pure function in lib/metrics.ts
}
```
Rules:
- Top navigation is generated from `registry/experiences.ts`, grouped by `group`, filtered by the current role. Adding an entry adds a tab. No other nav code.
- **No numeric KPI literal inside any component.** Every tile, card, and chart value comes from `useMetric(id)`. Every tile has an info popover showing formula, target, source, and owner from the registry.
- `Roadmap` experiences appear only in the **Experience Catalog drawer** (header button) as non-clickable cards with their question and status.
- The drawer also shows an **Assumptions and Open Questions** list (see end of this prompt).

## ADD-EXPERIENCE procedure (use for every backlog entry and every future idea)
1. Read the entry. If it lacks any of: question, persona, metrics, view description, done-when, **build nothing** and list the missing fields at the end of your reply.
2. Add any new metrics to `registry/metrics.ts` with a pure `compute` function.
3. Extend types **additively** (new optional fields or new interfaces). Never rename or remove existing fields. Extend the seed so the new view has meaningful data, including at least one amber and one red case where thresholds exist.
4. Build the view in `views/<ViewName>.tsx`, reusing existing components (tiles, badges, tables, charts, drawers). Under ~400 lines per file.
5. Add every user action to the DELTA ledger with a new type. Add role permissions.
6. Register it in `registry/experiences.ts` with `status: 'Live'`.
7. Add one line to the Demo Guide and one entry to `CHANGELOG.md` (idea id, date, files touched).
8. Run the Definition of Done. Do not modify the behavior of existing views except to display an additional element the entry names.
9. Reply with 3 lines: what was added, what was simplified, what is open.

**Definition of Done for any experience:** loads without console errors; keyboard reachable; empty and loading states; every control does something; AI content badged; ledger entries written; firewall and role rules respected; metrics read from the registry; numbers match the seed.

# DATA MODEL
Seeded PRNG (`mulberry32`), reproducible. **36 candidates** (12 fully hand-written with distinct SOI text of 140 to 200 words each, 24 generated), **12 board cycles** (`BRD-001` to `BRD-012`; the 4 cycles behind the four selectable boards in the Queue carry candidate-level data, the other 8 carry milestone and risk data only), **12 billets**, **6 panelists**, **~200 generated milestones** across the 12 cycles, **~14 risks/issues/changes**, **8 endorsers**, **4 source pipelines**.

**Seed so computed values reproduce these reference values** (baseline dashboard): Board Status 6 On Track, 2 At Risk, 1 Delayed, 3 Complete. Milestones On Track 94%. Milestone timeliness Preparation 94%, Eligibility 92%, Documents 88%, Execution 97%. Packet readiness Eligibility 98%, Documents 93%, Completeness 96%, Exceptions 12%. Open issue aging: under 30 days 3, 30 to 60 days 2, over 60 days 1. Label all of these "Illustrative values until replaced with validated data".

Seed deliberately: 2 heartburn outliers, 3 hidden-talent candidates (low panel avg, high fit), 1 stale-ARMS candidate (amber), 1 candidate past the red threshold on one source, 1 TOS waiver, 1 pending adverse action, 1 harsh and 1 lenient panelist, 2 hard-to-fill billets, 1 candidate who is a strong fit for 3 billets, 2 records stuck over 24 hours, 1 failed ingest, 1 validated record with errors, 2 overdue commander actions, one endorser who places many of their unit in the top 3.

Tag vocabulary for SOI tags (use these): `Operational planning`, `OCONUS willing`, `Joint staff`, `SDE in residence`, plus 8 more of the same style. DAFSC values: `63A3`, `13S3`, `14N3`, `17S3`, `61C3`. Certifications span Orbital Warfare, Acquisition, and Cyber. `badgeLevel` is a Space Ops Badge level.

```ts
type Lvl = 'High'|'Med'|'Low';
type Tri = 'High'|'Medium'|'Low';
type StageGate = 'Ingested'|'Validated'|'Locked'|'Scoring'|'Finalized';
interface Candidate {
  id: string; dodId: string; name: string;
  grade: 'CAPT'|'MAJ'|'LT COL'|'MSGT'|'SMSGT';
  dafsc: string; pasCode: string; currentUnit: string; lastAssignment: string;
  tig: string; tafcsd: string; dor: string;
  lookStatus: '1st Look'|'2nd Look'|'Below Zone';
  performance: { year: number; strat: string; pctTop: number }[];   // 3 to 5 yrs
  ropTrend: 'Upward'|'Consistent'|'Mixed'; decorations: string[];
  deployments: number; jointTours: number; shortTours: number;
  timeline: { year: number; role: string; command: string; missionArea: string }[];
  skills: { cluster: string; proficiency: number; rarity: number }[];
  skillVector: number[];                       // 8 dims
  badgeLevel: string; certs: string[];
  pmeStatus: 'SDE Complete'|'ILE Eligible'|'ILE In Progress';
  highestDegree: string; academicDiscipline: string; pmeYear: number;
  llmFitScore: number; mScore: number;
  confidence: { soi: Lvl; rops: Lvl; arms: Lvl; surf: Lvl };
  featureDrivers: { feature: string; weight: number }[];
  soi: { text: string; date: string; wordCount: number; tags: string[]; endorserQuote: string;
         preferences: { billetIds: string[]; locations: string[]; willingOCONUS: boolean } };
  chatSnapshot: { id: string; summary: [string,string,string]; strengths: string[];
                  considerations: string[]; signedOffBy: string|null; signedOffAt: string|null };
  eligibility: { adverseActions: string; waivers: string; limitationCode: string };
  freshness: { soi: string; rops: string; arms: string; surf: string; milpds: string; myvector: string }; // ISO
  scores: { panelistId: string; value: number; at: string }[];     // 6.0 to 10.0, 0.5 steps
  myScore: number|null; flagged: boolean;
  packetReadiness: { eligibility: boolean; documents: boolean; complete: boolean; exceptions: string[] };
  endorserId: string; endorserRank: number|null;                   // rank within endorser's unit list
  flow: { stage: StageGate; ingestedAt: string; validatedAt?: string; lockedAt?: string;
          scoringStartedAt?: string; finalizedAt?: string; errors: string[]; failedIngest: boolean };
  talent?: { riskOfLoss: Tri; impactOfLoss: Tri; talentScore: number; nBoxCell: string;
             advancementReadiness: 'Ready now'|'1-2 yrs'|'3+ yrs' };   // firewalled, see rule 5
}
interface Billet { id: string; title: string; unit: string; location: string; oconus: boolean;
  dafscRequired: string[]; gradeRange: [string,string];
  requiredSkills: { cluster: string; minProficiency: number }[];
  preferredTags: string[]; priority: 'Critical'|'High'|'Routine'; fillByDate: string; skillVector: number[] }
interface Board { id: string; name: string; status: 'On Track'|'At Risk'|'Delayed'|'Complete';
  startDate: string; lockDate: string; releaseDate: string; ownerId: string }
interface Milestone { id: string; boardId: string; phase: 1|2|3|4|5|6|7|8; name: string; owner: string;
  planned: string; actual: string|null; dependsOn: string[]; status: 'Done'|'On Track'|'At Risk'|'Delayed' }
interface Risk { id: string; boardId: string; type: 'Risk'|'Issue'|'Change'; title: string; owner: string;
  opened: string; suspense: string; mitigation: string; resolutionDate?: string; escalate: boolean; status: 'Open'|'Closed' }
interface SourcePipeline { id: 'ARMS'|'MilPDS'|'MyVector'|'SURF'; authoritativeSource: string; lastSyncAt: string }
interface Endorser { id: string; name: string; unit: string }
interface StaffSlot { boardId: string; role: 'Board President'|'Panelist'|'Scribe'; assigned: boolean; name?: string }
interface CommanderAction { id: string; candidateId: string; commander: string; due: string; status: 'Pending'|'Done' }
interface ImprovementAction { id: string; phase: number; title: string; owner: string; due: string; done: boolean; lessonLearned?: string }
```
`heartburn` is **computed**: |myScore - panelAvg| in the top 1% of the pool (use >= 1.5 points with this pool size; say so in a tooltip).

# METRICS REGISTRY (seed `registry/metrics.ts` with every row; "Surfaced in" says where it appears)
Source screens (`bmo.png`, `dataset board view.png`, `Personnel file.jpg`, `202605 1 dash.png`, `202605 2 dash.png`, `Dashboard Board Execution.png`, `Dashboard Readiness ILE-SLE.png`, `Impcat Loss.png`) are not attached; values below come from the catalog text.

| ID | Metric | Formula / logic | Target / thresholds | Surfaced in |
|---|---|---|---|---|
| M-101 | Records Processed Today | count processed in last 24h, show % change vs prior 24h | none | IDEA-001 |
| M-102 | Average Processing Time | mean(lockedAt - ingestedAt), minutes | <= 5.0 min | IDEA-001 |
| M-103 | Stuck Records | records in a non-complete stage > 24h | 0; amber "Needs Attention" if > 0 | IDEA-001 |
| M-104 | Failed Ingests | records with hard integration error | 0; red "Retry Available" if > 0 | IDEA-001 |
| M-105 | On-Track for Deadline | Yes if days remaining to lock > estimated processing days; show days remaining | Yes | IDEA-001 |
| M-106 | Throughput Trend | daily processed, 7-day rolling; slope of linear fit shown as "Topological Slope" | monitor peak days | HELIOS A (core) |
| M-201 | Ingested | packages ingested into active cycle | n/a | IDEA-001 |
| M-202 | Validated | packages passing schema validation; also show error count | n/a | IDEA-001 |
| M-203 | Locked | population locked from automated source updates | n/a | IDEA-001 |
| M-204 | Scoring | records in active panel evaluation | n/a | IDEA-001 |
| M-205 | Finalized | scores submitted, verified, closed | n/a | IDEA-001 |
| M-301 | ARMS freshness | hours since last sync | amber > 12h, red > 24h | core freshness badges, IDEA-002 |
| M-302 | MilPDS freshness | hours since last sync | amber > 24h, red > 48h | core freshness badges, IDEA-002 |
| M-303 | MyVector freshness | hours since last sync | amber > 24h, red > 72h | core freshness badges, IDEA-002 |
| M-304 | SURF freshness | days since last sync | amber > 1 day, red > 7 days | core freshness badges, IDEA-002 |
| M-401 | Pool Average Score (Board Avg) | sum of all submitted scores / total scored candidates | 6.0 to 10.0 | Queue |
| M-402 | Panel Average Score | sum of panel scores / total scored on panel | 6.0 to 10.0 | Queue, dock |
| M-403 | Your Average Score | sum of user scores / total scored by user | 6.0 to 10.0 | Queue |
| M-404 | Endorsee Scale | % of an endorser's unit candidates placed in Top 3 by that endorser | reference 18% Top 3 / 82% Other | IDEA-004 |
| M-405 | Stratification Overview | share of DP, P, Non-Strat across candidate unit | reference DP 29%, P 62%, Non-Strat 18% (sums to 109%, see Open Questions) | IDEA-004 |
| M-406 | Confidence Ratings | High / Med / Low for SOI, ROPs, ARMS, SURF, color coded | n/a | Panel Workspace |
| M-501 | Portfolio Readiness | % of cycle candidates with "Ready" packages | 100% before start date | IDEA-003 |
| M-502 | At-Risk Cycles | % of cycle candidates with missing core documents | 0% before start date | IDEA-003 |
| M-503 | Commander Actions | count of overdue command endorsement tasks | 0; amber if > 0 | IDEA-003 |
| M-504 | Packet Completeness | verified documents (SOI + ROP + ARMS + SURF) / total required | >= 95% to lock | Command Center, IDEA-003 |
| M-505 | Messaging Flow Status | assigned Board President, Panelists, Scribes | President assigned; Panelists 12/14 | IDEA-003 |
| M-601 | Risk of Loss | predicted likelihood of separating | High / Medium / Low | IDEA-005 |
| M-602 | Impact of Loss | operational impact if candidate departs position | High / Medium / Low | IDEA-005 |
| M-603 | Talent Score | multi-dimensional potential rating | 0 to 100 (assumed) | IDEA-005 |
| M-604 | N-Box Cell Assignment | strategic talent classification cell (for example "Key Talent") | grid cell | IDEA-005 |
| M-605 | Advancement Readiness | readiness for next higher grade | Ready now / 1-2 yrs / 3+ yrs | IDEA-005 |
| K-01 to K-11 | KPI catalog rows | see KPI table in `VIEW-CommandCenter` | see table | Command Center |

# DELTA LEDGER (build first; every action goes through `logDelta`)
Entry: `DELTA-000123`, Zulu timestamp (local toggle), actor, role, target ID, type, summary, payload, `prevHash`, `hash` (SHA-256 via `crypto.subtle`). Append only. "Chain verified" indicator on load. BMO Lead tamper test turns it red.
Types: `SCORE_SUBMIT, SCORE_REVISE, FLAG, SNAPSHOT_SIGNOFF, SHUFFLE, MATCH_APPROVE, MATCH_OVERRIDE, SCENARIO_RUN, SCENARIO_PROMOTE, HEARTBURN_ADJUDICATE, ROLLBACK, PSDM_EXPORT, FILE_ROUTED, MILESTONE_UPDATE, RISK_UPDATE, KPI_EDIT, IMPROVEMENT_ACTION`. The ADD-EXPERIENCE procedure adds more types as needed (for example `PIPELINE_RETRY`, `POPULATION_LOCK`, `COMMANDER_ACTION_UPDATE`, `TALENT_NOTE`).
Shuffle entries display the label `DELTA-ShuffleSeed`.

# VIEW-CommandCenter (leadership landing page)
- **Header:** title `ETM Boards Workflow and KPI Dashboard`. Subheader: `Active model for workflow visibility, performance measurement, risk management, and leadership decision support.`
- **Disclaimer banner:** tag `Editable demonstration model:` then `Click a KPI-table cell to update the text. Dashboard visuals use illustrative values and should be replaced with validated reporting data before operational use.`
- **Executive cards (computed, clickable, drill to the list):** Active Boards (12, "Active Boards / FYTD", blue top border), Milestones On Track (94%, green), Packet Readiness (96%, purple), Open Issues Requiring Attention (3, amber). Rule for "requiring attention": open items aged 30 days or more (2 + 1 from the aging buckets). Production reference strip under the cards: `Scored 112/449 · Pending 337 · Flagged 12 · Board Avg 7.8 · Panel Avg 8.1 · My Avg 8.4` labeled "Production pool reference".
- **8-phase workflow bar** (arrows between phases; phases 1 to 4 phase-blue, 5 to 8 phase-purple). Click a phase to filter the KPI catalog and milestones; click again or "Reset Filters" to clear. Each phase shows a live health dot and the count of candidates currently in it. Phases and scope text:

| # | Phase | Scope |
|---|---|---|
| 1 | Board Initiation | Establish board cycle, population, governance, owner, and initial schedule. |
| 2 | Preparation | Plan milestones, coordinate requirements, and prepare board execution. |
| 3 | Ingestion & Eligibility | Ingest submissions, validate candidate information, and determine eligibility. |
| 4 | Document Assembly / eBOSS Setup | Retrieve documents, validate packets, and prepare records for review. |
| 5 | Scoring & Board Execution | Review, score, assess, and capture board decisions and rationale. |
| 6 | OOM / Results Finalization | Finalize Order of Merit outputs, validate results, and prepare release. |
| 7 | Post-Board Matching & Activations | Complete handoffs, matching, activations, declinations, and deferments. |
| 8 | Closeout, Lessons Learned & KPI Reporting | Resolve issues, capture improvement actions, and report performance. |

- **Two supporting panels under the bar:** (1) *Milestones, Dates, and Completion Status*: track milestones, suspense dates, dependencies, completion; compare planned vs actual; identify delays and accountable owners. (2) *Risks, Issues, and Change Requests*: log them; track mitigation, resolution dates, escalation; use aging to focus leadership on overdue items. Each shows live top items and links to `VIEW-OpsRegisters`.
- **Data architecture strip (clickable):** six registers (`Board Register`, `Milestone Register`, `Candidate / Packet Readiness Register`, `Action Register`, `Risk / Issue / Change Register`, `Results and Post-Board Register`) then `STANDARDIZED ETM DATA LAYER` (Standard data definitions, Consistent workflow tables, Data quality and validation, Single source of truth) then `ETM POWER BI BOARDS DASHBOARD (ORBIT concept equivalent)` with pills: Status, Performance, Trends, Decisions, Timeliness, Readiness, Issue Aging. Each register opens its table in `VIEW-OpsRegisters`.
- **Four visuals:** Board Status donut (center label "Board Status"; On Track, At Risk, Delayed, Complete), Milestone Timeliness bar gauges (Preparation, Eligibility, Documents, Execution), Packet Readiness segmented bars (Eligibility, Documents, Completeness, Exceptions), Open Issue Aging segmented bar with legend (Less than 30 days, 30 to 60 days, More than 60 days).
- **KPI catalog and accountability model.** Controls: search placeholder `Search KPI, owner, source, timeframe, or phase...`, dropdown `All workflow phases`, `Reset Filters`, `Print / Save as PDF` (`window.print` plus print stylesheet). Cells editable for BMO Lead (logs `KPI_EDIT`). Each row also shows a **computed live value** and a "Drill to candidates" link where applicable (for example Packet Completeness opens the Queue filtered to incomplete packets). Seed exactly:

| Workflow Phase | KPI | Numerator | Denominator | Timeframe | Likely Data Source | Accountable Data Owner |
|---|---|---|---|---|---|---|
| Preparation | Preparation Timeliness | Boards completing required preparation milestones by suspense | Boards with preparation milestones due | Monthly / FYTD | Boards workflow tracker; MyVector/eBOSS process records as validated | Boards Management Lead |
| Ingestion & Eligibility | Eligibility Completion Rate | Candidates with eligibility status completed by required deadline | Candidates requiring eligibility determination | Per board / monthly | eBOSS ingestion template or validated Boards tracker | Ingestion/Eligibility Process Owner |
| Document Assembly | Packet Completeness Rate | Candidate packets complete at readiness review | Candidate packets required for the board | Per board | eBOSS; document-completeness tracker | Document Assembly Owner |
| Document Assembly | Document Retrieval Timeliness | Required documents received by suspense | Required document requests due | Per board / monthly | ARMS/AFPC request log; workflow tracker | Document Retrieval Owner |
| Execution | Board Execution Timeliness | Boards completing scoring within planned execution window | Boards scheduled for execution | Per board / quarterly | eBOSS board execution record | Board Execution Lead |
| Execution | Scoring Throughput | Records scored | Board execution hours or days | Per board | eBOSS scoring record | Board Execution Lead |
| Results Finalization | OOM Finalization Timeliness | OOM exports finalized and validated within standard | Boards requiring OOM finalization | Per board / monthly | eBOSS OOM export; validated results tracker | Results Finalization Owner |
| Post-Board | Post-Board Handoff Timeliness | Results handed to downstream team within standard | Boards requiring post-board handoff | Monthly / FYTD | Results handoff log; workflow tracker | Post-Board/Matching Owner |
| Post-Board | Activation Resolution Timeliness | Declinations, deferments, and activations resolved within standard | Cases requiring action | Monthly / FYTD | Centralized results/activation tracker | Assignments/Activation Process Owner |
| Cross-Cutting Management | Open Issue Aging | Open issues beyond established suspense | Total open issues | Weekly / monthly | Boards risk-and-issue register | Boards Management Lead |
| Closeout / Improvement | Improvement Action Completion Rate | Improvement actions completed by approved suspense | Improvement actions due in period | Monthly / quarterly | Improvement action register | Assigned Improvement-Action Owner |

- **Pipelines and Agents row (summary):** `PIPE-Ingest-Roster` (99.4%), `PIPE-MyVector-SOI` (5 min, ticking last-sync), `PIPE-Ingest-ROPE`, `AGENT-Validator`; one shown degraded amber. Links to `VIEW-HeliosCommand`.
- **Board Health Forecast (new):** per active board, projected completion date from milestone slippage (simple linear projection) with a confidence band and the top 2 driving milestones. Amber or red if projected past the required release date.

# VIEW-CandidateQueue
- Board selector: `ILE AY 26-27 - Panel 1`, `CY26 O4/O5 DT Scoring`, `DARPA SCFP`, `Space Test Course`. Counters computed live: Scored, Pending, Flagged, plus `Board Avg (M-401) | Panel Avg (M-402) | My Avg (M-403)`. Demo pool is 36; production reference 449.
- Search (`Ctrl+J`, show hint), filters (LLM Fit > 75, Status All/Pending/Scored/Flagged, PME status, Packet Readiness Complete/Exceptions), column sort, empty state.
- **True Random Shuffle:** Fisher-Yates seeded from `crypto.getRandomValues`; log seed, pre and post order hashes (label `DELTA-ShuffleSeed`); display "Seed: 0x...".
- Row click opens `VIEW-PanelReview`. Row hover shows a mini card (top 3 skills, trend).

# VIEW-PanelReview
- **8 lenses** (`Alt+1` to `Alt+8`): Overview (4-Panel), Skills, Experience, Performance, Education, Predictive, Narrative & SOI, Eligibility & Freshness. Selected candidate persists. One-line question under each lens name. Lenses reorganize the central workspace around that question.
- Overview 4-Panel: SOI card (date, word count, intent pills, full text), ROPs card (3 cycle stratification cards), ARMS and Freshness card (per-source badges using the thresholds in M-301 to M-304, limitation and waiver status), SURF card (Grade, DAFSC, TIG, TAFCSD, PME, PAS, Deployments, Joint Tours, Decorations, Adverse Actions).
- **Similar Candidates:** 5 nearest by cosine similarity of `skillVector`, with Compare (sends to `VIEW-CrossAnalytics`).
- **Possible Fits:** top 3 billets with a one-line reason, linking to Match Lab.
- **LLM Scribe drawer (Req 145):** `CHAT-SNAPSHOT-vNN`, 3-paragraph synthesis, confidence breakdown (SOI, ROPs, ARMS, SURF), Highlighted Strengths, Considerations (for example TIG `4y 2m` vs pool avg `4y 9m`). **Approve & Sign Off CHAT-SNAPSHOT** logs `SNAPSHOT_SIGNOFF`, stamps who and when, revocable with its own entry. Optional live Gemini via `VITE_GEMINI_API_KEY`; on missing key or error, silently use seeded text labeled "Cached synthesis". Never block the UI.
- **Scoring dock (sticky):** 6 to 10 in 0.5 steps; keys `1` to `5` = 6 to 10, `Shift` + key = +0.5, arrows nudge 0.5 (documented in a "?" popover). **Submit & Next Candidate**, **Flag for Discussion** (reason required), Previous. After submit reveal model score and delta vs panel average. Revision needs confirm and logs `SCORE_REVISE`.

# VIEW-HeliosCommand (core, three sub-tabs; backlog ideas extend them)
- **A. Pipeline & Topological Slope:** cards for `PIPE-Ingest-Roster` (99.4% ingest success), `PIPE-MyVector-SOI` (5 min real-time sync), `PIPE-Ingest-ROPE`, `AGENT-Validator`, each with status dot, throughput sparkline, last run, error count, expander with 5 fake runs. One degraded amber. Show the slope arrow from M-106.
- **B. Source Freshness:** table of the four source pipelines with last sync and amber/red status from M-301 to M-304 (IDEA-002 deepens it).
- **C. Portfolio Readiness:** summary of M-501 to M-505 (IDEA-003 deepens it).

# VIEW-ScoringAudit (Req 150)
Table of computed heartburn candidates: My Score, Panel Avg, Model Score, gap. Actions: **Reconcile** (new score + required note), **Uphold** (required note), **Escalate to Chair**; logs `HEARTBURN_ADJUDICATE`. BMO Lead only: **Parity Breach Rollback Playbook** 4-step stepper (Freeze scoring, Identify affected PDOs, Revert to last verified DELTA, Re-open for rescore), type `ROLLBACK` to confirm, state visibly reverts.

# VIEW-MatchLab (Req 149; contains `VIEW-MatchReview`)
Header toggle: **Official Proposal** vs **Sandbox Scenario**.
- **4A Match Board (`VIEW-MatchReview`):** candidates ranked by OoM left, 12 billets right, matches as links. Top 20% machine-matched get the purple AI badge. Approve, Override (justification), Unmatch. Contention highlight when one candidate is top choice for several billets.
- **4B Weight Sliders and Live Rerank:** sliders for OoM rank, SOI tag overlap, skill fit, DAFSC and grade fit, candidate stated preference, billet mission priority (sum locked to 100%). Dragging re-runs matching and shows a **diff panel** (who moved, gained, lost, net fill rate, unfilled critical billets). Reset, and Save scenario (named, up to 5). Default weights: 0.60 OoM, 0.25 SOI overlap, 0.15 DAFSC and grade fit with hard constraints on DAFSC and grade; other sliders start at 0.
- **4C Stable Matching Mode:** Greedy by OoM vs Stable (Gale-Shapley with candidate preferences). Show side by side: candidates with a top-3 preference, billets filled, blocking pairs. Plain-language popover.
- **4D Sensitivity:** per match, run 200 perturbations of weights (+/- 10%), show % of runs the match holds (green >= 80, amber 50 to 79, red < 50). Fragile matches listed first.
- **4E Near-Fit Finder:** candidates just outside constraints with a gap card (requirement missed, by how much, what training or tour would close it). Label "Not eligible as written; shown for human consideration only."
- **4F Billet Coverage and Supply Risk:** qualified count per billet, supply bar, thin-supply flags, "if candidate X declines" cascade.
- **4G Explainability:** Explain drawer with stacked weighted score, constraints passed, tags matched, top competitor and why they lost.
- **4H Promote and Publish:** Promote scenario to proposal (Assignments Officer, logs `SCENARIO_PROMOTE`). **Generate Draft PSDM & Export Audit Package** (`AGENT-Publisher`): modal with memorandum preview (USAF/USSF Personnel Services Delivery Memorandum: MEMORANDUM FOR, SUBJECT, numbered paragraphs, signature placeholder, DRAFT watermark), "Copy text", "Download .json audit package" (matches, overrides, scenario weights, ledger entries, chain hash). Disabled until at least one approved match. Logs `PSDM_EXPORT`.

# VIEW-CrossAnalytics
- **Comparator:** 2 to 4 candidates, radar (Skills, Experience, Performance, Education, Predictive Fit, each 0 to 100 with tooltip formula) plus synchronized metric table, best-in-row highlighted.
- **Scatter:** Panel Avg vs LLM Fit, quadrant lines at medians, labeled quadrants (Consensus Strong, Hidden Talent, Model Skeptic, Consensus Weak), heartburn ringed red, click adds to comparator.
- **Talent Constellation:** 2D projection of `skillVector` (PCA or fixed seeded layout), dots by status, sized by Fit, billets as diamonds, hover and select clusters; shows skill clusters with no billet and billets with no nearby candidate.
- **Panelist Calibration:** per panelist mean, spread, z-score comparison, panelist x candidate heatmap, flag harsh or lenient, **preview** of OoM under normalization (never applied without BMO Lead approval and a logged entry).
- **Inter-rater Agreement:** score spread per candidate; 5 most contested.
- **Score Distribution Monitor:** histograms by DAFSC, look status, unit; advisory note that large unexplained differences need human review.
- A computed one-line plain-language takeaway above each chart.

# VIEW-OpsRegisters
Sub-tabs for the six registers.
- **Milestones:** Gantt-style planned vs actual bars, dependency arrows, owner, status filters, late items on top. Editable by Board Recorder and BMO Lead (`MILESTONE_UPDATE`).
- **Risk / Issue / Change:** aging buckets, mitigation, suspense, resolution date, escalate flag (`RISK_UPDATE`).
- **Improvement Actions and Lessons Learned (Phase 8):** add entries linked to a phase and an owner with due dates; completion-rate KPI (`IMPROVEMENT_ACTION`).
- Board, Candidate/Packet Readiness, Action, Results and Post-Board registers as sortable read-only tables with CSV export.

# VIEW-DeltaLedger
- **Ledger:** paginated, filters (type, actor, target, date range), search, CSV export, chain status. **Replay mode:** timeline scrubber that rewinds state to any DELTA entry in read-only view.
- **ORBIT-NIPR File Canonicalizer (`00SRT` Gate):** dropzone plus "Load sample files" (8 messy mock names such as `Smith SOI final v3 (1).docx`, `ROP_2025.pdf`, `psdm draft.docx`, `SURF export 10-14.xlsx`, one bad extension, one duplicate). Read name, size, extension only. Rename to `[NodeCode]_YYYYMMDD_Slug.ext` (lowercase hyphen slug, max 60 chars). Result **PASS / WARN / FAIL** with reason text. Route to the SharePoint `ORBIT-BMO` tree: `00SRT, 01INT, 02SOI, 03ROP, 04SURF, 05ARMS, 06SCR, 07MAT, 08PUB, 09REF, 88ARC`. Attach to the PDO, log `FILE_ROUTED`. BMO Lead commits routing. Raw file types: SURF, SOI, ROPs, PSDM.

# QUALITY BAR
- No placeholder text. No `any` in domain types. No console errors. Every control works.
- Loading and empty states for every list and chart.
- Files: `types.ts`, `data/seed.ts`, `data/kpis.ts`, `registry/experiences.ts`, `registry/metrics.ts`, `state/reducer.ts`, `state/delta.ts`, `lib/match.ts`, `lib/stable.ts`, `lib/stats.ts`, `lib/metrics.ts`, `components/*`, `views/*`, `CHANGELOG.md`. Under ~400 lines each.
- **Demo Guide** side panel (toggle), one step per Live experience plus: read Command Center, drill from a late KPI to candidates, score a candidate, sign off a snapshot, change weights in Match Lab and read the diff, run stable matching, review near-fit candidates, export a PSDM, verify the chain.

# ASSUMPTIONS AND OPEN QUESTIONS (show in the Experience Catalog drawer; do not silently resolve)
1. Stratification reference values (DP 29%, P 62%, Non-Strat 18%) sum to 109%. Seed values must sum to 100%; note the discrepancy.
2. N-Box axes are not defined in the source. Default to a 3x3 grid, Performance by Potential, with cell labels configurable (for example "Key Talent").
3. The catalog expands ARMS as "Aviation Resource Management". Use the label `ARMS` only; confirm the expansion with the data owner.
4. "Open Issues Requiring Attention" is assumed to mean open items aged 30 days or more.
5. Talent Score scale is not defined; 0 to 100 assumed.
6. Source screenshots were not attached; metrics values come from catalog text.

# OUTPUT
Deliver each stage as a working app. After each stage list in 5 lines or fewer anything simplified.
