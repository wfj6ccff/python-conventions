# Async and lifespan

- FastAPI 启动关闭使用 lifespan。
- 连接池、HTTP client、缓存 client 在 lifespan 内创建和关闭。
- async route 不要调用阻塞 I/O、同步 DB 驱动或 CPU 重活。
- 上传文件、stream、request body 只能读一次，读过后显式传递结果。
- 多 worker 下内存缓存和全局锁不共享。
