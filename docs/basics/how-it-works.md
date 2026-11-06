# 渲染原理

理解这张图，就理解了 docsify 的全部。

![docsify 渲染流程](images/flow.svg)

## 四步流程

假设访问的地址是：

```text
https://baofengxue27.github.io/docsify-demo/#/guide/configure?id=常用配置项
└────────── 静态文件路径，服务器只看这部分 ──────────┘└──── hash，不会发给服务器 ────┘
```

**第一步：只请求 index.html。**
`/docsify-demo/` 这个路径下服务器返回 `index.html`，里面 `<div id="app">正在加载…</div>` 是空的。
hash 后面的内容（`#/guide/configure?id=...`）**浏览器不会发给服务器**——这点很关键，意味着服务器完全不知道你打开了哪一页。

**第二步：docsify 读 hash，算出要哪个 md。**
docsify 监听 `hashchange` 事件，把 `#/guide/configure` 里的 `guide/configure` 拼成请求地址：

```text
./guide/configure.md
```

**第三步：fetch 拿到 Markdown 原文，用 marked 转成 HTML。**

```js
// docsify 内部的逻辑，简化版大约是这样
const md = await fetch('./guide/configure.md').then(r => r.text())
const html = marked.parse(md)      // Markdown → HTML 字符串
document.querySelector('.markdown-section').innerHTML = html
```

**第四步：后处理。**
生成标题锚点（`?id=` 后面那段就是把标题文字转成 slug：小写、空格变 `-`、去标点）、加载 `_sidebar.md`、给代码块加高亮和复制按钮、跑各个插件的 `doneEach` 钩子。

## 自己验证一遍

打开任意一个 docsify 站，F12 → Network 面板，然后点侧边栏切换页面。你会看到：

- 每次切页只多出 **1 个 `.md` 请求**
- 页面**没有整页刷新**（DOM 只是被替换）

再试试直接访问 Markdown 文件本身，它和网站内容是同一份东西：

```bash
# 返回 200，content-type 是 text/markdown
curl -I https://baofengxue27.github.io/docsify-demo/guide/configure.md
```

## 由此推导出的三个坑

| 现象 | 根因 |
| --- | --- |
| 双击 `index.html` 打开是空白 | `file://` 协议下 `fetch` md 被 CORS 拦截，必须走 HTTP |
| 图片裂开 / 404 | 浏览器按 `location.pathname` 解析相对路径，不含 hash，路径会少算一层 |
| `_sidebar.md` 404 | GitHub Pages 默认用 Jekyll 处理站点，Jekyll 忽略下划线开头的文件，需要 `.nojekyll` |

前两个坑本 demo 都处理了，详见 [插件生态](advanced/plugins.md)。

## 一句话

**前端路由 + 前端 fetch markdown + 前端 marked 渲染**，GitHub Pages 只是个静态文件服务器。
