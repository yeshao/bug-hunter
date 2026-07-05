# Bug-Hunter Deep Review

## Original Findings

### H1. Phase 6 has no template, merge step, or reconciliation

Phase 6 (Parallel Audit) specifies *when* to use parallel lanes (≥3 disjoint trust boundaries) but defines almost nothing about how. Missing: prompt template for lane execution, merge/reconciliation step, conflict resolution, lane count bounds. "Share no context" is unworkable since almost all codebases share imports, types, and runtime.

### H2. Phase 4 "one full Phase 4 cycle" is ill-defined

The cap says "attempt a fix → run → observe" which contradicts the evidence template ("Do not fix the code yet"). "A fix" is ambiguous. No guidance on what "attempt" counts toward the cap.

### H3. No pre-flight / build sanity check

The skill assumes the target is buildable and runnable. Real-world scenarios (project won't compile, pre-existing failing tests, no test framework) can't produce executable evidence but consume audit effort before discovery.

### H4. Final report format unspecified

Phase 8 defines required sections but no schema. No severity framework, no output format specification.

### M1. "Weak" verdict is a dead-end state

Weak appears throughout with no clear action path. Can't be fixed (not confirmed), can't be rejected (not rejected).

### M2. Audit surface has no size cap

Phase 1 identifies entrypoints with no upper bound. Risk of unbounded scope on large codebases.

### M3. Risk scale half-defined

SKILL.md defines 1-5 inline under Phase 2, but prompt-templates.md only defines 1 and 5.

### M4. No language-specific guidance

model-adaptation.md exists but SKILL.md doesn't incorporate it. Evidence types differ dramatically by language.

### M5. No overall time budget

Per-candidate cap exists but no global bound on total hypotheses tested.

### M6. No worked example

No concrete walkthrough on toy code to anchor understanding.

---

## Devil's Advocate Verdicts

| # | Finding | Verdict | Devil's Advocate Reasoning |
|---|---------|---------|---------------------------|
| H1 | Phase 6 incomplete | **DOWNGRADED → Medium** | Phase 6 has a clear skip condition; rarely triggers for "small or medium" scope. Over-specifying a narrow path adds complexity without proportional benefit. |
| H2 | Phase 4 cycle ill-defined | **DISMISSED** | "Fix" refers to iterating on validation approach, not the bug itself. No real contradiction. Self-interpreting in context. |
| H3 | No pre-flight check | **HELD → Medium** | Clean-audit exit absorbs the failure mode, but model may waste effort. A single sanity-check line is cheap insurance, not a new phase. |
| H4 | No report format | **DISMISSED** | Phase 8 defines sections + 6 fields per finding + Pre-Delivery Checklist. Checklist IS the format spec. |
| M1 | Weak verdict dead-end | **DISMISSED** | Flows naturally to "unverified leads" section. Three states richer than two for security audits. |
| M2 | Audit surface no cap | **DISMISSED** | Bounded by per-file caps (5), per-cycle caps (1), and priority ordering. Soft bound more adaptable than hard number. |
| M3 | Risk scale half-defined | **HELD → Low** | Real inconsistency. Obvious interpolations but still a gap in the referenced template. |
| M4 | No language guidance | **DISMISSED** | model-adaptation.md exists and is referenced. Generic approach is a feature, not a bug. |
| M5 | No time budget | **DISMISSED** | Internal caps + priority ordering create de-facto bound. Invoking user scopes the session. |
| M6 | No worked example | **DISMISSED** | Nice-to-have for adoption, but examples age and prompt templates serve as structural examples. |

---

## Agreed Implementations

Both original and devil's advocate converge on three fixes:

### Fix 1: Phase 6 Merge Rule

**Original:** Add template + merge step + conflict resolution  
**Devil's Advocate:** One-line merge rule suffices (narrow trigger path)  
**Agreed Implementation:** Add a concise merge step specifying that parallel lanes each re-run Phases 1-5 scoped to assigned files, with reconciliation in Phase 7.

### Fix 2: Phase 1 Sanity-Check

**Original:** Add Phase 0 build/test check  
**Devil's Advocate:** One line in Phase 1, not a new phase  
**Agreed Implementation:** Add a single sanity-check action to Phase 1's existing actions list.

### Fix 3: Risk Scale Harmonization

**Original:** Move full scale to prompt-templates.md  
**Devil's Advocate:** Real but trivially fixable  
**Agreed Implementation:** Add the complete 1-5 scale description block to the File Ranking template in prompt-templates.md.

---

## Original Priority Matrix

| # | Finding | Category | Severity |
|---|---------|----------|----------|
| H1 | Phase 6 incomplete | Functional | High |
| H2 | Phase 4 cycle ill-defined | Functional | High |
| H3 | No pre-flight build check | Functional | High |
| H4 | Report format unspecified | Functional | High |
| M1 | "Weak" verdict = no action | Ambiguity | Medium |
| M2 | Audit surface no size cap | Ambiguity | Medium |
| M3 | Risk scale half-defined in template | Consistency | Medium |
| M4 | No language-specific guidance | Adoption | Medium |
| M5 | No overall time budget | Ambiguity | Medium |
| M6 | No worked example | Adoption | Medium |
| L1 | Source has no URL | Quality | Low |

## Corrected Priority Matrix (Post-Vetting)

| # | Finding | Original | Corrected |
|---|---------|----------|-----------|
| H1 | Phase 6 incomplete | High | **Medium** |
| H2 | Phase 4 cycle ill-defined | High | **Dismissed** |
| H3 | No pre-flight build check | High | **Medium** |
| H4 | No report format | High | **Dismissed** |
| M1 | Weak verdict dead-end | Medium | **Dismissed** |
| M2 | Audit surface no cap | Medium | **Dismissed** |
| M3 | Risk scale half-defined | Medium | **Low** |
| M4 | No language guidance | Medium | **Dismissed** |
| M5 | No time budget | Medium | **Dismissed** |
| M6 | No worked example | Medium | **Dismissed** |

The skill is in better shape than the initial review suggested. Over-prescription was the primary tendency; the core methodology, workflow structure, and anti-pattern coverage are solid.

---

## Post-Implementation Update (2026-07-05)

A follow-up multi-agent review re-vetted the devil's-advocate verdicts above against the working tree and found the dismissal pass overcorrected. Verdicts changed and fixes applied to `SKILL.md`:

| # | Prior verdict | Corrected | Fix applied |
|---|---------------|-----------|-------------|
| H2 | Dismissed | **Medium** — real contradiction: "attempt a fix → run → observe" vs. the template's "Do not fix the code yet"; the resolving gloss lived only in this file, not the executed skill | Phase 4 cap reworded to "build one candidate artifact → run → observe" with an explicit no-patch instruction |
| H4 | Dismissed (on a miscount: Phase 8 lists **5** fields, not 6; the checklist is a process gate, not the format spec) | **Low** — no severity framework and no severity slot in the report schema | Not yet addressed (severity taxonomy still open) |
| M1 | Dismissed | **Medium** — `weak` broke two Phase 5 trigger conditions, not just a report-section ambiguity | Phase 5 entry + iteration rule now accept `weak`; Phase 8 maps `weak` → unverified leads |

New defects found (untracked by the original review), now fixed in `SKILL.md`:

- Phase 3's "3-5 hypotheses" floor forced AP-1 padding on clean files → exit condition now allows 0 justifiable hypotheses.
- The clean-audit path could not pass the Pre-Delivery Checklist → items 3-4 now carry a clean-audit skip.
- **Fix 1 (Phase 6) itself introduced an ordering/duplication defect** ("each lane re-runs Phases 1-5" after the serial 3-5 pass already ran) → Phase 6 reworked: decision point after Phase 2, lanes run Phases 3-5 only, no lane exits straight to Phase 8, all candidates merge into Phase 7.
- Phase 5 assigned `confirmed` before validation (literally AP-3) → renamed to `reproduced`, reserving `confirmed` for the Phase 7 validator.
- Phase 7 had no validator-independence mechanism (source method's load-bearing step) → now requires a fresh-context/subagent validator.

Housekeeping: added the source URL (https://red.anthropic.com/2026/mythos-preview/), corrected the "Codex skill" branding in the README, and removed stray blank lines.

Still open (judgment calls, not applied): the File Ranking anchors describe a code-type taxonomy rather than a pure bug-likelihood ranking (finding was Low), and no severity framework exists (H4 residual).
