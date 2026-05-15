# Performance and resources

- 大文件默认避免一次性读入内存。
- 先定位瓶颈，再改实现；不要凭感觉引入缓存或并发。
- 外部数据源必须设置 timeout，并校验响应格式。
- 文件、DB session、HTTP client、对象存储 client 必须有明确生命周期。
- 缓存不是事实来源；必须明确 key、TTL 和失效条件。
