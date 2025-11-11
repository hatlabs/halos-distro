# Claude Code Project Planning Guide

This document describes the project planning and development workflow for Claude Code-driven projects backed by GitHub repositories.

## Philosophy

- **Specs in files**: Detailed specifications, architecture, and documentation live in markdown files in the repository
- **Tasks in issues**: Development tasks are tracked as GitHub issues for clear state management and git history
- **Direct conversation**: Actual development happens through natural language conversation with Claude Code
- **Minimal process**: Just enough structure to be useful, not so much that it slows us down

## Project Repository

- Create in the "hatlabs" GitHub organization unless otherwise specified.
- The repository should be initially _private_.
- Set the repo to automatically delete head branches
- Create a "Protect main branch" ruleset with:
  - Branch targeting criteria: `main` (default branch)
  - Require a pull request before merging
  - Block force pushes
  - Automatically request Copilot code review
  - Copilot sub-option: Review new pushes


## Project Structure

```
project-root/
├── docs/
│   ├── SPEC.md
│   └── ARCHITECTURE.md
├── src/
│   └── ...
└── README.md
```

Note: Actual documentation structure will depend on the specific project needs.

## Initial Planning Workflow

### Step 0: Create Repository

Create and configure the GitHub repository as described above.
Create a local clone for Claude Code to work in.

### Step 1: Technical Specification

**Human provides:** High-level project description and requirements

**Claude Code creates:** `/docs/SPEC.md`

**Contents:**
- Project overview and goals
- Core features and functionality
- Technical requirements
- Key constraints and assumptions
- Non-functional requirements (performance, security, etc.)
- Out of scope items

**Format:** Prose-style documentation. DO NOT include ANY code snippets.

**Human reviews and approves:** Review SPEC.md and request changes if needed before proceeding to next step

### Step 2: System Architecture

**Claude Code creates:** `/docs/ARCHITECTURE.md`

**Contents:**
- System components and their relationships
- Data models and schemas
- Technology stack decisions and rationale
- Integration points and APIs
- Deployment architecture
- Security considerations
- File tree structure

**Format:** Prose with diagrams where helpful. DO NOT include ANY code snippets.

**Human reviews and approves:** Review ARCHITECTURE.md and request changes if needed before proceeding to next step

### Step 3: Task Breakdown

**Claude Code creates:** `TASKS.md` (temporary file, not committed to repo)

**Contents:**
For each major component, create tasks with:
- Implementation details and approach
- Test scenarios
- Acceptance criteria
- Caveats and considerations
- Dependencies on other tasks

**Purpose:** This temporary file helps plan and discuss the implementation before creating GitHub issues.

**Human reviews and approves:** Review TASKS.md and request changes if needed before proceeding to create issues

### Step 4: Create GitHub Issues

After reviewing and refining TASKS.md:

**Claude Code creates:** GitHub issues for each task

**Issue format:**
```
Title: [Brief, clear description]

Body:
[Detailed implementation plan, specs, and approach]

Implementation details:
[Specific approach and technical decisions]

Test scenarios:
- [Test case 1]
- [Test case 2]

Acceptance criteria:
- [ ] [Specific deliverable 1]
- [ ] [Specific deliverable 2]
- [ ] [Tests pass]

Caveats:
[Any considerations, edge cases, or dependencies]
```

**Labels to apply:**
- `type:feature` for new functionality
- `priority:high` for critical/blocking work (sparingly)
- `priority:low` for nice-to-haves
- Unlabeled issues are implicit medium priority

**After issue creation:** Delete TASKS.md as it has served its purpose (or archive outside repository if historical reference is needed)

## Development Workflow

### Typical Development Session

```
Human: "Work on issue #42"
Claude Code:
  - Reads issue from GitHub
  - Implements the feature according to the plan in the issue
  - Writes tests
  - Commits with message: "feat: implement [feature]\n\nImplements #42"
  - Creates PR if needed

Human: [Reviews code via direct conversation]
Human: "The session handling needs adjustment..."
Claude Code: [Makes changes immediately]
Human: "Looks good. Close issue #42"
Claude Code: Closes issue with summary of changes
```

### Iteration Patterns

**All development iteration happens through direct conversation**, not through GitHub comments:

- Quick fixes: Just tell Claude Code what to change
- Code review feedback: Discuss directly, no need to document in issue
- Design discussions: Talk it through, update docs/ files if needed

### Ad-hoc Work

```
Human: "The auth is crashing on empty passwords. Fix it"
Claude Code:
  - Creates issue #47 [type:bug, priority:high] with details
  - Fixes the bug immediately
  - Commits with "fix: handle empty passwords\n\nFixes #47"
  - Closes issue with summary
```

