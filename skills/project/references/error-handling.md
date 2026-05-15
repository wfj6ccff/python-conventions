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

## 重试是例外，不是默认

**默认 fail fast，业务路径不要包重试。** 上重试需要同时满足：

1. 外部 HTTP / 网络 / DB 瞬时失败，业务对短暂失败可容忍；
2. 操作幂等（写操作必须先有幂等键 / 唯一约束）；
3. 不在 LLM 同步调用路径里（instructor 自带 schema 重试，不要叠加）。

不满足以上条件就让异常向上抛，由边界层转译。不要为了"健壮"在中间层兜底重试。

## 用 stamina（仅例外路径）

确认需要重试时统一用 stamina：

- 默认参数：`attempts=3, wait_initial=1s, wait_max=10s, wait_jitter=1s, timeout=30s`。
- 用 `@stamina.retry(on=...)` 显式声明可重试异常类，不要 `on=Exception`。
- 总耗时上限通过 `timeout` 强约束，避免无限退避。

## Gotchas

- 不要把重试当默认健壮性手段；多数业务函数不该有重试装饰器。
- 不要散落 `tenacity` 装饰器；要重试就统一 stamina。
- 不要捕获 `Exception` 后再 `raise`；要么具体类型，要么转译为边界异常。
- 不要重试鉴权失败 / 限流以外的 4xx。
- 不要在重试装饰器外再加 try/except 吞异常。
- 不要在 instructor 调用外再包一层 stamina；schema 重试已经在 instructor 内。
