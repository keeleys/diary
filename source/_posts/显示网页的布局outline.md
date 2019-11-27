---
title: 显示网页的布局outline
date: 2019-11-13 10:02:53
tags: css
---

```js
[].forEach.call($$("*"),function(a){ a.style.outline="1px solid #"+(~~(Math.random()*(1<<24))).toString(16)})
```