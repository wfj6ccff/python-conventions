# Performance and lightweight testing

## Performance

- 只在用户明确提到慢、性能、内存、吞吐、延迟、大文件或 profiling 时深入优化。
- 先定位瓶颈，再改实现；不要凭感觉引入缓存、并发或重构。
- 大文件默认避免一次性读入内存。
- 缓存必须明确 key、TTL、失效条件和权限维度。

## Testing

- 修 bug 优先补最小回归测试。
- 行为变更或重构后跑最小相关验证。
- 不为了覆盖率写无意义测试。
- 不引入繁杂测试框架；默认 pytest 足够。
