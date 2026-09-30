# Agent 操作手册：rwchen 博客维护

> 本文档是给 AI Agent 看的内部操作手册。
> 博客线上版：[《通过 Agent 维护本博客》](https://blog.rwchen.top/2026/06/22/agent-maintain-blog/)
> 用户（rwchen）要求的所有操作，最终都要通过 `git push` 到 `main` 分支生效。

---

## 1. 仓库结构速览

```
rwchen4k/rwchen4k.github.io/      ← 用户站点仓库，main 分支根目录 = 网站根
├── CNAME                         ← blog.rwchen.top
├── 404.html                      ← NexT 自带 QQ 公益 404
├── index.html                    ← 首页
├── about/index.html              ← 关于页
├── schedule/index.html           ← 日程表页
├── archives/                     ← 归档（含 YYYY/、YYYY/MM/）
├── categories/                   ← 分类
├── tags/                         ← 标签聚合页（含 单标签/index.html）
├── 2022/ 2023/ ... 2026/         ← 文章，按 年/月/日/短标题/  组织
├── css/  js/  images/            ← 静态资源
└── _agent/BLOG-MAINTENANCE.md    ← 本文件
```

**关键事实**：
- 这是 **Hexo 已构建产物**，**没有** `_config.yml` / `source/_posts/` / `themes/`
- 每篇文章是一个**完整 HTML 页面**，含 head / sidebar / footer / next-config JSON
- 改一处配置（如 hostname）要扫**所有 17+ 个 HTML 文件**同步替换

---

## 2. 网络与拉取约束

⚠️ **本机 `github.com` 主页不通**（HTTP=000，超时 10s）。但以下可用：

| 资源 | 状态 | 用法 |
|---|---|---|
| `raw.githubusercontent.com` | ✅ 通 | 直接 `curl` 拉单文件 |
| `api.github.com` | ✅ 通 | 走 REST API |
| `github.com`（HTTPS） | ❌ 不通 | git clone / `gh repo clone` 都卡 |
| `git@github.com`（SSH） | ❌ 报"Cloning into..."假成功 | 不要用 |

**推荐拉取方式**：zip 下载

```bash
curl -sL -o blog.zip https://api.github.com/repos/rwchen4k/rwchen4k.github.io/zipball/main
unzip -q blog.zip -d /tmp/rwchen4k-blog/
mv /tmp/rwchen4k-blog/rwchen4k-rwchen4k.github.io-* /tmp/rwchen4k.github.io
```

**推送走 SSH**（已配 key）：

```bash
git push origin main
```

---

## 3. 推送流程（标准操作）

```bash
cd /tmp/rwchen4k.github.io

# 1. 修改（按需）
# 2. 提交
git add -A
git commit -m "<type>: <scope>: <subject>

- <body bullet 1>
- <body bullet 2>

<footer>"

# 3. 推送
git push origin main

# 4. 验证（可选）
sleep 90  # GitHub Pages 重建 + CDN 推送
curl -sI https://blog.rwchen.top/<新文章路径>/ | head -5
```

**commit message 规范**（参考 Conventional Commits）：

- `fix:` 修 bug（如 hostname 修正）
- `feat:` 新增功能（如新文章）
- `docs:` 仅文档（README、_agent/）
- `style:` 样式调整
- `refactor:` 重构

**每篇文章一个独立 commit**，便于 rollback。

---

## 4. 写一篇新文章

### 4.1 路径约定

```
2026/06/22/短横线连接的英文标题/index.html
```

- 用 `YYYY/MM/DD`（北京时间）
- 中文标题也要转成拼音或英文短标题作路径（参考现有：`gauss-elimination`、`mysql-note`、`hello-world`）
- 文件名必须是 `index.html`，目录里只能有这一个文件

### 4.2 必须替换的字段

复制 `2023/01/02/高斯消元/index.html` 作骨架，然后改：

| # | 字段 | 示例值 |
|---|---|---|
| 1 | `<title>` (head) | `<title>通过 Agent 维护本博客 \| rwchen的博客</title>` |
| 2 | `<meta name="description">` | 一句话摘要 |
| 3 | `<meta property="og:title">` | 同文章标题 |
| 4 | `<meta property="og:url">` | `https://blog.rwchen.top/2026/06/22/agent-maintain-blog/index.html` |
| 5 | `<meta property="og:description">` | 同 description |
| 6 | `<meta property="article:published_time">` | ISO-8601 UTC，如 `2026-06-21T17:30:00.000Z` |
| 7 | `<meta property="article:modified_time">` | 同上 |
| 8 | `<link rel="canonical">` | `https://blog.rwchen.top/2026/06/22/agent-maintain-blog/` |
| 9 | `<script class="next-config" data-name="page">` JSON | title / permalink / path |
| 10 | `<h1 class="post-title">` | 文章标题 |
| 11 | `<div class="post-body">` 内全部 | 正文 HTML |
| 12 | `<div class="post-tags">` | 标签 `<a>` 列表（带 `/tags/<encoded>/` 链接） |

### 4.3 正文 HTML 规范

- 用 `<h2 id="锚点">` 分章节，id 用拼音或英文短横线
- `<h2>` 下挂 `<a href="#锚点" class="headerlink" title="..."></a>` 锚点链接
- 代码块：`<figure class="highlight bash">...</figure>`，用 `hexojs/highlight.js` 渲染
- 行内代码：`<code>...</code>`
- 链接：外部加 `target="_blank" rel="noopener"`，内部用相对路径
- 引用：`<blockquote><p>...</p></blockquote>`

### 4.4 标签同步

每篇文章的标签必须**同时**做 3 件事：

1. 文章页底部 `<div class="post-tags">` 写 `<a>` 链接
2. 给每个新标签建 `tags/<标签名>/index.html`（复制 `tags/个人笔记/index.html` 作模板）
3. 在 `tags/index.html` 的标签云里追加新条目

不建标签页 → 标签链接 404。

### 4.5 归档同步

新文章路径 `2026/06/22/xxx/` 需要：

1. 建 `archives/2026/index.html`（如果 `archives/2026/` 不存在）
2. 建 `archives/2026/06/index.html`
3. 在 `archives/index.html` 文章列表里追加新条目

模板都用现有 `archives/2023/01/index.html`，替换 title/permalink/path + 把里面的文章列表改成只有我们这一篇。

---

## 5. 改全局配置

⚠️ **最坑的坑**：NexT 主题的 `next-config` JSON 是**每个 HTML 文件内嵌一份**。

例：改 hostname `rwchen.xyz` → `blog.rwchen.top`，要：

```bash
find . -name "*.html" -exec sed -i \
  -e 's/"hostname":"rwchen.xyz"/"hostname":"blog.rwchen.top"/g' \
  -e 's|https://rwchen.xyz/|https://blog.rwchen.top/|g' {} \;
```

**变更前**永远先 grep 确认影响范围：

```bash
grep -rl '"hostname":"rwchen.xyz"' . --include="*.html"
```

---

## 6. 禁止事项

| 操作 | 原因 |
|---|---|
| ❌ 改 `CNAME` 内容 | 用户没要求换域名就不动 |
| ❌ 删 `404.html` | NexT 自带的 QQ 公益 404，别删 |
| ❌ 改 `js/` `css/` 里的 hash 文件 | NexT 编译产物，主题升级会被覆盖 |
| ❌ 强推保护分支之外的内容 | main 是唯一 publish 分支 |
| ❌ 改 `_config.yml` / 源文件 | 仓库里根本没有，别自己造 |
| ❌ 用 `git clone` 拉仓库 | 本机 github.com 不通，会假成功 |

---

## 7. 验证清单

每次 push 完，**按顺序**检查：

- [ ] `git log --oneline -1` 看到 commit
- [ ] `git log origin/main --oneline -1` 看到远端也有（确认 push 成功）
- [ ] 等 90 秒（GitHub Pages 重建）
- [ ] `curl -sI https://blog.rwchen.top/<新路径>/` 返回 200
- [ ] `curl -s https://blog.rwchen.top/<新路径>/ | grep "<title>"` 看到新标题
- [ ] 浏览器强刷（Ctrl+Shift+R）肉眼检查

---

## 8. 工具/技能备忘

- `gh` CLI 已配置（token `ghp_UB...E1e4`，全权限）
- `git` 已配 SSH key，可直接 push
- 国内 github.com 主页不通 → 用 raw / api 替代
- OpenClaw `openclaw message send` 可给用户微信发推送通知（操作完告诉他一声）

---

## 9. 已知问题

| 问题 | 缓解方案 |
|---|---|
| 国内 github.com 主页封锁 | 用 zipball + api.github.com |
| 仓库是产物不是源 | 改完自己渲染 HTML，不依赖 Hexo |
| archives/categories 索引页同步工作量大 | 每次新增文章都要补 3 个索引文件 |
| 无评论系统 | NexT 评论开关 `comments.style="tabs"` 但 `active=null`，实际无评论功能 |

---

_Last updated: 2026-06-22 by rwchen + AI Agent_