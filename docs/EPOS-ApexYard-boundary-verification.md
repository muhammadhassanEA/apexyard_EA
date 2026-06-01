# EPOS × ApexYard — Boundary Verification

> Verifying how the two systems connect when EPOS owns product strategy
> (hypotheses, metrics, prioritization) in Asana and ApexYard owns engineering
> execution (SDLC, code, reviews, gates) per-developer on the same project.

---

## The Architecture You Described

```
┌─────────────────────────────────────────────────────────────────────┐
│  EPOS (Asana)                                                       │
│  The canonical place for:                                           │
│  • Product context (§1)                                             │
│  • Strategy & hypotheses (§2)                                       │
│  • Milestones (§2.3)                                               │
│  • Metrics & innovation accounting (§5, §2.2)                      │
│  • Prioritization scores (§8)                                       │
│  • Weekly review packets (§7)                                       │
│  • Individual sheets (§6)                                           │
│                                                                     │
│  Asana = source of truth for WHAT and WHY                           │
│  Asana tasks = hypothesis IDs, experiment IDs, milestone IDs       │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                 The Seam: task ID cross-reference
                 (Asana task URL ↔ GitHub Issue number)
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  ApexYard (per developer fork)                                      │
│  The canonical place for:                                           │
│  • Ticket creation (/feature, /bug, /task, /spike)                  │
│  • Code SDLC (branch → code → review → merge → QA)                 │
│  • Mechanical enforcement (hooks, gates, reviews)                    │
│  • Architecture decisions (AgDRs)                                   │
│  • Per-project docs (C4, DFD, threat models)                        │
│                                                                     │
│  GitHub Issues = source of truth for HOW (code, reviews, gates)     │
└─────────────────────────────────────────────────────────────────────┘
```

**The key boundary**: Asana owns *what to build and why* (EPOS). GitHub Issues own *how it's built and whether it passes gates* (ApexYard). They connect via cross-references in the ticket body.

---

## What Works As-Is

### 1. Dual-tracker model — Asana + GitHub Issues

ApexYard already supports this. The `ticket-manager` agent says explicitly:

> "Teams that prefer a different tracker (Linear, Jira, etc.) can substitute the equivalent commands — but adopt this only as a deliberate deviation from the default."
> — `.claude/agents/ticket-manager.md`

