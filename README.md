# docsify-demo

用 **docsify + GitHub Pages** 把一个 Markdown 目录直接变成文档网站的最小示例。
零构建、零依赖、零产物——只有 1 个 `index.html`，其余全是 `.md`。

## 在线预览

<https://baofengxue27.github.io/docsify-demo/>

## 核心原理

```text
浏览器打开 /#/guide/configure
  → hash 不发给服务器，Pages 只返回 index.html
  → docsify 监听 hash，fetch ./guide/configure.md
  → marked 解析成 HTML，插入 #app
```

也就是说 GitHub Pages 只是个静态文件服务器，真正干活的是浏览器里的 docsify。

## 本地跑起来

```bash
cd docs && python3 -m http.server 3000
# 打开 http://localhost:3000
```

必须走 HTTP，直接双击 `index.html` 会因为 `file://` 的跨域限制拿不到 `.md`。

## 目录结构

```text
docsify-demo/
├── docs/                 # ← GitHub Pages 发布根目录
│   ├── index.html        # 唯一的"代码"，所有配置在这里
│   ├── .nojekyll         # 关键：禁用 Jekyll，否则 _sidebar.md 会被忽略
│   ├── _sidebar.md       # 左侧目录
│   ├── _navbar.md        # 顶部导航
│   ├── README.md         # 首页
│   ├── images/
│   ├── basics/           # docsify 是什么 / 渲染原理
│   ├── guide/            # Markdown 写法 / 配置项
│   └── advanced/         # 插件 / 部署踩坑
└── README.md
```

## 演示的功能

- 多级侧边栏 + 顶部导航 + 上一篇/下一篇
- 全文搜索（localStorage 缓存索引）
- 代码高亮 + 复制按钮 + Mermaid 图表 + 图片放大
- 日间/夜间模式切换
- 图片相对路径自动修正

## 发布

仓库 → Settings → Pages → Source: `Deploy from a branch` → `main` / `/docs`。

## 三条最容易踩的坑

1. `docs/.nojekyll` 不能删，删了侧边栏就 404（Jekyll 会忽略下划线开头的文件）
2. 不能双击 `index.html` 打开，必须用 HTTP 服务预览
3. `.md` 必须放在 Pages 发布目录内，否则 docsify 抓不到

更详细的内容见站内《部署与踩坑》一篇。
