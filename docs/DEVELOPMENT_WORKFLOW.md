# Development Workflow Guide for Claude Code

**Last modified:** 2025-11-16

This document defines the implementation workflows for Claude Code when working on HaLOS projects. Following these workflows ensures high-quality, well-tested implementations that match specifications.

**IMPORTANT**: To help Claude Code adhere to these workflows, always include a reference to this document in GitHub issues. Each issue should also contain an explicit implementation checklist copied from `/docs/IMPLEMENTATION_CHECKLIST.md`.

## Core Principle: Think Before You Code

**STOP** - Do not jump straight into coding. Follow the structured workflows below for every implementation task.

## Workflow 1: Explore-Plan-Code-Commit

This workflow applies to ALL feature implementations and bug fixes.

### Phase 1: Explore (No Coding!)

**What to do:**
1. Read the GitHub issue completely
2. Read relevant existing code WITHOUT writing any code
3. Use subagents for complex exploration:
   - `Task tool with subagent_type=Explore` for codebase understanding
   - Investigate specific questions about the architecture
4. Read `/docs/SPEC.md` for requirements context
5. Read `/docs/ARCHITECTURE.md` for design decisions

**Example:**
```
Human: "Work on issue #42 - Add user authentication"
Claude Code:
  1. Reads issue #42
  2. Uses Explore agent: "Find all authentication-related code"
  3. Reads SPEC.md authentication requirements
  4. Reads ARCHITECTURE.md for auth design decisions
  5. Explores existing user models and session handling
```

**Key Point:** NO CODE IS WRITTEN during exploration phase.

### Phase 2: Plan

**What to do:**
1. Use thinking commands to evaluate approaches:
   - `think` - Basic planning
   - `think hard` - Complex features
   - `think harder` - Architecture decisions
   - `ultrathink` - Critical system changes
2. Create a detailed implementation plan
3. Consider creating a plan document or GitHub comment
4. Identify which tests need to be written

**Example Plan Structure:**
```markdown
## Implementation Plan for Issue #42

### Approach
- Use JWT tokens for stateless authentication
- Store refresh tokens in secure httpOnly cookies
- Implement middleware for route protection

### Test Strategy
- Unit tests for JWT generation/validation
- Integration tests for login/logout flow
- E2E tests for protected routes

### Implementation Steps
1. Write authentication tests (TDD)
2. Implement JWT token service
3. Create authentication middleware
4. Add login/logout endpoints
5. Update protected routes

### Verification Points
- All tests pass
- No security vulnerabilities (check OWASP)
- Matches SPEC.md requirements
- Follows ARCHITECTURE.md patterns
```

### Phase 3: Code

**What to do:**
1. Implement the solution following your plan
2. Verify reasonableness as you code
3. Reference SPEC.md and ARCHITECTURE.md during implementation
4. Use subagents to verify specific implementation details

**Key Points:**
- Follow the plan created in Phase 2
- Don't deviate without going back to planning
- Regularly verify against requirements

### Phase 4: Commit

**What to do:**
1. Run all tests to ensure they pass
2. Create meaningful commit message referencing the issue
3. Update README or CHANGELOG if needed
4. Create PR with summary of changes

## Workflow 2: Test-Driven Development (TDD)

**MANDATORY** for all testable features. This workflow dramatically improves implementation quality.

### Step 1: Write Tests First

**What to do:**
1. Based on requirements, write comprehensive tests
2. Include unit tests, integration tests, and E2E tests as appropriate
3. Tests should cover expected inputs/outputs
4. DO NOT create mock implementations

**Example:**
```python
# Write this FIRST, before any implementation
def test_user_authentication():
    # Test successful login
    response = login("user@example.com", "password123")
    assert response.status == 200
    assert "token" in response.json

    # Test invalid credentials
    response = login("user@example.com", "wrong")
    assert response.status == 401

    # Test token validation
    token = get_token("user@example.com", "password123")
    assert validate_token(token)
```