And the `require-skill-for-issue-create.sh` hook blocks `asana task create` by default (it's in the `create_command_patterns` list), which means structured ticket creation through ApexYard is enforced regardless of which tracker you use. The hook blocks *raw* `asana task create` but *allows* it when a skill (`/feature`, `/task`, etc.) is in flight.

**Your workflow:**
```
1. EPOS process (Asana):
   - Create hypothesis in Asana (e.g., "We believe X")
   - Create experiment task in Asana (e.g., "Test X with spike")
   - Assign to developer
   - Track progress in Asana

2. When a developer picks up an Asana task:
   /start-ticket GH-42             ← ApexYard tracks the GitHub side
   # OR
   /spike "Test hypothesis X"      ← Creates GitHub Issue + links to Asana

3. Cross-reference:
   - In the GitHub Issue body: "Asana: https://asana.com/task/120345"
   - In the Asana task body: "GitHub: muhammadhassanEA/MyAgent#42"
```

### 2. Each developer has their own Apex fork

This is the intended model. Each developer:
- Has their own fork of `me2resh/apexyard` (or your org's fork)
- Has their own `onboarding.yaml` (can customize quality bar, pre-push commands)
- Has their own `.claude/project-config.json` (can add custom skills, trackers)
- Has their own `workspace/<name>/` clone of the project
- Shares the same `apexyard.projects.yaml` (committed, identical across forks)

**Customization each developer can make independently:**
| File | Per-developer | Commits to fork? |
|---|---|---|
| `onboarding.yaml` | Yes (team can share) | Yes |
| `.claude/project-config.json` | Yes | Yes |
| `projects/<name>/` docs | Yes (shared via git) | Yes |
| `.claude/session/` | Ephemeral (gitignored) | No |
| `custom-skills/` (split-portfolio) | Per-developer possible | Yes |
| `workspace/<name>/` local clone | Per-developer (gitignored) | No |
| Pre-push commands | Per developer in project-config | Yes |

**What must stay synchronized across developers:**
- `apexyard.projects.yaml` — same registry, same project entries
- `.claude/hooks/` — same enforcement rules (or different, per developer preference)
- `.claude/settings.json` — same hook wiring (or customized)

### 3. Asana integration in ApexYard

ApexYard doesn't have a native Asana skill, but the framework is tracker-agnostic in three places:

| Where | What Adapts | How |
|---|---|---|
| `ticket.create_command_patterns` | Blocked raw CLI commands | `asana task create` is already in the default block list |
| Branch validation | `validate-branch-name.sh` | Already accepts `[A-Z]+-[0-9]+` (covers `ASANA-42`, `GH-42`, etc.) |
| PR validation | `validate-pr-create.sh` | Same regex — accepts any uppercase prefix |
| `ticket_prefix` | Per-project in `apexyard.projects.yaml` | Change from `GH` to `ASANA` or keep `GH` for GitHub Issues |

**Recommendation: Keep both trackers. Use GitHub Issues for code-tracking (ApexYard's enforcement layer) and Asana For product-tracking (EPOS's strategy layer). Cross-reference URLs in ticket bodies.** Do not try to make ApexYard create Asana tasks — the enforcement hooks (branch names, PR titles, commit refs) all use GitHub Issues, and replacing them with Asana would break the mechanical gates.

### 4. The EPOS → ApexYard handoff points

| EPOS Artifact | Where It Lives | How It Connects to ApexYard |
|---|---|---|
| Hypothesis (`hyp-sunapse-001`) | Asana task | Referenced in `/spike` ticket body: `"Testing hypothesis: hyp-sunapse-001 (Asana: https://asana.com/task/120345)"` |
| Experiment (`exp-sunapse-001`) | Asana task | 1:1 with GitHub spike issue. Asana task tracks the business question; GitHub Issue tracks the code execution. |
| Milestone (`mil-sunapse-001`) | Asana project/section | Referenced in `/roadmap` and in feature ticket ACs: `"Milestone: mil-sunapse-001 (Asana: https://asana.com/task/120346)"` |
| Metric (`met-lean-activation`) | Asana (custom field or goal) | Referenced in `/launch-check` audit and `/feature` ticket ACs |
| Priority (EPOS §8 score) | Asana (priority field) | Maps to ApexYard P0/P1/P2/P3 labels on GitHub Issues |
| Individual sheet | Asana (task assignment + time tracking) | Cross-referenced with `/status` and `/tasks` |
| Weekly review | Asana project / dashboard | Synthesized from `/stakeholder-update` output enriched with EPOS structure |

---

## What Needs Adaptation

### Adaptation 1: The Ticket Prefix

Your `apexyard.projects.yaml` currently uses `ticket_prefix: GH` for all projects. If you want Asana task IDs in branch names and PR titles, change it to whatever Asana uses (typically a numeric ID or a custom prefix like `ASANA-`). But since ApexYard's enforcement hooks use GitHub Issues for validation (they call `gh issue view` to verify tickets exist), you should **keep GitHub Issues as the code-tracking tracker** and use Asana only for strategy/management.

**Recommended setup:**
```
apexyard.projects.yaml:
  ticket_prefix: GH    ← Keep this. GitHub Issues are the code tracker.

Asana task body:       "GitHub Issue: muhammadhassanEA/MyAgent#42"
GitHub Issue body:     "Asana task: https://app.asana.com/0/120345/120346"
```

This way:
- ApexYard hooks validate GitHub Issue numbers (they call `gh issue view`)
- Branch names use `feature/GH-42-...` (validated by `validate-branch-name.sh`)
- PR titles use `feat(#42): ...` (validated by `validate-pr-create.sh`)
- Asana tracks the EPOS metadata (hypothesis, experiment, metric, priority)
- GitHub tracks the code metadata (branch, PR, review, merge, CI)

### Adaptation 2: The EPOS Product Instance Files

ApexYard doesn't have native support for EPOS YAML instances. You need a place to store them. Two options:

**Option A: Inside the ApexYard ops repo (recommended)**
```
projects/myagent/
  epos/
    product-instance.yaml       ← §1-§6 consolidated
    hypotheses.yaml             ← §2.2 hypothesis registry
    metrics.yaml                 ← §5 metric definitions
    weekly-reviews/
      2026-W22.yaml              ← §7 weekly review packet
  handover-assessment.md         ← ApexYard native
  architecture/
    container.md                ← ApexYard native
  docs/
    agdr/                        ← ApexYard native
```

Pros: Version-controlled alongside code, accessible to ApexYard skills.
Cons: If the project is private, these files are visible (use split-portfolio mode).

**Option B: Inside Asana (as custom fields)**
```
Asana project: Sunapse
  Custom fields:
    EPOS Stage: (dropdown) Problem-Solution Fit / PMF / GTM / Scale
    Hypothesis ID: (text) hyp-sunapse-001
    Metric: (text) met-lean-activation
    Priority Score: (number) calculated from §8 weights
```

Pros: Single source of truth for EPOS data, team-visible.
Cons: Not accessible to ApexYard skills, can't be version-controlled.

**Recommendation: Use both.** Store the EPOS YAML instances in the ops repo (Option A) for version control and ApexYard access. Sync key fields (stage, hypothesis status, priority) into Asana custom fields for team visibility. The weekly review ritual updates both.

### Adaptation 3: Per-Developer Customization

Each developer's fork can have its own configuration. Here's how to set it up for a 2-developer team:

```
Developer 1 (Tech Lead / Hisham — you):
  .claude/project-config.json:
    {
      "ticket": { "prefix_whitelist": ["Feature", "Bug", "Chore", "Spike", ...] },
      "pre_push": {
        "commands": ["pytest", "ruff check .", "mypy ."]
      },
      "ui_paths": [""],
      "custom additions":
        - Handbooks for review style (thorough)
        - Custom skill: /epos-sync (push weekly metrics to Asana)
    }

Developer 2 (Backend Engineer / Karim):
  .claude/project-config.json:
    {
      "ticket": { ... same whitelist ... },
      "pre_push": {
        "commands": ["pytest", "ruff check .", "mypy ."]
      },
      - Same hooks, same enforcement
      - Maybe lighter review style preference
    }
```

**Custom skills** are the main per-developer customization point. A developer can create:
```
custom-skills/                     ← Developer-specific skills directory
  epos-sync/SKILL.md               ← Pushes metrics from YAML to Asana
  weekly-review/SKILL.md            ← Reads Asana tasks + GitHub issues, generates EPOS §7 packet
```

These get symlinked into `.claude/skills/` at session start by `link-custom-skills.sh`.

### Adaptation 4: Spike ↔ Hypothesis Cross-Reference

The `/spike` skill doesn't have a built-in field for "EPOS hypothesis ID". You need to add it manually in the spike ticket body:

```
/spike "Test weather API integration for BMS optimization"

# In the spike body, add:
## EPOS Cross-Reference
- Hypothesis: hyp-sunapse-001
- Asana task: https://app.asana.com/0/120345/120346
- Target metric: met-lean-activation
- Innovation accounting cycle: 2-week window
```

This is a manual addition, not automated. If you want to automate it, you'd create a custom skill that:
1. Reads the EPOS hypothesis ID from `projects/<name>/epos/hypotheses.yaml`
2. Pre-fills the spike body with the cross-reference
3. On `/spike-close`, updates the hypothesis status in the YAML file

---

## The Complete Workflow: Asana-Driven EPOS + ApexYard-Driven Engineering

### Phase 1: Product Strategy (Asana / EPOS)

```
EPOS §1-2: Define the product and its hypotheses in Asana
────────────────────────────────────────────────────────────
1. Product Manager creates Asana project "Sunapse"
2. Creates sections for each EPOS stage: "Problem-Solution Fit"
3. Creates tasks for each hypothesis:
   - Task: "hyp-sunapse-001: Weather aggregation saves 15% on energy"
   - Custom fields: Stage, Metric, Status (open/validated/invalidated)
   - Priority: EPOS §8 weighted score
4. Creates milestones for §2.3:
   - Milestone: "mil-sunapse-001: 20 active users with W4 retention ≥ 40%"
   - Due date, leading/lagging indicators
5. Assigns developers based on EPOS §8 prioritization
```

### Phase 2: Handoff to Engineering (Asana → GitHub Issues)

```
Developer picks up an Asana task → creates matching GitHub Issue
────────────────────────────────────────────────────────────
Developer opens ApexYard (their fork):

# For a hypothesis that needs testing:
/spike "Test weather API integration for BMS optimization"

# The spike is created in GitHub with cross-references:
# Body includes:
#   "EPOS Hypothesis: hyp-sunapse-001
#    Asana: https://app.asana.com/0/120345/120346
#    Budget: 2 days
#    Kill criteria: <1% energy savings in simulation
#    Disposition: PROMOTE"

# For validated hypotheses moving to production:
/feature "Integrate weather forecast data into BMS charge controller"

# Body includes:
#   "EPOS Hypothesis: hyp-sunapse-001 (validated via spike GH-5)
#    Asana: https://app.asana.com/0/120345/120347
#    Milestone: mil-sunapse-001
#    Target metric: met-lean-activation"

# Link is bidirectional:
#   Asana task body → "GitHub: muhammadhassanEA/MyAgent#7"
#   GitHub Issue body → "Asana: https://app.asana.com/0/120345/120347"
```

### Phase 3: Engineering Execution (ApexYard-Enforced SDLC)

```
Developer works through the ApexYard SDLC:
────────────────────────────────────────────────────────────
/start-ticket GH-7                    ← Declare active (required by hook)

git checkout -b feature/GH-7-weather-integration   ← Branch naming enforced

# ... code, code, code ...

git add src/weather.py tests/test_weather.py       ← Hook blocks git add -A
git commit -m "feat(GH-7): integrate weather API"  ← Commit format enforced
git push origin feature/GH-7-weather-integration   ← Pre-push gate runs

gh pr create --title "feat(#7): integrate weather API"   ← PR format enforced
                                              ← Auto-triggers Rex code review

# In Asana: update task status to "In Review"
/approve-merge 7                      ← CEO approves (per-PR, explicit)

# After merge:
gh issue edit GH-7 --add-label "qa"  ← QA verification (not auto-closed)

# After QA signs off:
gh issue close GH-7                   ← Now done

# In Asana: update hypothesis task status, update metric values
```

### Phase 4: Measurement Back to EPOS (GitHub → Asana)

```
Weekly review ritual:
────────────────────────────────────────────────────────────
1. Developer fills EPOS Individual Sheet (§7.2) in Asana:
   - Time allocation this week: 40% feature, 30% spike, 20% infra, 10% learning
   - Valuable work %: 70%
   - Top accomplishment: "Validated hyp-sunapse-001 via spike GH-5"
   - Blockers: "Ollama API rate limiting"
   - Burnout self-rating: 3/5

2. ApexYard generates the engineering side:
   /status                              ← Git state, open PRs
   /tasks                               ← Actionable items with URLs
   /stakeholder-update                  ← Narrative summary

3. Team lead merges ApexYard output into EPOS weekly review:
   - DORA metrics from CI (deployment count, lead time, CFR)
   - Hypothesis dispositions from /spike-close results
   - Milestone progress from GitHub Issues closed
   - Updated metric values (manually or from analytics)

4. EPOS §7 weekly review produces:
   - Product Sheet (one per product per week)
   - Individual Sheet (one per developer per week)
   - Leader Dashboard (aggregated)
```

---

## Boundary Clarification: What EPOS Owns vs What ApexYard Owns

```
                    EPOS OWNS                          APEXYARD OWNS
                    ─────────                           ─────────────
                    Product context (§1)                Code SDLC (§3 of EPOS ≈ all of ApexYard)
                    Strategy & hypotheses (§2)           Branch naming, commit format, PR format
                    Milestones & stage-gates (§2.3)     Pre-push gates (lint, test, build)
                    Metrics model (§5)                   Code review (Rex + human)
                    People & team health (§6)            Security review (Hatim)
                    Weekly assessment (§7)              Merge gates (CEO approval)
                    Prioritization scoring (§8)          Ticket-first enforcement
                    Exceptions (§9)                      AgDR enforcement
                    Front page (§10)                     Architecture docs (C4, DFD)

                    SHARED (requires manual sync)
                    ─────────────────────────────────
                    Priority labels: EPOS §8 score → GitHub P0/P1/P2/P3
                    Hypothesis status: /spike-close → EPOS §2.2 update
                    Sprint goals: EPOS weekly review → GitHub milestones
                    Developer time: EPOS individual sheet ↔ Asana/Calendar
```

---

## Verdict: Does This Work?

**Yes, this works.** Here's the summary:

### What works perfectly out of the box:

1. **Each developer has their own Apex fork** — this is the intended model. `onboarding.yaml` can be shared (committed to the fork) or per-developer.

2. **ApexYard's mechanical enforcement is tracker-agnostic** — `asana task create` is blocked by the skill gate, same as `gh issue create`. The structured ticket skills (`/feature`, `/spike`, etc.) create GitHub Issues, which is exactly what you want for code-tracking.

3. **The boundary between EPOS (Asana) and ApexYard (GitHub Issues)** is clean: Asana owns the *what* and *why*, GitHub owns the *how* and *whether*. Cross-reference URLs in ticket bodies.

4. **Developers can customize their own forks** — add custom skills, adjust `project-config.json`, modify handbooks. The shared enforcement (hooks, rules) stays in the fork's `.claude/` directory and propagates via `/update`.

5. **The `/spike` ↔ EPOS hypothesis mapping** works naturally — spikes have hypothesis, budget, kill criteria, and disposition fields that map 1:1 to EPOS §2.2 experiment objects.

### What needs manual setup (not built-in):

1. **EPOS YAML instances in `projects/<name>/epos/`** — create these manually (or write a custom `/epos-sync` skill). They're not generated by any existing skill.

2. **Cross-reference URLs** — add Asana links to GitHub Issue bodies and vice versa. No skill automates this yet.

3. **Priority mapping** — EPOS §8 produces a numeric score; GitHub Issues use P0/P1/P2/P3 labels. The mapping is manual (or write a custom skill that reads `hypotheses.yaml` and sets GitHub Issue labels).

4. **Metrics collection** — EPOS §5 DORA metrics need a data pipeline. Start manual, automate later with GitHub Actions + Datadog.

5. **Weekly review generation** — `/stakeholder-update` produces a narrative but not a structured EPOS §7 dashboard. Enrich manually with EPOS YAML data.

### What to watch for:

1. **Don't duplicate tickets** — Asana tasks and GitHub Issues should be 1:1. One Asana task maps to one GitHub Issue. Never create two GitHub Issues for one Asana task, or vice versa.

2. **Status synchronization** — When a GitHub Issue moves to `qa`, the Asana task should move to "In QA". This is manual unless you build a webhook/sync.

3. **Don't fight the ApexYard gates** — They're there for a reason. If Asana says "start coding" but ApexYard says "no active ticket", the hook blocks. The flow is: Asana task → GitHub Issue (`/feature` or `/spike`) → `/start-ticket` → code. Never skip the GitHub Issue creation step.

4. **Spike exemptions are intentional** — When EPOS says "run an experiment" (Problem-Solution Fit stage), use `/spike`. Spikes skip AgDR gates and 80% coverage because the code is throwaway. Don't use `/feature` for experiments.

5. **The `/start-ticket` requirement is absolute** — Even if Asana has a task assigned to you, ApexYard's `require-active-ticket.sh` hook will block code edits until you run `/start-ticket GH-42` to declare which GitHub Issue you're working on. This is by design: it enforces the "every code change has a tracker reference" rule.

---

## Quick Reference: The Developer's Daily Flow

```
MORNING:
  1. Check Asana: What tasks are assigned to me this sprint?
  2. Check /inbox: What PRs, issues, comments need my attention?
  3. Pick top task from Asana (prioritized by EPOS §8 score)

STARTING WORK:
  4. Create GitHub Issue if not already created:
     /spike "Test hypothesis X"       ← if experiment
     /feature "Build feature Y"       ← if validated
     /task "Refactor auth middleware"  ← if tech debt
  5. Link Asana task URL in GitHub Issue body
  6. Add GitHub Issue URL in Asana task body
  7. /start-ticket GH-42              ← Required before any code edits

DURING WORK:
  8. Code on branch (name enforced by hook)
  9. Commit (format enforced by hook)
  10. Push (pre-push gate: lint, test, build)
  11. Create PR (format enforced by hook)
  12. Rex auto-reviews

REVIEW & MERGE:
  13. Address Rex feedback
  14. CEO approves: /approve-merge PR-N
  15. Merge (hook checks: Rex approved? CEO approved? CI green?)

AFTER MERGE:
  16. Update Asana task status
  17. If spike: /spike-close --promote or --discard
  18. Update EPOS hypotheses.yaml if hypothesis validated/invalidated

FRIDAY REVIEW:
  19. Fill EPOS Individual Sheet in Asana (time, blockers, burnout)
  20. /stakeholder-update → enrich with EPOS metrics
  21. Prioritize next week using EPOS §8 blended score
```