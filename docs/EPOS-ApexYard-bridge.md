# EPOS × ApexYard — The Planning-to-Production Bridge

> How two systems — EPOS (product strategy + operating standard) and ApexYard (SDLC governance + mechanical enforcement) — connect at the seam where hypotheses become tickets and learning becomes code.

---

## Part 1 — The Big Picture: Who Owns What

```
┌─────────────────────────────────────────────────────────────────────┐
│                         EPOS                                       │
│   "What to build and why — measured learning before scaling"       │
│                                                                    │
│   §1  Product Context ───────── What is this? Who is it for?      │
│   §2  Strategy & Stages ──────── Where are we? What must be true? │
│   §3  Process & SDLC ──────────── How does work flow?              │
│   §4  Engineering Practices ────── How do we build safely?         │
│   §5  Metrics ─────────────────── Are we moving the right dials?   │
│   §6  People & Health ────────── Is the team sustainable?          │
│   §7  Weekly Assessment ────────── What did we learn this week?     │
│   §8  Prioritization ──────────── What do we do next?               │
│   §9  Exceptions ──────────────── Where and why do we deviate?      │
│   §10 Front Page ──────────────── One-page summary                  │
│                                                                    │
│   OUTPUT: per-product YAML instances + weekly review packets       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                  The Seam: hypothesis → ticket
                  (EPOS §2.2 Hypothesis → ApexYard `/spike`)
                  (EPOS §2.3 Milestone → ApexYard `/roadmap`)
                  (EPOS §8 Score → ApexYard P0/P1/P2/P3)
                               │
┌──────────────────────────────▼──────────────────────────────────────┐
│                        ApexYard                                    │
│   "How to build it — enforced gates from ticket to production"     │
│                                                                    │
│   /idea ──────── Raw capture (IDEA-NNN in backlog)                 │
│   /validate-idea  5-question gate before commitment                 │
│   /spike ──────── Hypothesis experiment (time-boxed, disposable)  │
│   /write-spec ─── PRD for validated ideas                          │
│   /feature ────── Structured ticket with user story + ACs          │
│   /start-ticket── Declare active work (hook-enforced requirement)  │
│   Build ───────── Code → PR → Review → Merge → QA → Deploy       │
│   /spike-close ── Disposition gate (promote or discard)           │
│   /launch-check── 8-dimension production readiness audit           │
│                                                                    │
│   OUTPUT: GitHub Issues, PRs, AgDRs, review markers, CI runs       │
└────────────────────────────────────────────────────────────────────┘
```

**The key insight:** EPOS ends at the ticket. ApexYard starts at the ticket. The connection point is the **hypothesis** — EPOS defines what must be learned, ApexYard's `/spike` is the vehicle to learn it, and the disposition (`/spike-close`) feeds the answer back into EPOS's §7 weekly assessment.

---

## Part 2 — Concept-by-Concept Mapping

### 2.1 EPOS Objects → ApexYard Artifacts

| EPOS Object | EPOS ID Pattern | ApexYard Equivalent | ApexYard ID Pattern | Where It Lives |
|---|---|---|---|---|
| Product | `prd-<slug>` | Project in `apexyard.projects.yaml` | Project `name` field | Ops repo root |
| Hypothesis | `hyp-<slug>-<nnn>` | `/spike` ticket + `/idea` backlog entry | `IDEA-NNN` → `[Spike] GH-NNN` | ideas-backlog.md + GitHub |
| Experiment | `exp-<hyp-slug>-<nnn>` | Spike branch + spike PR | `spike/GH-<N>-<slug>` | Git + GitHub |
| Milestone | `mil-<slug>-<nnn>` | `/roadmap` milestone entries | Roadmap H2 headings | `projects/<name>/roadmap.md` |
| Metric | `met-<slug>` | `/launch-check` dimensions + custom | Audit scores + manual | `projects/<name>/` |
| Person | `prs-<slug>` | `onboarding.yaml` team entries | `name` + `role` | Ops repo root |
| Action | `act-<yyyymmdd>-<nnn>` | GitHub Issue (feature/task/bug) | `GH-NNN` | GitHub Issues |

### 2.2 EPOS Sections → ApexYard Skills

