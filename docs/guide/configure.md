# 常用配置项

所有配置都写在 `index.html` 的 `window.$docsify` 对象里，没有第二个配置文件。

## 基础配置

```js
window.$docsify = {
  name: 'docsify 演示站',        // 左上角站点名
  repo: 'https://github.com/baofengxue27/docsify-demo',  // 右上角 GitHub 角标
  homepage: 'README.md',         // 首页文件，默认就是 README.md
  auto2top: true,                // 切换页面回到顶部
  maxLevel: 4,                   // 正文标题最多编号到几级
}
```

## 目录与导航

```js
window.$docsify = {
  loadSidebar: true,   // 左侧目录，读 _sidebar.md
  loadNavbar: true,    // 顶部导航，读 _navbar.md
  subMaxLevel: 3,      // 侧边栏自动展开到 h3（不写就只列手写的条目）
  alias: {             // 路径别名，可以让所有子目录共用同一个侧边栏
    '/.*/_sidebar.md': '/_sidebar.md'
  },
  relativePath: true   // 相对链接按当前路由解析
}
```

`alias` 是大型文档的常用技巧：`/.*/_sidebar.md` 表示「任何子目录请求 `_sidebar.md` 时，都返回根目录那份」，省得每个子目录放一份重复的侧边栏。

## 搜索

```js
window.$docsify = {
  search: {
    placeholder: '🔍 搜索本站…',
    noData: '😞 没找到相关内容',
    paths: 'auto',      // auto = 自动抓取所有 md；也可写成 ['/basics/', '/guide/']
    depth: 3,           // 索引到几级标题
    maxAge: 86400000,   // 索引在 localStorage 缓存 24 小时
  }
}
```

<div class="tip">

⚠️ 搜索是**前端全文索引**：首次搜索要抓一遍全站 `.md`，文档多了会很慢。它只适合中小型文档站。

</div>

## 分页

```js
window.$docsify = {
  pagination: {
    previousText: '上一篇',
    nextText: '下一篇',
    crossChapter: true,     // 允许跨目录翻页
    crossChapterText: true
  }
}
```

## 字体与主题

```js
window.$docsify = {
  themeColor: '#42b883',   // 主题色，会覆盖 CSS 变量
}
```

换整套主题就是换 `index.html` 里那个 `<link>`：

```html
<!-- vue / dark / buble / pure -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify@5/lib/themes/vue.css">
```

## 路由模式

默认 `hash` 模式（`/#/guide/configure`），URL 里带 `#`，但对静态托管最省心。
改成 `history` 模式可以把 `#` 去掉，但必须让服务器把所有未匹配路径都回退到 `index.html`，GitHub Pages 做不到，需要 404.html 兜底：

```js
window.$docsify = {
  routerMode: 'history'   // GitHub Pages 上不推荐
}
```

## 完整配置

本 demo 的完整配置就在 [docs/index.html](https://github.com/baofengxue27/docsify-demo/blob/main/docs/index.html)，每一行都有注释，可以直接抄。
