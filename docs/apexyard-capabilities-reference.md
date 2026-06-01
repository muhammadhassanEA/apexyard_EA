# ApexYard Capabilities & Usage Reference

> Flash-card format: each card is self-contained. Read any card in isolation.

---

## CARD 1 — What Is ApexYard?

ApexYard is a **multi-project governance framework** for Claude Code.
You fork it once, register all your repos in a portfolio registry, and get a **Chief-of-Staff agent** that enforces SDLC rules mechanically (hooks, not just prose).

Three layers of enforcement:
| Layer | What | Example |
|---|---|---|
| **Hooks** (shell scripts) | Run *before* tool calls; block or allow | `require-active-ticket.sh` blocks edits with no ticket |
| **Rules** (.md files) | Self-discipline rules Claude follows | "enter plan mode when >= 4 steps" |
| **Agents** (sub-personalities) | Specialized reviewers invoked at PR time | Rex reviews code; Hatim reviews security |

---

## CARD 2 — The Portfolio Model

```
Your Company (ops repo = fork of me2resh/apexyard)
├── apexyard.projects.yaml   ← THE registry: lists every project
├── projects/myagent/        ← Per-project docs (ADRs, architecture, etc.)
├── projects/bacnet-sim/      ← Another project's docs
├── workspace/myagent/        ← Live git clone (gitignored)
└── .claude/session/          ← Runtime markers (active ticket, skill state)
```

| Concept | File | Purpose |
|---|---|---|
| **Ops repo** | Root of your fork | Single governance hub for all projects |
| **Portfolio registry** | `apexyard.projects.yaml` | Lists name, repo, status, tier, tags for every managed project |
| **Per-project docs** | `projects/<name>/` | ADRs, architecture, feature inventories — committed to ops repo |
| **Workspace** | `workspace/<name>/` | Optional live clone of a managed project's repo — gitignored |
| **Session state** | `.claude/session/` | Ephemeral markers: active-ticket, active-bootstrap, active-issue-skill |

**Two modes:**

| Mode | Layout | When |
|---|---|---|
| **Single-fork** (default) | One repo = fork = everything | All projects public, or GitHub Pro/Team/Enterprise |
| **Split-portfolio** | Public fork + private sibling repo | GitHub Free with private projects — zero leaks to public fork |

Switch with `/split-portfolio` (one-way, destructive recovery with confirm gates).

---

## CARD 3 — Key Jargon Glossary

| Term | Meaning |
|---|---|
| **AgDR** | Agent Decision Record — what options were considered, why we chose X, and context |
| **Rex** | Code Reviewer agent — reviews every commit on every PR |
| **Hatim** | Security Reviewer agent — triggered on auth/crypto/secrets diffs |
| **Munir** | Dependency Auditor agent |
| **Tariq** | PR Manager agent |
| **Idris** | Ticket Manager agent |
| **Ops repo** | Your fork of apexyard — the governance hub |
| **Ticket-first** | No code edits without a real GitHub issue; enforced by hooks |
| **CEO approval** | Per-PR explicit merge authorization via `/approve-merge`; plan-level "go" does NOT authorize merges |
| **Bootstrap skills** | `/setup`, `/handover`, `/update`, `/split-portfolio` — exempt from ticket-first gate |
| **Spike** | Time-boxed hypothesis exploration; exempt from AgDR + 80% coverage gates |
| **Spike disposition** | `/spike-close --promote` (→ Feature ticket) or `--discard` (→ memo in `docs/spike-memos/`) |
| **Split-portfolio** | Two-repo layout preventing private project name leaks |
| **Fan-out** | Spawn N parallel agents via `/fan-out` with optional worktree isolation |
| **Plan mode** | Think-first-then-execute for >=4 steps, unclear paths, or hard-to-reverse actions |
| **Lifecycle** | `In Progress → In Review → QA → Done` (QA is mandatory, not auto-closed) |
| **Release-cut** | Framework: dev/main + semver; managed projects: trunk-based |
| **Skip markers** | Auditable bypasses for mechanical gates: `<!-- pr-sections: skip -->`, `<!-- private-refs: allow -->`, etc. |
| **Handbooks** | Adopter-authored coding standards in `handbooks/`; advisory default, `ENFORCEMENT: blocking` for must-fix |
| **Custom templates** | Override framework templates via `custom-templates/`; replace-not-append |

---