### Step 2: Verify Tests Fail

**What to do:**
1. Run the tests
2. Confirm they fail (no implementation exists yet)
3. DO NOT write any implementation code yet

**Command:**
```bash
./run test  # Should show failures
```

### Step 3: Commit Tests

**What to do:**
1. Commit the failing tests
2. This creates a clear target for implementation

**Commit message:**
```
test: add authentication tests for issue #42

Tests for JWT authentication, login/logout flow,
and protected route access.

Related to #42
```

### Step 4: Implement Until Tests Pass

**What to do:**
1. Write implementation code
2. Run tests
3. Fix failures
4. Repeat until all tests pass
5. DO NOT modify the tests (unless they have bugs)

**Iteration Example:**
```
Iteration 1: Implement basic JWT generation
  - Run tests: 3/10 pass

Iteration 2: Add token validation
  - Run tests: 6/10 pass

Iteration 3: Implement login endpoint
  - Run tests: 9/10 pass

Iteration 4: Fix edge case in token expiry
  - Run tests: 10/10 pass ✓
```

### Step 5: Verify Implementation

**What to do:**
1. Use subagents to verify implementation isn't overfitting to tests
2. Check for edge cases not covered by tests
3. Verify against SPEC.md requirements
4. Ensure follows ARCHITECTURE.md patterns

### Step 6: Commit Implementation

**What to do:**
1. Commit the working implementation
2. Reference the issue in commit message

**Commit message:**
```
feat: implement JWT authentication

- JWT token generation and validation
- Login/logout endpoints
- Protected route middleware
- Secure token storage

Fixes #42
```

## When to Use Each Workflow

### Use Explore-Plan-Code-Commit for:
- All new features
- Bug fixes
- Refactoring
- Any code changes

### Additionally use TDD for:
- Features with clear input/output requirements
- API endpoints
- Data processing functions
- Business logic
- Bug fixes (write test that reproduces bug first)

## Subagent Usage Guidelines

### When to Use Subagents

**Exploration Phase:**
- Complex codebase navigation: `Task tool with subagent_type=Explore`
- Finding all related code: "Find all authentication-related files"
- Understanding patterns: "How does error handling work in this codebase?"

**Verification Phase:**
- Checking implementation correctness
- Verifying security considerations
- Ensuring no regression

### Example Subagent Usage

```
# During exploration
Task tool with subagent_type=Explore
Prompt: "Find all files related to user authentication and session management.
Look for JWT implementations, middleware, and protected routes."

# During verification
Task tool with subagent_type=general-purpose
Prompt: "Verify that the JWT implementation in auth.js follows security best practices
and doesn't have common vulnerabilities like weak secret keys or missing expiration."
```

## Thinking Commands

Use these to improve planning quality:

- **`think`** - Quick planning for simple features
- **`think hard`** - Detailed planning for complex features
- **`think harder`** - Architecture decisions, security considerations
- **`ultrathink`** - Critical system changes, breaking changes

## Common Pitfalls to Avoid

### ❌ DON'T
- Jump straight to coding without exploration
- Skip the planning phase
- Write implementation before tests
- Modify tests to make them pass (fix the code instead)
- Ignore SPEC.md and ARCHITECTURE.md during implementation
- Skip verification steps

### ✅ DO
- Always explore the codebase first
- Plan thoroughly before coding
- Write tests before implementation
- Iterate on code until tests pass
- Verify implementation with subagents
- Reference project documentation throughout

## Implementation Checklist

Use this checklist for every implementation:

