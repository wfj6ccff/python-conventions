# Code design

## 判断顺序

先识别核心数据、所有权、修改者、流向和边界。控制流复杂通常是数据结构没设计好。

## 高价值规则

- 少类，多函数；只有需要状态、生命周期或明确边界时才写类。
- 函数只做一件事；超过 3 层缩进优先重新设计数据结构或拆分步骤。
- 不为一次性需求创建抽象基类、插件系统、配置层、兼容层或 fallback。
- 默认不改变公共 API、CLI 参数、返回结构、异常类型、日志格式和用户可见行为。

## 类的使用边界

默认用函数 + 模块。只在两种场景写类：

1. **数据模型**：Pydantic `BaseModel`（API schema、LLM 输出、配置、检索结果等结构化边界）。
2. **运行时容器**：`@dataclass`，把 settings + 长生命周期客户端（DB、HTTP、向量库）打成一包注入。

不要为"有几个相关函数"造业务类，也不要写 `XxxManager` / `XxxService` / `XxxHelper` 这类名字——名字里带 "Manager/Helper" 通常意味着应该是模块级函数。

## 模块按阶段切，不按层切

业务 pipeline 优先按**阶段**切文件，每个文件只导出该阶段的公开函数：

```text
app/
  recall.py     # 召回
  fusion.py     # 融合
  rerank.py     # 重排
  judge.py      # 终审
  pipeline.py   # 编排
```

私有助手用 `_` 前缀紧贴公开函数下方，不为单文件再造子包。只有当一个阶段内部确实有多种实现需要并列（如 `recall/dense.py` + `recall/sparse.py`）时才升子包。

不要给小项目硬套 `routes / services / repositories / models` 的经典分层——分层抽象不为阶段做切分，反而把同一业务流程拆散到多个文件。

## Repository pattern

允许作为常规建议，但只在 I/O、查询和业务逻辑混杂，或多个 use case 复用同一数据访问边界时使用。不要给小脚本或简单模块硬套 repository。
