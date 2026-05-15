# Retry and state

- retry 必须有最大次数、退避、总耗时上限和可重试异常列表。
- 每次任务记录 job_id、业务 ID、重试次数、失败原因。
- 非幂等操作必须先设计幂等键或唯一约束。
- 用户可见状态要区分 pending、running、succeeded、failed、retrying。
- 不重试参数错误、权限错误和不可恢复业务错误。
