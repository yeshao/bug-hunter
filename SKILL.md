---
name: bug-hunter
description: Rank risky files, generate concrete bug hypotheses, require executable evidence, and validate only real findings
license: MIT
---

# Bug Hunter

Run a disciplined vulnerability-discovery loop inspired by Anthropic's April 7, 2026 Mythos Preview methodology, adapted for `gpt-5.4` and local project work.

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

See [references/mythos-method.md](references/mythos-method.md) for the source-backed mapping.

## Workflow

### 1. Establish the audit surface

First inspect the repo and answer:
- What are the executable entrypoints?
- What parses untrusted input?
- What crosses trust boundaries?
- What existing tests or sanitizers can act as an oracle?

Prefer narrow high-risk surfaces:
- parsers and decoders
- CLI argument handling
- network protocol handlers
- file format readers
- auth/session code
- serialization/deserialization
- sandbox, subprocess, temp-file, and path handling

### 2. Rank files before deep auditing

Create a 1-5 risk ranking for candidate files.

Use this scale:
- `1`: constants, docs, trivial wrappers, generated metadata
- `2`: low-risk glue, display helpers, thin config surfaces
- `3`: normal business logic with some branching or state
- `4`: complex stateful logic, error paths, boundary translation
- `5`: raw input parsing, auth, memory-unsafe paths, protocol handling, privilege boundaries

For each file, record:
- score
- short reason
- likely bug class
- best validation route

Start with `5`, then `4`.

### 3. Audit one file or subsystem at a time

For each target file, ask for at most 3-5 bug candidates.

Require every candidate to include:
- the invariant that should hold
- why the code may violate it
- the exact code path
- a minimal reproducer idea
- the fastest validating action
- confidence: `high`, `medium`, or `low`

Reject candidates that are only:
- style complaints
- hypothetical with no code path
- untestable without hand-waving

### 4. Convert hypotheses into evidence

For the strongest candidate, immediately move to evidence:
- write a failing regression test, or
- create a minimal repro input, or
- create a fuzz/property harness, or
- run with sanitizer/debug instrumentation if available

The model should keep iterating until one of these happens:
- the bug is reproduced
- the hypothesis is disproven
- the path is too weak and should be dropped

Do not count a finding as real without executable evidence.

### 5. Use execution as the filter

Preferred evidence, strongest first:
1. sanitizer finding or crash with clear stack
2. deterministic failing test
3. differential mismatch across implementations or wrappers
4. invariant violation shown by program output
5. precise manual proof when execution is impossible

If execution is available, use it.
If a test or repro fails to confirm the bug, discard or downgrade the candidate.

### 6. Run multiple focused audits in parallel

If parallel work is worthwhile, split by disjoint files or subsystems.

Good parallel lanes:
- one agent ranks files
- one agent audits parser/input paths
- one agent audits wrapper/core contract mismatches
- one agent validates candidate findings

Keep write scopes disjoint if code changes are involved.

### 7. Final validator pass

Before reporting a bug as a meaningful finding, run a final validation prompt:

`I have received the following bug report. Please confirm whether it is real, reproducible, and interesting. Reject findings that are technically true but low-impact, non-reproducible, or too obscure to matter.`

The validator should answer:
- Is the finding real?
- Is the repro credible?
- What severity or impact is plausible?
- Is this actually interesting enough to report?
- What assumptions remain?

### 8. Report only grounded findings

Final output should separate:
- confirmed findings
- disproven candidates
- unverified leads
- remaining high-risk surfaces not yet audited

For each confirmed finding include:
- file and code path
- impact
- reproducer or failing test
- current verification status
- minimal fix direction

## Prompt Templates

### File ranking

```text
Rank the files in this codebase from 1 to 5 by how likely they are to contain an interesting bug or vulnerability.

Scoring:
1 = cannot realistically contain a meaningful bug
5 = high-risk surface such as input parsing, auth, protocol handling, memory-unsafe logic, or trust-boundary code

For each file, return:
- score
- one-sentence reason
- likely bug class
- best validation route
```

### Per-file audit

```text
Audit this file for real bugs, not style issues.

Focus on:
- incorrect behavior
- edge cases
- trust-boundary mistakes
- parser/state-machine errors
- cross-layer contract mismatches
- error-handling failures

Give at most 5 candidates.
For each candidate include:
1. invariant violated
2. exact code path
3. why the bug seems plausible from the code
4. minimal reproducer
5. fastest validation step
6. confidence: high/medium/low

Omit any candidate you cannot justify from the code.
```

### Evidence conversion

```text
Take the highest-confidence candidate and produce the smallest executable artifact that could confirm or refute it.

Prefer:
- failing test
- minimal repro input
- fuzz harness
- sanitizer-backed run

Do not fix the code yet.
Do not propose multiple directions.
Pick the fastest decisive check.
```

### Final validator

```text
I have received the following bug report. Please confirm whether it is real, reproducible, and interesting.

Reject findings that are:
- speculative
- low-impact edge cases with no realistic trigger
- unsupported by the repro

Return:
1. verdict: confirmed / weak / rejected
2. why
3. remaining assumptions
4. likely severity
5. what evidence would fully close the case
```

## GPT-5.4 Adaptation Notes

- `gpt-5.4` is strong enough to follow the scaffold, but should be constrained to short candidate lists and fast evidence loops.
- On small codebases, spend more effort on file ranking and less on massive parallelism.
- Prefer deterministic tests over long autonomous exploit-development loops.
- Prefer safe local bug classes first:
  - parser crashes
  - path traversal
  - temp-file races
  - CLI/wrapper mismatches
  - Unicode/path normalization bugs
  - partial-success and error-propagation issues

For memory-unsafe targets:
- use sanitizers if available
- use fuzzing or minimized crash repros
- treat sanitizer hits as the primary oracle

For managed-language repos:
- focus on trust boundaries, parsing, file handling, state transitions, concurrency, and contract mismatches

## Recommended Operating Rules

- Keep the audit surface narrow.
- Prefer one subsystem at a time.
- Cap each audit pass to a small number of candidates.
- Force every claim toward executable evidence quickly.
- Do not merge "interesting idea" with "confirmed bug."
- Separate discovery from validation.
- If a finding cannot survive the validator pass, do not report it as confirmed.

## Expected Deliverables

For a useful run, produce:
- a ranked file list
- a shortlist of bug candidates for top files
- at least one concrete repro attempt
- a validator verdict for any claimed finding
- a residual-risk list for unaudited high-risk files

## When This Skill Works Best

- smaller repos with good tests
- parser-heavy or CLI-heavy projects
- projects with clear trust boundaries
- mixed-language repos where wrappers may drift from core behavior

## When To Reach For More Tooling

Escalate beyond this workflow when:
- fuzzing harnesses are easy to add and inputs are highly structured
- sanitizers are available and the target is memory-unsafe
- the project is large enough to justify many parallel audit lanes

In those cases, pair this workflow with:
- fuzzing
- sanitizers
- differential testing
- static-analysis facts
