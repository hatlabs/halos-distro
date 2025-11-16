# Human Development Guidance - Agentic Coding with Claude Code

**Last modified:** 2025-11-16

This guide shows you how to work effectively with Claude Code, from initial project planning through implementation.

## Starting a New Project or Major Feature

For new projects or major improvements, follow the project planning workflow:

### The Planning Process

1. **Human provides initial input** - Free-format description: paragraphs, bullet points, keywords, even drawings
2. **Claude creates SPEC.md** - Technical specification from your input
3. **Human reviews & iterates** - This is crucial! Refine until the spec is right
4. **Claude creates ARCHITECTURE.md** - System design based on approved spec
5. **Human reviews & iterates** - Architecture decisions have long-term impact
6. **Claude creates TASKS.md** - Detailed implementation breakdown
7. **Human reviews tasks** - Ensure completeness and proper ordering
8. **Claude creates GitHub issues** - Converts tasks to issues with workflow reminders
9. **Start implementation** - Work through issues systematically

**Key Point:** The higher-level the document, the more important iteration is. Spend time getting SPEC and ARCHITECTURE right - they guide everything else.

### Example: Planning Cockpit APT
```
You: I need a web-based package manager for Cockpit. Like Raspberry Pi's
     Add/Remove Software but for the web. Should handle APT packages,
     show categories, search, install/remove with progress.
     Target users are non-technical. Must work on Debian/Ubuntu.

Claude: [Creates initial SPEC.md]

You: Add requirements for safety checks - prevent removing essential packages.
     Also needs to handle locked package manager gracefully.

Claude: [Updates SPEC.md]

You: Good spec. Now create ARCHITECTURE.md based on this.

Claude: [Creates architecture with Python backend, React frontend]

You: The backend should use python-apt directly, not shell commands.
     Add a TypeScript API wrapper layer for type safety.

Claude: [Updates ARCHITECTURE.md]

You: Perfect. Break this into implementation tasks.

Claude: [Creates TASKS.md with ~15 tasks]

You: Tasks look good. Create GitHub issues with DEVELOPMENT_WORKFLOW.md reminders.

Claude: [Creates issues #1-#15]
```

See **@PROJECT_PLANNING_GUIDE.md** for the complete planning methodology.

## The Implementation Workflow & Why It Matters

### What Is The Workflow?
The DEVELOPMENT_WORKFLOW.md defines a structured process:
1. **Explore** - Understand existing code WITHOUT writing new code
2. **Plan** - Think through the approach using "think hard"
3. **Test** - Write tests FIRST (TDD)
4. **Implement** - Code until tests pass
5. **Verify** - Check edge cases and correctness

### Why This Workflow?
- **Explore first** → Prevents breaking existing functionality
- **Plan with thinking** → Better architecture, fewer rewrites
- **Test first** → Clear target, prevents drift from requirements
- **Verify** → Catches edge cases humans might miss

### The Problem: Claude Forgets
While Claude can read these workflows, **it often skips them unless reminded**. Without reminders, Claude tends to jump straight to coding, leading to bugs and rework.

## Day-to-Day Development Workflow

### Step 1: Let Claude Create the Issue
Claude is excellent at writing the boilerplate. You just provide direction:

```
You: We need to add rate limiting to our API endpoints.
     Create a GitHub issue for this, including the DEVELOPMENT_WORKFLOW.md reminder.
```

Claude creates a complete issue with:
- Detailed description
- Implementation workflow steps
- Test scenarios
- Acceptance criteria

### Step 2: Review & Start Work
```
You: Good issue. Now work on it, following DEVELOPMENT_WORKFLOW.md - explore first!
```

### Step 3: Provide Feedback
```
You: Use Redis for the rate limiting, not in-memory storage
You: Add a bypass for admin users
You: The rate limit should be configurable
```

Claude adjusts immediately without you writing any code.

## Key Examples