## CARD 4 — The 48 Slash Commands at a Glance

### Ticket & Issue Skills
| Command | What It Does |
|---|---|
| `/feature <title>` | Create a **[Feature]** ticket: user story + acceptance criteria + priority |
| `/bug <title>` | Create a **[Bug]** ticket: Given/When/Then + repro steps + severity |
| `/task <title>` | Create a **[Chore/Refactor/Testing/CI/Docs]** ticket: driver + scope + ACs |
| `/spike <title>` | Create a **[Spike]** ticket: hypothesis + budget + kill criteria + disposition |
| `/spike-close` | Disposition a spike: promote to feature or discard with memo |
| `/investigation <title>` | Create a **[Investigation]** ticket for sustained root-cause work |
| `/migration <title>` | Create a migration ticket + migration AgDR (required before editing migration files) |
| `/idea <title>` | Capture a product idea to the backlog (quick, lightweight) |
| `/tickets-batch` | Bulk-file 5–20 structured tickets in one flow |
| `/start-ticket <#N>` | Declare active ticket for this session (required before code edits) |
| `/validate-idea` | 5-question pre-spec gate (target user, alternative, smallest version, kill criteria, build/buy) |

### Planning & Architecture Skills
| Command | What It Does |
|---|---|
| `/write-spec` | Generate a PRD / feature spec from a problem statement |
| `/decide` | Create an AgDR for a technical decision |
| `/agdr <browse/search/show/stats>` | Search/browse the AgDR library |
| `/c4 <project>` | Generate C4 L1 (System Context) + L2 (Container) Mermaid diagrams |
| `/dfd <project>` | Generate Data Flow Diagram (Mermaid + Threat Dragon JSON) |
| `/tech-vision <project>` | Interactive architecture vision authoring (target-state + migration path) |
| `/journey <project>` | Generate user-journey HTML (clickable boxes-and-arrows) |
| `/process <project>` | Extract business process → BPMN 2.0 file |
| `/extract-features <project>` | Scan codebase → Feature Inventory (6 axes) |
| `/roadmap` | Create/update/prioritize the product roadmap |

### Portfolio & Status Skills
| Command | What It Does |
|---|---|
| `/setup` | First-run bootstrap (company, stack, team, quality, LSP) |
| `/handover <repo>` | Onboard an external repo into ApexYard management |
| `/projects` | List all managed projects with status/branch/PRs/issues |
| `/status` | Current snapshot: git, CI, in-progress work |
| `/inbox` | Everything needing attention: PRs to review, issues assigned, blockers |
| `/tasks` | Actionable task list with direct URLs, prioritized |
| `/update` | Sync ops fork with upstream me2resh/apexyard |
| `/stakeholder-update` | Generate weekly/monthly/launch update from recent activity |

### Review & Quality Skills
| Command | What It Does |
|---|---|
| `/code-review` | Invoke Rex (Code Reviewer) on a PR |
| `/security-review` | Invoke Hatim (Security Reviewer) on a PR |
| `/audit-deps` | Audit dependencies for vulnerabilities, licenses |
| `/approve-merge <PR#>` | Record per-PR CEO approval (required by merge gate) |
| `/approve-design <PR#>` | Record per-PR design-review approval (required for UI PRs) |
| `/launch-check <project>` | 8-dimension production readiness audit with go/no-go verdict |

### Audit Deep-Dives (companion to /launch-check)
| Command | Dimension |
|---|---|
| `/threat-model` | Security — STRIDE threat modelling |
| `/accessibility-audit` | WCAG 2.1 AA compliance |
| `/compliance-check` | GDPR + ePrivacy |
| `/analytics-audit` | Event taxonomy + SDK coverage |
| `/seo-audit` | Technical SEO |
| `/performance-audit` | Bundle + Core Web Vitals |
| `/monitoring-audit` | Observability + incident readiness |
| `/docs-audit` | Diataxis documentation completeness |

### Other
| Command | What It Does |
|---|---|
| `/fan-out` | Spawn N parallel agents (worktree isolation, background mode) |
| `/split-portfolio` | Migrate single-fork → split-portfolio (10-step gated flow) |
| `/release` | Cut a new apexyard framework release (not for managed projects) |

---

## CARD 5 — The SDLC Flow (How Work Moves)

