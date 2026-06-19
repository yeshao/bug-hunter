# bug-hunter

A Codex skill for disciplined, evidence-driven bug hunting.

This skill adapts the transferable parts of Anthropic's April 7, 2026 cybersecurity research article into a practical workflow for local project audits with the underlying model:

- rank risky files before deep auditing
- generate concrete bug hypotheses
- convert the best hypothesis into executable evidence
- reject speculative findings that do not reproduce
- run a final validator pass before reporting

## Files

- `SKILL.md`: the skill definition and operating workflow
- `references/methodology.md`: source-backed notes on the methodology adaptation
- `references/prompt-templates.md`: audit prompt templates
- `references/model-adaptation.md`: model adaptation guidance

## Usage

Place this directory under your Codex skills path as `bug-hunter`, then invoke it when you want a structured bug-finding workflow.

Typical use cases:

- finding real bugs in a small or medium codebase
- auditing parser, CLI, auth, file-handling, or trust-boundary code
- producing grounded findings with repros instead of speculative review

## Source inspiration

- Anthropic, April 7, 2026 cybersecurity capabilities article