### Starting a New Feature
**Right Way:**
```
You: We need pagination for all API endpoints. Create an issue with DEVELOPMENT_WORKFLOW.md reminder.
Claude: [Creates detailed issue with workflow]
You: Work on it, follow the workflow - explore first!
```

**Wrong Way:**
```
You: Add pagination to the API
Claude: [Jumps to coding, misses edge cases]
```

### Bug Fixes
**Right Way:**
```
You: Users report duplicate search results. Create bug issue with workflow reminder,
     then fix using TDD.
```

**Wrong Way:**
```
You: Fix the duplicate search results
```

The difference? The right way ensures Claude writes a test that reproduces the bug first.

### When You Write Code
```
You: I've written the payment module. Add comprehensive tests and documentation.
You: I updated the database schema in schema.sql. Update the models to match.
```

This is perfectly fine! Claude excels at the "boring" work like tests and docs.

## Best Practices

### ✅ DO
- **Let Claude create issues** - It's faster and includes all boilerplate
- **Include workflow reminders** - "Follow DEVELOPMENT_WORKFLOW.md"
- **Write code yourself when you want** - Just tell Claude what you did
- **Batch similar tasks** - "Add validation to all endpoints"
- **Reference examples** - "Like the pattern in user-controller.js"

### ❌ DON'T
- **Assume Claude remembers** - Always remind about the workflow
- **Skip issue creation** - Issues with workflow reminders prevent problems
- **Write boilerplate** - Let Claude handle the tedious parts

## What The Workflow Provides

**With workflow reminders**, Claude:
- ✅ Explores before coding (prevents breaking changes)
- ✅ Writes tests first (clear target, no drift)
- ✅ Plans with deep thinking (better architecture)
- ✅ Verifies edge cases (fewer bugs)

**Without reminders**, Claude often:
- ❌ Jumps straight to coding
- ❌ Skips tests
- ❌ Misses edge cases
- ❌ Creates quick but brittle solutions

## Advanced Patterns

### Specification-Driven Development
```
You: Create SPEC.md for the new payments module
You: Create ARCHITECTURE.md for the payments module
You: Create implementation issues from the spec
You: Start with issue #1
```

### Test-Driven Bug Fixes
```
You: Users report the search returns duplicates sometimes
```
Claude Code writes a test that reproduces it, then fixes it.

### Continuous Improvement
```
You: Review the codebase for security issues
You: Optimize database queries for performance
You: Update all dependencies and fix any breaking changes
```

## Debugging Assistance

```
You: The deployment is failing on staging
You: Customer reports timeout errors after 30 seconds
You: The tests pass locally but fail in CI
```

Claude Code investigates, identifies root causes, and fixes issues.

## Quick Reference

### The Standard Flow
1. **Describe need**: "We need [feature/fix]"
2. **Claude creates issue**: "Create issue for this with DEVELOPMENT_WORKFLOW.md reminder"
3. **Start work**: "Work on it, follow the workflow"
4. **Provide feedback**: "Use Redis instead", "Add logging", etc.
5. **Ship**: "Good, create PR"

### The Magic Phrase
When starting any task: **"Remember to follow DEVELOPMENT_WORKFLOW.md"**

This simple reminder dramatically improves quality.

### Common Commands
- **New feature**: "Create issue for [feature] with workflow reminder"
- **Bug fix**: "Users report [bug]. Create issue and fix using TDD"
- **Your code**: "I wrote [module], add tests and docs"
- **Exploration**: "How does [system] work?"
- **Refactoring**: "Refactor [module] following the workflow"

## Summary

**Let Claude handle the boilerplate** - Issue creation, test writing, documentation.

**You provide direction** - What to build, architectural decisions, feedback.

**Always remind about the workflow** - It's the difference between quick hacks and quality code.

---

@DEVELOPMENT_WORKFLOW.md - Claude Code's internal workflow
@IMPLEMENTATION_CHECKLIST.md - Quick reference for implementation
