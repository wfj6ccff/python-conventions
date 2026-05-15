# API design

## Layering

- route：协议转换、依赖解析、调用 service。
- service：业务逻辑、事务边界、跨资源编排。
- data/repository：查询和存储边界。

## Database

- ORM 默认 SQLAlchemy 2.x。
- 同步还是异步按调用链决定，不强行统一。
- service 层决定 commit / rollback 边界。
- 避免 N+1；不要把 lazy load 风险留到响应序列化阶段。
- schema 变更必须说明影响；用什么迁移方式由项目自行决定。

## Repository pattern

只在查询复杂、I/O 与业务混杂、或多个 use case 复用数据访问时使用。
