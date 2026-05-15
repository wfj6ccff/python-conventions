# Config and secrets

pydantic-settings + .env，密钥用 `SecretStr`。

## 单一 Settings 类

- 一个项目一个 `Settings(BaseSettings)`，按 `env_prefix` 或嵌套子模型分组。
- 子模型：`DatabaseSettings`、`LLMSettings`、`RedisSettings` 等，挂在主 `Settings` 上。
- 实例化一次（`get_settings()` + `lru_cache`），全局只读。

## 加载顺序

优先级从低到高（高覆盖低）：

1. 字段默认值
2. `.env`
3. `.env.local`（本机覆盖，不入仓）
4. 环境变量
5. 显式构造参数（仅测试）

`SettingsConfigDict(env_file=(".env", ".env.local"), env_file_encoding="utf-8", extra="ignore")`。

## 密钥处理

- 所有 API key、token、密码字段类型为 `SecretStr`。
- 使用时 `.get_secret_value()`；日志输出默认是 `**********`。
- 不在仓库里留 `.env`；只留 `.env.example`。
- 生产从 secrets manager / KMS / k8s secret 注入环境变量。

## 校验与启动

- 启动早期实例化 Settings；缺失或非法字段立即抛错退出。
- 不在请求路径里读 env；只读 settings 对象。
- 关键字段加 `field_validator`（URL、枚举、范围）。

## 配置与代码边界

- Settings 只描述外部输入，不放业务策略常量。
- 业务常量放模块顶层（如 prompt 常量、阈值常量）。
- 不在 Settings 里塞 prompt 文本；prompt 由调用方在业务模块里管理。

## 多环境

- 通过 `APP_ENV=dev|staging|prod` 切换。
- 不为每环境写一个 Settings 子类；同一类，环境变量决定值。

## Gotchas

- 不要把密钥默认值写在代码里（哪怕是占位）。
- 不要直接 `os.getenv` 散落业务函数里。
- 不要在 import 时做网络 / 文件 I/O 校验配置；移到启动钩子。
- `.env.example` 必须与字段同步更新。
- 不在日志或异常里 repr Settings 对象（除非确认所有 secret 字段都是 SecretStr）。
