# Client lifecycle

## 客户端

- OpenAI / 兼容服务：`AsyncOpenAI`，按 provider 封装（base_url + key + 默认 timeout）。
- Anthropic：`AsyncAnthropic`。
- Vector DB：`pymilvus` 的 `MilvusClient` / `AsyncMilvusClient`。
- 同一进程内复用，不要每次调用新建。

## Timeout 拆分

- LLM client：connect=5s / read=30s / write=10s / pool=5s。
- 长上下文 / 大输出可单独 override read 到 60–120s，但要在调用点显式声明。
- 不要全局 `timeout=None`。

## 生命周期

- FastAPI：在 lifespan 中创建 / 关闭。
- Worker：进程启动创建，退出关闭。
- 脚本 / CLI：`async with` 包住主流程。
- DI 注入而非全局变量；测试可替换。

## Streaming

- 必须 `async with stream(...) as s:` 或 try/finally 关闭。
- 半消费后必须显式 close。
- 不在 streaming 中途做长时间同步处理；要么转发分片，要么累积后整体处理。

## 工具并发

- 工具调用并发用 `asyncio.Semaphore`，默认 5–10。
- 工具内部的 LLM / 向量 / DB 调用各自有自己的 timeout 和 retry。
- 不在工具循环里并发"全部展开"；按层级分批。

## Gotchas

- 不要在请求路径里 `AsyncOpenAI(...)`；只读全局/lifespan 实例。
- 不要混用同步 / 异步客户端。
- 不要忘记 `await client.close()`；用 `async with` 最稳。
- 流响应必须完整消费或显式取消，避免连接泄漏。
