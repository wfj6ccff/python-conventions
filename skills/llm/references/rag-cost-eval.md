# RAG, cost, eval

## 检索默认参数

| 项 | 默认 |
|---|---|
| chunk size | 512 tokens |
| chunk overlap | 64 tokens |
| top_k（召回） | 5 |
| top_n（rerank 后取） | 3 |
| similarity 阈值 | 视模型而定，必须显式声明 |

调整任一参数需在调用点或配置中记录原因。

## Embedding

- 仅使用 API，不部署本地模型。
- 默认 Qwen3-Embedding-0.6B。
- 使用 instruction 模板 `Instruct: {task_description}\nQuery: {query}`；provider 支持 `instruction` / `task_description` 入参时优先用入参。
- 记录 `model_name + dim + version + task_description`；任一变化分集合。
- embedding 缓存 key = `hash(content) + model_name + version + task_description`。

## 向量库

- 默认 Milvus（pymilvus）。
- 集合按 embedding 模型 + task_description 切分，不共用。
- 写入要带业务过滤字段（doc_id / tenant_id / source）。
- 不要临时换 Chroma / FAISS / pgvector。

## Rerank

- 仅使用 API，不部署本地模型。
- 默认 Qwen3-Reranker-0.6B。
- 使用模板 `<Instruct>: {instruction}\n\n<Query>: {query}\n\n<Document>: {doc}`；provider 支持 `instruction` 入参时优先用入参。
- 顺序：召回 top_k → rerank → 取 top_n。
- 命中分数和 rerank 分数都要可观测。

## 成本与延迟

- 只在用户关心成本、延迟或回归时深入做 token、吞吐或评估设计。
- 关键路径打点：embedding tokens、completion tokens、检索耗时、rerank 耗时、总耗时。
- 按 provider + model + 用途聚合。

## 日志

- 检索结果只保留摘要 + 业务 ID，不存全文。
- 命中分数保留前几位关键值，不打印全量 distance 数组。
- 不记原始用户 query 完整内容；用 hash + 长度。

## Eval

- 重要流程保留最小人工验收样例（pytest fixture）。
- 不为了 eval 引入 langchain。

## Gotchas

- 不要把 chunk overlap 设到 ≥ chunk size 的一半。
- 不要混用不同 embedding 模型的向量。
- 不要把 query 整体扔给 rerank；必须先召回再 rerank。
- 不要在请求路径里跑全量 eval。
