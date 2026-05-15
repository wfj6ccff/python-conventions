# Async and lifespan

- FastAPI 启动关闭使用 lifespan。
- 连接池、HTTP client、缓存 client 在 lifespan 内创建和关闭。
- async route 不要调用阻塞 I/O、同步 DB 驱动或 CPU 重活。
- 上传文件、stream、request body 只能读一次，读过后显式传递结果。
- 多 worker 下内存缓存和全局锁不共享。

## Runtime 注入

把配置 + 长生命周期客户端放进一个 `@dataclass`，在 lifespan 里建好挂到 `app.state.runtime`，route 通过 `request.app.state.runtime` 取：

```python
from contextlib import asynccontextmanager
from dataclasses import dataclass

@dataclass
class AppRuntime:
    settings: AppSettings
    milvus_client: MilvusClient   # 示例字段，按项目所需挂

@asynccontextmanager
async def lifespan(app: FastAPI):
    settings = get_settings()
    app.state.runtime = AppRuntime(
        settings=settings,
        milvus_client=create_milvus_client(settings),
    )
    yield

@app.post("/search")
async def search(payload: SearchRequest, request: Request):
    return run_pipeline(query=payload.query, runtime=request.app.state.runtime)
```

业务函数只接收 `runtime`，不读全局 settings、不自己 new 客户端。这样：

- 测试可注入 mock runtime；
- 客户端生命周期由 lifespan 唯一管理；
- route 层只做协议转换，业务函数与 FastAPI 解耦。
