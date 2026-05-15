---
name: llm
description: Load when user builds or reviews Python LLM, agent, RAG, structured output, tool calling, prompt design, retrieval, embeddings, or cost-sensitive AI code.
---

## Purpose

LLM 代码优先守住结构化输出、边界隔离、成本、限流、日志脱敏和可评估性。

## Required boundaries

- 结构化输出必须落到明确 Pydantic 模型。
- 使用 `instructor` 时统一 `mode=instructor.Mode.MD_JSON`。
- `base_url`、model、timeout、retry、temperature 集中配置。
- prompt、工具调用、状态、记忆、评估分清边界。
- 不记录完整 prompt、用户隐私、密钥或全文档内容。
- RAG 要记录 embedding 模型、top_k、阈值、rerank 参数和过滤条件。

## Reference loading rules

- Read `references/llm-stack.md` for package selection, agent framework choice, document parsing, chunking, embedding/rerank library, observability, or "用什么库做 X" decisions.
- Read `references/prompt-design.md` for writing or reviewing prompts, system/user split, few-shot, jinja2 升级, or prompt versioning.
- Read `references/structured-output.md` before defining LLM outputs, tool results, parsing behavior, or schema boundaries.
- Read `references/failure-handling.md` for timeout, rate limit, auth failure, content filtering, provider error, or parsing failure.
- Read `references/rag-cost-eval.md` for retrieval quality, embeddings, Milvus, cost, token usage, latency, or eval cases.
- Read `references/client-lifecycle.md` for reusable clients, streaming, async calls, vector DB clients, or tool concurrency.

## Gotchas

- 不要引入 langchain / llamaindex / haystack / guidance 等重型框架（详见 `references/llm-stack.md`）。
- 不要把 LLM 返回先当 dict 改完再塞回模型。
- 不要把 prompt、工具调用、状态、记忆、评估混在一个函数里。
- 不要在业务函数里散落模型名。
- 不要在 prompt 里手抄 JSON Schema；由 instructor 注入。
- 不要记录完整 prompt 或原始用户隐私内容。
- 不要临时换向量数据库；默认 Milvus，除非用户明确要求。
