# Datetime

所有 datetime 必须 aware；存储 UTC。

## 默认形态

- 取当前时间：`datetime.now(UTC)`，不要 `datetime.utcnow()`。
- 时间戳转 datetime：`datetime.fromtimestamp(ts, UTC)`，必须带 tz。
- 时区库：`zoneinfo`（stdlib，3.9+），不引入 pytz。
- 序列化：ISO 8601 带 offset（Pydantic v2 默认行为）。

## 边界规则

- 入参：API / 配置接受 ISO 8601 字符串，由 Pydantic 校验解析。
- 内部：函数签名用 `datetime`，必须 aware。
- 存储：DB 列统一 `TIMESTAMP WITH TIME ZONE`；存 UTC。
- 展示：仅在最末端输出层做 `astimezone(ZoneInfo("Asia/Shanghai"))` 之类换算。

## 比较与算术

- 不混用 naive / aware 比较；混用会抛异常或得错误结果。
- 时间差用 `timedelta`，不要手算秒。
- 月 / 年差额用 `dateutil.relativedelta`（按需引入）。

## Gotchas

- `datetime.utcnow()` 返回 naive，已弃用，禁用。
- `time.time()` 适合性能计时，不适合业务时间。
- DB 驱动可能默认丢 tz；显式声明列类型并校验回读结果。
- 不把字符串日期靠 `split` / 切片解析；用 `datetime.fromisoformat` 或 Pydantic。
- 跨进程 / 跨服务传递时间一律 UTC ISO 8601 字符串。
