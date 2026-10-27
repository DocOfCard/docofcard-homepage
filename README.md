# DocOfCard 首页 · Cloudflare Workers v1.2.0

以用户修改过的 `preview.html` 为基础，保留原文案和四个入口；将配色更新为**暖白 + 深灰 + 品牌珊瑚红**。支持桌面、手机、平板和具备双屏视口 API 的折叠设备。

## 文件

- `CHANGELOG.md`：版本更新记录。

- `worker.js`：Worker 完整首页、路由及社交分享元信息。
- `social-image.js`：分享图片 PNG 的 Base64 二进制数据模块（由 PNG 生成），无需配置 R2、KV 或静态网站资产绑定。
- `social-card.png`：实际的 **1200 × 630 PNG**，用于 Telegram / Facebook / X（Twitter）链接分享卡片。
- `logo.svg`：**用户本次上传的 5.svg 原始红色横版 Logo**，用于网页页头和分享卡片左上角（未重绘）。
- `favicon.svg`：**用户之前上传的红底白字小图标**，仅用作网站 favicon，并以图标本体形式放在分享卡片右侧，不增加白底、外框或装饰性描边。
- `preview.html`：可离线打开的静态页面快照；修改 HTML/CSS/JS 时，应同步 `worker.js` 中的 `HOME_HTML` 和 CSP SHA-256。
- `wrangler.jsonc`：不绑定生产域名的安全配置。
- `tests/worker.test.mjs`：Node 内建测试。

无框架、无运行时外部网络访问、无第三方字体和追踪代码。PNG 已在 Worker 内置，图片始终与当前 Worker 一同发布。

## 日间与夜间模式

页面右上角提供 **自动 / 浅色 / 深色** 三档：

- 默认 **自动**：跟随设备或浏览器的 `prefers-color-scheme`，系统主题变化时立即响应；不按固定时间或地理位置切换。
- **浅色 / 深色**：手动选择立即生效，会通过浏览器 `localStorage`（`docofcard-theme`）记住偏好。无法访问本地存储时，本次标签页仍可切换。
- 启动脚本同步设置 `theme-color`、按钮的 `aria-pressed` 状态。切换不请求服务器，不需要 KV、Cookie 或外部 JavaScript。
- Worker 的 CSP 使用启动脚本的固定 SHA-256 散列，只开放这一段必要的内联脚本。
- **社交分享的 Open Graph 预览图仍是固定 PNG**，由社交平台读取，和访问者浏览器的日夜模式无关。
- 若你修改了 HTML 中的启动脚本，应同步更新 Worker `Content-Security-Policy` 中的 SHA-256 哈希；在修改后运行 `npm test` 进行校验。

## GitHub / Cloudflare 自动部署

1. 在 GitHub 建立仓库，如 `docofcard-homepage`。
2. 将 ZIP 解压后**项目文件夹内的所有文件**上传至仓库根目录（不要直接上传 ZIP）。
3. 在 Cloudflare Workers & Pages 中连接这个 GitHub 仓库，使用 `main` 分支。
4. Root directory 选仓库根目录，Build command 留空，Deploy command：`npx wrangler deploy`。
5. 首次部署先通过 `*.workers.dev` 检查页面和 `/social-card.png`。
6. 此后将修改提交 Pull Request，合并到 `main` 后自动部署。提交 PR 不要求直接修改生产站点。

本地检查：`npm test`；本地启动：`npm run dev`。主题预览可直接打开 `preview.html`，桌面与手机版截图见 `screenshot-*.png`。

## 自定义主域名的特殊注意事项

**你仍有 WordPress 旧文章 `/archives/...` 的 301 跳转需求。** 不要用整个 `docofcard.com/*` 通配路由覆盖所有路径；不要把 Worker Custom Domain 直接接管原有网站，除非已将所有旧跳转整合进同一个 Worker。

如果仍由其他规则处理非首页路径，可以在 Cloudflare 为 Worker 分别建立如下 *精确* Routes：

```text
# 必须：网站首页
docofcard.com
# 必须：分享预览图片，社交爬虫独立抓取
docofcard.com/social-card.png
# 必须：首页页头 Logo
docofcard.com/logo.svg
# 推荐：浏览器图标
docofcard.com/favicon.svg
# 可选：robots
docofcard.com/robots.txt
```

这样 Worker 仅接管上述路径，其他 `/archives/...` 的路由仍由你的旧 301 跳转方案处理。如果使用 `www.docofcard.com`，需相应新增匹配的 Routes。**部署后验证首页带查询参数的 URL**，因为 Cloudflare Route 的精确匹配可能受查询字符串影响。

> 特别注意：如果只给首页绑定精确 Route，而没有让 `/social-card.png` 进入 Worker，那么 Telegram、X、Facebook 的卡片图片会是 404。必须为图片单独加 Route，或使用一个正确处理旧跳转的统一 Worker。

## 社交分享卡片

在 `<head>` 中输出抓取器可直接读取的静态元数据：

- `og:title` / `og:description` / `og:url` / `og:image` / `og:image:width` / `og:image:height` / `og:locale`。
- `twitter:card=summary_large_image`、`twitter:image` 和相关标题与描述。
- `og:image` 指向当前域名 `/social-card.png?v=1.1.1`（版本参数便于平台重新抓图），无需任何 JS 或第三方图片服务。

*社交平台最终如何裁切、显示由平台控制；首次上线以及更换图片后可能遇到缓存，需要等待重新抓取或使用平台的分享调试工具。请确保 WAF、机器人拦截和访问规则没有阻挡爬虫访问首页及图片。*

线上检查：

```bash
curl -I https://docofcard.com/social-card.png
curl -s https://docofcard.com/ | grep -E 'og:image|twitter:card'
```

## 维护

- **四个链接**：目前保留用户上传 HTML 中原来的链接（TG 群组入群导航、频道、论坛及微博客）。
- **文案**：以用户提供的 `preview(1).html` 为准，已保留首页标题、说明、各入口标签与页脚。
- **品牌资源区分**：`logo.svg` 是红色横版完整品牌 Logo，`favicon.svg` 是红底白字正方形图标。更新 Logo 后，需要同步 Worker 的 `LOGO_SVG` 常量，并重新生成 `social-card.png` / `social-image.js`；更新小图标则同步 `FAVICON_SVG`。
- 页面 CSS 嵌在 HTML 中；无需部署 CSS 构建服务。

Cloudflare 文档：https://developers.cloudflare.com/workers/configuration/routing/routes/
