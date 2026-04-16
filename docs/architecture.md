# Research Collaborator — Architecture Reference

The Research Collaborator is a Claude Code framework that transforms Claude from a stateless chat assistant into a persistent, self-correcting research partner. It is designed for empirical researchers running multi-session, months-long projects.

## Architecture Layers

The framework decomposes a monolithic system prompt into nine specialized layers, each with distinct context cost characteristics:

| Layer | Mechanism | Context Cost |
|---|---|---|
| **Core prompt** | `CLAUDE.md` — non-negotiable protocols (~110 lines) | Always loaded |
| **Skills** (15) | `.claude/skills/*/SKILL.md` — on-demand behavioral norms | 0 at rest |
| **Subagents** (13) | `.claude/agents/*.md` — isolated context windows | 0 (separate windows) |
| **Hooks** (11) | `.claude/hooks/*.sh` + `kb_maintain.py` — deterministic automation | 0 (shell scripts) |
| **Project brief** | `project_brief.md` — swappable project scope | Loaded at session start |
| **State document** | `research_state.md` — live anti-drift record | Read on demand |
| **Knowledge base** | `knowledge/*.md` — 6 categories, append-only | Read on demand |
| **Lessons** | `lessons/*.md` — 6 topic files + index with decay | Surfaced by hook |
| **Episodes** | `episodes/*.md` — narrative session summaries | Read on demand |

### Design Principles

- **Progressive disclosure**: Only load context when needed. The core prompt stays lean; skills inject full behavioral norms on demand.
- **Deterministic enforcement**: Hooks (shell scripts) handle mechanical guarantees — no prompt-based enforcement for things that can be automated.
- **Context isolation**: Subagents run in separate context windows. The main agent never forwards its full system prompt to them.
- **Append-only persistence**: Knowledge base and lessons are never deleted — only marked as superseded, resolved, or deprecated, then auto-archived.

---

## Non-Negotiable Protocols

Three protocols in `CLAUDE.md` govern all interactions:

### 1. Plan-Review-Execute Protocol

Every task follows: PLAN → CLARIFY → REVIEW → APPROVE → EXECUTE.

The PLAN step has two modes:

- **Complex tasks**: Main agent launches a Plan subagent (Sonnet) to draft the plan, then an Adversarial Review subagent (Sonnet) to critique it. Main agent synthesizes both into the final plan. This mirrors `/deliberate` but is lighter-weight and focused on execution planning.
- **Simple tasks**: Main agent plans directly.

Both modes require reading `lessons/lessons_index.md` first and incorporating relevant lessons into the proposed plan (referenced by ID).

### 2. Async Input Protocol ("noted")

When the researcher provides ideas or context:

1. Respond only with **"noted"**
2. After each "noted", a background Haiku subagent (`note-taker-classifier`) classifies the idea and writes it to the correct KB category or stages it for review
3. Continue acknowledging until the researcher says **"I am done."**
4. Only then engage with the full set of accumulated inputs

### 3. Research Decision Protocol

After every experiment, analysis, or milestone:

- **PROCEED**: Results validate the approach — move forward
- **REFINE**: Partially promising — tweak parameters and re-run (with a budget limit)
- **PIVOT**: Core assumptions invalidated — propose alternatives, require researcher approval

Always version artifacts before the decision. For PIVOT decisions, prompt the researcher to consider `/deliberate`.

---

## Skills (15)

Skills are behavioral norms and workflows loaded on demand via slash commands.

