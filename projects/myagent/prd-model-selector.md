# PRD: Model Selector — Local & Cloud Ollama Models

**Status**: Draft
**Author**: Mariam (Product Manager)
**Created**: 2026-05-21
**Last Updated**: 2026-05-21
**Ticket**: [#3](https://github.com/muhammadhassanEA/MyAgent/issues/3)

---

## Overview

### Problem Statement

MyAgent currently hardcodes `smollm2:135m` as the only available model. Users who have multiple Ollama models installed locally — or who have an Ollama cloud subscription — cannot switch models without editing source code. This blocks experimentation with different model capabilities and prevents effective use of a model pool.

### Target User

**Primary**: Developer or power user running Ollama locally, with one or more models pulled, who wants to pick the best model for a given task at session start.
**Secondary**: Ollama cloud subscriber who wants cloud models surfaced alongside local ones in the same interface.

### Goals

1. On startup, fetch and display all locally available Ollama models with their availability status.
2. Let the user select a model before the chat loop begins (default pre-selected).
3. Surface cloud models when an Ollama subscription is active; grey them out (with reason) when not.
4. Fall back to the default model automatically if the selected model becomes unavailable.
5. Show enough model metadata (context length, best-use) for an informed choice.

### Non-Goals

- Full model management UI (pull, delete, update models).
- Complex GUI or web dashboard — CLI-only.
- Mid-session model switching (future).
- Fine-grained cost tracking per token (future).
- Support for non-Ollama providers (OpenAI, Anthropic, etc.).

### Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
| Model selection works with ≥1 local model | 100% | Manual test / CI |
| Startup time with selection screen | < 2s on local Ollama | Timer in tests |
| Graceful fallback fires when selected model missing | 100% | Unit test |
| Cloud model list shown when subscription active | Verified in integration | Manual test |

---

## User Stories

### US-1: Select model at startup

> As a developer, I want to see all available models at startup and pick one, so that I can use the best model for my current task without editing code.

**Acceptance Criteria**:

- [ ] A table displays all available models grouped by type: **Local** / **Cloud**
- [ ] A default model is pre-selected (configurable, falls back to first available local model)
- [ ] User can confirm default with Enter or select another with arrow keys / number
- [ ] Selected model is used for the entire chat session

### US-2: Model availability indicators

> As a developer, I want to see at a glance which models are ready to use and which require action, so that I don't pick a model that will fail.

**Acceptance Criteria**:

- [ ] Local models with status `available` shown in green
- [ ] Cloud models unavailable due to no subscription shown in orange with reason `subscription required`
- [ ] If the model list fetch fails, show error with fallback to hardcoded default — no crash

### US-3: Subscription check for cloud models

> As an Ollama cloud subscriber, I want cloud models listed alongside my local models, so that I can use them from the same interface.

**Acceptance Criteria**:

- [ ] On startup, a subscription check is performed against the Ollama API
- [ ] If subscription is active, all subscribed cloud models appear in the Cloud group
- [ ] If subscription is inactive/absent, Cloud group shows placeholder row: `No active subscription`
- [ ] Subscription check has a timeout (≤ 3s); failure is non-fatal

### US-4: Automatic fallback

> As a user, I want the app to recover gracefully if my selected model disappears (e.g. pulled mid-session), so that I'm not left with a broken session.

**Acceptance Criteria**:

- [ ] If the chat API returns a model-not-found error, app falls back to the default model
- [ ] User sees a one-line warning: `[Model X unavailable — falling back to Y]`
- [ ] Chat continues uninterrupted after fallback

---

### Edge Cases

| Scenario | Expected Behavior |
|----------|-------------------|
| Ollama not running | Error message at model fetch; exit with code 1 and clear instructions |
| No local models installed | Show empty Local group + prompt to `ollama pull <model>` |
| Only one local model | Skip interactive selection; auto-select with notice |
| Subscription API unreachable | Cloud group hidden; local-only mode; non-fatal warning |
| Selected model removed during session | Fallback to default (US-4) |

---

## Requirements

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1 | Fetch local models via `GET /api/tags` on Ollama | Must |
| FR-2 | Display model table (name, type, context length, best-use, status) before chat loop | Must |
| FR-3 | Pre-select default model (first local model, or configurable override) | Must |
| FR-4 | Interactive model selection (keyboard-navigable) | Must |
| FR-5 | Subscription check via Ollama API on startup | Must |
| FR-6 | Color-code availability (green/orange) | Must |
| FR-7 | Auto-fallback on model-not-found API error during chat | Must |
| FR-8 | Configurable default model via env var `MYAGENT_DEFAULT_MODEL` | Should |
| FR-9 | Model description/best-use metadata (static map for known models; `—` for unknown) | Should |
| FR-10 | Timeout on all network calls (model fetch ≤ 3s, subscription check ≤ 3s) | Must |

### Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | Startup to first prompt (including model fetch) | < 3s on localhost Ollama |
| Reliability | Graceful degradation if Ollama API is slow/down | Non-fatal; fallback to default |
| Usability | CLI-only; no extra dependencies beyond `rich` + `requests` | No new deps unless justified |
| Testability | Model fetch + fallback logic unit-testable without a live Ollama | Mock-able HTTP layer |

---

## Design

### User Flow

```
[App starts]
      |
      v
[Fetch local models via GET /api/tags]
      |
      +-- error --> [Print error, exit 1]
      |
      v
[Check Ollama subscription (async, ≤3s timeout)]
      |
      v
[Render model table]
  Local group  (green = available)
  Cloud group  (orange = subscription required | green = available)
      |
      v
[Pre-select default; user confirms / picks]
      |
      v
[Chat loop with selected model]
      |
      +-- model-not-found error --> [Fallback warning + continue with default]
```

### Model Table (CLI sketch)

```
┌─ Available Models ────────────────────────────────────────────────────────┐
│  #  Name                  Type   Context   Best For        Status         │
│  ── ──────────────────    ─────  ───────   ─────────────   ─────────────  │
│  1  smollm2:135m [*]      Local  8k        Quick tasks     ● Available    │
│  2  llama3.2:3b           Local  128k      General chat    ● Available    │
│  3  qwen2.5-coder:7b      Cloud  32k       Coding          ○ Subscription │
└───────────────────────────────────────────────────────────────────────────┘
[*] default   Enter to confirm, or type number to select:
```

---

## Technical Notes

### Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| `GET /api/tags` Ollama endpoint | External (local) | Available in Ollama ≥ 0.1 |
| Ollama cloud subscription API | External (cloud) | Endpoint TBD — see Open Questions |
| `rich` (table + prompt) | Python library | Already in pyproject.toml |

### Technical Constraints

- Must run on Python 3.12+ with `uv`-managed deps.
- No new mandatory runtime dependencies — `rich` + `requests` cover the implementation.
- Ollama's cloud subscription API is not publicly documented. Implementation must be behind a feature flag or gracefully no-op until the endpoint is confirmed. See Open Questions.
- Model metadata (description, best-use) is not returned by `/api/tags`. A static lookup table ships in-code for common models; unknown models show `—`.

---

## Open Questions

| Question | Owner | Status |
|----------|-------|--------|
| What is the actual Ollama cloud subscription API endpoint and auth mechanism? | Tech Lead | Open — blocks FR-5/FR-9 |
| Should model selection persist across sessions (config file)? | PM | Resolved — per-session only, no persistence |
| Should `MYAGENT_DEFAULT_MODEL` also be settable via a config file? | Tech Lead | Resolved — env var only (no config file) |

---

## Timeline

| Milestone | Target | Status |
|-----------|--------|--------|
| PRD Approved | 2026-05-21 | Draft |
| Tech Design | TBD | Pending |
| Dev Complete | TBD | Pending |
| QA Complete | TBD | Pending |

---

## Approvals

| Role | Name | Status |
|------|------|--------|
| Product Manager | Mariam | Author |
| Tech Lead | Hisham | Pending |