### Updating Documentation

When implementation reveals better approaches or additional details:

```
Human: "Update ARCHITECTURE.md with the new caching approach"
Claude Code: Updates the documentation file based on what was implemented
```

Specs and plans can be added to issues as needed during development. Documentation files get updated to reflect the actual implementation.

### Closing Issues

Issues close automatically when:
- Commit message includes "Fixes #N", "Closes #N", or "Resolves #N"
- PR containing that commit is merged

Manual closing:
```
Human: "Close issue #42"
Claude Code: Closes with summary comment
```

## Open Source Maintenance Stage

Once the project is public and has external users/contributors, the workflow adapts:

### User-Reported Issues

Users create issues via templates. You or automated systems add labels:

- User reports bug → `type:bug` (+ `priority:high` if critical)
- User requests feature → `type:feature` (+ priority as appropriate)
- User asks question → `type:question`

### Triage

Periodically review new issues:

```
Human: "Triage the unlabeled issues from this week"
Claude Code: Reviews each, suggests labels and priority
You: Approve or adjust
```

## GitHub Labels Reference

### Type Labels (What kind of work)

**`type:bug`**
- Something is broken or incorrect
- Applies to code, docs, tests, anything
- Examples: "Crash on empty input", "Wrong command in setup docs"

**`type:feature`**
- New functionality to add
- Examples: "Add CSV export", "Add API rate limiting"

**`type:enhancement`**
- Improve existing functionality
- Examples: "Faster query performance", "Clearer error messages"

**`type:documentation`** (optional)
- Specifically about documentation improvements
- Use if you want to distinguish docs work from other types
- Alternative: Just use `type:bug` or `type:feature` for doc issues

**`type:question`**
- User needs help or clarification
- Often reveals documentation gaps
- Can be closed after answering

### Priority Labels (When to work on it)

**`priority:high`**
- Critical bugs blocking users
- Core features needed for MVP/release
- Use sparingly to maintain signal

**`priority:low`**
- Nice-to-haves
- Polish and optimizations
- Can be deferred

**(no label)**
- Implicit medium priority
- Normal importance work

### Optional Community Labels

**`good first issue`**
- Easy for new contributors
- Well-defined, limited scope
- Good for building community

**`help wanted`**
- Actively seeking external contributors
- Signals maintainer openness to PRs

## Best Practices

### Do

- Use TASKS.md as detailed temporary planning file, then delete after creating issues
- Put implementation details and plans in GitHub issues
- Use natural language with Claude Code
- Reference issues in commits for git history
- Review critical code paths manually
- Update docs/ files to reflect actual implementation
- Use semantic versioning for releases

### Don't

- Commit TASKS.md to the repository (it's temporary)
- Over-label or create complex status workflows
- Use GitHub comments for development iteration
- Create issues for every tiny task
- Let documentation drift from code

### Natural Language Over Process

The power of Claude Code is natural language interaction. Use direct, clear instructions:

```
"Work on issue #42"
"Here's what we need. Check the architecture docs and implement it"
"Update ARCHITECTURE.md with this approach"
```

## Example: Full Project Lifecycle

### Planning

```
You: "I want to build a task management API. Create the project plan."
Claude Code: Creates SPEC.md, ARCHITECTURE.md
You: Review and refine docs
Claude Code: Creates TASKS.md with implementation tasks
You: Review and discuss TASKS.md, make adjustments
Claude Code: Creates GitHub issues from TASKS.md
Claude Code: Deletes TASKS.md
```

### Initial Development

```
You: "Work on issue #1 - database setup"
Claude Code: Implements, commits "feat: setup database schema\n\nImplements #1"
You: Review, iterate via conversation
You: "Good. Next one."
Claude Code: Moves to issue #2
[Repeat for core features]
```

### First Release

```
You: "We're ready for v1.0.0. Create the release."
Claude Code:
  - Reviews all closed issues since last release
  - Generates changelog
  - Creates release tag
  - Updates README with installation instructions
```

### Maintenance

```
User creates issue: "API returns 500 on invalid JSON"
Human: Add label type:bug, priority:high
Human: "Claude, fix issue #47"
Claude Code: Fixes, tests, commits, closes issue

User creates issue: "Add pagination to list endpoint"
Human: Add label type:feature
Human: "Implement issue #48"
Claude Code: Implements according to spec
Human: Review and request changes via conversation
Claude Code: Adjusts
Human: "Ship it"
Claude Code: Merges, closes issue
```
