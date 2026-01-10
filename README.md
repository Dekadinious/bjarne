# Bjarne

Autonomous AI development loop. Give it an idea, it builds the project.

## How It Works

```
idea.md → INIT → [PLAN → EXECUTE → REVIEW → FIX] × N → Done
                              ↑
                    notes.md → REFRESH (add more tasks)
```

Bjarne reads your idea, creates a task list, then loops through each task autonomously until everything is built. After testing, you can add more tasks with `refresh`.

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- **macOS or Linux** (Windows users: use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install))

## Install

```bash
sudo curl -o /usr/local/bin/bjarne https://raw.githubusercontent.com/Dekadinious/bjarne/main/bjarne && sudo chmod +x /usr/local/bin/bjarne
```

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

## What Happens Each Iteration

1. **PLAN** - Picks first unchecked task, searches codebase, writes plan
2. **EXECUTE** - Implements the plan, marks task done
3. **REVIEW** - Runs tests, checks for issues
4. **FIX** - Fixes any problems found

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

## Standing on the Shoulders of Ralph

Bjarne is inspired by the [Ralph Wiggum technique](https://ghuntley.com/ralph/), created by [Geoffrey Huntley](https://ghuntley.com/) — a goat farmer in rural Australia who proved that "dumb things can work surprisingly well."

The original Ralph was beautifully simple: a bash loop that keeps running Claude until the job is done. Geoffrey once ran it for three months straight and woke up to a fully functional programming language with Gen Z slang keywords (`slay` for function, `sus` for variable, `based` for true).

Bjarne adds structure to the chaos — task planning, code review, and fix cycles — but the spirit is the same: *naive persistence wins*.

Thanks Geoffrey. Your goats would be proud.

## License

MIT
