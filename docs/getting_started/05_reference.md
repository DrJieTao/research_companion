# Quick Reference & Troubleshooting

## Slash Commands Cheat Sheet

Type these during a session to invoke specific capabilities:

| Command | What It Does | When to Use |
|---------|-------------|-------------|
| `/research-collaborator` | Load full behavioral norms and state template | Brainstorming, planning, writing for publication |
| `/deliberate` | Multi-perspective analysis (4 simulated reviewers) | High-stakes decisions, paper framing, PIVOT evaluation |
| `/knowledge-base` | Record or retrieve structured knowledge entries | After decisions, experiments, findings |
| `/lessons-learned` | Extract and record generalizable lessons | Session end, after unexpected results |
| `/episodic-summary` | Generate narrative summary of the session | Session wrap-up, onboarding new sessions |
| `/research-decision` | Evaluate results: PROCEED / REFINE / PIVOT | After every experiment or milestone |
| `/data-loader` | Load datasets from HuggingFace or Kaggle | Setting up data pipelines |
| `/sub-project` | Manage concurrent research streams | When juggling multiple threads |
| `/adopt-to-sub-project` | One-time retroactive migration of existing root material into a new sub-project | When formalizing an existing stream that was started at the project root |
| `/propose-lessons` | Scan the tool-activity buffer for candidate lessons and write drafts to `.proposed_lessons.md` | When the `[LESSON EXTRACT]` trigger appears, or on demand |
| `/review-figure` | Two-pass review of a manuscript figure (style + factual grounding) | Before sending a figure to co-authors or pasting it into the paper |
| `/slack` | Enable async Slack communication | When you need to step away but want updates |

**Subagents** (delegated by the main agent, not invoked directly):

| Subagent | What It Does | When the Main Agent Delegates |
|----------|-------------|-------------------------------|
| `literature-researcher` | Searches academic sources, summarizes papers | Literature review tasks |
| `code-implementer` | Writes and runs code against your project conventions | Implementation work |
| `data-analyst` | Loads, profiles, and analyzes datasets | Analysis and statistics |
| `figure-specialist` | Creates, modifies, and renders publication-quality figures following `FIGURE_AGENT_PROMPT.md` | Any figure create / modify / bootstrap-from-Mermaid task |
| `subproject-capture` | Records an idea into a target sub-project's KB without polluting your current session's context | When you say `/sub-project capture <slug> "<idea>"` during work on a different stream |
| `subproject-adopt-classifier` | Inventories root-level material (KB, lessons, manuscript, code) and classifies each item into adoption buckets for a new sub-project | When you run `/adopt-to-sub-project <slug>` to formalize an existing research stream |

---

## Glossary

| Term | Plain Language Meaning |
|------|----------------------|
| **Async Input Protocol** | The "noted" system — share multiple inputs before the agent engages. Say "I am done." to trigger response. |
| **Plan-Review-Execute** | The workflow where the agent proposes a plan, you review it, and only then does it execute. Never skips approval. |
| **Research Decision** | A formal evaluation after experiments: PROCEED (move forward), REFINE (tweak and retry), or PIVOT (change direction). |
| **Knowledge Base (KB)** | A structured collection of your project's decisions, experiments, findings, literature, questions, and reviews. Persists across sessions. |
| **Lessons Learned** | Process insights that capture WHY things worked or didn't — not just what happened. Surfaced before the agent takes action. |
| **State Document** | `research_state.md` — the single source of truth for what you're working on, what's been decided, and what's pending. |
| **Hook** | An automatic check that runs at specific moments (session start, before actions, after responses). You don't trigger these — they happen on their own. |
| **Breath Check** | A self-check that fires every 10 minutes during long tasks. Ensures the agent stays on plan and records intermediate results. |
| **KB Prime** | A brief summary of your knowledge base shown to the agent before it processes your message. Helps it find relevant past context. |
| **Subagent** | A specialized helper that handles delegated tasks (literature search, code writing, data analysis). Works in the background. |
| **Deliberation** | A structured process where 4 simulated perspectives (Collaborator, Editor, Reviewer 2, Strategist) analyze a decision before the agent recommends an action. |
| **Sub-project** | An isolated research stream within your project. Has its own state, knowledge, and lessons. |
| **Episode** | A narrative summary of a session — what was discussed, decided, and what remains open. |
| **Version/Snapshot** | A saved checkpoint of your project state. Created before major decisions so you can roll back if needed. |
| **Figure Style Guide** | `FIGURE_AGENT_PROMPT.md` — the single source of truth for figure palette, typography, matplotlib conventions, and graphviz patterns. Automatically symlinked into every new project by `init_project.sh`. Required for any figure work. |
| **Source-level figure workflow** | Figures are edited by modifying their Python generator script (`.py`), never by editing the rendered PDF, SVG, or PNG. Re-running the generator produces fresh artifacts. |

