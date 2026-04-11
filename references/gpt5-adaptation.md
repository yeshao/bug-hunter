# GPT-5.4 Adaptation Notes

Mythos 方法论在 GPT-5.4 上的适配建议。

## 核心适配原则

- `gpt-5.4` 足够强大以遵循此框架，但应限制为短候选列表和快速证据循环
- 在小型代码库上，投入更多精力进行文件排序，减少大规模并行审计
- 优先选择确定性测试而非长自主漏洞开发循环
- 优先处理安全的本地 bug 类型

## 优先 bug 类型

| 类型 | 说明 |
|------|------|
| parser crashes | 输入解析器崩溃 |
| path traversal | 路径遍历 |
| temp-file races | 临时文件竞态 |
| CLI/wrapper mismatches | CLI 和包装器不匹配 |
| Unicode/path normalization | Unicode/路径规范化问题 |
| partial-success & error-propagation | 部分成功和错误传播问题 |

## 内存不安全目标

- 如可用，使用 sanitizers
- 使用 fuzzing 或最小化崩溃复现
- 将 sanitizer 命中作为主要 oracle

## 托管语言仓库

重点关注：
- trust boundaries (信任边界)
- parsing (解析)
- file handling (文件处理)
- state transitions (状态转换)
- concurrency (并发)
- contract mismatches (契约不匹配)

## 与 Mythos 原版的区别

| Mythos 原版 | GPT-5.4 适配 |
|------------|-------------|
| 自主漏洞开发 |  grounded bug discovery |
| 大规模并行 | 窄审计面，单文件深入 |
| 复杂利用链 | 回归测试/最小复现 |
| 报告所有可疑 | 区分 confirmed/weak/rejected |
