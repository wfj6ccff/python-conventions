# Logging

默认 loguru。

## 默认配置

- 单进程默认一个 stderr sink（loguru 自带）。
- 长期运行进程加 file sink：`rotation="50 MB"` 或 `"1 day"`，`retention="14 days"`，`compression="gz"`。
- 开发环境保留彩色；生产环境关闭颜色，输出 JSON 或固定 plain 格式。
- 第三方库使用标准 logging 时，桥接到 loguru（InterceptHandler 模式）。

## 结构化字段

固定字段（按场景出现）：

- `request_id`：API 请求追踪。
- `job_id`：后台任务。
- `trace_id` / `span_id`：分布式追踪。
- 业务 ID：`user_id` / `order_id` / `doc_id` 等，按 use case 命名稳定。
- `duration_ms`：关键操作耗时。
- `error_type`：异常类名（不是堆栈）。
- `provider` / `model`：LLM 调用。

使用 `logger.bind(...)` 或 `logger.contextualize(...)` 注入，不拼字符串。

## 日志级别

| 级别 | 用途 |
|---|---|
| DEBUG | 本地调试细节，生产关闭 |
| INFO | 关键业务节点（请求开始/结束、任务完成） |
| WARNING | 可恢复异常、降级、限流命中 |
| ERROR | 业务失败、外部调用失败、需要人介入 |
| CRITICAL | 进程级故障 |

## 边界与脱敏

- 同一异常只在边界层记一次；下层抛，上层捕获 + 记录。
- 不记：完整 prompt、原始用户文本、token、密码、API key、完整请求体、SQL 参数明文。
- 必要时记摘要：长度、hash、字段名列表、关键 ID。
- API 错误响应不要把日志内容透传给用户。

## 异步与多进程

- 多进程 worker 使用 `enqueue=True`（loguru sink 参数），避免日志竞态。
- async 上下文用 `logger.contextualize` 在 `async with` 内绑定字段。

## Gotchas

- 不要 `print` 调试上线代码。
- 不要在循环里 `logger.info`，对热点路径用 `logger.opt(lazy=True)` 或抽样。
- 不要在每个模块 `logger.add` 重复加 sink；只在程序入口配置一次。
- 不要把 traceback 完整打到用户面前；只对内部日志保留。
- 文件 sink 路径用 `pathlib.Path`，目录不存在要预先创建。
