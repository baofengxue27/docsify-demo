# docsify 是什么

> 一个「运行时」的 Markdown 文档站框架：不构建、不生成 HTML，浏览器打开页面后现场把 `.md` 拉下来渲染成网页。

## 和其它方案的区别

文档站工具分成两个流派，理解这条分界线，后面所有“坑”都能自己推出来。

| | 运行时渲染（本 demo，docsify） | 构建型（VitePress / Docusaurus / mkdocs） |
| --- | --- | --- |
| 产物 | 1 个 `index.html` + 一堆 `.md` | 每个 `.md` 编译成独立 `.html` |
| 发布 | `git push` 即上线 | 要跑 `npm run build` 再发布 |
| 首屏 | 等 JS 执行 + 拉 md（有短暂白屏） | 直接返回 HTML，最快 |
| SEO | 差，爬虫看到空壳 | 好 |
| 上手成本 | 极低，会写 Markdown 就能用 | 中，需要懂 Node 构建链 |
| 适合 | 内部文档、教程、笔记、教程类开源书 | 官网、需要 SEO 的产品文档 |

## 三个关键事实

**1. 没有构建产物。**
仓库里没有任何 `.html`（除了 `index.html`），你读到的每一页都是 `.md` 原文件。这也是为什么像 [all-in-rag](https://github.com/datawhalechina/all-in-rag/tree/main/docs) 这类教程项目喜欢它——贡献者只提 Markdown，不用碰前端。

**2. 网站根目录 = Pages 发布目录。**
docsify 用相对路径请求 `.md`，所以 Markdown **必须**放在 Pages 的发布根目录里。本 demo 把 Pages 指向 `/docs`，因此只有 `docs/` 下的文件能被渲染；仓库里其它目录（比如源码）只能外链到 GitHub。

**3. 它在浏览器里做的事，等价于你自己写的那几行。**
读下一节 [渲染原理](basics/how-it-works.md)，你会看到它其实只依赖 `fetch` + `marked` 两个最普通的 API。

## 什么时候别用它

- 页面要被搜索引擎收录、要分享到社交平台出卡片 → 换 VitePress / Docusaurus
- 文档几百篇以上，搜索索引构建会很卡 → 换带服务端搜索的方案
- 需要多语言、版本化文档 → Docusaurus 更成熟