---

## Common Issues & Fixes

### "command not found: claude"

**Cause**: Claude Code CLI is not installed or not in your PATH.

**Fix**:
```bash
npm install -g @anthropic-ai/claude-code
```
If `npm` is not found, install Node.js first: `brew install node` (macOS).

---

### "command not found: jq"

**Cause**: The `jq` tool (used by hooks) is not installed.

**Fix**:
```bash
# macOS:
brew install jq

# Linux:
sudo apt install jq
```

---

### Hooks are not firing / No [LESSONS CHECK] messages

**Possible causes**:
1. **Settings not loaded**: Close and restart your Claude session. Hooks are loaded at session start.
2. **Symlinks broken**: Check that `.claude/hooks/` points to the right place:
   ```bash
   ls -la .claude/hooks/
   # Should show files like breath_check.sh, lessons_check.sh, etc.
   ```
3. **Not executable**: Make hooks executable:
   ```bash
   chmod +x .claude/hooks/*.sh
   ```

---

### "research_state.md does not exist yet"

**Cause**: The state document was not created during project initialization.

**Fix**: The agent will guide you to create one. Or create it manually:
```bash
# From your project directory, ask the agent:
"Create a research_state.md using the template from /research-collaborator"
```

---

### The agent seems to have forgotten previous sessions

**Possible causes**:
1. **State document outdated**: Check `research_state.md` — when was it last updated?
2. **Knowledge base empty**: Check `knowledge/` — are there entries?
3. **Different directory**: Make sure you're in the right project folder (`pwd`)

**Fix**: Tell the agent: "Re-read research_state.md and the knowledge base indexes."

---

### "I see [BREATH CHECK] messages but I didn't do anything"

**This is normal.** The breath check fires based on elapsed time (every 10 minutes), not based on your actions. If the agent has been working on a task for 10+ minutes, you'll see this message. The agent handles it automatically — you don't need to respond.

---

### The agent is not challenging my ideas

**Possible causes**:
1. You may be in a short exchange where critical thinking isn't triggered
2. The behavioral norms may not be loaded

**Fix**: Say: "I want you to challenge this assumption: [your assumption]. What could go wrong?"

Or invoke the full behavioral norms: `/research-collaborator`

---

### Knowledge base feels messy / too many entries

**This is expected** in active projects. The framework auto-archives entries marked as "superseded" or "resolved" at the end of each session.

To manually clean up:
- Mark outdated decisions: change their status to "SUPERSEDED by D[newer]"
- The `kb_index_hook` will archive them on next session end

### `/review-figure` reports "FIGURE_AGENT_PROMPT.md not found"

This means the figure style guide is not reachable from your project. The review skill fails loud rather than silently falling back to "general publication conventions" (that would produce an incomplete review with no §1–§11 traceability).

**Fix**: symlink the style guide into your project root:

```bash
cd <your-project>
ln -s ../research-collaborator/FIGURE_AGENT_PROMPT.md FIGURE_AGENT_PROMPT.md
```

Projects created by `init_project.sh` get this symlink automatically. If you have an older project, this one-time backfill is all you need.

### `figure-specialist` refuses to edit a PDF or SVG directly

