---
name: mythos-bug-hunt
description: "Mythos-style 漏洞发现工作流，基于 Anthropic 2026 方法论。触发词：mythos bug hunt, bug hunt, vulnerability audit, 漏洞挖掘, 找bug, 安全审计, bug hunt workflow。NOT: 普通功能开发（用 code-pro）、代码风格审查（用 code-review-v2）、简单问答（直接问）。"
---

# Mythos-Style Bug Hunt

**IRON LAW: 无执行证据不报告；未经验证不确认。**

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

## Anti-Patterns (禁止行为)

| # | 反模式 | 后果 |
|---|--------|------|
| AP-1 | 报告无代码路径的推测性 bug | 浪费验证时间 |
| AP-2 | 将风格问题标记为安全漏洞 | 噪音 findings |
| AP-3 | 未经验证就报告为 confirmed | 虚假阳性 |
| AP-4 | 追求数量而非可复现性 | 报告无价值 |
| AP-5 | 忽视 validator 的 reject 意见 | 降低可信度 |

## Pre-Delivery Checklist (输出前必检)

- [ ] 已输出风险文件排序列表 (1-5 分)
- [ ] 每个候选 bug 有具体代码路径
- [ ] 最强候选已生成可执行复现 (test/fuzz/sanitizer)
- [ ] 通过 validator 验证 (confirmed/weak/rejected)
- [ ] 区分三类结果，不混为一谈

## Workflow

| 阶段 | 入口条件 | 出口条件 |
|------|---------|---------|
| 1. Audit Surface | 收到漏洞审计请求 | 明确可执行入口点和信任边界 |
| 2. File Ranking | Audit Surface 完成 | 输出 1-5 分风险排序 |
| 3. Per-File Audit | 有风险文件列表 | 生成 3-5 个可验证假设 |
| 4. Evidence Conversion | 有候选假设 | 生成可执行复现或证伪 |
| 5. Execution Filter | 有复现/证伪 | 确认 confirmed/weak/rejected |
| 6. Parallel Audit | 审计面足够大 | 各 lane 完成验证 |
| 7. Validator Pass | 有疑似 confirmed | validator 给出 verdict |
| 8. Report | Validator 完成 | 输出结构化报告 |

### 1. Establish the audit surface

**入口条件：** 收到漏洞审计请求  
**操作：**
- 检查可执行入口点
- 识别解析不可信输入的代码
- 识别跨越信任边界的位置
- 检查现有测试或 sanitizer

**高优先级目标：**
- parsers and decoders
- CLI argument handling
- network protocol handlers
- file format readers
- auth/session code
- serialization/deserialization
- sandbox, subprocess, temp-file, path handling

**出口条件：** 明确可执行入口点和信任边界

### 2. Rank files before deep auditing

**入口条件：** Audit Surface 完成  
**操作：** 使用 [references/prompt-templates.md](references/prompt-templates.md) 中的 File ranking 模板

风险等级 (1-5):
- `1`: constants, docs, trivial wrappers
- `2`: low-risk glue, display helpers
- `3`: normal business logic
- `4`: complex stateful logic, error paths
- `5`: input parsing, auth, memory-unsafe, trust boundaries

**出口条件：** 输出风险文件排序 (从 5 分开始)

### 3. Audit one file or subsystem at a time

**入口条件：** 有风险文件列表  
**操作：** 使用 Per-file audit 模板，每文件最多 5 个候选

每个候选必须包含：
- invariant violated
- exact code path
- why plausible
- minimal reproducer idea
- fastest validation step
- confidence: high/medium/low

**出口条件：** 生成 3-5 个可验证假设

### 4. Convert hypotheses into evidence

**入口条件：** 有候选假设  
**操作：** 使用 Evidence conversion 模板

生成：
- failing regression test, or
- minimal repro input, or
- fuzz/property harness, or
- sanitizer run

**出口条件：** bug reproduced / hypothesis disproven / path dropped

### 5. Use execution as the filter

**入口条件：** 有复现/证伪  
**证据强度排序：**
1. sanitizer finding / crash with stack
2. deterministic failing test
3. differential mismatch
4. invariant violation in output
5. precise manual proof (if execution impossible)

**出口条件：** 确认 confirmed / weak / rejected

### 6. Run multiple focused audits in parallel

**入口条件：** 审计面足够大  
**并行 lane：**
- file ranking
- parser/input paths
- wrapper/core contract mismatches
- candidate validation

**出口条件：** 各 lane 完成验证

### 7. Final validator pass

**入口条件：** 有疑似 confirmed  
**操作：** 使用 Final validator 模板

Validator 回答：
- Is the finding real?
- Is the repro credible?
- What severity?
- Is this interesting enough?
- What assumptions remain?

**出口条件：** validator 给出 verdict

### 8. Report only grounded findings

**入口条件：** Validator 完成  
**输出结构：**
- confirmed findings
- disproven candidates
- unverified leads
- remaining high-risk surfaces

每个 confirmed finding 包含：
- file and code path
- impact
- reproducer or failing test
- verification status
- minimal fix direction

**出口条件：** 输出结构化报告，通过 Pre-Delivery Checklist

## References

| 文件 | 内容 |
|------|------|
| [references/mythos-method.md](references/mythos-method.md) | Mythos 方法论来源与映射 |
| [references/prompt-templates.md](references/prompt-templates.md) | 审计提示模板 |
| [references/gpt5-adaptation.md](references/gpt5-adaptation.md) | GPT-5.4 适配建议 |

## Best Practices

- 保持审计面狭窄
- 优先单文件深入而非大规模并行
- 强制每个假设快速走向可执行证据
- 不将"有趣的想法"与"确认的 bug"混为一谈
- 发现与验证分离
- 无法通过 validator 的 finding 不报告为 confirmed
