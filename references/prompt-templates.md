# Prompt Templates

Predefined audit prompt templates.

## File ranking

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

## Per-file audit

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

## Evidence conversion

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

## Final validator

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
