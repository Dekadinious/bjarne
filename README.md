# Bjarne

Autonomous AI development loop. Give it an idea, come back to a finished project.

**TL;DR:** Write what you want in a markdown file → run `bjarne init idea.md` → run `bjarne` → wait → done.

## Quick Start

```bash
# Install
sudo curl -o /usr/local/bin/bjarne https://raw.githubusercontent.com/Dekadinious/bjarne/master/bjarne && sudo chmod +x /usr/local/bin/bjarne

# Create an idea file
echo "A CLI tool that converts markdown to PDF" > idea.md

# Initialize (creates CONTEXT.md and TASKS.md)
bjarne init idea.md

# Run the loop
bjarne
```

That's it. Bjarne will loop through tasks until your project is built.

## The Key Concept: Outcome-Focused Tasks

Bjarne isn't a dumb "keep running until done" loop. It uses **verifiable outcomes**.

Every task in `TASKS.md` is a multi-line block:

```markdown
- [ ] Action summary (imperative verb)
  Optional context: guidance, references, constraints.
  > Verification point 1 (machine-checkable)
  > Verification point 2 (if needed)
```

The checkbox line stays short and scannable. Description lines provide context for implementation. Verification lines (`> ` prefix) define machine-checkable outcomes. Simple tasks can still be one-liners.

**Examples at different complexity levels:**

```markdown
- [ ] Install dependencies and verify dev server starts
  > node_modules/ directory exists
  > npm run dev starts without errors

- [ ] Add login button to navbar
  > Button with href="/login" exists in header component

- [ ] Create /api/users endpoint
  Follow existing /api/posts pattern.
  Use auth middleware from src/middleware/auth.ts.
  > GET /api/users returns 200 with JSON array
  > Response includes id, name, email fields
  > Unauthenticated requests return 401
```

Outcomes must be **machine-verifiable**. Bjarne's REVIEW phase actually checks if each outcome was achieved (grep for elements, curl endpoints, verify files exist) before moving on. A task isn't done until all its outcomes are confirmed.

This is what separates Bjarne from naive loops:

| Issue | Dumb Loop | Bjarne |
|-------|-----------|--------|
| Security vulnerabilities | Undetected until prod | Caught in REVIEW |
| DRY violations | Copy-paste spreads | Flagged and refactored |
| Growing monoliths | Files balloon unchecked | Architecture reviewed |
| Broken tests | Ignored or disabled | Must pass to continue |
| Dead code | Accumulates silently | Cleaned up in FIX |

## How It Works

```
idea.md → INIT → VALIDATE → [PLAN → EXECUTE → REVIEW → FIX] × N → Done
                                          ↑
                            notes.md → REFRESH → VALIDATE

        "fix X" → TASK → [PLAN → EXECUTE → REVIEW → FIX] → Branch + PR
                         (isolated state, auto-cleanup)
```

Each iteration:
1. **PLAN** - Picks first unchecked task (or batch of related tasks with `-b`), extracts expected outcomes, writes plan with verification steps
2. **EXECUTE** - Implements the plan, marks task(s) done
3. **REVIEW** - Verifies outcome(s) achieved, then checks code quality
4. **FIX** - Fixes failed outcomes first, then other issues

## Files Bjarne Creates

When you run `bjarne init`, it creates these files that drive the entire process:

| File | Purpose |
|------|---------|
| `CONTEXT.md` | Static project reference - tech stack, architecture, constraints |
| `TASKS.md` | The task list with checkboxes - this is the "brain" |
| `specs/` | Detailed specifications for complex features |
| `.task` | Current task state (temporary, per-iteration) |

`TASKS.md` is the source of truth. Bjarne reads it, picks the next unchecked task, works on it, and marks it done.

## Using Claude to Manage Bjarne

Here's something powerful: **you can use Claude Code to set up and manage Bjarne projects**.

> **Tip:** Before asking Claude to help with Bjarne, have it read this README first:
> ```bash
> claude "Read https://github.com/Dekadinious/bjarne/blob/master/README.md then help me set up a Bjarne project for [your idea]"
> ```

### Let Claude Write Your Idea File

Not sure how to describe what you want? Have Claude interview you:

```bash
claude "I want to build [brief description]. Ask me questions to understand what I need, then write a detailed idea.md file I can use with Bjarne."
```

Claude will ask about tech preferences, features, constraints, and produce a well-structured idea file.

