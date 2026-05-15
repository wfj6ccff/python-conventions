# Prompt design

## 默认形态

- 纯字符串 + 模块顶层常量。
- 命名 `*_PROMPT`（系统 / 主指令）、`*_EXAMPLES`（few-shot）、`*_SCHEMA_HINT`（仅当 instructor 不能覆盖时）。
- 一个 use case 一个 `prompts.py` 或 `prompts/<usecase>.py`，不跨业务复用。

## 文件组织

单文件 `prompts.py` 默认结构：

- 顶层常量 `XXX_SYSTEM_PROMPT = """..."""`，多行；规则编号 1./2./3.，示例放末尾。
- 用户消息用函数构造 `build_xxx_user_prompt(...)`，签名带类型，返回字符串。
- 业务模块只 import 常量和 builder，**不在业务函数里拼 prompt 字符串**。
- 条件分支多到难维护再升 jinja2，不要为 1–2 个变量直接上 jinja2。

```python
# prompts.py
ANALYSIS_SYSTEM_PROMPT = """你是检索入口分析器。

规则：
1. ...
2. ...

示例：
用户请求："..."
正确拆解：...
"""

def build_analysis_user_prompt(query: str) -> str:
    return f"用户请求：\n{query}\n"
```

业务侧：

```python
from app.prompts import ANALYSIS_SYSTEM_PROMPT, build_analysis_user_prompt

result = call_structured_llm(
    system_prompt=ANALYSIS_SYSTEM_PROMPT,
    user_prompt=build_analysis_user_prompt(query),
    response_model=AnalysisLLMOutput,
)
```

## 系统 / 用户边界

- system 写角色、约束、输出格式硬要求。
- user 写本次任务的具体输入。
- few-shot 单独命名常量，便于版本化与 A/B。
- 不把动态用户数据塞进 system。

## 何时升级到 jinja2

满足下列任一即升级：

- 条件分支 ≥ 3 个。
- 循环渲染示例 / 文档块。
- 同一模板多语言或多角色变体。

升级形态：

- 模板独立文件 `prompts/<name>.j2`。
- 渲染入口集中（`render(name, **vars)`），不在业务函数内拼字符串。
- 变量必须类型化（Pydantic 输入模型 → `model_dump()` 喂模板）。

## 版本管理

- 常量后缀 `_V1` / `_V2`；jinja2 文件按目录 `prompts/v2/<name>.j2`。
- 切换通过配置开关，旧版保留至下游全部迁移完。
- 日志记录模板名 + 版本号 + 关键变量 hash。

## Schema 注入

- 不在 prompt 里手抄 JSON Schema。
- 由 instructor 根据 Pydantic 模型注入。
- 仅在 instructor 失效或非结构化任务里写 `*_SCHEMA_HINT`。

## 日志与脱敏

- 不记录完整 prompt。
- 只记：模板名、版本、输入字段名列表、输入内容 hash 或长度、token 计数。
- 用户隐私字段在进入 prompt 前由边界层脱敏。

## Gotchas

- 不要在 prompt 里堆"请确保""一定要"等套话；用 instructor 模型约束代替。
- 不要为 1 个变量上 jinja2。
- 不要把 prompt 散落在 service 函数体里。
- 不要复用别人项目的 prompt 而不写版本号。
