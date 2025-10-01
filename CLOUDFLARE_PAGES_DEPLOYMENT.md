# Cloudflare Pages 部署指南

## 部署问题修复

### 问题1：Edge Runtime 配置错误

**错误信息：**

```text
The following routes were not configured to run with the Edge Runtime:
  - /api/test/simple

Please make sure that all your non-static routes export the following edge runtime route segment config:
  export const runtime = 'edge';
```

**解决方案：**

✅ 已修复：删除了空的 `/api/test/simple/route.ts` 文件和相关目录。

**验证：**

所有API路由现在都正确配置了 `export const runtime = 'edge';`

### 问题2：Windows环境下的bash依赖问题

**错误信息：**

```text
Error: spawn bash ENOENT
```

**原因：**

`@cloudflare/next-on-pages` 在Windows环境下需要bash来执行构建过程。

**解决方案选项：**

#### 选项1：使用 WSL (推荐)

1. 安装 Windows Subsystem for Linux (WSL)
2. 在WSL环境中运行构建命令

#### 选项2：使用 Git Bash

1. 确保已安装 Git for Windows
2. 在Git Bash中运行构建命令：

   ```bash
   pnpm run pages:build
   ```

#### 选项3：云端构建

直接在Cloudflare Pages的CI/CD环境中构建，因为云环境通常是Linux系统。

## 正确的部署步骤

### 1. 本地验证构建

```bash
# 生成运行时配置
pnpm run gen:runtime

# 生成manifest
pnpm run gen:manifest

# Next.js 构建
npx next build

# Cloudflare Pages 适配 (在Linux/WSL环境中)
npx @cloudflare/next-on-pages
```

### 2. Cloudflare Pages 配置

在Cloudflare Pages控制台中设置：

**构建配置：**

- 构建命令: `pnpm install --frozen-lockfile && pnpm run pages:build`
- 构建输出目录: `.vercel/output/static`
- Node.js 版本: `20.x`

**环境变量：** (已在 `wrangler.toml` 中配置)

- `NEXT_PUBLIC_STORAGE_TYPE=d1`
- `NEXT_PUBLIC_SITE_NAME=KatelyaTV`
- 其他变量见 `wrangler.toml`

### 3. 验证部署

部署成功后，检查：

1. 所有API路由是否正常工作
2. 静态页面是否正确生成
3. Edge Runtime是否正常运行

## 常见问题排查

### API路由问题

确保所有API文件都包含：

```typescript
export const runtime = 'edge';
```

### 构建失败

1. 检查所有依赖是否安装完整
2. 确认TypeScript编译无错误
3. 验证环境变量配置

### 性能优化

- 已启用默认代码分割
- PWA缓存策略已配置
- 静态资源优化已开启

## 部署状态验证

部署完成后，访问以下端点验证：

- `/api/server-config` - 服务器配置
- `/api/debug/env` - 环境变量 (开发时)
- 主页 `/` - 前端页面

## 后续维护

1. 定期更新依赖
2. 监控部署日志
3. 备份数据库配置
4. 关注Cloudflare Pages更新
