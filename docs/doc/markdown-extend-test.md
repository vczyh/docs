---
title: Markdown拓展
post: false
date: 2020-4-9 01:07:47
---

## 自定义容器

**输入**

```md
::: tip
这是一个提示
:::

::: warning
这是一个警告
:::

::: danger
这是一个危险警告
:::

::: details
这是一个详情块，在 IE / Edge 中不生效
:::
```

**输出**

::: tip
这是一个提示
:::

::: warning
这是一个警告
:::

::: danger
这是一个危险警告
:::

::: details
这是一个详情块，在 IE / Edge 中不生效
:::

你也可以自定义标题：

~~~md
::: danger STOP
危险区域，禁止通行
:::

::: details 点击查看代码
```js
console.log('你好，VuePress！')
```
:::
~~~

::: danger STOP
危险区域，禁止通行
:::

::: details 点击查看代码
```js
console.log('你好，VuePress！')
```
:::

## Emoji

:+1: :grinning: :heart_eyes: :yum: :hugs: :neutral_face: :relieved: :mask: :cowboy_hat_face: :sunglasses: :confused: :triumph: :shit: :smiley_cat: :see_no_evil: :cupid: :wave: :ok_hand: :point_left: :clap: :writing_hand: :muscle: :baby: :frowning_woman: :frowning_man: :tada: :100:

- [全部可用的Emoji](https://github.com/markdown-it/markdown-it-emoji/blob/master/lib/data/full.json)

- [Github Markdown语法支持的Emoji 不保证全部可用](https://github.com/ikatyang/emoji-cheat-sheet)

## 目录

**输入**

```
[[toc]]
```

**输出**

[[toc]]

## 代码块中的行高亮

**输入**

~~~text
``` js {4}
export default {
  data () {
    return {
      msg: 'Highlighted!'
    }
  }
}
```
~~~

**输出**

``` js {4}
export default {
  data () {
    return {
      msg: 'Highlighted!'
    }
  }
}
```