| Skill | Slash Command | Purpose |
|---|---|---|
| Research Collaborator | `/research-collaborator` | Full behavioral norms: critical thinking directives, audience modes (researcher/publication), academic rigor guidelines, communication norms, state document template |
| Knowledge Base | `/knowledge-base` | Maintain a structured KB across 6 categories (decisions, experiments, findings, literature, questions, reviews). Three tiers per category: active, archive, index |
| Lessons Learned | `/lessons-learned` | Extract generalizable lessons via a 7-question self-reflection protocol. Categories: methodology, data, tooling, design, collaboration, domain. Recency decay: fresh → recent → aging → archived |
| Research Decision | `/research-decision` | PROCEED/REFINE/PIVOT decision framework with artifact versioning to `versions/` and rollback support |
| Episodic Summary | `/episodic-summary` | Narrative session checkpoint with structured sections (arc, decisions, assumptions, open threads, artifacts). Triggers post-summary maintenance (literature triage, findings triage, KB validation) |
| Deliberate | `/deliberate` | Multi-perspective deliberation: 4 sequential personas (Collaborator → EiC → R2 → Strategist), each with evidence retrieval, word budget tracking, and tension synthesis. Presets: quick, full, deep |
| Data Loader | `/data-loader` | Load datasets from HuggingFace Hub or Kaggle with authentication, format detection, validation, and profiling |
| Sub-Project | `/sub-project` | Manage concurrent research streams. Commands: create, switch, capture, list, status. Each sub-project has isolated knowledge, lessons, episodes, and state |
| Adopt to Sub-Project | `/adopt-to-sub-project` | One-time retroactive migration of root material into a new sub-project via propose/review/execute workflow. Physical file moves with backup, path-fixup detection, and rollback |
| Note Taker | `/note-taker` | Inspect and triage ideas auto-captured by the `note-taker-classifier` during "noted" sessions. Commands: status, review |
| Propose Lessons | `/propose-lessons` | Scan the tool-activity buffer (`.tool_use_buffer.jsonl`) for candidate lessons, deduplicate against existing lessons, write drafts to `.proposed_lessons.md` for researcher review |
| House-Keeping | `/house-keeping` | KB hygiene: scan findings/decisions for validation status, propose converting unvalidated entries to open questions. Non-destructive scan → researcher review → execute |
| Review Figure | `/review-figure` | Two-pass figure review: Pass 1 (style compliance vs. `FIGURE_AGENT_PROMPT.md` sections 1-11), Pass 2 (factual grounding vs. KB, config, source code). Never modifies files |
| PM | `/pm` | Lightweight project management: backlog of tracked tickets (KB-QST entries), effort tracking, dispatch to front/backend agents, close-with-housekeeping |
| Slack | `/slack` | Async Slack DM communication with 3-minute polling via CronCreate. Treats DM responses as terminal input |

---

## Subagents (13)

Subagents run in isolated context windows and are delegated by the main agent, never invoked directly by the researcher.

### Research Execution Agents (Sonnet)

| Agent | Tools | Role |
|---|---|---|
| `literature-researcher` | Read, Grep, Glob, WebFetch, WebSearch, Scholar Gateway | Academic literature search using tiered strategy: Scholar Gateway first, then web search. Prioritizes last 18 months, top-tier venues. Quality over quantity |
| `code-implementer` | Read, Write, Edit, Bash, Grep, Glob | Write, test, and debug implementation code. PEP 8, type hints, pytest, `uv run python` only |
| `data-analyst` | Read, Write, Edit, Bash, Grep, Glob | Data processing, statistical analysis, experiment execution, visualization. Reproducible: random seeds, confidence intervals, per-class metrics |
| `figure-specialist` | Read, Write, Edit, Bash, Grep, Glob | Create, modify, and render publication-quality figures following `FIGURE_AGENT_PROMPT.md`. Supports matplotlib and graphviz. Source-level editing only (never touches PDF/SVG directly) |

### Deliberation Agents (Sonnet, sequential)

| Agent | Sequence | Role |
|---|---|---|
| `deliberate-collaborator` | 1st | Friendly outside perspective from an adjacent discipline. Picks ONE discipline per deliberation. Brings genuine alternative framing |
| `deliberate-editor` | 2nd | Editor-in-Chief evaluation: contribution clarity, positioning, audience, ethical implications. Must engage with Collaborator's reframing |
| `deliberate-reviewer2` | 3rd | Hostile peer reviewer: fatal flaws, statistical rigor, threats to validity. Must respond to EiC and challenge Collaborator |
| `deliberate-strategist` | 4th (always last) | Integrates all perspectives into pragmatic recommendation. Resolves tensions. Only persona allowed to do so explicitly |

### Deliberation Support Agents (Haiku)

| Agent | Role |
|---|---|
| `deliberate-context-distiller` | Extract compact evidence index (< 700 words) for persona workflow. Reads KB headers only, never full entries |
| `deliberate-evidence-retriever` | Select top 5 KB entries per persona and synthesize evidence brief (< 600 words). Opposing evidence mandatory |

### Classification and Capture Agents (Haiku)

| Agent | Role |
|---|---|
| `note-taker-classifier` | Classify async input ideas → write KB entries directly (`Status: captured`) or stage lessons/uncertain to `.proposed_notes.md`. Returns single-line confirmation. Uses `fcntl.flock()` for concurrency |
| `subproject-capture` | Capture ideas into target sub-project KB without polluting main session context. Context firewall. Global ID allocation |
| `subproject-adopt-classifier` | Classify root material for adoption into new sub-project. Produces proposal at `.adopt_proposal.md`. Advisory only |

### Delegation Rules