```
   ┌─────────┐    ┌──────────┐    ┌───────┐    ┌────────┐    ┌────┐    ┌──────┐    ┌────────┐
   │ Planning │───►│ Design   │───►│ Build │───►│ Review │───►│ QA │───►│Deploy│───►│Monitor │
   └─────────┘    └──────────┘    └───────┘    └────────┘    └────┘    └──────┘    └────────┘
    Gate 1         Gate 2          Gate 3        Gate 4        Gate 5    Gate 6
    PRD approved   Design ok       Ticket +      Rex + CEO    QA signs  Prod deploy
    epic exists    AgDR done        branch       CI green     off all   + monitor
                                   created
```

**Every gate is a hard stop.** If you're missing something, you don't skip ahead.

---

## CARD 6 — The Hooks: What Gets Enforced Mechanically

| Hook | When | What It Blocks |
|---|---|---|
| `require-active-ticket.sh` | Before Edit/Write/Bash-write | Edits to source code without an active ticket (`/start-ticket` first) |
| `require-skill-for-issue-create.sh` | Before `gh issue create` | Raw ticket creation outside a structured skill (`/feature`, `/bug`, `/task`, etc.) |
| `require-migration-ticket.sh` | Before editing migration files | Migration edits without a migration ticket + AgDR |
| `require-agdr-for-arch-changes.sh` | Before `git commit` on arch paths | Architecture-changing commits without an AgDR |
| `block-main-push.sh` | Before `git push` | Direct pushes to main/master/dev/develop |
| `block-git-add-all.sh` | Before `git add -A` / `git add .` | Staging all files (must stage specific files) |
| `block-unreviewed-merge.sh` | Before `gh pr merge` | Merges without Rex + CEO approval markers |
| `block-merge-on-red-ci.sh` | Before `gh pr merge` | Merges with failing CI |
| `check-secrets.sh` | Before `git commit` | Commits containing secret patterns (API keys, tokens, passwords) |
| `validate-branch-name.sh` | Before `git push` | Branches not matching `{type}/{TICKET-ID}-{slug}` |
| `validate-commit-format.sh` | Before `git commit` | Commits not matching `type: subject` conventional format |
| `validate-pr-create.sh` | Before `gh pr create` | PRs missing ticket ID, wrong format, or missing required sections |
| `pre-push-gate.sh` | Before `git push` | Pushes without passing lint/typecheck/test/build |
| `block-private-refs-in-public-repos.sh` | Before `gh issue/pr create/comment` | Private project names leaking into public repos |
| `suggest-ticket-template.sh` | Before `gh issue create` | Missing structured template sections |
| `require-design-review-for-ui.sh` | Before `gh pr merge` on UI PRs | UI PRs without design approval |

---

## CARD 7 — The Marker Files (Session State)

| Marker | Written By | Read By | Cleaned Up By |
|---|---|---|---|
| `.claude/session/active-bootstrap` | `/setup`, `/handover`, `/update`, `/split-portfolio` (Step 0) | `require-active-ticket.sh` (exempts bootstrap skills from ticket gate) | Skill's final step, or `clear-bootstrap-marker.sh` on next session start |
| `.claude/session/active-issue-skill` | `/feature`, `/bug`, `/task`, `/spike`, `/idea`, `/investigation`, `/migration` (Step 0) | `require-skill-for-issue-create.sh` (allows `gh issue create` through) | Skill's final step, or `clear-issue-skill-marker.sh` on next session start |
| `.claude/session/current-ticket` | `/start-ticket` | `require-active-ticket.sh` (proves a ticket is active) | Manually, or when switching tickets |
| `.claude/session/tickets/<project>` | `/start-ticket` (per-project variant) | `require-active-ticket.sh` (per-project ticket tracking) | Manually, or when switching tickets |

---

---

## USAGE CASE 1 — Adding a New Product (Portfolio Extension)

> Scenario: Your company already has one product. You want to add a second product under the same company umbrella.

### Step-by-step

```
1. Create the new repo on GitHub (or reuse an existing one)

2. Run /handover to register it in the portfolio:
   /handover my-new-product

3. This skill will:
   - Ask for the repo URL (owner/repo format)
   - Clone it into workspace/my-new-product/ (or use existing clone)
   - Scan the codebase (routes, models, tests, features)
   - Generate a handover assessment at projects/my-new-product/handover.md
   - Add the project entry to apexyard.projects.yaml

4. Configure project-specific settings in .claude/project-config.json if needed:
   - pre_push.commands (lint, test, build)
   - ui_paths (if it has a UI)
   - agdr_trigger_paths (architecture-sensitive paths)

5. Start working:
   /idea "New product MVP feature"
   /validate-idea IDEA-NNN
   /write-spec "Feature description"
   /feature "First feature"
   /start-ticket GH-1
```

