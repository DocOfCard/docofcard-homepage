# DocOfCard 首页 v1.3.0 · Cloudflare Workers Static Assets

这是从 **v1.2.0 动态 Worker** 转换而来的 **纯静态项目**。保留你调整过的首页文案、Telegram 群与频道、Discourse 论坛、微博客、完整 Logo 与独立 favicon、社交分享 PNG、响应式布局以及 **自动 / 浅色 / 深色** 三档主题。

**不含 `worker.js`，没有 Worker JavaScript 运行请求。** 仅在用户浏览器里运行一个小脚本，用来切换主题及保存偏好。

## 目录

```text
public/
  index.html           完整首页 HTML/CSS/主题切换 JS
  logo.svg             用户提供的完整横版 Logo
  favicon.svg          用户提供的独立图标
  social-card.png      Telegram / X / Facebook 分享预览图
  robots.txt           机器人访问声明
  _headers             CSP 等安全响应头
wrangler.jsonc         Cloudflare 纯静态资源配置
package.json           本地检查与部署命令
tests/                 静态版本检查
```

## 关键行为：所有不存在的页面显示首页

在 `wrangler.jsonc` 中：

```json
"assets": {
  "directory": "./public",
  "not_found_handling": "single-page-application"
}
```

- `https://docofcard.com/` 和 `https://docofcard.com/?utm_source=tg`：加载首页。
- `https://docofcard.com/anything` 或 `/some/deep/path`：返回 **HTTP 200 + 首页 HTML**，**不会跳转**，浏览器地址栏保留原路径。
- `/logo.svg`、`/favicon.svg`、`/social-card.png`：返回对应静态文件。
- 你如果仍要保留老 WordPress `/archives/...` 的 **301 跳转**，需要在 Cloudflare Redirect Rules / Bulk Redirects 中明确设置，或在 `public/_redirects` 文件添加单独的映射（不要为所有未知地址设置重定向）。没有显式重定向的 `/archives/...` 会显示首页。

因为会为未知路径返回首页，`index.html` 引用图标时使用绝对路径 `/logo.svg`、`/favicon.svg`，避免深层路径下资源错误。

## GitHub 自动发布到 Workers

1. 在 GitHub 建立仓库，例如 `docofcard-homepage`，将此 ZIP 解压后的文件**直接上传到仓库根目录**。
2. Cloudflare → **Workers & Pages** → 创建 Worker → 连接 GitHub 仓库；生产分支 `main`。
3. **Root directory** 为仓库根目录；**Build command** 留空；**Deploy command** 为 `npx wrangler deploy`。
4. 先打开对应的 `*.workers.dev` 预览：检查 `/`、`/abc`、`/deep/link`、主题切换和分享图片 URL。
5. 完成检查后，在 Worker 的 **Domains & Routes** 中设置 `docofcard.com` Custom Domain，或者为现有 DNS 已代理的域名配置 Worker Route `docofcard.com/*`。**不要同时让多个 Worker 争用同一个 Route。**

该版本使用静态资源服务：匹配静态文件时不会运行 Worker 脚本；未匹配文件时也由原生 SPA fallback 返回首页 HTML。静态资源请求不计入 Worker 脚本调用额度。

原来的 v1.2.0 Worker 项目连接了相同的 GitHub 仓库时，将根目录替换为本项目即可；首次变更建议在 `*.workers.dev` 确认是否正常后再调整主域名绑定。

## 社交分享与主题

- 社交网站爬虫直接读取 `index.html` 内的 `og:*` 和 `twitter:*`，从 `/social-card.png` 获取 **1200 × 630** 的 PNG；社交卡片是静态图像，不是浏览器 CSS。
- 网站页头使用 `logo.svg`，标签图标使用 `favicon.svg`，二者分离。
- 页面主题按钮默认 **自动**，跟随系统；用户手动选择后保存在浏览器 `localStorage` 的 `docofcard-theme`，不向服务器发送设置。
- 深色和浅色共用一份 HTML/CSS，原四个入口和文案未修改。

### 更新文件后

- 直接更新 `public/index.html`、SVG 或 PNG，提交并合并 GitHub PR 即会触发 Cloudflare 自动部署。
- 如修改 `index.html` 内联主题脚本，必须同步更新 `public/_headers` 中的 `script-src 'sha256-...'`，否则 CSP 会拦截脚本。
- 修改社交分享 PNG 后，更新 HTML 的 `og:image`/`twitter:image` 查询参数，帮助社交平台重新抓取新图。

## 本地验证

```bash
npm test
npm run dev
```

参考：
- https://developers.cloudflare.com/workers/static-assets/
- https://developers.cloudflare.com/workers/static-assets/routing/single-page-application/
- https://developers.cloudflare.com/workers/static-assets/headers/
