---
name: cli
description: Load when user builds DevOps scripts, CI-invoked scripts, Python CLI tools, reusable automation commands, or asks about arguments, stdout/stderr, exit codes, or machine-readable output.
---

## Purpose

CLI 只在长期复用、自动化调度或多环境调用时设计。普通脚本不要强行 CLI 化。

## Rules

- 简单命令用 argparse；子命令 ≥ 3 **或** 参数 ≥ 5 **或** 需要类型补全 → typer。
- stdout 输出正常结果，stderr 输出诊断信息。
- 失败必须非零退出码。
- rich 只用于人读输出；机器消费输出必须稳定、简单、可解析。
- 参数保持少而稳定；复杂配置优先配置文件或脚本顶部常量。
- DevOps 脚本关注幂等、可重复执行、失败退出码和敏感信息脱敏。

## Reference loading rules

- Read `references/io-contract.md` when command config, parsed inputs, stdout, stderr, JSON/CSV output, or CI compatibility matters.
- Read `references/error-exit.md` when mapping exceptions to exit codes or stderr messages.
- Read `references/resource-safety.md` when commands open files, subprocesses, HTTP clients, DB sessions, temp files, or locks.

## Gotchas

- 不要为一次性脚本设计一堆参数。
- 不要让机器消费 rich 表格。
- 不要吞掉异常后返回 0。
- 不要把 secrets、token、内部路径打印到 stderr。
- 不要把 typer 套到只有 1–2 个参数的脚本上。
