# Error handling

## 边界与脱敏

- 边界层把底层异常转换为稳定、可理解的错误。
- 用户可见错误用中文说明原因；不暴露 SQL、路径、token、完整请求体、内部堆栈。
- 同一个异常只在边界层记录一次。

## 外部 I/O 分类

区分：timeout、连接失败、鉴权失败、限流、参数错误、不可恢复错误。

| 类别 | 是否重试 |
|---|---|
| `httpx.TimeoutException` / `httpx.ConnectError` | 是 |
| 5xx 服务端错误 | 是（限次数） |
| 429 限流 | 是（遵守 Retry-After） |
| 4xx 参数 / 鉴权 / 权限错误 | 否 |
| 非幂等写操作的任何失败 | 否（除非有幂等键） |
| 业务级不可恢复错误 | 否 |

## 重试（统一 stamina）

- 默认参数：`attempts=3, wait_initial=1s, wait_max=10s, wait_jitter=1s, timeout=30s`。
- 用 `@stamina.retry(on=...)` 显式声明可重试异常类，不要 `on=Exception`。
- 总耗时上限通过 `timeout` 强约束，避免无限退避。
- 写操作必须先有幂等键 / 唯一约束才能重试。

## Gotchas

- 不要散落 `tenacity` 装饰器；统一 stamina。
- 不要捕获 `Exception` 后再 `raise`；要么具体类型，要么转译为边界异常。
- 不要重试鉴权失败 / 限流以外的 4xx。
- 不要在重试装饰器外再加 try/except 吞异常。
