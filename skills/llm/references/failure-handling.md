# LLM failure handling

LLM 同步路径**默认不外加重试**：instructor 自带 schema 重试，外层叠加只会放大延迟和成本。仅在 timeout / 限流 / 5xx 这类纯瞬态错误确实需要兜底时，再用 stamina，并显式列可重试异常。

## 错误分类

| 类别 | 是否重试 | 备注 |
|---|---|---|
| timeout | 是 | 默认超时 30s |
| rate limit (429) | 是 | 遵守 Retry-After，限总耗时 |
| 5xx provider error | 是 | 限次数 |
| auth failure (401/403) | 否 | 直接抛错 |
| 参数错误 (400) | 否 | 调用方修复 |
| 内容过滤 | 否 | 走业务降级路径 |
| instructor 解析失败 | 视情况 | instructor 已内置有限重试，外层不再叠加 |

## 默认参数

- 单次调用 timeout：30s（拆 connect=5s / read=30s / write=10s / pool=5s 见 client-lifecycle）。
- stamina：attempts=3, wait_initial=1s, wait_max=10s, wait_jitter=1s, timeout=30s。
- instructor `max_retries=2`，与外层 stamina 加总不超过 5 次。

## 日志

- 必记：provider、model、request_id、duration_ms、error_type、模板版本、token 计数。
- 不记：完整 prompt、原始响应、用户隐私字段。
- 同一异常只在边界层记一次。

## 用户可见错误

- 中文说明原因，不暴露供应商原始响应中的敏感内容。
- 区分"暂时不可用，请重试"与"输入不合规"两类用户语义。

## Gotchas

- 不要 `on=Exception` 重试 LLM 调用；显式列异常类。
- 不要在 instructor 外再叠加完整重试链；只叠瞬态错误。
- 不要在 streaming 中途 raise 后忘记 close 流。
