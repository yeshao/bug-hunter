# bug-hunter Improvement Plan

## Approved Fixes (from devil's advocate round)

### 1. Iteration loop between Phase 5 and Phase 3
**Problem:** If execution rejects a hypothesis, the workflow says nothing. Model stalls or hallucinates.
**Fix:** Add explicit reject-and-backtrack path in Phase 5.

### 2. Clean-audit exit (no findings path)
**Problem:** Every phase assumes a bug will be found. Model manufactures findings.
**Fix:** If no candidate survives evidence, produce a "clean audit" report.

### 3. Remove `gpt-5.4` hardcoding
**Problem:** Skill ages poorly, tied to specific model.
**Fix:** Replace `gpt-5.4` with "the underlying model" throughout.

### 4. Phase 6 rewrite (honest skip condition)
**Problem:** "Audit surface is broad enough" is undefined. Parallel lanes list topics, not slices.
**Fix:** Define when to skip Phase 6 (small codebases) and when to use it (≥3 disjoint trust boundaries).

### 5. Per-candidate evidence cap (principle, not number)
**Problem:** Model can spend unbounded time on a single hypothesis.
**Fix:** "If no executable artifact exists after one Phase 4 cycle, classify as weak and move on."

## Rejected Fixes (with reasoning)

| # | Proposal | Reason to skip |
|---|----------|----------------|
| 1 | Phase 1 template | Existing 4 bullets are sufficient for a scoping pass |
| 4 | Risk scale 0 | Schema already says "only rank files in audit surface" |
| 5 | 4th validator verdict | `weak` already covers it; 4th state adds branching without action |
| 7 | Split checklist | Adds length, not clarity |
| 10 | Seed prompt | SKILL.md is the seed |

## Lower-priority (not planned)

- Phase table formatting — cosmetic
- AP-3 / AP-5 overlap — different directions of same failure, leave both
- Dead source link — sufficient provenance
