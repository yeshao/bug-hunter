# GPT-5.4 Adaptation Notes

Guidance for adapting this methodology to GPT-5.4.

## Core adaptation principles

- `gpt-5.4` is strong enough to follow this workflow, but it should be constrained to short candidate lists and fast evidence loops
- on small codebases, spend more effort on file ranking and less on large parallel audits
- prefer deterministic tests over long autonomous exploit-development loops
- prioritize safe, local bug classes first

## Priority bug classes

| Type | Description |
|------|-------------|
| parser crashes | Input parser failures |
| path traversal | Unsafe path resolution |
| temp-file races | Temporary file race conditions |
| CLI/wrapper mismatches | CLI and wrapper behavior drift |
| Unicode/path normalization | Unicode and path normalization bugs |
| partial-success and error-propagation | Incorrect partial success or error propagation |

## Memory-unsafe targets

- use sanitizers when available
- use fuzzing or minimized crash repros
- treat sanitizer hits as the primary oracle

## Managed-language repositories

Focus on:
- trust boundaries
- parsing
- file handling
- state transitions
- concurrency
- contract mismatches

## Differences from the original research method

| Original research method | GPT-5.4 adaptation |
|--------------------------|--------------------|
| autonomous exploit development | grounded bug discovery |
| large-scale parallelism | narrow audit surface with deeper single-file work |
| complex exploit chains | regression tests and minimal repros |
| report every suspicious case | separate confirmed, weak, and rejected |
