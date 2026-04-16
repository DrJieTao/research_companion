# Research Collaborator — What It Is and How It Works

## The Problem

When you use ChatGPT or Claude on the web, every conversation starts from scratch. The AI forgets your decisions, loses track of your experiments, and drifts off-plan during long sessions. For a research project that spans weeks or months, this means constantly re-explaining context and manually catching mistakes.

## What the Research Collaborator Does

It turns Claude into a persistent research partner that **remembers everything, follows protocols, and checks its own work**. You work in a project folder on your computer. Inside that folder, the framework maintains structured files that accumulate your project's knowledge across sessions.

## How It's Organized

The framework has three types of components:

**Things the AI reads** (your project's memory):
- **Project Brief** — your research goals, datasets, success criteria (you write this once)
- **State Document** — what you're working on right now, all decisions made, open questions. Updated every session. Includes a **Roadmap** (table of major direction changes) and a **Task Queue** (tracked work items)
- **Knowledge Base** — organized entries for decisions, experiments, findings, literature, questions, and peer reviews. Entries are never deleted, only archived
- **Lessons Learned** — insights about *why* things worked or didn't (not just what happened). Older lessons gradually fade from prominence

**Things that happen automatically** (background guardrails):
- At **session start**: the AI loads your project context and recent lessons
- Before **each prompt**: it sees a summary of your knowledge base and any open tickets
- Before **each action**: it reviews relevant lessons from past work
- Every **10 minutes**: it checks whether it's still on-plan
- After **each response**: it verifies it updated the state document
- After **experiments/milestones**: it formally evaluates whether to proceed, refine, or change direction

**Specialized helpers** (delegate tasks without losing focus):
- A **literature researcher** searches academic databases
- A **code implementer** writes and tests Python code
- A **data analyst** runs experiments and produces statistics
- A **figure specialist** creates publication-quality diagrams
- During high-stakes decisions, four simulated **peer review personas** (friendly collaborator, journal editor, hostile reviewer, pragmatic strategist) debate the options before recommending an action

## Key Workflows

**Dropping ideas** ("noted" protocol): You can rapidly share multiple ideas. The AI responds "noted" to each one and silently classifies them in the background — questions become tracked tickets, observations become findings, references become literature entries. When you say "I am done," the AI engages with everything at once.

**Planning**: For complex tasks, the AI drafts a plan using a subagent, then has a second subagent critique it. The AI synthesizes both into a final proposal. For simple tasks, it plans directly. Either way, it checks past lessons before proposing and waits for your explicit approval before executing. You always have the final say.

**Decisions**: After experiments finish, the AI recommends one of three paths: **proceed** (results are good, move forward), **refine** (partially promising, tweak and retry with a budget), or **pivot** (assumptions are wrong, change direction). Each decision is recorded with its rationale and versioned so you can roll back.

**Project management**: Research requests become tracked tickets in a backlog. The AI surfaces open tickets before each session and asks whether to prioritize existing work or take on something new. When a decision resolves a question, the ticket is automatically closed.

## What You Control vs. What the AI Controls

| You decide | The AI handles |
|---|---|
| Research direction and goals | Maintaining structured records |
| Which experiments to run | Running background checks and maintenance |
| Whether to proceed, refine, or pivot | Indexing, archiving, and surfacing past knowledge |
| When to start and stop tasks | Drafting plans for your approval |
| Approving or rejecting plans | Delegating work to specialized helpers |

The AI proposes; you approve. It never starts work, closes tickets, or changes direction without your explicit permission.

## What Makes This Different from a Chatbot

A chatbot answers questions. The Research Collaborator is closer to a junior researcher who keeps meticulous lab notes, remembers every past experiment, challenges your assumptions, and manages the project backlog — but always defers to your judgment on what to do next.
