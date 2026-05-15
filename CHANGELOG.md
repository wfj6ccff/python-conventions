# Changelog

本项目遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [0.1.0] - 2026-05-15

### 首次发布

- 6 个自包含 skill：`project` / `api` / `llm` / `data-pipeline` / `worker` / `cli`
- Perplexity 风格路由：description 描述用户意图，按需条件加载 references
- 默认栈固化：uv / Pydantic v2 / httpx + stamina / loguru / FastAPI / SQLAlchemy 2.x / instructor / Qwen3-Embedding/Reranker（仅 API）/ Milvus / fastmcp
- 明确禁用 langchain / llamaindex / haystack / guidance / Celery / requests / 标准 logging
- 不涉及：文档解析、Chunking、本地 embedding/rerank、DB 迁移工具
