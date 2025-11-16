---
name: Feature Implementation
about: Template for new feature implementation tasks
title: ''
labels: 'type:feature'
assignees: ''
---

## Description

[Clear description of the feature to be implemented]

## Requirements

[List specific requirements from SPEC.md that this feature addresses]

- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

## Implementation Workflow

**IMPORTANT**: Follow the [DEVELOPMENT_WORKFLOW.md](/docs/DEVELOPMENT_WORKFLOW.md) guide. Do NOT skip steps.

### Phase 1: Explore & Plan ⚠️ NO CODING YET!
1. **Explore**: Read all relevant files WITHOUT writing code. Use `Task tool with subagent_type=Explore` for complex exploration
2. **Plan**: Use `think hard` to create detailed approach
3. **Document**: Comment your plan before proceeding

### Phase 2: Test-Driven Development
1. **Write Tests First**: Create comprehensive tests based on requirements
2. **Verify Tests Fail**: Run tests to confirm they fail
3. **Commit Tests**: Commit tests before implementation
4. **Implement**: Write code to pass tests (don't modify tests)
5. **Iterate**: Run tests, fix issues, repeat until all pass
6. **Verify**: Use subagents to verify implementation
7. **Commit Implementation**: Commit once tests pass

## Test Scenarios

[List specific test cases that must be implemented]

- [ ] Test case 1: [description]
- [ ] Test case 2: [description]
- [ ] Test case 3: [description]
- [ ] Edge case: [description]

## Acceptance Criteria

- [ ] All tests pass
- [ ] Implementation matches `/docs/SPEC.md` requirements
- [ ] Code follows patterns in `/docs/ARCHITECTURE.md`
- [ ] No security vulnerabilities
- [ ] Documentation updated if needed
- [ ] Meaningful commit messages with issue reference

## References

- **Specification**: See `/docs/SPEC.md` section [X]
- **Architecture**: See `/docs/ARCHITECTURE.md` section [Y]
- **Related Issues**: #[number]

## Implementation Checklist

Before starting, confirm you will:
- [ ] Read this issue completely
- [ ] Explore without coding first
- [ ] Write tests before implementation
- [ ] Follow TDD workflow
- [ ] Reference SPEC and ARCHITECTURE documents
- [ ] Use subagents for verification

---
⚠️ **Reminder**: Quality > Speed. Follow the workflow for successful implementation.