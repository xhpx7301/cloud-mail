# 本仓库的 Cloudflare 部署绑定

本 Fork 的 `mail-worker/wrangler.toml` 已配置现有的 `cloud-mail` Worker 使用的数据库：

| 类型 | 绑定名 | 现有资源 |
| --- | --- | --- |
| D1 | `db` | `cloudmail_db`，ID 为 `d7e1f57f-c9bf-4767-ad2f-da63c2d0f92b` |
| KV | `kv` | ID 为 `ed97055b0b77485a94362963e7333302` |

资源 ID 不是访问凭据。部署到其他 Cloudflare 账号时，必须替换成自己的资源 ID。

## Cloudflare 连接 GitHub 自动构建

- 构建根目录使用 `mail-worker`，部署使用 `pnpm run deploy` 或 `npx wrangler deploy`。
- 配置中的 `[build]` 会构建 `mail-vue`，静态资源目录为 `mail-worker/dist`。
- 更新上游代码时，保留本 Fork 在 `wrangler.toml` 中的 D1、KV 绑定。不要使用全部注释掉绑定的上游模板覆盖此文件。
- `keep_vars = true` 用于保留在 Cloudflare 控制台配置的普通环境变量，不能保留部署配置里缺少的 D1、KV 资源绑定。
- `domain`、`admin` 继续使用控制台现有值；`jwt_secret` 保存在 Cloudflare 加密 Secret 中，不要提交到 GitHub。

## 使用 GitHub Actions 时

仓库自带的 Actions 工作流使用 `wrangler-action.toml`，与默认配置是两个独立的部署入口。若启用它，应设置以下 Repository Variables 或 Secrets，复用现有数据库：

```text
NAME=cloud-mail
D1_DATABASE_ID=d7e1f57f-c9bf-4767-ad2f-da63c2d0f92b
KV_NAMESPACE_ID=ed97055b0b77485a94362963e7333302
CUSTOM_DOMAIN=cloudmail.333797.xyz
DOMAIN=["333797.xyz"]
ADMIN=admin@333797.xyz
```

另需配置工作流要求的 `CLOUDFLARE_ACCOUNT_ID`、`CLOUDFLARE_API_TOKEN` 和 `JWT_SECRET`。密钥使用 Secrets，`JWT_SECRET` 必须与现有 Worker 保持一致；不要重新生成来代替现有值。

## 更新后出现“数据库未绑定”

这表示 Worker 缺少资源连接，不表示数据被清空。先在 Worker 的“绑定”中恢复 `db` 和 `kv` 到上面的原有资源，再修复仓库配置并重新部署。不要为此删除数据库或创建空库替换。

普通代码更新不需要重新创建数据库；涉及数据库结构升级时，按对应上游版本的迁移说明执行。
