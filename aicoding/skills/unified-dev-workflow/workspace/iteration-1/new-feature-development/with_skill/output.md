# Unified Dev Workflow Test Output

## Scenario Detection

**Detected Scenario:** 新功能开发 (New Feature Development)

**Reasoning:**
- User keywords: "实现" (implement), "功能" (feature)
- Task is to build new user login functionality
- Will affect multiple files (controllers, services, data layer, configuration)
- Requires team review and historical traceability

**Selected Flow:** Flow A - 新功能开发

---

## Flow A Execution Plan

### Phase 1: 需求探索 (OpenSpec)

**Next Step:** Execute `/opsx:propose user-login`

**Expected Artifacts:**
- `proposal.md` - Goals, scope, exclusions
- `specs/` - Given/When/Then scenarios for:
  - Phone number login flow
  - Email login flow
  - Error handling (invalid credentials, rate limiting)
  - Session management
- `design.md` - Architecture diagram, data flow, API contracts
- `tasks.md` - Granular tasks (1-2 hours each)

**Key Questions to Clarify:**
1. Authentication method: JWT? Session-based? OAuth?
2. Password requirements and validation rules
3. Rate limiting and brute-force protection
4. Multi-device login support?
5. Integration with existing user system?

---

### Phase 2: 实现规划 (桥接)

**Next Step:** Create Superpowers plan based on tasks.md

**Expected Output:**
- `docs/superpowers/plans/2026-04-15-user-login.md`

**Plan Structure:**
- Map each task to specific file paths
- Define test verification commands
- Identify dependencies between tasks

---

### Phase 3: TDD 实现 (Superpowers)

**Next Step:** Execute RED-GREEN-REFACTOR cycle for each task

**Key Tasks (anticipated):**
1. **Controller Layer**
   - `controllers/http/auth.go` - Login handler
   - Request/Response types in `integrity/`

2. **Service Layer**
   - `service/process/auth.go` - Authentication logic
   - Token generation/validation

3. **Data Layer**
   - `data/user.go` - User lookup by phone/email
   - Password verification
   - Session/token storage

4. **Configuration**
   - `conf/mount/api.yaml` - Auth service endpoints
   - `conf/mount/custom.yaml` - JWT secret, token expiry

**TDD Requirements:**
- Write failing test FIRST
- Implement minimum code to pass
- Refactor if needed
- Target 80%+ coverage
- Code review after each task

---

### Phase 4: 归档 (OpenSpec)

**Next Step:** Execute `/opsx:archive` after completion

**Expected Outputs:**
- Updated `specs/` with final behavior
- Archive directory with change history
- Commit message with full context

---

## Checklist for Phase 1

Before proceeding, ensure:
- [ ] `proposal.md` contains clear goals, scope, and exclusions
- [ ] `specs/` contains Given/When/Then scenarios for both login methods
- [ ] `design.md` contains architecture diagram and data flow
- [ ] `tasks.md` has granular tasks (each ~1-2 hours)

---

## Anti-Pattern Warnings to Avoid

| Anti-Pattern | Correct Approach |
|-------------|------------------|
| Starting implementation without `/opsx:propose` | Create OpenSpec proposal first |
| Skipping TDD | Use Superpowers TDD for all business logic |
| Not referencing tasks.md during implementation | Use tasks.md as source of truth |
| Forgetting to archive | Execute `/opsx:archive` before completion |

---

## Summary

**Flow Selected:** Flow A (新功能开发)

**Immediate Next Step:** Execute `/opsx:propose user-login` to begin Phase 1 (需求探索)

**Why Flow A:**
1. This is a new feature with clear "implement" keyword
2. Will affect multiple files and layers
3. Requires design decisions and team alignment
4. Benefits from historical traceability via OpenSpec
5. Requires TDD discipline via Superpowers for authentication security

**Estimated Complexity:** Medium-High
- 2 authentication methods (phone + email)
- Security considerations (passwords, tokens, rate limiting)
- Integration with existing user data model
