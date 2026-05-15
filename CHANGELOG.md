# Changelog

本项目遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [0.1.1] - 2026-05-15

### Removed

- LLM 栈收敛到主干：移除 `pydantic-ai`、`logfire` / `langfuse`、`ragas` 推荐。
- `llm-stack.md`：Agent 段移除 `pydantic-ai`；观测段移除 logfire/langfuse 二选一；评估段移除 ragas。
- `rag-cost-eval.md`：Eval 段移除 ragas / langfuse 内置 eval 推荐。
- `stack.md`：反模式表 "可选 pydantic-ai" 尾缀删除。

### 设计取向

LLM 栈现在是**纯主干**：原生 SDK（OpenAI / Anthropic）+ instructor (`MD_JSON`) + Qwen3-Embedding/Reranker（仅 API） + Milvus + fastmcp + loguru。可选工具一律不进推荐位。

## [0.1.0] - 2026-05-15

### 首次发布

- 6 个自包含 skill：`project` / `api` / `llm` / `data-pipeline` / `worker` / `cli`
- Perplexity 风格路由：description 描述用户意图，按需条件加载 references
- 默认栈固化：uv / Pydantic v2 / httpx + stamina / loguru / FastAPI / SQLAlchemy 2.x / instructor / Qwen3-Embedding/Reranker（仅 API）/ Milvus / fastmcp
- 明确禁用 langchain / llamaindex / haystack / guidance / Celery / requests / 标准 logging
- 不涉及：文档解析、Chunking、本地 embedding/rerank、DB 迁移工具