### What happens under the hood

```
/handover reads apexyard.projects.yaml
    │
    ├─ Asks: "What's the repo?"
    ├─ Clones into workspace/<name>/
    ├─ Scans 6 axes (routes, models, async, tests, screens, docs)
    ├─ Generates handover assessment
    ├─ Appends entry to apexyard.projects.yaml
    │       name: my-new-product
    │       repo: your-org/new-product
    │       status: active
    │       tier: P1
    │
    └─ Next: /projects shows both projects
```

---

## USAGE CASE 2 — Adding an Internal Project (Different Perspective)

> Scenario: You have a customer-facing SaaS product. You want to add an internal tool (like an admin dashboard) with different quality gates or visibility.

### Step-by-step

```
1. If the internal project is PRIVATE and you're on GitHub Free:
   /setup -- needs split-portfolio mode
   (or run /split-portfolio to migrate)

2. Register the internal project:
   /handover my-admin-dashboard

3. In apexyard.projects.yaml, tag it:
   tags:
     - internal
     - inherited
   tier: P2

4. Customize per-project config in .claude/project-config.json:
   - Maybe lower pre-push checks (skip E2E for internal tool)
   - Maybe skip design review (no external UI)
   - Maybe adjust quality gates

5. Use the internal tag to filter:
   /projects     ← shows both products with status
   /inbox        ← shows work across both
   /tasks        ← prioritized across portfolio
```

### Tags and Tiers

| Field | Purpose | Values |
|---|---|---|
| `tier` | Business priority | P0 (critical), P1 (important), P2 (nice-to-have), P3 (backlog) |
| `tags` | Free-form labels | `internal`, `inherited`, `experimental`, `customer-facing`, etc. |
| `status` | Project lifecycle | `active`, `maintenance`, `paused`, `sunset` |

### Filtering by project

All portfolio-aware skills (`/projects`, `/inbox`, `/tasks`, `/status`) read `apexyard.projects.yaml` and can filter by project. Skills that need a project context (`/c4`, `/threat-model`, `/launch-check`, etc.) accept the project name as an argument.

---

## USAGE CASE 3 — Ideas & Hypothesis Validation Backlog

> Scenario: You had a brainstorming session. You want to capture raw ideas without validation, then optionally validate them later.

### The Idea Pipeline

```
Raw Idea ──► /idea ──► ideas-backlog.md (IDEA-NNN, status: NEW)
                                        │
                              /validate-idea ◄── 5-question gate
                                        │
                              /write-spec ◄── full PRD
                                        │
                              /feature ◄── structured ticket
```

### Capturing an idea (no validation, just hypothesis)

```
/idea Weather API integration for BMS optimization
```

This skill:
1. Asks a few clarifying questions (category, description)
2. Appends to `projects/ideas-backlog.md` with a unique ID (`IDEA-001`)
3. Status is `NEW` — no validation performed
4. Idea is now tracked and can be referenced later

### Validating an idea (when you're ready to decide)

```
/validate-idea IDEA-001
```

This skill asks 5 questions:
1. **Who is the target user?** (specific persona or "everyone")
2. **What alternative exists today?** (build/buy/rent analysis)
3. **What is the smallest version?** (MVP scope)
4. **What would make us kill it?** (kill criteria)
5. **Should we build, buy, or rent?** (make vs. buy decision)

The output:
- `VALID` → proceed to `/write-spec`
- `KILL` → mark as `REJECTED` in the backlog
- `UNCERTAIN` → more research needed or run a `/spike`

### Converting to a feature spec

```
/write-spec "Weather API integration for BMS optimization"
```

Generates a full PRD from the problem statement.

### Quick reference: idea vs. spike vs. feature

| Tool | Purpose | When | Output |
|---|---|---|---|
| `/idea` | Capture raw idea | Brainstorm, no commitment | `IDEA-NNN` in backlog, status NEW |
| `/validate-idea` | 5-question gate | Decide if idea is worth speccing | VALID / KILL / UNCERTAIN |
| `/spike` | Time-boxed experiment | Test a hypothesis before committing | `[Spike]` ticket with budget + kill criteria |
| `/write-spec` | Full PRD | Idea validated, ready to spec | Product Requirements Document |
| `/feature` | Structured feature ticket | Ready to build | `[Feature]` GitHub Issue |

