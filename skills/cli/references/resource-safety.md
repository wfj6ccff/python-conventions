# Resource safety

- 临时文件使用上下文管理或明确清理。
- subprocess 必须检查退出码和 timeout。
- 文件路径用 pathlib.Path，读写明确 encoding。
- HTTP client 设置 timeout。
- 生成结果文件优先临时文件 + 原子替换。
