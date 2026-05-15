# Lifecycle and resources

- worker 启动时创建进程内客户端，退出时关闭。
- 任务参数只传可序列化小对象。
- 分布式锁必须有 TTL 和唯一 token。
- 定时任务在多实例下要防重复执行。
- async worker 不要调用阻塞 I/O。
- subprocess 必须有 timeout，并检查退出码。