| EPOS Section | What It Asks | ApexYard Skill That Answers It | How |
|---|---|---|---|
| §1 Product Context | What is this product? Who owns it? | `/setup` + `onboarding.yaml` | Company, team, tech stack |
| §1 Product Context | What capabilities does it have today? | `/extract-features` | 6-axis scan of codebase |
| §1 Product Context | What systems depend on it? | `/c4` + `/dfd` | Architecture diagrams |
| §2.1 Stage Model | What stage are we in? | `onboarding.yaml` → doesn't store stage | Store in `projects/<name>/epos/` |
| §2.2 Hypotheses | What are our leap-of-faith assumptions? | `/idea` → `/validate-idea` → `/spike` | Ideas backlog → validation → experiment ticket |
| §2.3 Milestones | What gates must we clear? | `/roadmap` | Milestones with dates |
| §3 Process & SDLC | How does work flow? | ApexYard hooks + `workflows/sdlc.md` | Mechanically enforced gates |
| §4 Engineering | How do we build safely? | ApexYard hooks + `onboarding.yaml` quality | Lint, test, coverage, review |
| §5 DORA Metrics | Deployment frequency, lead time, CFR? | `/launch-check` (monitoring dimension) | Audit-based; no live DORA pipeline yet |
| §5 Lean Metrics | Activation, retention, engagement? | `/launch-check` (analytics dimension) | Event taxonomy audit |
| §7 Weekly Assessment | Product + individual sheets? | `/stakeholder-update` + `/status` | Narrative, not structured YAML |
| §8 Prioritization | Weighted scoring? | P0/P1/P2/P3 labels | Simplified; no formula scoring |
| §9 Exceptions | Where do we deviate? | AgDR (`/decide`) + skip markers | Decision records in code |

---

## Part 3 — The Planning Flow for a 2-Developer Team with Unvalidated Ideas

This is your exact situation. Let's walk through it step by step.

### 3.1 Where You Are in EPOS Terms

Based on your `onboarding.yaml` (Sunapse, BMS + PV Solar, 2 developers):

```
EPOS Stage Assessment for Sunapse:
┌──────────────────────────────────────────────────────┐
│  Stage: Problem-Solution Fit (or early PMF-search)  │
│                                                      │
│  Evidence:                                           │
│  • Product exists but has few features               │
│  • Team: 2 developers (1 backend, 1 frontend)        │
│  • Ideas backlog has unvalidated hypotheses          │
│  • No DORA metrics pipeline yet                      │
│  • No Lean metrics (activation, retention) yet        │
│                                                      │
│  EPOS prioritization weights for this stage:         │
│  w_learn = 0.55  (55% — learning dominates)          │
│  w_flow  = 0.15  (15% — basic flow)                  │
│  w_econ  = 0.10  (10% — economics don't matter yet) │
│  w_cost  = 0.20  (20% — keep costs low)              │
└──────────────────────────────────────────────────────┘
```

**What this means:** More than half your capacity should go to **learning** (validating hypotheses), not building production features. Spikes and experiments are your primary vehicle, not feature tickets.

### 3.2 The Planning Pipeline: From Raw Idea to Code

```
EPOS §2.2 HYPOTHESIS           ApexYard SKILLS               OUTPUT ARTIFACT
─────────────────────         ─────────────────              ────────────────
                                │
 "We believe that                │
  weather aggregation            │
  will save users                │
  15% on energy bills"           │
                                │
                        ┌───────▼────────┐
                        │   /idea         │  Capture raw, no commitment
                        │                 │  IDEA-001 in ideas-backlog.md
                        └───────┬────────┘
                                │
                        ┌───────▼────────┐
                        │ /validate-idea  │  5-question gate:
                        │                 │   Q1: Target user (concrete)
                        │                 │   Q2: Current alternative
                        │                 │   Q3: Smallest testable version
                        │                 │   Q4: Kill criteria (falsification)
                        │                 │   Q5: Build / buy / rent
                        └───────┬────────┘
                                │
                   ┌────────────┼────────────┐
                   │            │             │
              GREEN verdict  YELLOW       RED verdict
              (validated)    (needs work) (kill)
                   │            │             │
                   ▼            ▼             ▼
           ┌──────────┐   Park for N     Update backlog:
           │ /write-  │   days; revisit    status → WONTDO
           │  spec    │   sharpening
           │          │
           │ Full PRD │
           └────┬─────┘
                │
        ┌───────▼────────┐
        │    /spike       │  Time-boxed experiment (EPOS §2.2 Experiment)
        │                 │   - Hypothesis statement
        │                 │   - Budget: 2-3 days
        │                 │   - Kill criteria ← maps to EPOS falsification
        │                 │   - Disposition: PROMOTE or DISCARD
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  Build spike    │  Spike branch: spike/GH-N-slug
        │  (throw-away    │  EXEMPT from:
        │   code)         │   • AgDR gates
        │                 │   • 80% test coverage
        │                 │  STILL requires:
        │                 │   • Code review (Rex)
        │                 │   • Security review (if auth/crypto)
        │                 │   • QA hypothesis verification
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │ /spike-close    │  Feed the answer BACK to EPOS
        │                 │
        │ --promote:      │  Hypothesis CONFIRMED
        │                 │  → Create [Feature] ticket
        │                 │  → Update EPOS §2.2: status → validated
        │                 │
        │ --discard:      │  Hypothesis REJECTED
        │                 │  → Write memo to docs/spike-memos/
        │                 │  → Update EPOS §2.2: status → invalidated
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │   /feature      │  Only NOW do you create a production feature
        │                 │  — user story, ACs, priority, design notes
        │                 │  → Full SDLC applies (AgDR, 80% coverage, etc.)
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  /start-ticket  │  Declare active work (hook-enforced)
        │                 │  → Write .claude/session/current-ticket
        └───────┬────────┘
                │
           Full SDLC:
           branch → code → PR → review → merge → QA → done
```

