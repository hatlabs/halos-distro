# Implementation Checklist - Quick Reference

**Last modified:** 2025-11-16

**⚠️ STOP! Before implementing ANY feature, complete this checklist in order:**

@DEVELOPMENT_WORKFLOW.md - Full workflow documentation

## □ Phase 1: EXPLORE (No Coding!)
- [ ] Read GitHub issue completely
- [ ] Read `/docs/SPEC.md` relevant sections
- [ ] Read `/docs/ARCHITECTURE.md` relevant sections
- [ ] Explore existing code WITHOUT writing anything
- [ ] Use `Task tool with subagent_type=Explore` for complex navigation

## □ Phase 2: PLAN
- [ ] Use `think hard` to evaluate approaches
- [ ] Create detailed implementation plan
- [ ] Identify test scenarios
- [ ] Document plan in comment/file

## □ Phase 3: TEST (TDD)
- [ ] Write comprehensive tests FIRST
- [ ] Verify tests FAIL (no mocks)
- [ ] Commit tests: `git commit -m "test: add X tests"`

## □ Phase 4: IMPLEMENT
- [ ] Write code to pass tests
- [ ] Run tests after each change
- [ ] DO NOT modify tests to pass
- [ ] Iterate until ALL tests pass

## □ Phase 5: VERIFY
- [ ] Use subagents to verify correctness
- [ ] Check for edge cases
- [ ] Verify matches SPEC.md
- [ ] Ensure follows ARCHITECTURE.md

## □ Phase 5.5: DOCUMENT
- [ ] Run `technical-writer` agent to update AGENTS.md if significant changes
- [ ] Scrub any inline comments for temporal contamination
- [ ] Verify no marketing/hedging language in docstrings

## □ Phase 6: COMMIT
- [ ] All tests pass
- [ ] Code linted and formatted
- [ ] Type checks pass
- [ ] Commit: `git commit -m "feat: implement X\n\nFixes #N"`
- [ ] Create PR if needed

---

## 🚨 Common Mistakes to Avoid

❌ **DON'T:**
- Jump to coding immediately
- Skip writing tests first
- Modify tests to make them pass
- Ignore SPEC/ARCHITECTURE docs
- Skip verification steps

✅ **DO:**
- Explore thoroughly before coding
- Plan with thinking time
- Write tests before implementation
- Follow TDD cycle religiously
- Verify with subagents

---

## 🎯 Success Criteria

A successful implementation has:
- ✅ All tests written first and passing
- ✅ Matches SPEC.md requirements
- ✅ Follows ARCHITECTURE.md patterns
- ✅ No security vulnerabilities
- ✅ Clean, maintainable code
- ✅ Updated documentation
- ✅ Meaningful commit messages

---

**Remember:** Quality > Speed. Following this checklist ensures successful implementations.
