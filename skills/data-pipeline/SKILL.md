---
name: data-pipeline
description: Load when user processes data with Python, builds ETL or batch jobs, cleans CSV/Parquet/Excel/JSON, validates schemas, handles large files, or needs idempotent data outputs.
---

## Purpose

数据处理优先保证 schema、幂等、可追踪和不静默丢数据。

## Rules

- 先定义输入输出 schema，再写清洗逻辑。
- 小规模表格用 pandas；文件 > 100MB **或** 行数 > 10⁶ 切 polars / pyarrow / duckdb。
- 列式持久化优先 Parquet；与外部交换才用 CSV；Excel 仅用于人读交付，不作为中间格式。
- 大文件默认流式或分块；不一次性读入未知大小文件。
- 批处理要考虑断点续跑、幂等和重复执行。
- 结果文件先写临时文件，再原子替换。
- 丢弃、填充、截断必须有记录。

## Reference loading rules

- Read `references/schema-quality.md` for schemas, cleaning rules, bad rows, rejects, validation, or type conversion.
- Read `references/idempotency.md` for batch jobs, resumability, checkpoints, output replacement, or repeated execution.
- Read `references/performance-resource.md` for large files, memory pressure, object storage, DB reads, external HTTP sources, or slow pipelines.
- Read `assets/rejects-template.md` when bad rows must be reported.

## Gotchas

- 不要静默 drop 异常数据。
- 不要用 pandas 读未知大小或可能超内存的文件。
- 不要靠字符串切割解析结构化格式。
- 不要为一次性小清洗任务引入复杂 pipeline 框架。
- 不要把 Excel 当中间格式在多步骤之间传递。