### 3.3 What EPOS Calls "Experiment" vs What ApexYard Calls "Spike"

These are the **same concept** expressed in different vocabularies:

| EPOS Experiment | ApexYard Spike |
|---|---|
| ID: `exp-<hyp-slug>-<nnn>` | Branch: `spike/GH-<N>-<slug>` |
| References one Hypothesis | Must include hypothesis statement |
| Build-Measure-Learn cycle | Budget + Kill Criteria + Disposition |
| Time-boxed (days to weeks) | Time-boxed (1-3 days recommended) |
| Produces validated/invalidated/inconclusive | Produces PROMOTE or DISCARD |
| Feeds back to §2.2 hypothesis status | Feeds back via `/spike-close` |

### 3.4 The Critical Loop: EPOS Weekly Review Feeds ApexYard Priorities

```
     EPOS §7 Weekly Review
     ┌────────────────────────┐
     │ Hypothesis disposition │─── validated ────► /feature (production ticket)
     │ (from /spike-close)    │─── invalidated ──► Update EPOS §2.2, kill related ideas
     │                        │─── inconclusive ─► Extend spike or new spike
     │                        │
     │ DORA metrics           │─── bottleneck ──► /task (tech debt / infra ticket)
     │ (from /launch-check)  │─── healthy ──────► Continue feature work
     │                        │
     │ Prioritization score   │─── high score ──► P0 label on next ticket
     │ (§8 blended rule)     │─── medium ──────► P1 label
     │                        │─── low ─────────► P2 or deprioritize
     └────────────────────────┘
```

---

## Part 4 — Where EPOS Has Depth That ApexYard Doesn't (Gaps)

ApexYard is an SDLC enforcement framework. EPOS is a product operating standard. The gaps are real:

### 4.1 What ApexYard Does NOT Cover (that EPOS Requires)

| EPOS Concept | ApexYard Gap | What to Do |
|---|---|---|
| **Hypothesis lifecycle** (open → validated → invalidated → retired) | Ideas backlog has `NEW/WONTDO` but no `VALIDATED/INVALIDATED/INCONCLUSIVE/RETIRED` lifecycle | Track in `projects/<name>/epos/hypotheses.yaml` |
| **Innovation accounting** (hypothesis validation rate over rolling 4-8 weeks) | No metrics pipeline | Track manually in weekly review; future: custom hook |
| **EPOS §2.1 Stage Model** (Problem-Solution Fit → PMF → GTM → Scale) | `onboarding.yaml` has no stage field | Add to `projects/<name>/epos/product-instance.yaml` |
| **Stage-dependent prioritization weights** (w_learn/flow/econ/cost) | Priority labels are static P0/P1/P2/P3 | Compute weighted scores alongside labels |
| **DORA metrics** (DF, LTFC, CFR, FDRT, rework rate) | No automated collection; `/launch-check` is audit-based | Start manual; automate via GitHub Actions + Datadog |
| **Lean metrics** (activation, retention, engagement) | `/launch-check` analytics-audit checks event taxonomy but doesn't collect data | Instrument your app; `/analytics-audit` validates coverage |
| **Business metrics** (LTV, CAC, payback) | No collection or modeling | Store in `projects/<name>/epos/metrics.yaml` |
| **Individual sheet** (time allocation, valuable-work %, burnout signals) | `/tasks` shows what needs attention but not per-person allocation | Manual; consider adding to weekly ritual |
| **Weekly review ritual** (Wednesday ingest → Thursday self-reports → Friday review) | `/stakeholder-update` generates narrative but not structured EPOS dashboard | Use `/stakeholder-update` output as starting point, enrich with EPOS structure |
| **Blended prioritization scoring** (learning × w_learn + flow × w_flow + econ × w_econ − cost × w_cost) | Priority labels are categorical, not formula-based | Compute scores manually in `projects/<name>/epos/prioritization.yaml` |

