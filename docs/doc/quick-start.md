---
title: 快速上手
post: false
date: 2020-4-5 00:47:12
tags:
  - vuepress-theme-something
---

## 安装

```bash
npm install vuepress-theme-something --save-dev
```

通过 `config.js` 使用主题

```js
module.exports = {
  theme: 'vuepress-theme-something'
}
```

## 使用

除了支持默认主题的所有配置外，本主题提供了多个Layout

| layout          | 功能         |
| --------------- | ------------ |
| PostsLayout     | 显示文章列表 |
| TagsLayout      | 标签云       |
| ArchiveLayout   | 归档         |
| WaterfallLayout | 照片瀑布流   |

### PostsLayout

使用 `PostsLayout` 的页面会显示为博客列表页，先根据官方文档配置导航栏，例如 `/blog/`，然后创建 `docs/blog/index.md` ，那么列表的内容就是 `blog` 文件夹下的所有文章

```
---
layout: PostsLayout
---
```

### TagsLayout

使用 `TagsLayout` 的页面会显示为标签云

```
---
layout: TagsLayout
---
```

### ArchiveLayout

```
---
layout: ArchiveLayout
---
```

### WaterfallLayout

```
---
layout: ArchiveLayout
pictures:
  - src: http://p.vczyh.com/blog/IMG_1096(20200128-152110).JPG
  	info: 科比绝杀
  - src: http://p.vczyh.com/blog/IMG_1097(20200128-153100).JPG
  - src: http://p.vczyh.com/blog/IMG_1102.GIF
---
```

| 参数     | 描述               |
| -------- | ------------------ |
| pictures | 表示图片数组       |
| src      | 图片链接           |
| info     | 图片描述，**可选** |


