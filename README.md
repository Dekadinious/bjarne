# Bjarne

Autonomous AI development loop. Give it an idea, come back to a finished project.

**TL;DR:** Write what you want in a markdown file → run `bjarne init idea.md` → run `bjarne` → wait → done.

## How It Works

```
idea.md → INIT → [PLAN → EXECUTE → REVIEW → FIX] × N → Done
                              ↑
                    notes.md → REFRESH (add more tasks)

        "fix X" → TASK → [PLAN → EXECUTE → REVIEW → FIX] → Branch + PR
                         (isolated state, auto-cleanup)
```

Bjarne reads your idea, creates a task list, then loops through each task autonomously until everything is built. After testing, you can add more tasks with `refresh`. For quick isolated fixes, use `task` mode.

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- **macOS or Linux** (Windows users: use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install))
- [Docker](https://docs.docker.com/get-docker/) (optional, for safe mode)

## Install

```bash
sudo curl -o /usr/local/bin/bjarne https://raw.githubusercontent.com/Dekadinious/bjarne/main/bjarne && sudo chmod +x /usr/local/bin/bjarne
```

Or just give Claude this repo and ask it to set things up:
```
Install bjarne from https://github.com/Dekadinious/bjarne
```

## Commands

| Command | What it does |
|---------|--------------|
| `bjarne init idea.md` | Create project from idea file |
| `bjarne init --safe idea.md` | Same, but enables Docker sandbox |
| `bjarne` | Run the development loop |
| `bjarne 50` | Run with 50 iterations (default: 25) |
| `bjarne refresh notes.md` | Add tasks from feedback notes |
| `bjarne task "description"` | Run isolated single-task fix |
| `bjarne --rebuild` | Rebuild Docker image (safe mode) |

## Usage

### 1. Write an idea file

Create `idea.md` describing what you want to build.

**Simple example:**
```markdown
A CLI tool that converts markdown files to PDF
```

**Detailed example:**
```markdown
# Invoice Generator

A web app for freelancers to create and manage invoices.

## Tech Stack
- Next.js 14 with App Router
- SQLite database with Drizzle ORM
- Tailwind CSS
- PDF generation with @react-pdf/renderer

## Features
- Dashboard showing all invoices with status (draft/sent/paid)
- Create invoice form: client name, line items, tax rate, due date
- Auto-calculate totals and tax
- Generate PDF with professional template
- Mark invoices as sent/paid
- Filter by status and date range

## Data Model
- Invoice: id, client_name, status, issue_date, due_date, tax_rate, created_at
- LineItem: id, invoice_id, description, quantity, unit_price

## Pages
- / - Dashboard with invoice list
- /new - Create invoice form
- /invoice/[id] - View invoice with PDF download
- /invoice/[id]/edit - Edit draft invoice

## Constraints
- No authentication (single user)
- All amounts in USD
- Tax rate per invoice (not per item)
```

### Let Claude write your idea

Not sure how to write a good idea file? Let Claude help:

```bash
claude "I want to build [brief description]. Ask me questions to understand what I need, then write a detailed idea.md file I can use with an autonomous coding agent. Ask about tech preferences, features, constraints, and anything else that would help define the project clearly."
```

Claude will interview you and produce a well-structured idea file.

### 2. Initialize

```bash
bjarne init idea.md
```

This creates:
- `CONTEXT.md` - Project overview
- `TASKS.md` - Checkbox task list
- `specs/` - Detailed specs (if needed)

**Works on existing projects too!** If you run `init` in a folder with existing code, Bjarne will:
- Detect and explore your codebase
- Understand what's already built
- Create tasks that build ON your existing code

### 3. Run

```bash
bjarne
```

Bjarne loops through tasks until done (default: max 25 iterations).

Want more iterations?
```bash
bjarne 50
```

### Safe Mode (recommended for unattended runs)

Running Bjarne overnight? Use safe mode to prevent any chance of accidental damage to files outside your project:

```bash
bjarne init --safe idea.md   # Creates Docker config
bjarne                        # Runs in isolated container
```

**What safe mode does:**
- Runs Claude inside a Docker container
- Container can ONLY see your project directory
- Rest of your system is completely protected
- Your Claude credentials are mounted read-only

**Customize the container:**
```bash
vim .bjarne/Dockerfile       # Edit as needed
bjarne --rebuild             # Rebuild with changes
```

Safe mode auto-detects your tech stack (Node.js, Python, Rust, Go, PHP) and uses the appropriate Docker image.

> **Requires:** [Docker](https://docs.docker.com/get-docker/) installed

### 4. Refresh (optional)

After Bjarne finishes, test your project manually. Found bugs? Want new features? Write freeform notes:

```markdown
# notes.md

The login button doesn't work on mobile
Add a dark mode toggle
The API returns 500 when email is empty
Would be nice to have a loading spinner
```

Then refresh:

```bash
bjarne refresh notes.md
bjarne  # run again to work through new tasks
```

Bjarne reads your notes, adds tasks to `TASKS.md`, and you're back in the loop.

### 5. Task Mode (isolated fixes)

Need a quick fix without touching your main project state? Use task mode:

```bash
bjarne task "Fix the login button not responding"
```

**What task mode does:**
- Creates isolated state in `.bjarne/tasks/<task-id>/`
- Breaks your request into subtasks
- Runs the full PLAN → EXECUTE → REVIEW → FIX loop
- Creates a git branch and PR (if git/gh available)
- Cleans up after itself

**Options:**
```bash
bjarne task "description"              # Basic usage
bjarne task -f bugfix.md               # Read description from file
bjarne task --safe "description"       # Run in Docker sandbox
bjarne task --branch feature/foo "..." # Custom branch name
bjarne task --no-pr "..."              # Skip PR creation
bjarne task -n 10 "..."                # Limit to 10 iterations
```

**When to use task mode vs regular mode:**
| Scenario | Use |
|----------|-----|
| Building a new project | `bjarne init` + `bjarne` |
| Adding features to existing project | `bjarne refresh` + `bjarne` |
| Quick isolated fix | `bjarne task` |
| Running multiple fixes in parallel | Multiple `bjarne task` in different terminals |

Task mode is sandboxed — it never touches your `CONTEXT.md`, `TASKS.md`, or root `.task` file. You can run multiple task instances simultaneously on different parts of your codebase.

## What Happens Each Iteration

1. **PLAN** - Picks first unchecked task, extracts expected outcome, writes plan with verification steps
2. **EXECUTE** - Implements the plan, marks task done
3. **REVIEW** - Verifies outcome was achieved, then checks code quality
4. **FIX** - Fixes failed outcomes first, then other issues

## Outcome-Focused Tasks

Tasks use the format: `- [ ] Action → Outcome`

```markdown
- [ ] Add login button to navbar → Button with href="/login" exists in header
- [ ] Create /api/users endpoint → GET /api/users returns 200 with JSON array
- [ ] Add email validation → Invalid email shows error message
```

The outcome must be **machine-verifiable**. REVIEW actually checks if the outcome was achieved (grep for elements, curl endpoints, verify files exist) before moving on. A task isn't done until its outcome is confirmed.

## Why Not Just a Dumb Loop?

Pure "keep running until done" loops can build working code, but they accumulate problems:

| Issue | Dumb Loop | Bjarne |
|-------|-----------|--------|
| Security vulnerabilities | Undetected until prod | Caught in REVIEW |
| DRY violations | Copy-paste spreads | Flagged and refactored |
| Growing monoliths | Files balloon unchecked | Architecture reviewed |
| Broken tests | Ignored or disabled | Must pass to continue |
| Dead code | Accumulates silently | Cleaned up in FIX |

The REVIEW step acts as a quality gate. Code doesn't just need to *work* — it needs to pass inspection before moving on.

## Tips

- **Simple ideas** get sensible defaults (tech stack, testing, etc.)
- **Detailed ideas** are respected exactly as written
- Put constraints in your idea if you care (e.g., "use Python", "no dependencies")
- Bjarne runs headless - it can't ask you questions, so be clear upfront
- The more detail you provide, the closer the result matches your vision

## Files Bjarne Uses

| File | Purpose |
|------|---------|
| `CONTEXT.md` | Static project reference |
| `TASKS.md` | Checkbox task list (main state) |
| `specs/` | Detailed specifications |
| `.task` | Current task state (temporary) |
| `.bjarne/Dockerfile` | Docker config (safe mode) |
| `.bjarne/logs/` | Session logs and failure details |
| `.bjarne/tasks/<id>/` | Task mode state (isolated, auto-cleaned) |

## Standing on the Shoulders of Ralph

Bjarne is inspired by the [Ralph Wiggum technique](https://ghuntley.com/ralph/), created by [Geoffrey Huntley](https://ghuntley.com/) — a goat farmer in rural Australia who proved that "dumb things can work surprisingly well."

The original Ralph was beautifully simple: a bash loop that keeps running Claude until the job is done. Geoffrey once ran it for three months straight and woke up to a fully functional programming language with Gen Z slang keywords (`slay` for function, `sus` for variable, `based` for true).

Bjarne adds structure to the chaos — task planning, code review, and fix cycles — but the spirit is the same: *naive persistence wins*.

Thanks Geoffrey. Your goats would be proud.

## License

MIT
