---
name: bug-hunter
description: "Bug discovery workflow based on Anthropic's 2026 methodology. Triggers: bug-hunter, bug hunt, vulnerability audit, security audit, find bugs, bug hunt workflow. NOT: normal feature development (use code-pro), style reviews (use code-review-v2), or simple Q&A."
---

# Bug Hunter

**IRON LAW: Do not report without executable evidence; do not confirm without validation.**

Run a disciplined vulnerability-discovery loop inspired by Anthropic's April 7, 2026 research article, adapted for `gpt-5.4` and local project work.

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

## Pre-Delivery Checklist

- [ ] Produced a ranked 1-5 risk file list
- [ ] Every bug candidate includes a concrete code path
- [ ] The strongest candidate has an executable repro, test, fuzz case, or sanitizer run
- [ ] A validator pass classified findings as confirmed, weak, or rejected
- [ ] The report separates confirmed findings, disproven candidates, and unverified leads

## Workflow

| Phase | Entry condition | Exit condition |
|------|-----------------|----------------|
| 1. Audit Surface | Received a vulnerability audit request | Clear entrypoints and trust boundaries identified |
| 2. File Ranking | Audit surface defined | 1-5 risk ranking produced |
| 3. Per-File Audit | Risky files identified | 3-5 testable hypotheses produced |
| 4. Evidence Conversion | Candidate hypothesis exists | Executable repro or disproof produced |
| 5. Execution Filter | Repro or disproof exists | Confirmed, weak, or rejected status assigned |
| 6. Parallel Audit | Audit surface is broad enough | Parallel lanes complete validation |
| 7. Validator Pass | Candidate looks confirmed | Validator returns a verdict |
| 8. Report | Validator pass complete | Structured report delivered |

### 1. Establish the audit surface

**Entry condition:** a vulnerability audit request has been received.

**Actions:**
- inspect executable entrypoints
- identify code that parses untrusted input
- identify trust-boundary crossings
- inspect existing tests and available sanitizers

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

**Exit condition:** 3-5 testable hypotheses are produced.

### 4. Convert hypotheses into evidence

**Entry condition:** candidate hypotheses exist.

**Action:** use the Evidence Conversion template.

Produce one of:
- a failing regression test
- a minimal repro input
- a fuzz or property harness
- a sanitizer-backed run

**Exit condition:** the bug is reproduced, disproven, or dropped.

### 5. Use execution as the filter

**Entry condition:** a repro or disproof exists.

**Evidence strength, strongest first:**
1. sanitizer finding or crash with a clear stack
2. deterministic failing test
3. differential mismatch
4. invariant violation in output
5. precise manual proof when execution is impossible

**Exit condition:** assign confirmed, weak, or rejected.

### 6. Run multiple focused audits in parallel

**Entry condition:** the audit surface is broad enough.

**Parallel lanes:**
- file ranking
- parser and input paths
- wrapper and core contract mismatches
- candidate validation

**Exit condition:** each lane finishes validation.

### 7. Final validator pass

**Entry condition:** a candidate appears confirmed.

**Action:** use the Final Validator template.

The validator should answer:
- Is the finding real?
- Is the repro credible?
- What severity is plausible?
- Is the finding interesting enough to report?
- What assumptions remain?

**Exit condition:** a validator verdict exists.

### 8. Report only grounded findings

**Entry condition:** validator pass is complete.

**Output structure:**
- confirmed findings
- disproven candidates
- unverified leads
- remaining high-risk surfaces

Each confirmed finding should include:
- file and code path
- impact
- reproducer or failing test
- verification status
- minimal fix direction

**Exit condition:** a structured report passes the Pre-Delivery Checklist.

## References

| File | Purpose |
|------|---------|
| [references/methodology.md](references/methodology.md) | Source methodology and mapping |
| [references/prompt-templates.md](references/prompt-templates.md) | Audit prompt templates |
| [references/gpt5-adaptation.md](references/gpt5-adaptation.md) | GPT-5.4 adaptation guidance |

## Best Practices

- keep the audit surface narrow
- prefer deep single-file work over broad parallel noise
- drive each hypothesis toward executable evidence quickly
- do not confuse an interesting idea with a confirmed bug
- separate discovery from validation
- do not report findings as confirmed if they fail validator review