The main agent **may delegate**: literature search, code implementation, data analysis, dataset loading, figure creation.

The main agent **never delegates**: design decisions, protocol enforcement, state document management, researcher interaction, synthesizing/evaluating subagent outputs, or plan approval. For complex tasks, the main agent delegates plan *drafting* and *adversarial review* to subagents but always **synthesizes the final plan itself**.

---

## Hooks (11)

Hooks are event-driven shell scripts that provide mechanical guarantees without consuming prompt context.

### Session Lifecycle

| Hook | Event | What It Does |
|---|---|---|
| `session_start.sh` | SessionStart | Loads `project_brief.md` and `research_state.md` into context. Surfaces recent lessons. Identifies active sub-project |
| `pre_compact.sh` | PreCompact | Creates timestamped backup of state document to `.state_backups/`. Injects recovery guidance before context compression |
| `slack_stop_reminder.sh` | Stop | Checks for active `.slack_session.json` and reminds to close |

### Per-Prompt and Per-Action

| Hook | Event | What It Does |
|---|---|---|
| `kb_prime.sh` | UserPromptSubmit | Injects KB summary (entry counts + index paths) before each prompt. Skips short messages (< 30 chars) |
| `lessons_check.sh` | PreToolUse (Task, Write, Edit, Bash) | Surfaces fresh/recent lessons from `lessons_index.md` before significant actions |
| `breath_check.sh` | PreToolUse (Task, Write, Edit, Bash) | Time-gated (10 min): nudges self-check for plan compliance, task health, intermediate findings. Detects long-running foreground commands via pattern matching |

### Post-Action

| Hook | Event | What It Does |
|---|---|---|
| `post_tool_use_buffer.sh` | PostToolUse (all tools) | Logs tool activity to `.tool_use_buffer.jsonl`. Debounced trigger: fires `[LESSON EXTRACT]` after 120s AND (10 events OR 3 errors) |
| `stop_state_check.sh` | Stop | Checks if state document was modified within 120s. Emits reminder if not updated during substantive exchange |
| `kb_index_hook.sh` | Stop | Rebuilds KB index tables if `*_active.md` files modified since last rebuild. Invokes `kb_maintain.py` for auto-archival |
| `subagent_stop.sh` | SubagentStop | Logs subagent completion, prompts artifact review |

### Python Maintenance

| Script | Invoked By | What It Does |
|---|---|---|
| `kb_maintain.py` | `kb_index_hook.sh` | Parses KB entries, regenerates index markdown tables, auto-archives entries with status SUPERSEDED/RESOLVED/DEPRECATED/ADDRESSED. Preserves "Cited" flags for literature entries |

---

## Knowledge Architecture

### Knowledge Base (6 Categories)

Each category has three tiers of files:

| Tier | Pattern | Purpose |
|---|---|---|
| Active | `*_active.md` | Current entries — the working set |
| Archive | `*_archive.md` | Superseded/resolved entries — preserved for history |
| Index | `*_index.md` | Compact lookup tables, auto-generated by `kb_maintain.py` |

Categories:

| Category | Entry Prefix | Tracks |
|---|---|---|
| Decisions | `D-NNN` | Design choices, methodology selections, parameter settings with rationale |
| Experiments | `EXP-NNN` | Every run: config, results, interpretation, success/failure criteria |
| Findings | `F-NNN` | Validated insights from experiments or synthesis |
| Literature | `LIT-NNN` | Papers reviewed: key claims, relationships to project work |
| Questions | `Q-NNN` | Research questions: active, parked, or resolved |
| Reviews | `REV-NNN` | Feedback from agents, researcher critiques, peer comments |

Entry format: `[PREFIX-NNN] Short Title` with Date, Phase, Content, Evidence, Connections, Status fields.

Rules:
- **Append-only**: Never delete entries; mark as superseded or deprecated
- **Index-first retrieval**: Read index files before accessing full entries
- **Global IDs**: IDs are unique across shared + all sub-project scopes
- **Sub-project scoping**: Sub-project-specific entries go to `subprojects/<slug>/knowledge/`; generalizable knowledge stays at project root

### Lessons Learned (6 Topic Categories)

| Category | File | Covers |
|---|---|---|
| Methodology | `methodology.md` | Research methods, experimental design patterns |
| Data | `data.md` | Data handling, preprocessing, quality issues |
| Tooling | `tooling.md` | Tool/workflow insights, environment problems |
| Design | `design.md` | Architecture, prompt design, system decisions |
| Collaboration | `collaboration.md` | Human-AI interaction, communication patterns |
| Domain | `domain.md` | Domain-specific insights (mental health, NLP, etc.) |

