# Structured output

- 先定义输出模型，再写调用逻辑。
- 字段说明服务稳定解析，不写散文式 prompt。
- 解析失败保留可诊断摘要和原始错误类型。
- 不直接依赖裸 dict。
- 外部输入、配置和 LLM 输出都要经过明确边界。

## 双层模型：LLMOutput 与对外 Response 分离

LLM 直接产出的模型 ≠ 给外部 API 的模型。两者各负责一件事：

- `XxxLLMOutput`：LLM 当前产出的真实形态，含校验器；字段类型按 LLM 实际输出（int、原始枚举等）。
- `XxxResponse`：对外稳定契约；类型按调用方需要（id 转字符串、字段裁剪、字段重排）。

```python
class JudgeLLMOutput(BaseModel):
    type: Literal["single_best", "multiple_valid", "no_confident_match"]
    selected_ids: list[int]
    selected_scores: list[float]
    reason: str

class JudgeResponse(BaseModel):
    type: Literal["single_best", "multiple_valid", "no_confident_match"]
    selected_ids: list[str]   # 对外稳定的字符串 ID
    selected_scores: list[float]
    reason: str
```

转换在一个 builder 函数里一次性完成，不要在多处 `model_dump()` 后各自补字段。

禁止：

- 把 `LLMOutput` 直接当 `response_model` 漏出去；
- LLM 输出类型变化时直接改对外 schema 让调用方一起跟着变。

## 交叉字段一致性用 model_validator

多字段联动用 `@model_validator(mode="after")`，校验失败抛 `ValueError`，让 instructor 触发 schema 重试：

```python
class JudgeLLMOutput(BaseModel):
    type: Literal["single_best", "multiple_valid", "no_confident_match"]
    selected_ids: list[int]
    selected_scores: list[float]

    @model_validator(mode="after")
    def check(self) -> "JudgeLLMOutput":
        if self.type == "single_best" and len(self.selected_ids) != 1:
            raise ValueError("single_best 必须返回 1 个 id")
        if len(self.selected_scores) != len(self.selected_ids):
            raise ValueError("selected_scores 数量必须和 selected_ids 一致")
        return self
```

不要把这种校验放到业务函数里 if-else——一旦放业务侧，instructor 看不到失败，无法触发 schema 重试，模型就拿不到反馈。
