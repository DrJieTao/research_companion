# Setting Up Your Environment

This guide walks you through everything from scratch. If any step feels unfamiliar, that's expected — just follow along and you'll be fine.

## Step 0: Understanding the Terminal

The **terminal** (also called "command line" or "shell") is a text-based way to interact with your computer. Instead of clicking folders and buttons, you type commands. It looks something like this:

```
$ ls
Documents  Downloads  Desktop  Pictures
$ cd Documents
$ ls
my_papers  research_data  notes.txt
```

That's it — you type a command, press Enter, and the computer responds. Here are the only commands you need to know:

| Command | What It Does | Example |
|---------|-------------|---------|
| `ls` | List files in the current folder | `ls` |
| `cd` | Change to a different folder | `cd Documents` |
| `cd ..` | Go up one folder | `cd ..` |
| `pwd` | Show which folder you're in | `pwd` |

### Opening the Terminal

**macOS:**
1. Press `Cmd + Space` to open Spotlight
2. Type "Terminal" and press Enter
3. A window with a text prompt appears — that's your terminal

**Linux:**
- Press `Ctrl + Alt + T` (works on most distributions)

> **Don't worry**: The terminal can't break your computer. If you type something wrong, you'll just get an error message. You can always close the window and start over.

---

## Step 1: Choose Your Setup

You have three options, depending on your budget and comfort level:

### Option A: Claude Code (Recommended)

**Cost**: Claude Pro subscription ($20/month) or Max ($60/month)
**Best for**: Full research collaboration experience with all framework features

