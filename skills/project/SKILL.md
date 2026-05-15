---
name: project
description: Load when user asks to implement, refactor, review, or structure Python project code for maintainability, production readiness, or team conventions.
---

## Purpose

先判断项目形态，再施加最小必要约束。简单脚本保持简单；服务型项目才使用服务型边界。

## Defaults

栈基线（包、版本、反模式、量化阈值、默认参数）见 `references/stack.md`。

不要强制 FastAPI、SQLAlchemy、Milvus、LLM 工具、`src/` 布局或复杂打包。

## Reference loading rules

- Read `references/stack.md` for stack selection, package alternatives, anti-patterns, quantitative thresholds, or default parameters.
- Read `references/code-design.md` for refactor, architecture, structure, "是否合理", "生产可用", unclear data flow, excessive abstraction, or repository pattern decisions.
- Read `references/error-handling.md` for exceptions, retries (stamina), external I/O failures, boundary errors, or user-facing messages.
- Read `references/async-resource.md` for async/await, DB/HTTP clients, files, subprocesses, streams, locks, sockets, or lifecycle resources.
- Read `references/type-boundaries.md` for public functions, config models, service boundaries, schemas, or structured outputs.
- Read `references/datetime.md` for time, timezone, scheduling, timestamps, ISO 8601, or DB time columns.
- Read `references/logging.md` for loguru config, sinks, structured fields, sensitive filtering, or async/multi-process logs.
- Read `references/config-secrets.md` for `.env`, pydantic-settings, API keys, multi-env, or `SecretStr` usage.
- Read `references/performance-testing.md` only for slow code, memory, large files, profiling, bug fixes, behavior changes, or verification.

## Gotchas

- 不要输出完整最佳实践清单；只处理当前任务会出错的点。
- 不要为了规范新增抽象、目录、配置层、兼容层或 fallback。
- 不要建议 Celery；复杂到像是需要 Celery 时，先重新评估系统拆分。
- 不要用 `requests`、标准 `logging`、`os.path`、`datetime.utcnow()`、裸 dict 跨边界。
- 不要主动加入 Docker/K8s、复杂发布、复杂 Python 包分发或完整可观测性体系。
- 不要顺手重构无关代码或改变公共 API、CLI 参数、返回结构、异常类型、日志格式。
- 不要把一次性脚本工程化。
