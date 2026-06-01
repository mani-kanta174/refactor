---
name: refactor-plan
description: "Create a concrete plan before starting a multi-file or cross-cutting refactor. Use when the user asks to plan, sequence, scope, or safely execute a refactor that spans multiple files; always investigate first, output the plan, and wait for confirmation before making code changes. For a single localized change, the `refactor` skill alone is enough — this skill is for when sequencing across files matters."
---

# Refactor Plan

Create a detailed plan before making any code changes.

## Instructions

1. Do not edit files while preparing the plan.
2. A refactor plan covers structure changes only — it must not include behavior, feature, or bug-fix changes. If the goal requires any of those, flag them as separate work in the Risks section rather than folding them into refactor phases.
3. Search the codebase to understand the current state. Read enough implementation, tests, configuration, and docs to make the plan specific to the repository.
4. Identify affected files, ownership boundaries, dependencies, and likely hidden coupling. Also note whether a test safety net exists for the affected code; if it does not, the plan's first phase should be adding characterization tests before any behavior-adjacent change.
5. Note the language and framework versions detected from the project (manifests/lockfiles or the code itself). The plan must not assume features newer than those versions; if a version can't be determined and a phase depends on it, list that as an open question rather than guessing.
6. Plan changes in a safe sequence so each phase leaves the code working: prefer contracts/types first (where the language has them), then implementations, then callers, then tests, then cleanup. For languages without a separate type layer, start from the implementation — do not force a types-first phase that doesn't apply.
7. Include verification steps between phases and a final validation command. Each phase's verification must confirm behavior is unchanged (tests still pass / characterization diff clean), not just that the code compiles. Use the repository's actual file extensions and its own test/build commands — detect these from the project during the investigation step; the Output Format below uses neutral placeholders (`<ext>`, generic phase names) that you must replace with the real language's specifics.
8. Include rollback or recovery steps for the riskiest phases.
9. Output the complete plan using the format below.
10. Stop after the plan and ask for confirmation before implementing. If the user already asked you to implement, still produce the plan first and wait for confirmation unless they explicitly said to continue without review after the plan.
11. Only after the user approves the plan, begin execution — never start changing code without that explicit go-ahead. Execute one phase at a time: for each individual change, apply the `refactor` skill (if available) so the change follows its safe process — small steps, tests re-run after each, behavior preserved. After completing each phase, report against this plan's verification step and pause for the user's confirmation before starting the next phase; do not run all phases straight through.

If the request is too ambiguous to plan safely, ask concise clarifying questions instead of editing files.

## Output Format

```markdown
## Refactor Plan: [title]

### Current State

[Brief description of how things work now]

### Context

- **Versions:** [detected language + framework versions, or "unknown — see open questions"]
- **Safety net:** [existing tests cover the affected code / characterization tests needed first / none — flag as risk]

### Target State

[Brief description of how things will work after]

### Affected Files

| File | Change Type          | Dependencies           |
| ---- | -------------------- | ---------------------- |
| path | modify/create/delete | blocks X, blocked by Y |

### Execution Plan

#### Phase 1: Contracts / Interfaces / Types

(Skip or rename this phase if the language has no separate contract/type layer — e.g. for dynamically typed code, start from the implementation.)

- [ ] Step 1.1: [action] in `path/to/file.<ext>`
- [ ] Verify: [how to check it worked — confirm behavior unchanged]

#### Phase 2: Implementation

- [ ] Step 2.1: [action] in `path/to/file.<ext>`
- [ ] Verify: [how to check — tests still pass / characterization diff clean]

#### Phase 3: Tests

- [ ] Step 3.1: Update tests in `path/to/test_file.<ext>`
- [ ] Verify: run the project's test suite (use the repo's own command — e.g. `pytest`, `npm test`, `go test ./...`, `mvn test`, `cargo test`)

#### Phase 4: Cleanup

- [ ] Remove deprecated code
- [ ] Update documentation

### Rollback Plan

If something fails:

1. [Step to undo]
2. [Step to undo]

### Risks

- [Potential issue and mitigation]
```

After the plan, ask: "Shall I proceed with Phase 1?"
