---
name: bug-hunter
description: "Bug discovery workflow based on Anthropic's 2026 methodology. Triggers: bug-hunter, bug hunt, vulnerability audit, security audit, find bugs, bug hunt workflow. NOT: normal feature development (use code-pro), style reviews (use code-review-v2), or simple Q&A."
---

# Bug Hunter

**IRON LAW: Do not report without executable evidence; do not confirm without validation.**

Run a disciplined vulnerability-discovery loop inspired by Anthropic's April 7, 2026 research article, adapted for the underlying model and local project work.

Use this skill when the user wants to:

- find real bugs or vulnerabilities in a codebase
- reproduce the "LLM finds bugs" effect on a smaller repo
- run an agentic audit with proofs instead of speculative review
- generate a repeatable bug-hunting workflow

Do not use this skill for:

- normal feature implementation
- broad style reviews
- cleanup-only refactors without bug-hunting intent

## Core Idea

The model should not be trusted as the oracle.

Instead, use the model to:

1. rank risky files
2. generate concrete bug hypotheses
3. produce repro steps, failing tests, fuzz harnesses, or sanitizer-triggering inputs
4. run the target and reject unsupported hypotheses
5. validate that surviving findings are both real and important

This follows the transferable part of Anthropic's scaffold:

- isolated project-under-test
- simple vulnerability-finding prompt
- agentic code reading plus execution
- file-prioritized parallelization
- final validator pass

See [references/methodology.md](references/methodology.md) for the source-backed mapping.

## Anti-Patterns

| # | Anti-pattern | Consequence |
|---|--------------|-------------|
| AP-1 | Reporting speculative bugs with no code path | Wastes validation time |
| AP-2 | Labeling style issues as security bugs | Creates noisy findings |
| AP-3 | Marking findings confirmed before validation | Produces false positives |
| AP-4 | Optimizing for quantity over reproducibility | Lowers report value |
| AP-5 | Ignoring validator rejection | Reduces credibility |
| AP-6 | Spending unbounded time on one hypothesis | Starves other candidates |

## Pre-Delivery Checklist

- [ ] Produced a ranked 1-5 risk file list
- [ ] Every bug candidate includes a concrete code path
- [ ] If any candidate survived, the strongest has an executable repro, test, fuzz case, or sanitizer run (skip on a clean audit)
- [ ] If any candidate survived, a validator pass classified findings as confirmed, weak, or rejected (skip on a clean audit)
- [ ] The report separates confirmed findings, disproven candidates, and unverified leads
- [ ] If no finding survived evidence, a clean audit report lists files examined and surfaces mapped

## Workflow

| Phase | Entry condition | Exit condition |
|------|-----------------|----------------|
| 1. Audit Surface | Received a vulnerability audit request | Clear entrypoints and trust boundaries identified |
| 2. File Ranking | Audit surface defined | 1-5 risk ranking produced |
| 3. Per-File Audit | Risky files identified | 0-5 justifiable hypotheses produced |
| 4. Evidence Conversion | Candidate hypothesis exists | Executable repro or disproof produced |
| 5. Execution Filter | Repro or disproof exists | Reproduced, weak, or rejected status assigned |
| 6. Parallel Audit | Audit surface has ≥3 disjoint trust boundaries | Parallel lanes complete validation |
| 7. Validator Pass | Candidate looks confirmed | Validator returns a verdict |
| 8. Report | Validator pass complete | Structured report delivered |

### 1. Establish the audit surface

**Entry condition:** a vulnerability audit request has been received.

**Actions:**

- inspect executable entrypoints
- identify code that parses untrusted input
- identify trust-boundary crossings
- inspect existing tests and available sanitizers
- sanity-check that the project builds and is runnable (record any blockers before proceeding)

**High-priority targets:**

- parsers and decoders
- CLI argument handling
- network protocol handlers
- file format readers
- auth and session code
- serialization and deserialization
- sandbox, subprocess, temp-file, and path handling

**Exit condition:** executable entrypoints and trust boundaries are clearly mapped.

### 2. Rank files before deep auditing

**Entry condition:** the audit surface is defined.

**Action:** use the File Ranking template from [references/prompt-templates.md](references/prompt-templates.md).

Risk scale:

- `1`: constants, docs, trivial wrappers
- `2`: low-risk glue, display helpers
- `3`: normal business logic
- `4`: complex stateful logic and error paths
- `5`: input parsing, auth, memory-unsafe logic, and trust boundaries

**Exit condition:** a risk-ranked file list exists, starting from score `5`.

### 3. Audit one file or subsystem at a time

**Entry condition:** a ranked file list exists.

**Action:** use the Per-File Audit template and cap each file at 5 candidates.

Each candidate must include:

- invariant violated
- exact code path
- why the bug is plausible
- minimal reproducer idea
- fastest validation step
- confidence: high, medium, or low

**Exit condition:** up to 5 justifiable hypotheses are produced for this file. Zero is a valid result — if the file yields no candidate you can justify from the code, record that and move to the next ranked file. Do not invent candidates to hit a quota; padding is AP-1.

