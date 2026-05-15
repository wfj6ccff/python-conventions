# Idempotency

## 原则

- 批处理任务必须能重复执行，结果一致。
- 非幂等写操作不能随便自动重试；先设计幂等键或唯一约束。
- 失败原因和关键业务 ID 要可追踪。

## Checkpoint

- 文件命名约定：`<job_name>.ckpt.json`，与输出同目录。
- 内容至少包含：last_processed_id / last_offset、started_at（UTC ISO 8601）、updated_at、run_id。
- 写 checkpoint 用临时文件 + 原子替换（`tempfile` 同目录 + `os.replace`），不要直接覆盖。
- 启动时读 checkpoint，校验 schema，损坏时显式报错而非默认从头跑。

## 输出文件

- 命名带运行时间戳 + 业务 ID：`<job>_<YYYYMMDDTHHMMSSZ>_<id>.parquet`。
- 优先临时文件 + 原子替换；同目录 `.tmp` → `os.replace` 到目标路径。
- 不在写入过程中暴露未完成文件给下游。

## 重复执行

- 输入侧：用 `(source, id, version)` 之类幂等键判断是否已处理。
- DB 写入：`INSERT ... ON CONFLICT DO UPDATE / DO NOTHING`，或先查再写带唯一约束。
- 对象存储：先 head 再 put；put 用确定性 key。

## Gotchas

- 不要把 checkpoint 写到 `/tmp` 或与输出不同盘（原子替换跨盘失效）。
- 不要把幂等键设成时间戳；时间戳不稳定。
- 不要把 checkpoint 与业务输出混在同一文件。
- 失败重跑前必须先确认 checkpoint 与外部副作用一致。