### Let Claude Create CONTEXT.md and TASKS.md Directly

You don't have to use `bjarne init`. You can have Claude create the files manually:

```bash
claude "Look at this codebase and create a CONTEXT.md and TASKS.md for adding [feature]. Use the multi-line task format with '> Verification point' lines."
```

This is useful when:
- You want more control over the task breakdown
- You're adding features to an existing project
- You want to review/edit tasks before Bjarne runs

### Let Claude Refine Tasks Mid-Project

Bjarne running but tasks aren't quite right? Stop it and ask Claude to help:

```bash
claude "Look at TASKS.md. The tasks aren't specific enough. Rewrite them with clearer, more verifiable outcomes."
```

### Let Claude Review Progress

After Bjarne finishes (or if you stop it):

```bash
claude "Review what Bjarne built. Check TASKS.md for completed items and verify the outcomes were actually achieved. List anything that needs fixing."
```

### The Meta-Workflow

The most powerful pattern:

1. **Claude** helps you write `idea.md`
2. **Bjarne** runs `init` and creates tasks
3. **Claude** reviews and refines `TASKS.md` if needed
4. **Bjarne** executes the loop
5. **Claude** reviews the output, writes `notes.md`
6. **Bjarne** runs `refresh notes.md` and continues
7. Repeat until satisfied

You're using Claude as the "project manager" and Bjarne as the "worker".

## Writing Good Idea Files

### Simple Ideas

For simple projects, a one-liner is fine:

```markdown
A CLI tool that converts markdown files to PDF
```

Bjarne will fill in sensible defaults for tech stack, testing, etc.

### Detailed Ideas

For specific requirements, be explicit:

```markdown
# Invoice Generator

A web app for freelancers to create and manage invoices.

## Tech Stack
- Next.js 14 with App Router
- SQLite database with Drizzle ORM
- Tailwind CSS

## Features
- Dashboard showing all invoices with status (draft/sent/paid)
- Create invoice form: client name, line items, tax rate
- Generate PDF with professional template

## Constraints
- No authentication (single user)
- All amounts in USD
```

### Tips

- **Simple ideas** get sensible defaults
- **Detailed ideas** are respected exactly as written
- Put constraints if you care (e.g., "use Python", "no dependencies")
- Bjarne runs headless - it can't ask questions, so be clear upfront
- The more detail you provide, the closer the result matches your vision

## Commands Reference

| Command | What it does |
|---------|--------------|
| `bjarne init idea.md` | Create project from idea file |
| `bjarne init --safe idea.md` | Same, but enables Docker sandbox |
| `bjarne` | Run the development loop |
| `bjarne 50` | Run with 50 iterations (default: 25) |
| `bjarne --batch` | Enable batch mode (up to 5 related tasks) |
| `bjarne --batch 50` | Batch mode, 50 iterations |
| `bjarne --batch=3` | Batch mode, custom size (up to 3 tasks) |
| `bjarne --batch=3 50` | Batch up to 3 tasks, 50 iterations |
| `bjarne refresh notes.md` | Add tasks from feedback notes |
| `bjarne task "description"` | Run isolated single-task fix |
| `bjarne --rebuild` | Rebuild Docker image (safe mode) |

## Workflow Details

### Initialize

```bash
bjarne init idea.md
```

Creates `CONTEXT.md`, `TASKS.md`, and optionally `specs/`. Then runs a **validation pass** that checks for vague tasks, ordering issues, contradictions, duplicates, and scope creep before the loop starts.

**Works on existing projects!** If you run `init` in a folder with existing code, Bjarne will detect your codebase, understand what's built, and create tasks that build on your existing code.

### Run

```bash
bjarne          # Default: max 25 iterations
bjarne 50       # Custom: max 50 iterations
```

### Batch Mode

By default, Bjarne processes one task per PLAN → EXECUTE → REVIEW → FIX cycle. Batch mode groups related tasks together:

```bash
bjarne --batch        # Batch mode, up to 5 related tasks
bjarne --batch 50     # Batch mode, 50 iterations
bjarne --batch=3      # Batch mode, up to 3 related tasks
bjarne --batch=3 50   # Up to 3 tasks, 50 iterations
bjarne -b3 50         # Short form: up to 3 tasks, 50 iterations
```

**How it works:** Instead of picking just the first unchecked task, Bjarne scans all pending tasks and groups ones that naturally belong together — same file, same feature, logical dependencies. It might batch 1 task (if standalone) or up to N tasks (if tightly coupled).

