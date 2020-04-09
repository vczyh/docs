---
home: true
heroImage: /avatar.jpg
heroText: VuePress Theme Something
tagline: 支持文章列表、标签、归档、图片瀑布流
actionText: 快速上手 →
actionLink: /doc/quick-start
features:
- title: 简洁至上
  details: 以 Markdown 为中心的项目结构，以最少的配置帮助你专注于写作。
- title: Vue驱动
  details: 享受 Vue + webpack 的开发体验，在 Markdown 中使用 Vue 组件，同时可以使用 Vue 来开发自定义主题。
- title: 高性能
  details: VuePress 为每个页面预渲染生成静态的 HTML，同时在页面被加载的时候，将作为 SPA 运行。
footer: Copyright © 2020-present Zhang Yuheng
---

```bash
npm install vuepress-theme-something --save-dev
```

通过 `config.js` 使用主题

```js
module.exports = {
  theme: 'vuepress-theme-something'
}
```