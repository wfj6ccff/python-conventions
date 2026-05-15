# Changelog

本项目遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [0.1.2] - 2026-05-15

### Changed

- **重试默认 fail fast**：把 stamina 从默认推荐降级为例外路径。`error-handling.md` / `retry-state.md` / `failure-handling.md` / `llm-stack.md` / `stack.md` / `worker/SKILL.md` / `README.md` 同步修订。LLM 同步路径不外加重试（instructor 自带 schema 重试）。

### Added

- `project/references/type-boundaries.md`：Keyword-only 参数（`*` 强制关键字传参）。
- `project/references/code-design.md`：类的使用边界（数据模型 / 运行时容器，不写业务类）；模块按阶段切，不按层切。
- `project/references/logging.md`：阶段耗时日志模板（`perf_counter` + `elapsed_ms`，每阶段一条）。
- `project/references/config-secrets.md`：`@lru_cache get_settings()` + `resolved_xxx` property 收敛多候选 key fallback。
- `api/references/async-lifespan.md`：Runtime 注入模式（`@dataclass` 容器 + lifespan + `app.state.runtime`）。
- `llm/references/structured-output.md`：双层模型（`XxxLLMOutput` / `XxxResponse`）；`@model_validator(mode="after")` 做交叉字段一致性。
- `llm/references/prompt-design.md`：prompt 文件组织（顶层常量 + `build_xxx_user_prompt` builder）。
- `llm/references/rag-cost-eval.md`：混合检索（dense + sparse RRF + 并行召回）；Milvus `in` filter 用 `json.dumps` 安全转义。

### 来源

基于真实 LLM/RAG 项目（FastAPI + instructor + Milvus 混合检索）抽取的通用编码习惯。所有规则跨项目复用，不绑定特定业务。

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