**Trade-offs:**

| | Single Task (default) | Batch Mode |
|---|---|---|
| Context usage | Higher (full cycle per task) | Lower (one cycle for multiple tasks) |
| Speed | Slower | Faster |
| Precision | Higher (focused attention) | Possibly lower (split attention) |
| Best for | Complex tasks, precision work | Related tasks, faster iteration |

**When to use batch mode:**
- Many small, related tasks (e.g., "add field X", "add field Y", "add field Z")
- Tasks grouped by phase in TASKS.md
- You want faster iteration and accept slight precision trade-off

**When to stick with single-task:**
- Complex architectural tasks
- Tasks requiring careful, focused implementation
- When precision matters more than speed

### Refresh

After testing your project, found bugs or want new features? Write freeform notes:

```markdown
# notes.md
The login button doesn't work on mobile
Add a dark mode toggle
Would be nice to have a loading spinner
```

Then:
```bash
bjarne refresh notes.md
bjarne  # run again
```

Bjarne converts your notes into properly formatted tasks, runs a validation pass to catch conflicts with existing tasks, and continues.

### Task Mode (Isolated Fixes)

Need a quick fix without touching your main project state?

```bash
bjarne task "Fix the login button not responding"
```

Task mode:
- Creates isolated state in `.bjarne/tasks/<task-id>/`
- Runs the full PLAN → EXECUTE → REVIEW → FIX loop
- Creates a git branch and PR (if git/gh available)
- Cleans up after itself

Options:
```bash
bjarne task "description"              # Text description
bjarne task bugfix.md                  # Read from file
bjarne task --safe "description"       # Docker sandbox
bjarne task --no-pr "..."              # Skip PR creation
bjarne task -n 10 "..."                # Limit to 10 iterations
```

**When to use which:**

| Scenario | Use |
|----------|-----|
| Building a new project | `bjarne init` + `bjarne` |
| Adding features to existing project | `bjarne refresh` + `bjarne` |
| Quick isolated fix | `bjarne task` |
| Multiple fixes in parallel | Multiple `bjarne task` in different terminals |

## Safe Mode

Running Bjarne unattended? Use safe mode:

```bash
bjarne init --safe idea.md   # Creates Docker config
bjarne                        # Automatically runs in container
```

You only need `--safe` on `init`. Once `.bjarne/Dockerfile` exists, all subsequent `bjarne` runs automatically detect it and use Docker.

Safe mode:
- Runs Claude inside a Docker container
- Container can ONLY see your project directory
- Rest of your system is completely protected
- Your Claude credentials are mounted read-only

Auto-detects your tech stack (Node.js, Python, Rust, Go, PHP) and uses the appropriate Docker image.

Customize:
```bash
vim .bjarne/Dockerfile       # Edit as needed
bjarne --rebuild             # Rebuild with changes
```

Requires [Docker](https://docs.docker.com/get-docker/).

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- **macOS or Linux** (Windows users: use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install))
- [Docker](https://docs.docker.com/get-docker/) (optional, for safe mode)

## All Files Reference

| File | Purpose |
|------|---------|
| `CONTEXT.md` | Static project reference |
| `TASKS.md` | Checkbox task list (main state) |
| `specs/` | Detailed specifications |
| `.task` | Current task state (temporary) |
| `.bjarne/Dockerfile` | Docker config (safe mode) |
| `.bjarne/logs/` | Session logs and failure details |
| `.bjarne/tasks/<id>/` | Task mode state (isolated, auto-cleaned) |

## Auto-Update

Bjarne checks for updates every 2 days:

```
A new version of bjarne is available.
Do you want to update? [y/N]
```

```bash
bjarne --disable-auto-update   # Disable
bjarne --enable-auto-update    # Re-enable
```

## Standing on the Shoulders of Ralph

Bjarne is inspired by the [Ralph Wiggum technique](https://ghuntley.com/ralph/), created by [Geoffrey Huntley](https://ghuntley.com/) — a goat farmer in rural Australia who proved that "dumb things can work surprisingly well."

The original Ralph was beautifully simple: a bash loop that keeps running Claude until the job is done. Geoffrey once ran it for three months straight and woke up to a fully functional programming language with Gen Z slang keywords.

Bjarne adds structure to the chaos — task planning, code review, and fix cycles — but the spirit is the same: *naive persistence wins*.

## License

MIT
