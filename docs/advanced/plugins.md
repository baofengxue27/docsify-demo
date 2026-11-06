# 插件生态

docsify 本体很小，能力几乎全靠插件。插件就是往 `window.$docsify.plugins` 数组里塞函数，函数的第一个参数是 `hook`。

## 钩子（hook）怎么用

```js
window.$docsify = {
  plugins: [
    function (hook, vm) {
      hook.init(() => {});                 // docsify 启动，最早
      hook.beforeEach((markdown) => {      // md 原文 → 返回处理后的原文
        return markdown.replace(/foo/g, 'bar');
      });
      hook.afterEach((html, next) => {     // HTML → 处理后再交给页面
        next(html);
      });
      hook.doneEach(() => {                // 页面已插入 DOM，最适合改 DOM
        console.log('渲染完成');
      });
    }
  ]
}
```

<div class="tip">

改动 DOM（插按钮、改图片 src、挂事件）一律用 `doneEach`；改 Markdown 原文用 `beforeEach`；改最终 HTML 用 `afterEach`。
每次切页这些钩子都会重新执行，所以**插入元素前必须判重**，否则会越插越多。

</div>

## 本 demo 用的插件

| 插件 | 作用 | 引入方式 |
| --- | --- | --- |
| `search` | 全文搜索 | 官方，`lib/plugins/search.min.js` |
| `zoom-image` | 点图片放大 | 官方，`lib/plugins/zoom-image.min.js` |
| `docsify-pagination` | 上一篇 / 下一篇 | 第三方 |
| `docsify-copy-code` | 代码块复制按钮 | 第三方 |
| `docsify-mermaid` | Mermaid 图表 | 第三方 |

CDN 引入顺序很重要：**docsify 核心必须最先**，插件在后。

```html
<script src="https://cdn.jsdelivr.net/npm/docsify@5/lib/docsify.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/docsify@5/lib/plugins/search.min.js"></script>
<!-- 第三方插件放后面 -->
```

## 自己写一个：图片路径归一化

docsify 会把相对图片路径拼上当前页面路由的前缀，于是 `../images/flow.svg` 在 DOM 里变成 `/basics/../images/flow.svg`。浏览器能自己消掉 `..`，功能没问题，但 src 很丑、也不好排查。用 `doneEach` 归一化一下：

```js
function (hook) {
  hook.doneEach(function () {
    document.querySelectorAll('.markdown-section img').forEach(function (img) {
      var src = img.getAttribute('src');
      if (!src) return;
      if (/^(https?:)?\/\//i.test(src)) return;      // 外链跳过
      try {
        // new URL 会自动消掉 . 和 .. 段
        var clean = new URL(src, location.origin).pathname;
        if (clean !== src) img.setAttribute('src', clean);
      } catch (e) { /* 忽略非法 URL */ }
    });
  });
}
```

## 自己写一个：暗黑模式切换

右下角那个 🌙 按钮就是这 20 行。关键是把状态存进 `localStorage`，否则每切一页都会被打回白天。

```js
function (hook) {
  hook.doneEach(function () {
    if (document.querySelector('.theme-toggle')) return;   // 判重！
    var btn = document.createElement('button');
    btn.className = 'theme-toggle';

    if (localStorage.getItem('docsify-theme') === 'dark') {
      document.body.classList.add('dark-mode');
    }
    btn.textContent = document.body.classList.contains('dark-mode') ? '☀️' : '🌙';

    btn.onclick = function () {
      document.body.classList.toggle('dark-mode');
      var dark = document.body.classList.contains('dark-mode');
      localStorage.setItem('docsify-theme', dark ? 'dark' : 'light');
      btn.textContent = dark ? '☀️' : '🌙';
    };
    document.body.appendChild(btn);
  });
}
```

## 常用第三方插件清单

- **docsify-copy-code** — 代码复制按钮
- **docsify-pagination** — 上下篇导航
- **docsify-mermaid** — 流程图、时序图
- **docsify-katex** — 数学公式（`$E=mc^2$`）
- **docsify-tabs** — 多标签页展示同一段代码的不同语言版本
- **docsify-themeable** — 换肤
- **docsify-darklight-theme** — 现成的暗黑模式

插件索引见 [awesome-docsify](https://github.com/docsifyjs/awesome-docsify)。

## 注意版本兼容

docsify 5 和 4 的插件 API 基本一致，但部分老插件只测过 4.x。遇到白屏，先看控制台报错，再把 `@5` 降到 `@4` 试试——本 demo 用的四个插件都已确认在 5.0.0 下正常。
