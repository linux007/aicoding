# Unified Dev Workflow - Refactor Scenario Response

## Task
重构用户模块，把 UserService 拆分成多个小服务，但保持对外接口不变

---

## Scene Detection

**Detected Scene Type:** 代码重构 (Code Refactoring)

**User Keywords:** "重构" (refactor), "拆分" (split), "保持对外接口不变" (keep external interface unchanged)

**Selected Flow:** **Flow D: 代码重构/迁移**

**Rationale:**
- User explicitly mentioned "重构" (refactoring)
- Goal is to improve internal structure (splitting UserService into smaller services)
- External behavior must remain unchanged ("保持对外接口不变")
- This matches Flow D criteria: "用户说'重构'、'优化'、'清理' 且 不改变行为"

---

## Flow D: Phase-by-Phase Execution Plan

### Phase 1: 现状分析 (OpenSpec)

**Actions:**
1. Run `/opsx:explore` to analyze:
   - Current UserService dependencies
   - Impact scope (which files/modules depend on UserService)
   - Risk assessment (what could break)
   
2. Run `/opsx:propose refactor-user-service`
   - Create `proposal.md` with refactoring goals
   - Create `specs/` defining "behavior unchanged" verification standards
   - Create `tasks.md` with step-by-step refactoring plan

**Key Questions to Answer:**
- What methods does UserService currently expose?
- What are the logical boundaries for splitting? (e.g., UserAuth, UserProfile, UserPreferences)
- What tests currently exist for UserService?

**Checklist:**
- [ ] proposal.md contains current state analysis and target state
- [ ] specs/ contains Given/When/Then scenarios verifying behavior unchanged
- [ ] tasks.md has granular steps (each 1-2 hours)

---

### Phase 2: 安全网构建 (Superpowers)

**Actions:**
1. Ensure all existing tests pass
2. Identify missing test coverage using `superpowers:test-driven-development`
3. Add "behavior anchor tests" that will verify interface contract is maintained
4. Record current behavior snapshot (method signatures, return types, error handling)

**Principle:** 没有测试覆盖的代码不允许重构
**Principle:** 每次提交后系统可正常运行

**Checklist:**
- [ ] All existing tests pass
- [ ] Test coverage for UserService >= 80%
- [ ] Behavior anchor tests in place for public interface
- [ ] Current behavior documented

---

### Phase 3: 分步重构 (Incremental Refactoring)

**Execution Pattern:**
```
For each split target:
  1. Create new service file (e.g., UserAuthService)
  2. Move related methods from UserService to new service
  3. Add delegation in UserService (maintain interface)
  4. Run tests -> All pass?
  5. Commit with message: "refactor: extract UserAuthService from UserService"
```

**Recommended Split Strategy:**
1. Identify cohesive method groups in UserService
2. Create new service files:
   - `UserAuthService` - authentication, login, logout, token management
   - `UserProfileService` - profile CRUD, avatar, settings
   - `UserPreferencesService` - user preferences, notifications config
3. UserService becomes a facade that delegates to smaller services
4. External callers see no change in UserService interface

**Checklist:**
- [ ] Each refactoring step has passing tests
- [ ] Each commit leaves system in runnable state
- [ ] No CRITICAL/HIGH code review issues

---

### Phase 4: 验证与归档 (Verification & Archive)

**Actions:**
1. Run full test suite
2. Verify test coverage has not decreased
3. Check performance has not regressed
4. Run `/opsx:archive`
5. Update any affected documentation

**Checklist:**
- [ ] All tests pass
- [ ] Coverage >= pre-refactor level
- [ ] No performance regression
- [ ] OpenSpec archive created
- [ ] Documentation updated

---

## Next Steps for User

1. **Confirm the refactoring approach:**
   - Do you agree with Flow D for this task?
   - What methods does UserService currently have?
   - What split boundaries make sense for your use case?

2. **Begin Phase 1:**
   - I will run `/opsx:explore` to analyze current UserService
   - Then `/opsx:propose refactor-user-service` to create the refactoring plan

3. **Questions to clarify:**
   - Where is UserService located in the codebase?
   - Are there existing tests for UserService?
   - What are the main functional areas you want to split?

---

## Tool Commands to Execute

| Phase | OpenSpec Command | Superpowers Skill |
|-------|-----------------|-------------------|
| Phase 1 | `/opsx:explore`, `/opsx:propose refactor-user-service` | - |
| Phase 2 | - | `superpowers:test-driven-development` |
| Phase 3 | - | `superpowers:requesting-code-review` (after each step) |
| Phase 4 | `/opsx:archive` | - |

---

## Summary

**Detected Scene:** 代码重构 (Code Refactoring)
**Selected Flow:** Flow D
**Key Principle:** Tests must pass before and after each refactoring step
**Outcome:** UserService split into smaller services while maintaining external interface compatibility
