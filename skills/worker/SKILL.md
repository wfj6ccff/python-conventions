---
name: worker
description: Load when user implements Python background jobs, scheduled tasks, retrying workers, APScheduler, RQ, arq, Redis queues, or long-running task orchestration.
---

## Purpose

后台任务必须有生命周期、幂等、退出机制、错误记录和状态追踪。

## Rules

- 轻量本地定时任务用 APScheduler；Redis 轻量队列用 RQ 或 arq。
- 不建议 Celery；复杂到像是需要 Celery 时先重新评估系统拆分。
- 任务函数必须幂等。
- 参数只传可序列化小对象，不传 DB session、客户端或大对象。
- 重试默认 stamina：3 次，退避 1/2/4s，总耗时 ≤30s；按业务调整需明示。
- 非幂等操作不能随便自动重试。
- 多实例定时任务要防重复执行（分布式锁带 TTL 和唯一 token）。

## Reference loading rules

- Read `references/retry-state.md` before implementing retry, failure state, idempotency keys, or user-visible task status.
- Read `references/lifecycle-resource.md` when jobs use clients, sessions, locks, subprocesses, files, async workers, or distributed resources.
- Read `references/performance-testing.md` only for queue backlog, throughput, latency, memory growth, or regression verification.

## Gotchas

- 不要传 DB session、HTTP client 或大对象给任务。
- 不要对非幂等写操作自动重试。
- 不要让后台任务异常静默失败。
- 不要把 FastAPI BackgroundTasks 当可靠队列。
- 不要用 Celery；如确需，必须先评估系统是否可拆分。