This is by design. Figures are modified at the source level — the subagent locates the `.py` generator, edits it, and re-runs. If only a PDF or SVG exists with no generator, the subagent will escalate.

**Fix**: if the figure has a `.mmd` Mermaid spec, ask for a BOOTSTRAP: the subagent reconstructs a Python generator from the spec, then applies your edit. If neither a `.py` nor a `.mmd` exists, the figure needs to be rebuilt from scratch.

---

## Frequently Asked Questions

### "Do I need to know Python?"

**No.** You can use the framework for literature review, experimental design, writing, and knowledge management without writing any code. If your research involves code (data analysis, running models), the agent can write and run it for you — but you should be able to review what it produces.

### "Can I use this for qualitative research?"

**Yes, with some adaptation.** The knowledge base categories (decisions, findings, questions) work for any research methodology. The experiment tracking is designed for quantitative work, but you can use it to track interview rounds, coding passes, or thematic analysis iterations.

### "How much does it cost?"

| Option | Monthly Cost | What You Get |
|--------|-------------|-------------|
| Claude Pro + Claude Code | $20/month | Full framework with all features |
| Claude Max + Claude Code | $60/month | Same features, higher usage limits |
| GitHub Copilot (alternative) | $10/month | Basic AI coding help, no framework features |

### "Can multiple people collaborate on one project?"

**Not simultaneously.** The framework is designed for one researcher + one AI agent. However, you can share your project folder (via git or shared drive), and different team members can run their own sessions at different times. The knowledge base and state document persist between sessions regardless of who runs them.

### "What if I want to switch to a different AI model?"

The framework is built specifically for Claude Code. It uses Claude-specific features (hooks, skills, agents) that don't exist in other tools. If you switch to a different AI, you'd lose the automatic checks, knowledge management, and research protocols — though your knowledge base files (plain markdown) remain readable by any tool.

### "Can I use this on Windows?"

**Limited support.** Claude Code runs on macOS and Linux. On Windows, you can use WSL (Windows Subsystem for Linux) to get a Linux environment. The setup is more involved — ask your IT department or a tech-savvy colleague for help with WSL installation.

### "What happens if I lose my project folder?"

If you're using git (recommended), your entire project history is backed up. If not, the project folder is your single source of truth — back it up regularly. The framework creates automatic backups of the state document in `.state_backups/`, but this doesn't cover everything.

**Recommendation**: Initialize your project as a git repository:
```bash
cd my_first_study
git init
git add -A
git commit -m "Initial project setup"
```

### "The agent is too slow / uses too many tokens"

Some tips:
- Use the "noted" protocol to batch inputs instead of sending many small messages
- For long experiments, the `run_in_background` convention avoids blocking
- Deliberation (`/deliberate`) is intentionally thorough — use `quick` budget for simpler decisions
- Close and restart sessions periodically to clear accumulated context

---

## File Quick Reference

| File | What It Is | You Edit It? |
|------|-----------|-------------|
| `project_brief.md` | Your research scope and goals | Yes — fill this in first |
| `research_state.md` | What's happening now | Rarely — the agent maintains it |
| `CLAUDE.md` | Framework protocols | No — this links to the shared framework |
| `knowledge/*.md` | Accumulated project knowledge | Rarely — the agent writes entries |
| `lessons/*.md` | Process insights and reflections | No — the agent writes these |
| `episodes/*.md` | Session narratives | No — auto-generated |
| `.claude/settings.json` | Hook wiring and configuration | Only during initial setup |
| `.claude/settings.local.json` | Your permissions | Yes — customize for your workflow |
| `.claude/agents/deliberate-*.md` | Reviewer personas | Yes — customize for your domain |
| `FIGURE_AGENT_PROMPT.md` | Figure style guide (symlink to framework) | No — shared style source of truth |
| `manuscript/figures/*.py` | Figure generator scripts (source of truth) | Yes — via `figure-specialist` |
| `manuscript/figures/*.pdf` / `*.png` / `*.gv` | Rendered figure outputs | No — regenerated from `.py` |