### 4.2 The Recommended Structure for EPOS Artifacts in ApexYard

Create a dedicated directory per project to store EPOS instances alongside ApexYard's existing project docs:

```
projects/
  myagent/
    epos/                              ← NEW: EPOS instance per project
      product-instance.yaml            ← §1-§6, §8-§9 consolidated
      hypotheses.yaml                  ← §2.2 hypothesis registry + status
      metrics.yaml                     ← §5 metric definitions + values
      weekly-reviews/                  ← §7 weekly review packets
        2026-W21.yaml                  ← One per ISO week
      front-page.md                    ← §10 one-page visual summary
    README.md                           ← Existing handover docs
    docs/
      agdr/                             ← Existing AgDRs
```

This structure keeps EPOS artifacts version-controlled (they're in the ops repo) and accessible to ApexYard skills without modifying the framework.

---

## Part 5 — Practical Playbook: 2 Developers, Unvalidated Hypothesis

### 5.1 Your EPOS Stage Determines Your ApexYard Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│  IF YOU ARE IN:               THEN YOUR APEXYARD WORKFLOW IS:      │
│                                                                     │
│  Problem-Solution Fit          HEAVY /spike usage                   │
│  (idea validity unknown)       LIGHT /feature usage                 │
│  w_learn = 0.55                Relaxed gates for spikes              │
│                                 50/50 discovery vs delivery          │
│                                                                     │
│  PMF Search                    MODERATE /spike + MODERATE /feature  │
│  (problem confirmed,           40/60 discovery vs delivery          │
│   solution evolving)           Milestones start to matter           │
│                                                                     │
│  Go-to-Market                 LOW /spike, HIGH /feature            │
│  (PMF found, scaling up)      20/80 discovery vs delivery           │
│  w_econ = 0.35                Full SDLC enforcement                  │
│                                 Launch checks become critical        │
│                                                                     │
│  Scale                        MINIMAL /spike, FULL /feature         │
│  (mature, optimizing)         10/90 discovery vs delivery            │
│  w_flow = 0.35                DORA metrics matter most               │
│                                 Tech debt reserve: 15-25%             │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 Week 1 Playbook: Bootstrapping Sunapse with EPOS

```
DAY 1 — Set up the EPOS product instance
─────────────────────────────────────────
1. /setup                          ← Already done
2. Create projects/myagent/epos/product-instance.yaml
3. Fill in §1 (Product Context):
   - id: prd-sunapse
   - name: Sunapse
   - domain: web-app
   - capabilities: [list from /extract-features myagent]
   - stage: pmf-search (or problem-solution-fit)
   - ownership: tech-lead: you
4. Fill in §2 (Strategy):
   - current stage: pick honestly
   - top 3-5 hypotheses
   - milestones for next 3-6 months
5. Store in Git: git add projects/myagent/epos/ && git commit

DAY 2 — Register every hypothesis
─────────────────────────────────
For EACH hypothesis in §2.2:
  /idea "We believe that [hypothesis]"
  /validate-idea IDEA-NNN

This creates the validation document and gives you GREEN/YELLOW/RED.

DAY 3 — Create experiments for GREEN hypotheses
────────────────────────────────────────────────
For each GREEN-validated idea:
  /spike "Test [hypothesis]"
  # Budget: 2-3 days
  # Kill criteria: pulled from EPOS §2.2 falsification condition
  # Disposition: PROMOTE (we want to build if validated)

For each YELLOW idea:
  Park in ideas-backlog.md; revisit next week.
  Mark with a date: "Revisit 2026-05-28 after [sharpening criterion]"

For each RED idea:
  Automatic: ideas-backlog.md status → WONTDO
  Record why in the validation doc.

DAY 4-5 — Run spikes
────────────────────
  /start-ticket GH-NN      ← Declare active spike
  git checkout -b spike/GH-NN-hypothesis-test
  # BUILD the smallest testable slice
  # MEASURE the outcome
  # LEARN: did the kill criteria fire?

END OF WEEK — Spike-close + EPOS weekly review
──────────────────────────────────────────────
For each completed spike:
  /spike-close
  # --promote if hypothesis confirmed → creates feature ticket
  # --discard if hypothesis rejected → writes memo

Update EPOS §2.2 hypothesis status:
  validated → the hypothesis was confirmed
  invalidated → the hypothesis was rejected
  inconclusive → need more data or new spike

Generate weekly review:
  /stakeholder-update myagent   ← Use as starting point
  Add EPOS structure: DORA snapshot, Lean snapshot, hypothesis dispositions
```

### 5.3 Ongoing Weekly Rhythm

```
MONDAY
├── Pick up to 2 items from the prioritized backlog
│   (weighted by EPOS §8 blended score for current stage)
├── /start-ticket GH-NN on the top item
└── Work through the SDLC (build → review → merge → QA)

WEDNESDAY (EPOS automated ingest)
├── Pull DORA data (deployment count, lead times, incidents)
├── Pull product analytics (activation, retention if instrumented)
└── Update metrics.yaml with current values

THURSDAY (Individual self-report — 5 min each)
├── Each developer: time allocation (feature/spike/bug/infra/learning)
├── Valuable-work % (self-rated)
├── Top accomplishment (linked to hypothesis or milestone ID)
├── Top friction point
└── Burnout self-rating (1-5)

FRIDAY (Review — 30 min for 2 devs)
├── Walk EPOS §7 dashboard
│   ├── Stage-gate: on-track / at-risk / off-track?
│   ├── Hypotheses: any validated/invalidated this week?
│   ├── DORA: deployment frequency, lead time, CFR
│   ├── Lean: activation trend, key retention point
│   └── Bottlenecks: what's blocking us?
├── Pick top 3 actions for next week
└── Commit updated EPOS product-instance.yaml
```

---

## Part 6 — The EPOS-ApexYard Feedback Loop in Detail

### 6.1 Hypothesis Lifecycle (EPOS §2.2 ↔ ApexYard)

```
       EPOS §2.2                              ApexYard
       ────────                              ────────

  Hypothesis CREATED
  (status: open)
       │
       │  ┌──────────────────────┐
       ├──│ /idea "hypothesis"   │──► IDEA-NNN (status: NEW)
       │  └──────────────────────┘
       │
       │  ┌──────────────────────┐
       ├──│ /validate-idea ID    │──► validation doc (GREEN/YELLOW/RED)
       │  └──────────────────────┘     Updates ideas-backlog.md status
       │
       ├── YELLOW → park, revisit
       ├── RED → WONTDO
       │
       │  GREEN ↓
       │
  Experiment RUNNING                Spike branch active
  (EPOS: exp-<hyp>-<nnn>)           (ApexYard: spike/GH-NN-slug)
       │
       ├── /start-ticket GH-NN
       ├── Build → Measure → Learn
       ├── /spike-close --promote ──► Hypothesis VALIDATED
       │   OR                             │
       ├── /spike-close --discard ──► Hypothesis INVALIDATED
                                          │
                                    Update EPOS §2.2:
                                    status: validated | invalidated
                                    validation_date: YYYY-MM-DD
                                    evidence: "spike result summary"
```

### 6.2 Milestone Lifecycle (EPOS §2.3 ↔ ApexYard)

```
  EPOS §2.3                              ApexYard
  ────────                              ────────

  Milestone: "Demonstrate 3-month                 │
  retention ≥ 40% for solar-plant segment"        │
  (status: at-risk)                               │
       │                                          │
       │  ┌──────────────────────┐               │
       ├──│ /roadmap add         │──────────────►│ Milestone in
       │  │ "3-month retention   │               │ projects/<name>/roadmap.md
       │  │  40% by Q3 2026"    │               │
       │  └──────────────────────┘               │
       │                                          │
       │  Linked metrics:                         │
       │  - met-lean-w4-ret (retention)          │
       │  - met-lean-activation (activation)     │
       │                                          │
       │  When a spike validates a hypothesis     │
       │  that moves this metric:                 │
       │  ┌──────────────────────┐               │
       ├──│ /spike-close --promote│──► [Feature] │ │
       │  └──────────────────────┘    ticket      │
       │                              linked to   │
       │                              milestone   │
       │                                          │
       ▼                                          ▼
  EPOS §7 weekly review:          /stakeholder-update:
  milestone status updated         narrative includes
  (on-track → at-risk →            milestone progress
   off-track → met)
```

### 6.3 Prioritization (EPOS §8 ↔ ApexYard Labels)

EPOS computes a weighted score. ApexYard uses categorical labels. The mapping:

```
EPOS §8 Blended Score         ApexYard Label    What It Means
──────────────────────         ──────────────    ────────────

score >= 4.0 (top quartile)   → P0              Must-have for current milestone
score 2.5 - 3.9               → P1              Ship soon after current milestone
score 1.0 - 2.4               → P2              Future / v2+
score < 1.0                   → Deprioritize    Park or kill

For your stage (Problem-Solution Fit / early PMF):
  w_learn = 0.55  → Learning value dominates
  w_flow  = 0.15  → Flow improvements take a back seat
  w_econ  = 0.10  → Economics barely matter yet
  w_cost  = 0.20  → Keep costs low

Example scoring:
  "Test weather API integration" → learning_value=5, flow=1, econ=2, effort=2
  score = 0.55×5 + 0.15×1 + 0.10×2 − 0.20×2 = 2.75 + 0.15 + 0.20 − 0.40 = 2.70
  → P1

  "Refactor auth middleware"      → learning_value=1, flow=4, econ=1, effort=3
  score = 0.55×1 + 0.15×4 + 0.10×1 − 0.20×3 = 0.55 + 0.60 + 0.10 − 0.60 = 0.65
  → Deprioritize (at this stage, auth refactor doesn't teach us much)

Contrast with the Scale stage:
  "Refactor auth middleware"      → w_learn=0.10, w_flow=0.35, w_econ=0.35, w_cost=0.20
  score = 0.10×1 + 0.35×4 + 0.35×1 − 0.20×3 = 0.10 + 1.40 + 0.35 − 0.60 = 1.25
  → P2 (starts to look more attractive at scale)
```

---

## Part 7 — EPOS §4 Engineering Practices ↔ ApexYard Hooks

EPOS §4 defines engineering standards declaratively. ApexYard enforces them mechanically.

| EPOS §4 Requirement | ApexYard Enforcement | Hook |
|---|---|---|
| "Merge rules: peer review, CI green" | `block-unreviewed-merge.sh` requires Rex + CEO approval markers | PreToolUse |
| "No direct pushes to main" | `block-main-push.sh` | PreToolUse |
| "Never stage all files" | `block-git-add-all.sh` | PreToolUse |
| "Secrets never in source control" | `check-secrets.sh` | PreToolUse |
| "Branch naming convention" | `validate-branch-name.sh` | PreToolUse |
| "Commit format convention" | `validate-commit-format.sh` | PreToolUse |
| "Tests pass before push" | `pre-push-gate.sh` | PreToolUse |
| "PR has glossary + testing sections" | `validate-pr-create.sh` | PreToolUse |
| "Architecture decisions require AgDR" | `require-agdr-for-arch-changes.sh` | PreToolUse |
| "No code edits without ticket" | `require-active-ticket.sh` | PreToolUse |
| "Ticket creation only through skills" | `require-skill-for-issue-create.sh` | PreToolUse |
| "Migration edits require migration ticket" | `require-migration-ticket.sh` | PreToolUse |
| "Security review on auth/crypto diffs" | `detect-role-trigger.sh` → Hatim activates | PreToolUse |
| "Design review on UI diffs" | `require-design-review-for-ui.sh` | PreToolUse |

### What EPOS Asks That ApexYard Doesn't Enforce (Yet)

| EPOS §4 Question | Gap | Suggestion |
|---|---|---|
| "What is the on-call path?" | No hook validates on-call config | Store in `projects/<name>/epos/product-instance.yaml` engineering.ownership |
| "What are the SLOs?" | `/launch-check` monitors dimension but no SLO enforcement hook | Define in EPOS; implement alerting in Datadog |
| "What is the tech-debt register?" | `/task` creates tickets but no debt tracking | Label tech-debt tickets with `tech-debt`; track in GitHub project board |
| "15-25% capacity for non-feature work?" | No capacity-allocation hook | Use EPOS weekly individual sheet to track time allocation by work type |
| "Definition of Done includes rollback plan?" | Not enforced by hook | Add to `onboarding.yaml` quality section or `.claude/project-config.json` PR required sections |

---

## Part 8 — The Complete Decision Tree

When you have an idea and aren't sure what to do next:

```
I have an idea.
│
├── Is it a raw hypothesis (unvalidated)?
│   │
│   ├── YES → /idea "description"
│   │         │
│   │         ├── /validate-idea IDEA-NNN
│   │         │   ├── GREEN → Ready for experiment
│   │         │   │   └── /spike "test this hypothesis"
│   │         │   │       └── /spike-close
│   │         │   │           ├── --promote → /feature (build production)
│   │         │   │           └── --discard → memo (hypothesis rejected)
│   │         │   ├── YELLOW → Park, revisit in N days
│   │         │   └── RED → Kill (WONTDO in backlog)
│   │         │
│   │         └── Or skip validation if you have strong conviction:
│   │             └── /spike directly
│   │
│   └── NO (it's already validated or obvious)
│       │
│       ├── Does it need a product spec?
│       │   │
│       │   ├── YES (new feature, major scope) → /write-spec "description"
│       │   │                                       └── /feature from spec
│       │   │
│       │   └── NO (small, well-understood) → /feature "title" directly
│       │                                           or /task "title" for tech debt
│
├── Is it a bug?
│   └── /bug "description" (Given/When/Then + repro + severity)
│
├── Is it a technical decision?
│   └── /decide "what to choose"
│       └── Creates AgDR (required before architecture changes)
│
├── Is it a database migration?
│   └── /migration "description"
│       └── Creates migration ticket + migration AgDR
│           (hook blocks migration file edits until ticket exists)
│
├── Is it an investigation (root cause unknown)?
│   └── /investigation "description"
│       └── Creates live-doc with hypotheses, evidence, follow-ups
│
├── Is it pure exploration with no clear hypothesis?
│   └── /spike "exploration topic" (budget: 1-2 days max)
│       └── Still requires: hypothesis, budget, kill criteria, disposition
│
└── Not sure? Ask:
    "Can I answer this with reasoning alone?"
    ├── YES → /task or /feature (well-defined, no experiment needed)
    └── NO → /spike (genuine uncertainty, need to build to learn)
```

---

## Part 9 — Quick Reference: EPOS-ApexYard Vocabulary Translation

| You're Thinking (EPOS language) | What to Type (ApexYard command) |
|---|---|
| "We have a hypothesis to test" | `/spike "We believe X. We will know we're right when Y."` |
| "We need to validate this idea quickly" | `/validate-idea IDEA-NNN` |
| "Just capture this idea, no commitment" | `/idea "description"` |
| "This idea passed validation, spec it" | `/write-spec "feature description"` |
| "We've spec'd it, create the ticket" | `/feature "feature title"` |
| "What stage should we mark this product?" | Create `projects/<name>/epos/product-instance.yaml` with `stage: <stage>` |
| "What's our blended prioritization score?" | Compute using EPOS §8 weights; store in `projects/<name>/epos/prioritization.yaml` |
| "Track this milestone" | `/roadmap add` |
| "What's our DORA looking like?" | `/launch-check <project>` (monitoring dimension) |
| "Is our analytics coverage good enough?" | `/launch-check <project>` (analytics dimension) |
| "What needs attention this week?" | `/inbox` or `/tasks` |
| "Give me a status update" | `/status` |
| "Generate a weekly stakeholder update" | `/stakeholder-update` (enrich with EPOS structure) |
| "We need to decide on a tech choice" | `/decide "question"` |
| "Then spike to test it" | `/spike "test this tech choice"` |
| "Spike confirmed, build production" | `/spike-close --promote` → `/feature` |
| "Spike rejected, record what we learned" | `/spike-close --discard` → memo in `docs/spike-memos/` |
| "Review architecture before building" | `/c4 <project>` + `/dfd <project>` |
| "Review security before launch" | `/threat-model <project>` |
| "Full production readiness check" | `/launch-check <project>` |

---

## Part 10 — Template: EPOS Product Instance YAML for Sunapse

This is what the EPOS product instance looks like when stored alongside ApexYard:

```yaml
# projects/myagent/epos/product-instance.yaml
# EPOS v1 instance for Sunapse — kept beside ApexYard project docs
# Updated: 2026-05-21

product:
  id: prd-sunapse
  name: Sunapse
  code_name: sunapse
  domain: web-app
  capabilities:
    - Ollama chat with system prompt
    - (more as features land)
  upstream_dependencies:
    - Ollama API
    - Open weather APIs (planned)
  downstream_consumers: []
  ownership:
    product_owner: "Muhammad Hassan"
    tech_lead: "Muhammad Hassan"
    service_owner: "Muhammad Hassan"
    business_stakeholder: "Muhammad Hassan"
  segments:
    - name: "PV solar plant operators"
      type: user
      jtbd: "Manage battery charge/discharge timing to maximise energy savings"
      success_definition: "Reduced energy cost vs manual operation"
      current_size: { value: 0, unit: "active_accounts" }
    - name: "Residential solar owners"
      type: user
      jtbd: "Automate home energy management"
      success_definition: "Lower electricity bill"
      current_size: { value: 0, unit: "active_accounts" }
  business_model:
    value_creation: "Optimise battery charge/discharge timing using weather + demand forecasting"
    value_capture: "SaaS subscription (planned)"
    monetization: subscription
    primary_value_metrics: [met-lean-activation]
  constraints:
    regulatory: []
    technical:
      - "Depends on Ollama API availability"
      - "Single developer capacity initially"
    stage: pmf-search

strategy:
  stage:
    current: pmf-search
    critical_risks: [desirability, feasibility]
    exit_criterion: "20 active users with W4 retention ≥ 40%"
    pivot_or_kill_conditions:
      - "No organic signups after 3 months of active promotion"
      - "Users don't return after first week"

  hypotheses:
    - id: hyp-sunapse-001
      statement: "We believe that aggregating weather forecasts with PV production data will save solar plant operators ≥ 15% on energy costs"
      target_metric: met-lean-activation
      threshold: { direction: gte, value: 20 }
      falsification_condition: "Fewer than 5 plant operators sign up for a free trial after 8 weeks of availability"
      owner: prs-muhammad-hassan
      status: open
      created: 2026-05-19
      deadline: 2026-07-19

    - id: hyp-sunapse-002
      statement: "We believe that BMS users will prefer an API-driven automated controller over manual setpoints"
      target_metric: met-lean-activation
      threshold: { direction: gte, value: 0.5 }
      falsification_condition: "Fewer than 50% of trial users enable auto-mode after 2 weeks"
      owner: prs-muhammad-hassan
      status: open
      created: 2026-05-19
      deadline: 2026-08-19

  innovation_accounting:
    target_bml_cycle_days: 14
    last_n_weeks_validation_rate:
      window_weeks: 4
      validated: 0
      invalidated: 0
      inconclusive: 0
      carried_over: 2

  milestones:
    - id: mil-sunapse-001
      statement: "20 active users with W4 retention ≥ 40%"
      type: learning
      gates_stage_transition: true
      leading_indicators: [met-lean-activation]
      lagging_indicators: [met-lean-w4-ret]
      target_date: 2026-08-01
      status: at-risk

  guardrails:
    pre_scale_conditions:
      - "W4 retention ≥ 40% for target segment"
      - "At least 3 validated hypotheses"
    target_ltv_cac: { min: 3 }
    runway_floor_months: 18

prioritization:
  active_constraint:
    flow_constraint: capacity
    discovery_delivery_split_pct: { discovery: 50, delivery: 50 }
    system_capacity_reserve_pct: 15
  weights:
    by_stage:
      pmf-search: { w_learn: 0.40, w_flow: 0.25, w_econ: 0.15, w_cost: 0.20 }
```

---

## Part 11 — Summary: The Two Systems as One Pipeline

```
    STRATEGIZE          VALIDATE            EXPERIMENT           BUILD             MEASURE
    ──────────          ─────────           ───────────          ─────             ───────
    EPOS §1-2           EPOS §2.2           EPOS §2.2            EPOS §3-4         EPOS §5-6
    Product context      Hypothesis          Experiment           SDLC              Metrics
    + strategy           validation           (spike)             (full gates)      + health

    ┌─────────┐    ┌───────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
    │ /setup   │    │/validate- │    │   /spike      │    │/start-ticket │    │/launch-check │
    │/c4       │    │  idea     │    │              │    │/feature      │    │/stakeholder- │
    │/extract- │    │           │    │ hone·budget· │    │/task         │    │  update      │
    │ features │    │ 5Q gate   │    │ kill·dispose │    │/bug          │    │/status       │
    │/write-   │    │           │    │              │    │/migration    │    │/tasks        │
    │  spec    │    │ GREEN     │    │ PROMOTE ───► │    │              │    │/inbox        │
    │/decide   │    │ YELLOW    │    │ DISCARD ───► │    │ full SDLC:   │    │              │
    │/roadmap  │    │ RED       │    │ memo only    │    │ branch → PR  │    │ EPOS weekly   │
    └─────────┘    └───────────┘    └──────────────┘    │ → merge → QA │    │ review ritual │
                                                          └──────────────┘    └──────────────┘

     ApexYard          ApexYard           ApexYard            ApexYard           ApexYard
     + EPOS §1-2       + EPOS §2.2        + EPOS §2           + EPOS §3-4       + EPOS §5-7

     ←──── PLANNING PHASE (EPOS-heavy) ────►←── EXECUTION PHASE (ApexYard-heavy) ──►←── FEEDBACK ──►

     For a 2-developer team with unvalidated hypotheses:
     ◄════════════ MOST OF YOUR TIME IS HERE ════════════►
     ◄═══ /idea → /validate-idea → /spike → /spike-close ═══►
     ◄═══ Then /feature only for validated hypotheses ═══►
```

**Your ratio at the Problem-Solution Fit / early PMF stage should be roughly:**

- **50% of capacity** → `/spike` work (validating hypotheses, running experiments)
- **30% of capacity** → `/feature` work (building what's already validated)
- **15% of capacity** → `/task` work (infra, tech debt, tooling that unblocks flow)
- **5% of capacity** → `/bug` work (fixing what's broken)

As you validate hypotheses and move toward Scale, the `/spike` ratio drops and `/feature` + `/task` increase. EPOS §8 gives you the exact weights for your current stage.

**The weekly feedback loop closes the circle:** every `/spike-close` result updates EPOS §2.2 (hypothesis status), which feeds into §8 (re-prioritization), which determines next week's `/spike` and `/feature` mix. This is how the two systems become one operating rhythm.