---

## USAGE CASE 4 — Registering Features from a Prioritized Meeting

> Scenario: You had a planning meeting. You have a list of features, already prioritized and defined. You want to register them all.

### Single feature

```
/feature Add Ollama chat with system prompt
```

The skill walks through one question at a time:
1. **User Story**: "As a user, I want to chat with Ollama so that I get concise answers"
2. **Acceptance Criteria**: "[ ] Chat sends prompt to Ollama, [ ] System prompt enforces concise style, [ ] Error handling..."
3. **Design Notes**: "No UI changes" or Figma links
4. **Priority**: P0 / P1 / P2
5. **Out of Scope**: (optional)

Review → confirm → creates GitHub Issue with `enhancement` + priority labels.

### Bulk registration (5–20 features)

If the meeting produced more than 5 features, use batch mode:

```
/tickets-batch
```

The skill:
1. Asks shared-context questions ONCE (repo, priority scheme, common context)
2. Runs a 3-question micro-interview per ticket (title, ACs, priority)
3. Confirms the full batch
4. Files each via `gh issue create`

### Priority mapping

| Meeting Priority | ApexYard Label | GitHub Label |
|---|---|---|
| Must-have for launch | P0 | `enhancement`, `P0` |
| Ship soon after | P1 | `enhancement`, `P1` |
| Future / v2+ | P2 | `enhancement`, `P2` |
| Nice-to-have / backlog | P3 | `enhancement`, `P3` |

---

## USAGE CASE 5 — Tracking Features

### Project-level tracking

```
/projects           ← list all projects, their status, open PRs, open issues
```

### Feature status lifecycle

```
IDEA-001 (NEW)           ← /idea
     │
     ├── /validate-idea  → VALID / KILL / UNCERTAIN
     │
     ├── /write-spec     → PRD written
     │
     └── /feature        → [Feature] GitHub Issue created
              │
              /start-ticket GH-1   ← declare active work
              │
              └── branch created, PR opened
                       │
                       /code-review           ← Rex reviews
                       /approve-merge GH-2    ← CEO approves
                       │
                       └── merged → QA → Done
```

### Tracking at any point

| Command | What You See |
|---|---|
| `/status` | Current project snapshot: git state, CI, in-progress issue |
| `/inbox` | Everything needing your attention across all projects |
| `/tasks` | Actionable task list with URLs, prioritized |
| `/projects` | All projects with open PRs and issue counts |
| `gh issue list` | Raw GitHub Issues list (filtered by labels) |

### GitHub labels for status

The SDLC uses these states:
- `in-progress` — someone is working on it
- `in-review` — PR is open and under review
- `qa` — merged, waiting for QA verification
- No special label = backlog or done

---

## USAGE CASE 6 — Creating Tasks That Belong to Features & Milestones

### The hierarchy

```
Epic / Feature (#42)
  ├── Sub-task (#43) — "Implement API endpoint"
  ├── Sub-task (#44) — "Add unit tests"
  └── Sub-task (#45) — "Update documentation"
```

ApexYard uses **GitHub Issues** for everything. The relationship is:
- **Features** = `[Feature]` issues created by `/feature`
- **Tasks** = `[Chore]`, `[Refactor]`, `[Testing]`, `[CI]`, `[Docs]` issues created by `/task`
- **Spikes** = `[Spike]` issues created by `/spike`
- **Bugs** = `[Bug]` issues created by `/bug`

### Creating a task under a feature

```
/feature Add real-time BMS dashboard
  → Creates: [Feature] GH-10

/task Write API endpoint for battery readings
  → Creates: [Chore] GH-11

# Link them:
gh issue edit GH-11 --body "- Parent: GH-10"
# Or use GitHub's native sub-issue linking:
gh issue develop GH-11 --name "GH-10"
```

### Milestones

GitHub Milestones map to sprint or release targets:

```
# Create a milestone
gh api repos/{owner}/{repo}/milestones \
  -f title="Sprint 12" \
  -f description="May 20-27" \
  -f due_on="2026-05-27T00:00:00Z"

# Assign an issue to a milestone
gh issue edit GH-10 --milestone "Sprint 12"
```

### The /roadmap skill

```
/roadmap
```

