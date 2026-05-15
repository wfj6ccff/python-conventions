---
name: api
description: Load when user designs, implements, refactors, or reviews Python web/API services, especially FastAPI routes, request/response schemas, pagination, errors, database access, or layering.
---

## Purpose

API 层只做协议转换：接收入参、调用业务函数、返回明确响应。业务逻辑和事务边界不要堆在 route 里。

## Required boundaries

- 入参和出参必须有明确 Pydantic 模型。
- 不直接返回 ORM 对象。
- 列表接口必须分页。
- 查询接口要明确过滤、排序和边界。
- 错误响应使用稳定 code 和中文 message。
- 不泄露内部异常、SQL、token、路径、完整请求体。

## Reference loading rules

- Read `references/api-design.md` for route/service/data layering, repository decisions, transaction boundaries, or N+1 risks.
- Read `references/error-responses.md` before designing exception handling or error responses.
- Read `references/async-lifespan.md` for lifespan resources, async clients, sessions, uploads, streams, or multi-worker concerns.
- Read `assets/response-template.md` when user asks for response format.

## Gotchas

- 不要把 ORM 对象穿透到 API 返回层。
- 不要在 route 中写复杂 SQLAlchemy 查询。
- 不要把内部异常原样返回给用户。
- 不要为简单 CRUD 制造多余层级。
- `BackgroundTasks` 只做轻量收尾，不当可靠队列。