Claude Code is Anthropic's tool that lets Claude work directly with files on your computer. It's available as:
- A **command-line tool** (what we'll install below)
- A **desktop app** for macOS and Windows
- A **VS Code extension** (see Option B)

**Install Claude Code CLI:**

```bash
# On macOS or Linux, run this in your terminal:
npm install -g @anthropic-ai/claude-code
```

If you don't have `npm`, install Node.js first:
```bash
# macOS (using Homebrew):
brew install node

# If you don't have Homebrew either:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install node
```

**Verify it worked:**
```bash
claude --version
# You should see something like: claude-code 1.x.x
```

**Sign in:**
```bash
claude
# Follow the prompts to sign in with your Anthropic account
```

### Option B: VS Code + Claude Code Extension (Beginner-Friendly)

**Cost**: Same as Option A (requires Claude subscription)
**Best for**: Researchers who prefer a visual interface over the terminal

VS Code is a free code editor with a built-in file explorer and terminal. The Claude Code extension runs inside it.

1. **Install VS Code**: Download from [code.visualstudio.com](https://code.visualstudio.com)
2. **Install the Claude Code extension**:
   - Open VS Code
   - Click the Extensions icon in the left sidebar (looks like four squares)
   - Search for "Claude Code"
   - Click "Install"
3. **Open your project folder**: File > Open Folder > select your project
4. **Open the terminal panel**: View > Terminal (or press `` Ctrl + ` ``)

You now have a file explorer on the left and a terminal at the bottom — all in one window.

### Option C: VS Code + GitHub Copilot (Budget Alternative)

**Cost**: $10/month
**Best for**: Researchers who want basic AI coding help without the full framework

> **Honest comparison**: GitHub Copilot is a general-purpose coding assistant. It's great for writing code, explaining code, and answering questions. However, it **cannot** use the Research Collaborator framework — no persistent knowledge base, no research protocols, no automatic self-checks. Think of it as a helpful but forgetful assistant, versus a collaborator with a shared notebook.

| Capability | Claude Code + Framework | GitHub Copilot |
|-----------|------------------------|----------------|
| Answer questions about code | Yes | Yes |
| Help write scripts | Yes | Yes |
| Remember decisions across sessions | Yes | No |
| Follow research protocols | Yes | No |
| Self-check during long tasks | Yes | No |
| Maintain structured knowledge base | Yes | No |
| Multi-perspective deliberation | Yes | No |
| **Monthly cost** | **$20** | **$10** |

**When Copilot makes sense**: You only need occasional help with Python/R scripts or data analysis, and you're comfortable managing your own notes and decisions.

**Upgrade path**: Start with Copilot, and when you find yourself re-explaining context every session, that's your signal to move to Claude Code + the framework.

To install: Open VS Code > Extensions > Search "GitHub Copilot" > Install > Sign in with GitHub.

---

## Step 2: Install Dependencies

The framework needs a few standard tools. Open your terminal and run:

**jq** (a tool for processing structured data — used by the framework's automatic checks):
```bash
# macOS:
brew install jq

# Linux (Debian/Ubuntu):
sudo apt install jq
```

**uv** (a fast Python package manager — used for running analysis scripts):
```bash
# macOS or Linux:
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Verify both:**
```bash
jq --version
# Should show: jq-1.x

uv --version
# Should show: uv 0.x.x
```

---

## Step 3: Get the Research Collaborator Framework

You need the `research-collaborator` folder on your computer. If someone shared it with you:

```bash
# Navigate to where you want your research projects:
cd ~/Documents

# Create a folder for your research projects (if you don't have one):
mkdir research_projects
cd research_projects

# Copy or clone the research-collaborator folder here
# (your PI or collaborator will tell you how to get it)
```

Your folder structure should look like:
```
research_projects/
  research-collaborator/    <-- the framework (shared)
```

---

## Step 4: Create Your First Project

This is the exciting part. From inside the `research-collaborator` folder, run:

```bash
cd research-collaborator
./init_project.sh my_first_study "My Research Title Here"
```

Replace `my_first_study` with a short name (no spaces) and `"My Research Title Here"` with your actual project title.

**What you should see:**
```
Initializing project: my_first_study
  Title: My Research Title Here
  Location: /home/you/Documents/research_projects/my_first_study
  Framework: /home/you/Documents/research_projects/research-collaborator

Project initialized at /home/you/Documents/research_projects/my_first_study

Next steps:
  1. Fill in project_brief.md with your research details
  2. Customize .claude/agents/deliberate-*.md personas for your domain
  3. Create .claude/settings.local.json with project-specific permissions
  4. Start a Claude Code session: cd ../my_first_study && claude
  5. The session_start hook will guide initial setup
```

---

## Step 5: Fill In Your Project Brief

Before starting your first session, tell the framework about your research. Open `project_brief.md` in any text editor (VS Code, TextEdit, nano — whatever you're comfortable with):

```bash
cd ../my_first_study

# Open in VS Code:
code project_brief.md

# Or open in the default text editor (macOS):
open project_brief.md
```

Fill in each section:

| Section | What to Write | Example |
|---------|--------------|---------|
| PI / Lead Researcher | Your name and affiliation | "Dr. Jane Smith, Dept. of Psychology, University X" |
| Research Question | What you're investigating | "How does social media language predict depression onset?" |
| Objectives | 2-4 concrete goals | "1. Build a classifier... 2. Evaluate against..." |
| Dataset | Where your data comes from | "ANGST dataset from HuggingFace (2,876 labeled posts)" |
| Methods | High-level approach | "Multi-agent LLM classification with iterative feedback" |
| Target Venues | Where you plan to publish | "HICSS 2027, JMIR Mental Health" |
| Timeline | Key milestones | "Phase 1: Prototype by March, Phase 2: Evaluation by May" |
| Success Criteria | How you'll know it worked | "Macro F1 > 0.45 on 4-class task" |

> **Tip**: You don't need to have everything figured out. Write what you know now — the framework will help you refine it as you go.

---

## Step 6: Start Your First Session

```bash
claude
```

That's it. Head to [02 First Session](02_first_session.md) to learn what happens next.

---

## Checklist

Before moving on, confirm:

- [ ] Claude Code is installed and you can run `claude --version`
- [ ] `jq` is installed (`jq --version`)
- [ ] `uv` is installed (`uv --version`)
- [ ] You've run `init_project.sh` and your project folder exists
- [ ] You've filled in at least the Research Question and Objectives in `project_brief.md`
