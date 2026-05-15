# Async and resources

## 协议选择

- async 函数里不要直接调用阻塞 I/O、同步 DB 驱动或 CPU 重活。
- 要么保持同步链路，要么保持异步链路，不要混用并发模型。
- CPU 重活在 async 链路里用 `asyncio.to_thread` 或进程池外推。

## 并发原语（Python 3.11+）

- 并发分组用 `asyncio.TaskGroup`（统一异常聚合 + 取消语义）。
- 控制并发数用 `asyncio.Semaphore`；默认外部 API 5–10，文件 I/O 视磁盘。
- 跨库统一原语用 `anyio`（兼容 asyncio / trio）。
- 单次等待多个任务用 `asyncio.gather(..., return_exceptions=True)` 时必须显式处理异常。

## 超时与取消

- 顶层用 `asyncio.timeout(...)`（3.11+）。
- 并发任务要有超时、取消和资源释放。
- 不让 fire-and-forget 任务静默失败：用 `TaskGroup` 或 `done_callback` 记录异常。

## 资源生命周期

- HTTP client、DB session、文件、锁、subprocess、stream 必须有明确生命周期。
- HTTP client 复用并设置 timeout（默认见 `stack.md`：connect=5s / read=30s / write=10s / pool=5s）。
- DB session 不做全局单例，按请求、任务或 use case 管理。
- 文件读写明确 encoding；结果文件优先临时文件 + 原子替换（`os.replace`）。

## Gotchas

- 不要在 `async def` 里调用同步 DB 驱动或 `requests`。
- 不要 `asyncio.create_task` 后丢弃返回值（任务异常会被静默吞掉）。
- 不要在 lifespan 外创建长生命周期 client。
- `gather` 不要传 generator 表达式；先 `list(...)` 再传，避免热启偏差。
