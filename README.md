# python-conventions

个人 Python 开发偏好的 Claude Code plugin。沿用指定的栈和编码习惯，按 Perplexity 风格做条件加载，**只在用户意图命中时才加载对应 skill**，不堆全量最佳实践。

## 适用场景

| 场景 | 命中 skill |
|---|---|
| 写 / 重构 / review Python 项目 | `project` |
| 设计 FastAPI 接口 | `api` |
| 写 LLM / RAG / Agent 代码 | `llm` |
| 处理 CSV / Parquet / ETL | `data-pipeline` |
| 写后台任务 / 定时任务 | `worker` |
| 写 CLI / DevOps 脚本 | `cli` |

混合场景（如 "FastAPI 接口里调 LLM 返回结构化结果"）会自动加载多个 skill。

## 默认栈

| 维度 | 选择 |
|---|---|
| 运行 | Python ≥ 3.11 / `uv` |
| 边界 | Pydantic v2 / pydantic-settings |
| HTTP | httpx + stamina |
| 日志 | loguru |
| 时间 | aware datetime + UTC |
| API | FastAPI + SQLAlchemy 2.x |
| LLM | OpenAI 兼容 SDK + instructor (`MD_JSON`) |
| Embedding / Rerank | Qwen3-Embedding-0.6B / Qwen3-Reranker-0.6B（仅 API） |
| 向量库 | Milvus（pymilvus） |
| MCP | fastmcp |
| Prompt | 纯字符串 + 顶层常量；条件多用 jinja2 |
| 数据 | pandas（小） / polars / pyarrow / duckdb（大） |
| 后台任务 | APScheduler / RQ / arq（不用 Celery） |
| CLI | argparse / typer / rich |

## 明确禁用

langchain / llamaindex / haystack / guidance、Celery、requests、标准 logging、`os.path`、`datetime.utcnow()`、裸 dict 跨边界。

## 安装

### 方式一：从 marketplace 安装（推荐）

```text
/plugin marketplace add wfj6ccff/python-conventions
/plugin install python-conventions@wfj6ccff
```

更新：

```text
/plugin marketplace update wfj6ccff
```

### 方式二：本地路径直接挂载（临时验证）

```bash
claude --plugin-dir /path/to/python-conventions
```

## 调用方式

加载后所有 skill 通过命名空间访问：

- `python-conventions:project`
- `python-conventions:api`
- `python-conventions:llm`
- `python-conventions:data-pipeline`
- `python-conventions:worker`
- `python-conventions:cli`

通常不需要手动调用。Claude 会按 SKILL.md 的 `description` 自动路由。

## 目录结构

```
python-conventions/
├── .claude-plugin/
│   └── plugin.json
├── .claude-plugin/marketplace.json    # 让本仓库直接作为 marketplace
├── skills/
│   ├── api/
│   ├── cli/
│   ├── data-pipeline/
│   ├── llm/
│   ├── project/
│   └── worker/
├── README.md
├── LICENSE
└── CHANGELOG.md
```

每个 skill 自包含：`SKILL.md` + `references/` (+ 可选 `assets/`)。

## License

MIT © wfj6ccff

## 维护

发版流程、版本号决策、cookbook、故障排除见 [MAINTENANCE.md](./MAINTENANCE.md)。