```markdown
## Pre-Implementation
- [ ] Read GitHub issue completely
- [ ] Explore relevant code (no coding!)
- [ ] Use subagents for complex exploration
- [ ] Read SPEC.md for requirements
- [ ] Read ARCHITECTURE.md for design
- [ ] Create implementation plan
- [ ] Use "think hard" for planning

## Test-Driven Development
- [ ] Write comprehensive tests first
- [ ] Verify tests fail (no mocks)
- [ ] Commit tests
- [ ] Implement until tests pass
- [ ] Don't modify tests

## Implementation
- [ ] Follow the plan
- [ ] Reference SPEC/ARCHITECTURE
- [ ] Verify as you code
- [ ] All tests pass

## Post-Implementation
- [ ] Use subagents to verify
- [ ] Check for edge cases
- [ ] Update documentation if needed
- [ ] Meaningful commit message
- [ ] Reference issue in commit
```

## Example: Full Feature Implementation

```
Human: "Implement issue #99 - Add rate limiting to API"

Claude Code Process:

1. EXPLORE (10 minutes)
   - Read issue #99 requirements
   - Use Explore agent: "Find all API endpoint definitions"
   - Read SPEC.md section on performance requirements
   - Read ARCHITECTURE.md for middleware patterns
   - Examine existing middleware implementations

2. PLAN (5 minutes)
   - think hard
   - Plan: Redis-based rate limiting with sliding window
   - Identify test scenarios
   - Document approach

3. WRITE TESTS (15 minutes)
   - Test rate limit enforcement
   - Test limit reset
   - Test exempted endpoints
   - Run tests - confirm they fail
   - Commit tests

4. IMPLEMENT (30 minutes)
   - Create rate limit middleware
   - Run tests - 2/8 pass
   - Add Redis integration
   - Run tests - 5/8 pass
   - Fix window calculation
   - Run tests - 8/8 pass

5. VERIFY (5 minutes)
   - Use subagent to check for bypass vulnerabilities
   - Test with high load
   - Verify Redis connection handling

6. COMMIT
   - "feat: add rate limiting to API endpoints

    Implements sliding window rate limiting using Redis.
    Configurable per-endpoint limits.

    Fixes #99"
```

## Workflow for Different Scenarios

### Bug Fix Workflow
1. Explore to understand the bug
2. Write test that reproduces the bug
3. Verify test fails
4. Fix the bug
5. Verify test passes
6. Check for similar bugs elsewhere
7. Commit fix with test

### Refactoring Workflow
1. Explore current implementation thoroughly
2. Write tests for current behavior (if missing)
3. Plan refactoring approach
4. Refactor in small steps
5. Run tests after each step
6. Verify behavior unchanged
7. Commit refactored code

### Documentation Update Workflow
1. Explore related code/features
2. Plan documentation structure
3. Write/update documentation
4. Verify accuracy against code
5. Commit documentation

### Using the Technical Writer Agent

For documentation tasks, invoke the `technical-writer` agent:
- Creating or updating AGENTS.md files
- Cleaning up comments after implementation
- Reviewing documentation for LLM-optimization
- Ensuring present-tense, token-efficient docs

The agent enforces:
- **No temporal contamination**: "Added", "Changed", "Previously" → present tense only
- **No marketing language**: "robust", "elegant", "seamless" → technical descriptions
- **WHY-focused comments**: Comments explain reasoning, not what code does
- **Tabular navigation**: AGENTS.md files use tables for quick lookup

Integration points in the workflow:

| Workflow Phase | Technical Writer Usage |
|----------------|------------------------|
| Post-implementation | Update AGENTS.md with new file/module entries |
| Before PR | Scrub plan comments that leaked into code |
| New repository setup | Create initial AGENTS.md structure |
| Refactoring | Update documentation to reflect new structure |

## Success Metrics

A successful implementation:
- ✅ All tests pass
- ✅ Matches SPEC.md requirements
- ✅ Follows ARCHITECTURE.md patterns
- ✅ No security vulnerabilities
- ✅ Code is readable and maintainable
- ✅ Documentation is updated
- ✅ Commit messages are meaningful

## Remember

**Quality > Speed**

Taking time to explore, plan, and write tests first leads to:
- Fewer bugs
- Better architecture
- Easier maintenance
- Happy users

Following these workflows is not optional - it's essential for maintaining code quality in AI-assisted development.