Maintains a `roadmap.md` per project with milestones, features, and priorities. Supports:
- `add` — add a milestone
- `remove` — remove a milestone
- `reorder` — change priority order
- Output is a markdown table per milestone

---

## USAGE CASE 7 — Assigning & Tracking Tasks

### Assigning a task

```
# Assign to a GitHub user
gh issue edit GH-11 --add-assignee @username

# Assign yourself
gh issue edit GH-11 --add-assignee @me
```

### Tracking task progress

```
# Current session — what am I working on?
/status

# Everything I need to act on
/inbox

# Prioritized across the portfolio
/tasks

# Per-project details
gh issue list --repo muhammadhassanEA/MyAgent --assignee @me

# Move through the lifecycle
gh issue edit GH-11 --add-label "in-progress"   # Start working
gh issue edit GH-11 --add-label "in-review"      # PR opened
gh issue edit GH-11 --add-label "qa"             # Merged, QA needed
gh issue close GH-11                             # QA verified, done
```

### The full lifecycle with ApexYard enforcement

```
/feature "Add battery monitoring"                # Create ticket
/start-ticket GH-1                               # Declare active (required before edits)
                                                 #   → writes .claude/session/current-ticket
git checkout -b feature/GH-1-battery-monitoring   # Branch (name enforced by hook)
                                                 #   Code, code, code...
git add src/battery.py tests/test_battery.py      # Stage specific files (hook blocks git add -A)
git commit -m "feat(GH-1): add battery monitoring" # Commit format enforced by hook
git push origin feature/GH-1-battery-monitoring   # Pre-push gate runs lint+test+build
gh pr create --title "feat(GH-1): battery monitoring"  # PR format enforced by hook
                                                 #   → Auto-triggers Rex code review
/approve-merge 2                                 # CEO approves THIS specific PR
gh pr merge 2                                    # Merge (hook checks: Rex approved? CEO approved? CI green?)
gh issue edit GH-1 --add-label "qa"              # → QA, not Done
                                                 # QA verifies all ACs
gh issue close GH-1                              # Now Done
```

---

## USAGE CASE 8 — Tracking Projects

### Portfolio-level view

```
/projects
```

Output includes every registered project:
- Name, repo, status, tier, tags
- Current branch
- Open PRs count
- Open issues count

### Per-project deep dive

```
/status                    # Current project snapshot
/inbox                     # Action items across all projects
/tasks                     # Prioritized task list
/roadmap                   # Milestones and priorities
```

### Project onboarding lifecycle

```
/handover <name>           # Onboard external repo → projects/<name>/handover.md
/extract-features <name>   # Deep scan → projects/<name>/feature-inventory.md
/c4 <name>                 # Architecture diagrams → projects/<name>/architecture/
/dfd <name>                # Data flow diagram → projects/<name>/architecture/
/tech-vision <name>        # Target-state vision → projects/<name>/architecture/vision.md
```

### Production readiness

```
/launch-check <name>       # 8-dimension audit → pass/conditional/fail verdict
```

Scores across: security, accessibility, compliance, analytics, SEO, performance, monitoring, documentation. Each dimension has a deep-dive skill.

### Stakeholder communication

```
/stakeholder-update        # Weekly/monthly/launch update from recent activity
```

---

## CARD 8 — The Rules (Self-Discipline, Not Hooked)

These rules aren't enforced by hooks — Claude follows them from `.claude/rules/*.md`:

