# bug-hunter

Evidence-first bug hunting for coding agents.

`bug-hunter` is a reusable skill that adapts the transferable parts of Anthropic's April 7, 2026 Mythos Preview methodology into a practical workflow for local project audits.

Version: `1.0.0`
License: MIT

## What it does

- ranks risky files before deep auditing
- generates concrete bug hypotheses instead of broad speculation
- converts the strongest hypothesis into executable evidence
- rejects findings that do not reproduce
- runs a final validator pass before reporting

## Repository layout

- `SKILL.md`: primary skill definition
- `references/mythos-method.md`: source-backed methodology notes
- `compat/claude-code/bug-hunter.md`: Claude Code subagent adaptation
- `LICENSE`: MIT license
- `CHANGELOG.md`: release notes

## Installation

### Codex

OpenAI's public skills catalog documents installing a skill via `$skill-installer` using a GitHub directory URL.

Use:

```text
$skill-installer install https://github.com/yeshao/bug-hunter/tree/main
```

Then restart Codex so the new skill is discovered.

### OpenCode

OpenCode documents direct `SKILL.md` discovery from project and global skill directories.

Project-local install:

```bash
git clone https://github.com/yeshao/bug-hunter.git /tmp/bug-hunter
mkdir -p .opencode/skills/bug-hunter
cp -R /tmp/bug-hunter/. .opencode/skills/bug-hunter/
```

Global install:

```bash
git clone https://github.com/yeshao/bug-hunter.git /tmp/bug-hunter
mkdir -p ~/.config/opencode/skills/bug-hunter
cp -R /tmp/bug-hunter/. ~/.config/opencode/skills/bug-hunter/
```

OpenCode also recognizes compatible `.claude/skills` and `.agents/skills` locations.

### OpenClaw

OpenClaw's docs describe skills in the agent workspace at `~/.openclaw/workspace/skills/<skill>/SKILL.md`.

Install with:

```bash
git clone https://github.com/yeshao/bug-hunter.git /tmp/bug-hunter
mkdir -p ~/.openclaw/workspace/skills/bug-hunter
cp -R /tmp/bug-hunter/. ~/.openclaw/workspace/skills/bug-hunter/
```

### Claude Code

Claude Code's official reusable prompt primitives are Markdown subagents in `.claude/agents/` or `~/.claude/agents/`, and custom slash commands in `.claude/commands/` or `~/.claude/commands/`.

This repository includes a Claude Code subagent adaptation at `compat/claude-code/bug-hunter.md`.

Project-local install:

```bash
git clone https://github.com/yeshao/bug-hunter.git /tmp/bug-hunter
mkdir -p .claude/agents
cp /tmp/bug-hunter/compat/claude-code/bug-hunter.md .claude/agents/bug-hunter.md
```

User-level install:

```bash
git clone https://github.com/yeshao/bug-hunter.git /tmp/bug-hunter
mkdir -p ~/.claude/agents
cp /tmp/bug-hunter/compat/claude-code/bug-hunter.md ~/.claude/agents/bug-hunter.md
```

After that, Claude Code can invoke it as the `bug-hunter` subagent when appropriate.

## Quick use

Use this workflow when you want to:

- find real bugs or vulnerabilities in a codebase
- audit parser, CLI, auth, file-handling, or trust-boundary code
- produce grounded findings with repros instead of speculative review

The expected output of a useful run is:

- a ranked file list
- a shortlist of bug candidates for top files
- at least one concrete repro attempt
- a validator verdict for any claimed finding
- a residual-risk list for unaudited high-risk files

## Sources

- Anthropic, [Assessing Claude Mythos Preview’s cybersecurity capabilities](https://red.anthropic.com/2026/mythos-preview/)
- OpenAI, [openai/skills README](https://github.com/openai/skills)
- Anthropic, [Claude Code slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- Anthropic, [Claude Code subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- OpenCode, [Agent Skills](https://opencode.ai/docs/skills)
- OpenClaw, [openclaw/openclaw](https://github.com/clawdbot/clawdbot)
