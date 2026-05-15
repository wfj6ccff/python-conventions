# Type boundaries

- 公开函数、service 函数、数据访问函数、API schema、配置模型必须写清入参和返回类型。
- 外部输入、配置和 LLM 输出使用 Pydantic v2 或明确 schema 校验。
- 不在模块边界之间传递无结构裸 dict。
- 关键字段避免过度依赖隐式类型转换。
- 不强制所有临时变量标注类型。
- 不强制 mypy/pyright；生产服务建议启用，但按项目实际决定。
