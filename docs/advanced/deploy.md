# 部署与踩坑

## 发布到 GitHub Pages

### 方式一：分支 + 目录（最省事，本 demo 用的）

仓库 → **Settings** → 左侧 **Pages** → Source 选 `Deploy from a branch` → 分支 `main`、目录 `/docs` → **Save**。

等 1 分钟左右，访问 `https://<用户名>.github.io/<仓库名>/`。

也可以用命令行一步搞定：

```bash
gh api -X POST repos/<用户名>/<仓库名>/pages \
  -f "source[branch]=main" -f "source[path]=/docs"
```

### 方式二：GitHub Actions（需要校验/预渲染时）

```yaml
name: Deploy docs
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: docs
      - uses: actions/deploy-pages@v4
```

此时 Settings → Pages → Source 要选 `GitHub Actions`。

## 五个必踩的坑

### 1. `_sidebar.md` 一直 404

GitHub Pages 默认用 Jekyll 处理站点，而 **Jekyll 会忽略所有下划线开头的文件**。`_sidebar.md`、`_navbar.md` 全被吞掉，表现就是侧边栏永远不出现。

解决：在 Pages 发布目录里放一个空的 `.nojekyll` 文件。

```bash
touch docs/.nojekyll
```

### 2. 本地双击 index.html 一片空白

`file://` 协议下 `fetch` 本地 `.md` 会被跨域策略拦截。必须起 HTTP 服务：

```bash
cd docs && python3 -m http.server 3000
```

### 3. 图片裂开

图片路径是相对**当前 md 文件所在目录**解析的。图片统一放 `docs/images/` 时，子目录页面必须写 `../images/xxx.png`；写成 `images/xxx.png` 会被解析到 `子目录/images/xxx.png`。

也不要用 `/images/xxx.png` 这种站点根绝对路径：项目部署在 `用户名.github.io/仓库名/` 子路径下，`/images/...` 会跑到域名根去 404。

### 4. 中文标题的旧链接失效

标题一改，slug 就变，别人收藏的 `?id=旧标题` 直接跳空。
要么少改标题，要么用 `alias` 做重定向。

### 5. 私有仓库开了 Pages 还是 404

GitHub Pages 对私有仓库需要付费计划（Pro/Team/Enterprise）。免费账号只能发布公开仓库。

## 常见问题速查

| 症状 | 排查方向 |
| --- | --- |
| 页面空白，控制台 404 md | 文件路径大小写、是否在 Pages 根目录下 |
| 侧边栏不显示 | 缺 `.nojekyll`，或 `loadSidebar` 没开 |
| 样式全丢 | CDN 被墙，换国内镜像或把 css/js 下载到本地 |
| 搜索无结果 | 索引还没建完（首次慢），或 `paths` 配置不对 |
| 切页后按钮重复出现 | 插件里 `doneEach` 忘了判重 |

## 换成国内可访问的 CDN

如果受众在国内，jsDelivr 偶尔会慢。可以把 `docsify.min.js`、主题 css 直接下载到 `docs/assets/` 里，用相对路径引入：

```html
<link rel="stylesheet" href="assets/vue.css">
<script src="assets/docsify.min.js"></script>
```

这样整站零外部依赖，离线也能看。