### 4. Convert hypotheses into evidence

**Entry condition:** candidate hypotheses exist.

**Action:** use the Evidence Conversion template.

Produce one of:

- a failing regression test
- a minimal repro input
- a fuzz or property harness
- a sanitizer-backed run

Per-candidate cap: one Phase 4 cycle is *build one candidate artifact → run it → observe the result*. Do **not** patch the target code at this stage (the Evidence Conversion template says "Do not fix the code yet") — a fix before a failing repro exists contaminates the oracle. If one cycle produces no runnable artifact (for example the harness or test will not even execute), classify the candidate as `weak` and move to the next candidate. Do not iterate on the same hypothesis indefinitely.

**Exit condition:** the bug is reproduced, disproven, or dropped.

### 5. Use execution as the filter

**Entry condition:** Phase 4 produced a repro, a disproof, or a `weak` classification for at least one candidate.

**Evidence strength, strongest first:**

1. sanitizer finding or crash with a clear stack
2. deterministic failing test
3. differential mismatch
4. invariant violation in output
5. precise manual proof when execution is genuinely impossible — this supports at most a `weak` classification (reported as an unverified lead), never a confirmed finding, so the IRON LAW's executable-evidence bar for reported findings still holds

**Iteration rule:** if all candidates for a file are rejected or classified `weak`, return to Phase 3 for the next highest-ranked file. If all files are exhausted with no `reproduced` finding, skip Phases 6–7 and proceed directly to the clean-audit report (Phase 8).

**Exit condition:** assign each candidate `reproduced`, `weak`, or `rejected`. Reserve `confirmed` for after the Phase 7 validator pass — marking a finding confirmed here is AP-3.

### 6. Run multiple focused audits in parallel

**Decision point:** choose between a serial pass and parallel lanes right after Phase 2, using the ranked file list. This phase is an *alternative* to a single serial pass through Phases 3-5, not an addition on top of one. If you have already completed a serial Phases 3-5 pass, do not also run this phase — it would re-audit the same files.

**Entry condition:** the ranked audit surface splits into ≥3 disjoint trust boundaries whose *audits* need no shared context (assessed from Phases 1-2 — code-level shared imports or types do not disqualify a split).

**When to skip:** for small codebases or fewer than 3 disjoint trust boundaries, skip this phase entirely, run the serial Phases 3-5 pass instead, and go to Phase 7.

**Parallel lanes:** each lane gets a bounded, explicit file list, not a topic.

**Scope rule:** each lane runs Phases 3-5 scoped to its assigned file list only (Phases 1-2 already ran once, globally — do not repeat them per lane). A lane never proceeds to the clean-audit report on its own; an empty lane simply contributes no findings.

**Merge rule:** after all lanes complete, reconcile findings. Deduplicate shared findings, and carry every surviving candidate — `reproduced`, `weak`, and cross-lane contradictions — into Phase 7 for the validator pass.

**Exit condition:** all lanes finish and their findings are merged into a single reconciled candidate set for Phase 7.

### 7. Final validator pass

**Entry condition:** at least one candidate is `reproduced`, or Phase 6 produced a reconciled candidate set to validate.

**Action:** run the Final Validator template in a fresh context — ideally a separate agent or subagent that did not perform the discovery — so the validator does not rubber-stamp its own Phase 5 work. Give it only the bug report and the code, not the discovery reasoning. This independent second pass is the load-bearing step of the source methodology; do not collapse it into the discovery context.

The validator should answer:

- Is the finding real?
- Is the repro credible?
- What severity is plausible?
- Is the finding interesting enough to report?
- What assumptions remain?

**Exit condition:** a validator verdict exists.

### 8. Report only grounded findings

**Entry condition:** validator pass is complete — or the iteration rule (Phase 5) produced no `reproduced` candidates.

**Output structure:**

- confirmed findings (may be zero)
- disproven candidates (`rejected`)
- unverified leads (candidates classified `weak`)
- remaining high-risk surfaces

Each confirmed finding should include:

- file and code path
- impact
- reproducer or failing test
- verification status
- minimal fix direction

**Clean-audit exit:** if no candidate survived evidence, the report states "no confirmed findings" and includes the audit surface map plus the risk ranking as the deliverable. Do not manufacture findings to fill the report.

**Exit condition:** a structured report passes the Pre-Delivery Checklist.

## References

| File | Purpose |
|------|---------|
| [references/methodology.md](references/methodology.md) | Source methodology and mapping |
| [references/prompt-templates.md](references/prompt-templates.md) | Audit prompt templates |
| [references/model-adaptation.md](references/model-adaptation.md) | Model adaptation guidance |

## Best Practices

- keep the audit surface narrow
- prefer deep single-file work over broad parallel noise
- drive each hypothesis toward executable evidence quickly
- do not confuse an interesting idea with a confirmed bug
- separate discovery from validation
- do not report findings as confirmed if they fail validator review