Entry format: `[LL-NNN] Short Title` with Date, Phase, Category, Trigger, sections for What Happened, Reasoning, Expectation vs. Reality, Root Cause, The Lesson, Applicability, Confidence.

Recency decay:
- **Fresh**: 0-14 days
- **Recent**: 15-30 days
- **Aging**: 31-60 days
- **Archived**: 60+ days

The `lessons_check` hook surfaces fresh and recent lessons before significant actions.

### Lesson Extraction Pipeline

```
Tool use → post_tool_use_buffer.sh (logs to .tool_use_buffer.jsonl)
         → Debounce gate (120s + 10 events or 3 errors)
         → [LESSON EXTRACT] trigger in context
         → /propose-lessons dispatches background subagent
         → Subagent writes candidates to .proposed_lessons.md
         → Researcher reviews candidates
         → /lessons-learned promotes approved candidates to lessons/*.md
```

---

## State Management

### State Document (`research_state.md`)

The primary anti-drift mechanism. Contains:

- **Active Phase**: Current task and status
- **Roadmap**: Curated progression view — Program Timeline (inflection points) and Sub-Project Status. Updated by `/research-decision` and `/sub-project` skills
- **Task Queue**: Compact index of active PM tickets, synced from `.pm_backlog.md` by `/pm` and `/research-decision`
- **Decisions Made**: Append-only table with rationale
- **Open Questions**: Active, parked, or resolved
- **Settled Assumptions**: Not re-questioned unless researcher reopens
- **Scope Boundaries**: In scope, out of scope, phase boundary
- **Key Artifacts**: Files, drafts, code outputs with status

Update rules:
- Update after every substantive exchange (not after "noted")
- Append-only for Decisions and Settled Assumptions
- Self-check every 5 turns, before plan execution, or on "check yourself"

### Roadmap

The Roadmap section serves a distinct purpose from Decisions Made: it captures only inflection points (phase transitions, direction changes) in a scannable table format designed for paper-writing and multi-week review. The full decision history lives in `knowledge/decisions_index.md`. Propagation between sub-project and root state is instruction-based (embedded in `/research-decision` Step 5), not hook-based.

Root-level Roadmap has two tables:
- **Program Timeline**: One row per inflection point across the whole project
- **Sub-Project Status**: One row per sub-project with current phase and last decision

Sub-project-level Roadmap has two tables:
- **Progression**: One row per inflection point within the sub-project
- **Gates**: Optional milestone gates with pass/fail criteria and evidence

### Sub-Projects

Research streams within a project, managed via `/sub-project`:

```
subprojects/
├── manifest.md          # Registry of all sub-projects
├── .active              # Contains active sub-project slug
├── <slug>/
│   ├── brief.md         # Sub-project scope
│   ├── state.md         # Sub-project state (same template)
│   ├── knowledge/       # 6 categories, scoped
│   ├── lessons/         # Scoped lessons
│   └── episodes/        # Scoped episode summaries
```

All hooks are sub-project-aware: session_start loads active sub-project state, lessons_check surfaces both shared and sub-project lessons, stop hooks check sub-project state freshness and maintain sub-project KB indexes.

### Versioning

Before major decisions (especially PIVOT), artifacts are snapshotted:

```
versions/
├── v_YYYYMMDD_HHMMSS_[descriptor]/
│   ├── research_state.md
│   ├── knowledge/
│   └── src/
```

---

## Project Bootstrapping

`init_project.sh` creates a sibling project directory with:

- Symlinked `.claude/` directory (skills, hooks, agents, settings)
- Scaffolded directories: `knowledge/`, `lessons/`, `episodes/`, `versions/`, `deliberations/`
- Template files: `project_brief.md`, `research_state.md`
- Symlinked `FIGURE_AGENT_PROMPT.md` style guide
- `.gitignore` for common exclusions

Usage:
```bash
cd research-collaborator
./init_project.sh my_project "My Research Title"
```

---

## Deliberation Workflow Detail

The `/deliberate` skill provides structured multi-perspective analysis for high-stakes decisions.

### Process

1. **Determine preset**: `quick` (skips EiC + R2), `full`, or `deep`
2. **Safety archive**: Backup any existing deliberation files
3. **Write frame**: Fill decision frame template at `deliberations/active_frame.md`
4. **Build evidence index**: `deliberate-context-distiller` creates compact index (< 700 words)
5. **Sequential personas**:
   - Collaborator (1st): Adjacent-discipline reframing
   - EiC (2nd, skip if quick): Publication viability assessment
   - R2 (3rd, skip if quick): Hostile critique targeting fatal flaws
   - Strategist (4th, always last): Integrates all perspectives, resolves tensions
