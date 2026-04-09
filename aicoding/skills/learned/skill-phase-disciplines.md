# Skill Phase Discipline

**Extracted:** 2026-03-31
**Context:** When using skills with multi-phase workflows (like /migrate-feature, /tdd-guide), always complete each phase in prescribed order

## Problem
When invoking a skill that has multiple phases (requirement collection → exploration → planning → implementation → verification), I sometimes skip ahead to exploration/implementation before gathering complete requirements from the user. This leads to wasted exploration effort when requirements are unclear.

## Root Cause
Skills like `/migrate-feature` explicitly define Phase 1 as "需求收集" with input validation rules. I should never bypass this phase, even if I think I know the answer or the user seems impatient.

## Solution
**Before any skill execution:**
1. Identify if the skill has multi-phase structure
2. If yes, explicitly announce which phase I'm in
3. Complete that phase fully before proceeding
4. For `/migrate-feature`: Always validate user input format before exploring code

**Multi-phase skills to watch:**
- `/migrate-feature` - 5 phases, Phase 1 is requirement collection
- `/migrate-feature-v2` - similar structure
- `/tdd-guide` - has write-test-first discipline
- Any skill with "阶段" in its description

## Example

**WRONG approach (this session's error):**
```
User: /migrate-feature @service/webplugin.go 完成featureList
Me: *immediately starts grepping for GetTeacherFeature*
User: STOP - you should have asked for requirements first
```

**CORRECT approach:**
```
User: /migrate-feature @service/webplugin.go 完成featureList
Me: Phase 1: 需求收集 - 请提供源文件、迁移范围、参考文件
User: *provides info*
Me: *validates input format*, then proceeds to Phase 2
```

## When to Use
- Any time a user invokes `/migrate-feature` or similar multi-phase skill
- When a skill description mentions "阶段" or has numbered phases
- When the skill has explicit input validation rules in Phase 1
