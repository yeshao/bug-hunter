# bug-hunter

A [Pi](https://github.com/badlogic/pi) agent skill for disciplined, evidence-driven bug hunting.

This skill adapts the transferable parts of Anthropic's April 7, 2026 cybersecurity research article into a practical 8-phase workflow for auditing local projects:

1. **Audit Surface** — identify entry points, trust boundaries, and data flows
2. **File Ranking** — rank files by risk (1–5) before deep auditing
3. **Per-File Audit** — generate concrete bug hypotheses per risky file
4. **Per-Candidate Audit** — develop the strongest hypothesis into a testable claim
5. **Execution** — produce executable evidence (repro, failing test, fuzz harness, sanitizer run)
6. **Parallel Audit** — (optional) run parallel auditor lanes for large attack surfaces
7. **Validation** — classify findings as confirmed, weak, or rejected
8. **Reporting** — deliver findings with evidence, disproven candidates, and unverified leads

**Iron Rule:** Do not report without executable evidence; do not confirm without validation.

## Files

- `SKILL.md` — skill definition, anti-patterns, pre-delivery checklist, and the 8-phase workflow
- `references/methodology.md` — source-backed notes on the methodology adaptation
- `references/prompt-templates.md` — prompt templates for each workflow phase
- `references/model-adaptation.md` — guidance on adapting the audit to the target language/framework

## Usage

Place this directory in your agent's skills directory (for example `~/.pi/extensions/bug-hunter`), then invoke it when you want a structured bug-finding workflow.

Typical use cases:

- finding real bugs in a small or medium codebase
- auditing parser, CLI, auth, file-handling, or trust-boundary code
- producing grounded findings with repros instead of speculative review

## Source inspiration

- Anthropic, "Assessing Claude Mythos Preview's cybersecurity capabilities," April 7, 2026 — https://red.anthropic.com/2026/mythos-preview/
