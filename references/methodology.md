# Bug Hunting Method Reference

This note extracts the directly transferable method from Anthropic's April 7, 2026 cybersecurity capabilities article.

## Source-backed scaffold

Anthropic describes a simple agentic vulnerability-finding scaffold:

1. Launch an isolated container containing the project-under-test and its source.
2. Prompt the model to find a security vulnerability.
3. Let the model read code, run the project, test hypotheses, add debug logic, and iterate.
4. To diversify findings, assign different agents to different files.
5. Before deep auditing, ask the model to rank files by likelihood of containing an interesting bug on a 1-5 scale.
6. Start with the highest-ranked files.
7. After a bug report is produced, run a final agent to confirm whether it is real and interesting.

Anthropic states this explicitly in the scaffold section of the article.

## Important details worth preserving

- They emphasize real execution, not just reasoning over code.
- They use file-level routing to reduce duplicate findings.
- They use a second validation pass to filter low-value or weak reports.
- They favor bug classes with strong oracles, especially memory-safety issues validated by sanitizers.

## Adaptation for local tooling

The original article focuses heavily on advanced zero-day discovery and, in many cases, memory-safety exploitation. For typical local project work with the underlying model, the practical adaptation is:

- keep the scaffold
- reduce ambition from "autonomous exploit development" to "grounded bug discovery"
- demand a repro, failing test, sanitizer finding, or other executable evidence
- prioritize small, high-signal audit targets

Recommended substitutions:

- Instead of long exploit-development loops, prefer:
  - regression tests
  - minimized repro inputs
  - property tests
  - fuzz harnesses
  - differential tests

- Instead of broad repo sweeps, prefer:
  - file risk ranking
  - top 5-20 files
  - parallel audits only for clearly disjoint slices

- Instead of reporting every plausible issue, split outputs into:
  - confirmed
  - weak
  - rejected

## Practical interpretation

The transferable lesson is not "the original system is special, therefore this cannot be reproduced."

It is:
- use the model as a hypothesis generator
- use the program as the oracle
- prioritize risky files
- parallelize narrowly
- validate findings twice

That methodology should transfer reasonably well to smaller codebases and to the underlying model, especially where the codebase has:
- structured inputs
- existing tests
- deterministic CLI behavior
- wrapper/core contracts
- sanitizer or crash visibility

## Source

- Anthropic, April 7, 2026 cybersecurity capabilities article
