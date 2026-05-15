# Error and exit behavior

- 成功返回 0。
- 用户输入错误、配置缺失、外部服务失败、权限错误要映射为稳定非零退出码。
- stderr 输出诊断信息，stdout 保持机器可消费结果。
- 不暴露 SQL、路径、token、完整请求体或内部堆栈。