| Rule | File |
|---|---|
| Enter plan mode for risky/multi-step work | `plan-mode.md` |
| Offer fan-out for parallel work | `parallel-work.md` |
| One ticket at a time; complete before starting next | `workflow-gates.md` |
| QA state is mandatory (never auto-close to Done) | `workflow-gates.md` |
| Use ticket vocabulary carefully (Ticket/#N = real GH issues) | `ticket-vocabulary.md` |
| Run `/decide` before any technical decision | `agdr-decisions.md` |
| Follow DDD, naming, testing conventions | `code-standards.md` |
| PR must have glossary + testing sections | `pr-quality.md` |
| Plan-level "go" does NOT authorize merges | `pr-workflow.md` |
| Activate roles when triggers fire | `role-triggers.md` |

---

## CARD 9 — The Role System

When a trigger fires, Claude reads the role file and acts as that persona:

| Department | Role (Persona) | Auto-Trigger |
|---|---|---|
| Engineering | Head (Khalid) | Architecture review, cross-project call |
| Engineering | Tech Lead (Hisham) | Design docs, AgDR triggers, dependency additions |
| Engineering | Backend (Karim) | Backend implementation |
| Engineering | Frontend (Yasmin) | UI implementation |
| Engineering | QA (Salim) | Ticket labeled `qa` |
| Engineering | Platform (Adel) | CI/CD, workflows, golden paths |
| Engineering | SRE (Saif) | Production incident, SLO breach |
| Product | Head (Omar) | Roadmap, strategy |
| Product | PM (Mariam) | PRD, user stories |
| Product | Analyst (Hanan) | Market research, metrics |
| Design | Head (Maha) | Design system changes |
| Design | UI (Nour) | Visual design, tokens |
| Design | UX (Iman) | User flows, wireframes |
| Security | Head (Faisal) | Strategy, compliance |
| Security | Auditor (Hakim) | PRs touching auth/crypto/secrets |
| Security | Pen Tester (Hamza) | Active testing |
| Data | Head (Khalil) | Analytics strategy |
| Data | Analyst (Nadia) | Dashboards, queries |
| Data | Engineer (Anwar) | ETL, data modeling |

---

## CARD 10 — Templates & Document Types

| You Want to Create | Use This | File Location |
|---|---|---|
| Product spec | `/write-spec` or `templates/prd.md` | `projects/<name>/docs/` |
| Technical design | `templates/technical-design.md` | `projects/<name>/docs/` |
| Architecture decision | `/decide` → `templates/agdr.md` | `projects/<name>/docs/agdr/AgDR-NNNN-slug.md` |
| Migration decision | `/migration` → `templates/agdr-migration.md` | `projects/<name>/docs/agdr/AgDR-NNNN-slug.md` |
| Spike ticket | `/spike` → `templates/spike.md` | GitHub Issue |
| Investigation | `/investigation` → `templates/investigation.md` | GitHub Issue + live-doc |
| C4 diagram | `/c4 <project>` → `templates/architecture/c4-context.md` + `c4-container.md` | `projects/<name>/architecture/` |
| Data flow diagram | `/dfd <project>` → `templates/architecture/dfd.md` | `projects/<name>/architecture/` |
| Architecture vision | `/tech-vision <project>` → `templates/architecture/vision.md` | `projects/<name>/architecture/vision.md` |
| Sequence diagram | `templates/architecture/sequence.md` | `projects/<name>/architecture/` |
| User journey | `/journey <project>` | Self-contained HTML |
| Feature inventory | `/extract-features <project>` | `projects/<name>/feature-inventory.md` |
| Business process | `/process <project>` | `projects/<name>/processes/<slug>.bpmn` |

---

## QUICK REFERENCE — The Most Common Workflows

### "I have an idea"
```
/idea "Your idea here"        → IDEA-NNN in backlog (no commitment)
/validate-idea IDEA-NNN       → 5-question gate (is it worth pursuing?)
/write-spec "Validated idea"  → Full PRD
/feature "Feature from PRD"   → [Feature] GH-NNN ticket
```

### "I have a bug"
```
/bug "Bug description"        → [Bug] GH-NNN ticket with Given/When/Then
/start-ticket GH-NNN           → Declare active (required before code edits)
```

### "I need to investigate something"
```
/spike "Hypothesis to test"   → [Spike] GH-NNN with budget + kill criteria
# ... do the spike ...
/spike-close                  → Promote to feature or discard with memo
```

### "I want to start coding"
```
/start-ticket GH-NNN           → Required! (hook blocks edits without this)
git checkout -b feature/GH-NNN-slug
# ... code, commit, push (hooks enforce format) ...
gh pr create ...
/code-review                   → Rex reviews
/approve-merge PR-N            → CEO approves (per-PR, explicit)
```

### "I want to add a new project"
```
/handover existing-repo        → Register + assess
# OR
/setup (first time)            → Bootstrap the whole framework
```

### "I want to see where things stand"
```
/projects                      → All projects, status, PRs, issues
/status                        → Current git + CI + in-progress work
/inbox                         → Everything needing attention
/tasks                         → Prioritized action list
```

### "I want to audit before launch"
```
/launch-check myagent           → 8-dimension audit with go/no-go
# Or deep-dive a single dimension:
/threat-model myagent
/accessibility-audit myagent
/performance-audit myagent
# etc.
```