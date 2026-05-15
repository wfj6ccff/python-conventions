# LLM stack

LLM 项目的第三方包白名单。

## 客户端

- OpenAI 生态 / 兼容服务：`openai` 官方 SDK，按 provider 用不同 `base_url` 封装。
- Anthropic：`anthropic` 官方 SDK。
- 不要为简单调用再包一层抽象基类；直接按 provider 暴露 `get_client(provider)`。

## 结构化输出

- 默认 `instructor`，`mode=instructor.Mode.MD_JSON`。
- 输出模型必须是 Pydantic v2 BaseModel。
- 受限制的 grammar / 强约束本地推理可考虑 `outlines`；非默认。

## Token 与成本

- OpenAI 系编码：`tiktoken`。
- Anthropic：使用 SDK 自带的 `count_tokens`，不自己估算。
- 成本聚合按 provider + model + 用途打点。

## 重试

- 默认 fail fast；instructor 已内置 schema 重试，外层不要叠加完整重试链。
- 确需兜底瞬态错误时统一 `stamina`，不要散落 `tenacity` 装饰器。
- 默认参数：attempts=3、wait_initial=1s、wait_max=10s、wait_jitter=1s、timeout=30s。
- 不重试鉴权失败、参数错误、内容过滤、解析错误。

## 流式

- 优先各 SDK 自带 stream。
- 自建 SSE 用 `httpx-sse`。
- 流必须 `async with` 或 try/finally 关闭，禁止半消费悬挂。

## Embedding

- 仅使用 API，不部署本地模型。
- 默认 Qwen3-Embedding-0.6B（API 形式）。
- 调用必须使用 instruction 模板：`Instruct: {task_description}\nQuery: {query}`。
- 若 provider API 直接支持 `instruction` / `task_description` 入参，优先用入参，不在 query 字符串里手拼。
- `task_description` 与业务用途强绑定（如 "Given a web search query, retrieve relevant passages that answer the query"），固化为模块顶层常量。
- 记录 model name + dim + version；同一集合不混不同模型或 task_description。

## Rerank

- 仅使用 API，不部署本地模型。
- 默认 Qwen3-Reranker-0.6B（API 形式）。
- 调用模板：`<Instruct>: {instruction}\n\n<Query>: {query}\n\n<Document>: {doc}`。
- 若 provider API 直接支持 `instruction` 入参，优先用入参，不在 query/document 字符串里手拼。
- `instruction` 与业务用途强绑定，固化为模块顶层常量。
- 召回 top_k=5；rerank 后取 top_n=3。

## 向量库

- 默认 `pymilvus`（Milvus / Zilliz）。
- 不要临时切 Chroma / FAISS / pgvector。
- 集合按 embedding model 切，不共用。

## Agent 与工具调用

- 优先原生：instructor + 自定义工具循环（while + tool dispatch）。
- **禁用**：langchain、llamaindex、haystack、guidance。

## MCP

- 默认 `fastmcp`（装饰器风格、批处理、生命周期友好）。
- 工具暴露按能力分组，避免一个 server 暴露 50 个工具。
- 工具签名用类型注解 + Pydantic 模型；fastmcp 会自动注入 schema。

## 观测

- 默认 loguru 已足够日常调试；只有需要 trace 树 / token 成本聚合 / 评估时再引入。

## 缓存

- 响应级：`aiocache`（多后端）或自建 key。
- Embedding 缓存：内容 hash + model name 为 key。
- 单进程 → `functools.cache` / `cachetools.TTLCache`；多进程 / 多实例 → Redis。

## 评估

- 轻量样例：人工 fixture + pytest。
- 不要为评估引入 langchain。

## 明确禁用

| 包 | 原因 |
|---|---|
| langchain | 抽象层过厚、API 漂移、调试链路长 |
| llamaindex | 同上，且与 langchain 重叠 |
| haystack | 体量过大，运维成本高 |
| guidance | 维护节奏不稳、与 instructor 重叠 |

如确有场景必须用上述任一，先在 PR 描述里说明替代方案为什么不行。
