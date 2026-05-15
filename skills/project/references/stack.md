# Stack baseline

本仓所有 skill 共享同一栈选择。其他 references 只引用本文件，不复读偏好。换包只改这里。

## 版本基线

- Python ≥ 3.11
- Pydantic v2
- SQLAlchemy 2.x
- FastAPI ≥ 0.110
- httpx ≥ 0.27

## 默认栈

| 维度 | 选择 |
|---|---|
| 依赖与运行 | uv |
| 配置入口 | pyproject.toml |
| 配置 / 密钥 | pydantic-settings + .env，SecretStr |
| 边界校验 | Pydantic v2 |
| HTTP 客户端 | httpx |
| HTTP 重试 | 默认 fail fast；仅瞬态错误且操作幂等时用 stamina |
| JSON | stdlib `json`；orjson 仅在性能敏感时启用 |
| 日志 | loguru |
| 路径 | pathlib.Path |
| 时间 | aware datetime + UTC（`datetime.now(UTC)`） |
| 测试 | pytest（+ pytest-asyncio / anyio） |
| 格式化 | 跟随项目；默认 ruff |
| Web/API | FastAPI |
| ORM | SQLAlchemy 2.x；同步或异步按调用链定 |
| 数据（小） | pandas |
| 数据（大 / 列式） | polars / pyarrow / duckdb |
| 后台任务 | APScheduler / RQ / arq |
| CLI（简单） | argparse |
| CLI（复杂） | typer |
| CLI 人读输出 | rich |
| LLM 客户端 | OpenAI SDK / Anthropic SDK / OpenAI 兼容 base_url |
| LLM 结构化输出 | instructor，`mode=instructor.Mode.MD_JSON` |
| Prompt 模板 | 纯字符串 + 顶层常量；条件多用 jinja2 |
| 向量库 | Milvus（pymilvus） |

## 反模式清单

| ❌ 旧 / 不推荐 | ✅ 替换 |
|---|---|
| `requests` | `httpx` |
| 标准 `logging` | `loguru` |
| `os.path` 字符串拼接 | `pathlib.Path` |
| `datetime.utcnow()` / `fromtimestamp(...)` 无 tz | `datetime.now(UTC)` / `fromtimestamp(..., UTC)` |
| `json.dumps(model.dict())` | `model.model_dump_json()` |
| `pip install` + `python -m venv` | `uv add` / `uv sync` |
| `print` 调试 | `loguru.logger.debug/info` |
| 裸 `dict` 跨模块边界 | Pydantic BaseModel |
| Celery（除非确证需要） | APScheduler / RQ / arq |
| `tenacity` 散落 retry 装饰器 | 默认 fail fast；确需重试时统一 `stamina` |
| langchain / llamaindex / haystack / guidance | 原生 SDK + instructor |
| `requirements.txt` 手维护 | `uv` + `pyproject.toml` |
| `model.dict()` / `model.json()`（v1 写法） | `model.model_dump()` / `model.model_dump_json()` |

## 量化判定阈值

| 决策 | 阈值 |
|---|---|
| pandas vs polars | 文件 > 100MB **或** 行数 > 10⁶ → polars |
| argparse vs typer | 子命令 ≥ 3 **或** 参数 ≥ 5 **或** 需要类型补全 → typer |
| 进程内缓存 vs Redis | 单进程 → `functools.cache` / `cachetools.TTLCache`；多进程 / 多实例 → Redis |
| stdlib json vs orjson | 单次 ≥ 1MB **或** 每秒 ≥ 1k 次 → orjson |
| 同步 vs 异步 SQLAlchemy | 与上层框架同协议；纯脚本默认同步 |
| BackgroundTasks vs 队列 | 几十毫秒收尾 → BackgroundTasks；其余 → APScheduler / arq / RQ |

## 默认参数

| 项 | 默认 |
|---|---|
| httpx timeout | connect=5s / read=30s / write=10s / pool=5s |
| stamina 重试 | attempts=3, wait_initial=1s, wait_max=10s, wait_jitter=1s, timeout=30s |
| 外部 API 并发 | Semaphore=5–10 |
| LLM timeout | 30s |
| LLM 重试 | 3 次，退避 1/2/4s |
| RAG chunk | 512 tokens，overlap 64 |
| RAG top_k | 5（rerank 后取 3） |

## 明确禁用 / 强不推荐

- LLM 框架：langchain、llamaindex、haystack、guidance（抽象层过厚，调试成本高）
- 后台任务：Celery（运维复杂；遇到必须用 Celery 的体量，优先重新评估系统拆分）
- HTTP：requests（无 async、无 HTTP/2、维护节奏慢）
- 时间：naive datetime 进入 DB / 序列化 / 跨服务边界
