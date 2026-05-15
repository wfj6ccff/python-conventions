# Retry and state

## 默认不自动重试

后台任务**默认 fail fast**：异常向上抛，让队列层（RQ/arq/APScheduler）按其策略处理失败任务，并在外部记录失败状态。不要在任务函数里包一层 try/except + 重试。

只有同时满足下列条件才在任务函数里加 stamina：

- 失败来自瞬时网络 / 外部服务抖动；
- 操作幂等（有幂等键 / 唯一约束）；
- 队列层的整体重试策略不够细粒度。

## 真要重试时

- retry 必须有最大次数、退避、总耗时上限和可重试异常列表。
- 默认参数：`attempts=3, wait_initial=1s, wait_max=10s, wait_jitter=1s, timeout=30s`。
- 显式列可重试异常类，不要 `on=Exception`。
- 不重试参数错误、权限错误和不可恢复业务错误。
- 非幂等操作必须先设计幂等键或唯一约束。

## 状态记录

- 每次任务记录 job_id、业务 ID、重试次数、失败原因。
- 用户可见状态要区分 pending、running、succeeded、failed、retrying。