6. **Validate transcript**: `validate_transcript.py`
7. **Synthesize**: Main agent writes tension map and synthesis
8. **Archive and report**: Finalize to `deliberations/` with timestamped slug

Each persona has its own evidence retrieval (top 5 KB entries via `deliberate-evidence-retriever`), word budget, and mandatory engagement with prior personas.

### Soft Auto-Trigger

When the main agent detects a qualifying high-stakes decision (via `assets/trigger_criteria.md`), it suggests `/deliberate` but never nags after a decline.

---

## Figure Workflow

Two components handle figures with a deliberate split:

| Component | Role | Invoked By |
|---|---|---|
| `figure-specialist` subagent | Create, modify, render figures | Main agent delegates |
| `/review-figure` skill | Two-pass review (style + factual) | Main agent, on researcher request |

### Figure Creation Flow

1. Subagent loads `FIGURE_AGENT_PROMPT.md` (mandatory — halts if missing)
2. Classifies figure type → selects tool (matplotlib or graphviz)
3. Writes `.py` generator script
4. Runs with `uv run python` → produces `.pdf` + `.png` (+ `.gv` for graphviz)
5. Walks pre-flight checklist (sections 1-11)

### Figure Review Flow

1. **Pass 1 — Style compliance**: Walk sections 1-11 checklist. Can be delegated to `figure-specialist`
2. **Pass 2 — Factual grounding**: Extract claims from figure, search authoritative sources (config → src → state → KB → brief), classify as match / discrepancy / unverifiable

Figures are always edited at the source level (`.py` generator), never by modifying rendered PDF/SVG/PNG directly.

---

## Remote Collaboration

The `/slack` skill enables async communication:

1. **Init**: Get researcher's Slack user ID, send init DM, start 3-minute polling
2. **DM triggers**: Input needed, or task complete (as thread replies)
3. **Process responses**: Treat DM text as raw terminal input
4. **Close**: End DM, cancel polling, delete `.slack_session.json`

---

## Integration Patterns

### Session Flow
```
SessionStart → session_start.sh (load context)
  → UserPromptSubmit → kb_prime.sh (inject KB summary)
    → PreToolUse → lessons_check.sh (surface lessons)
    → PreToolUse → breath_check.sh (time-gated self-check)
      → Tool execution
    → PostToolUse → post_tool_use_buffer.sh (log activity)
  → Stop → stop_state_check.sh (verify state update)
  → Stop → kb_index_hook.sh (rebuild indexes)
  → Stop → slack_stop_reminder.sh (remind to close Slack)
```

### Knowledge Flow
```
Researcher input → note-taker-classifier (Haiku, background)
                 → KB *_active.md (Status: captured)
                 → kb_index_hook.sh rebuilds indexes
                 → kb_prime.sh surfaces counts next prompt

Tool activity → post_tool_use_buffer.sh (buffer)
             → /propose-lessons (background subagent)
             → .proposed_lessons.md (staging)
             → Researcher review
             → /lessons-learned (permanent write)
```

### Delegation Pattern
```
Main agent (Opus) — orchestration, planning, decisions, state
  ├── literature-researcher (Sonnet) — read-only search
  ├── code-implementer (Sonnet) — write/test code
  ├── data-analyst (Sonnet) — analysis/statistics
  ├── figure-specialist (Sonnet) — figure rendering
  ├── deliberate-* (Sonnet) — persona deliberation
  ├── note-taker-classifier (Haiku) — idea capture
  ├── subproject-capture (Haiku) — cross-stream capture
  └── deliberate-context-distiller/evidence-retriever (Haiku) — lightweight extraction
```

### PM Ticket Lifecycle

```
"noted" session → note-taker-classifier creates KB-QST-NNN (Status: captured)
  → classifier AUTO-registers in .pm_backlog.md (status: open)
  → Task Queue in state doc synced
  → kb_prime.sh injects ticket counts before each prompt
  → On new task: agent surfaces backlog, asks prioritize-or-queue
  → Agent may query /pm next or /pm status (read-only, no approval needed)
  → /pm pick assigns to main agent or subagent (requires approval)
  → Work completed
  → /research-decision AUTO-closes resolved tickets (Step 6)
  → /pm close for manual close (creates KB-DEC/KB-FND, resolves KB-QST)
  → Task Queue in state doc synced on every change
  → kb_maintain.py auto-archives resolved KB-QST on next Stop cycle
```
