# Cloudflare Workers 部署指南

## 1. 前置准备

### 1.1 创建 D1 数据库（如果还未创建）

```bash
# 本地创建 D1 数据库
wrangler d1 create moontvplus

# 记下返回的 database_id，例如：
# ✅ Successfully created D1 database 'moontvplus'
# database_id = abc123def456ghi789jkl
```

### 1.2 获取必要的凭证

访问 [Cloudflare Dashboard](https://dash.cloudflare.com/)：

1. **Account ID**
   - 进入任意 Workers 页面
   - 左侧栏底部可看到 "Account ID"，或
   - 复制浏览器 URL 中的 ID：`https://dash.cloudflare.com/abc123xyz/workers/overview`

2. **API Token**
   - 进入 [Profile → API Tokens](https://dash.cloudflare.com/profile/api-tokens)
   - 点击 "Create Token"
   - 选择模板：**"Edit Cloudflare Workers"**
   - 点击 "Use template" → "Create Token"
   - 复制生成的 Token

## 2. GitHub Secrets 配置

在你的仓库：**Settings → Secrets and variables → Actions**

添加以下 Secrets：

| Secret 名称 | 值 | 说明 |
|---|---|---|
| `CLOUDFLARE_API_TOKEN` | 你的 API Token | 从上面第 1.2 步获取 |
| `CLOUDFLARE_ACCOUNT_ID` | 你的 Account ID | 从上面第 1.2 步获取 |

## 3. 更新 wrangler.toml

将 `wrangler.toml` 中的占位符替换为实际的数据库 ID：

```toml
[[d1_databases]]
binding = "DB"
database_name = "moontvplus"
database_id = "YOUR_ACTUAL_DATABASE_ID"  # 替换此处，例如：abc123def456
```

## 4. 本地测试

```bash
# 安装依赖
pnpm install

# 构建 Cloudflare 版本
pnpm build:cloudflare

# 本地预览
pnpm preview:cloudflare

# 部署到 Cloudflare
pnpm deploy:cloudflare
```

## 5. 自动部署

配置完成后，每次推送到 `main` 或 `master` 分支时，GitHub Actions 会自动：
1. 构建项目（`pnpm build:cloudflare`）
2. 部署到 Cloudflare Workers（`pnpm deploy:cloudflare`）

## 常见问题

### ❌ "The requested installation does not exist"
- 检查 `database_id` 是否正确
- 运行 `wrangler d1 list` 查看所有数据库

### ❌ API Token 权限不足
- 确保使用了 "Edit Cloudflare Workers" 模板
- 或创建自定义 Token，确保包含 Workers 和 D1 权限

### ❌ 构建失败
- 检查本地是否能成功运行 `pnpm build:cloudflare`
- 查看 Actions 日志获取详细错误信息

## 查看部署状态

1. GitHub: **Actions** 标签查看工作流运行状态
2. Cloudflare: [Workers & Pages](https://dash.cloudflare.com/?to=/:account/workers/overview) 查看部署记录

## 6. 版本校验与升级记录

本仓库已按上游 `Sailing540/MoonTVPlus` 的最新 `main` 分支校验到 `224.0.0` 版本（见 `VERSION.txt`），该版本包含 Turso/libSQL 支持、流量分析、客户端去广告配置等更新。

如果后续需要继续升级 Cloudflare Workers 部署，请先同步上游代码，再重新执行：

```bash
pnpm install
pnpm build:cloudflare
pnpm deploy:cloudflare
```

升级后建议在 Cloudflare Dashboard 检查 Worker 日志，并确认 D1/Upstash/Turso 等存储环境变量与当前 `wrangler.toml`、GitHub Secrets 保持一致。
