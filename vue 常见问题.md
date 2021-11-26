---
title: Vue 常见问题
date: 2021-05-26 16:11:53
tags:
- 前端
- Vue
- 常见问题
categories:
- 前端
- 前端框架
---

## 1.动态绑定 img 的 src 属性

**问题**：不同的status值，加载不同的图片，如下代码虽然动态绑定了src，但是并不能成功加载图片。

```js
<div>
    <img :src="imgUrl"/>
    <p>{{info}}</p>
</div>
computed: {
    imgUrl: function () {
        return './imgs/'+this.status+'.png'
    }
}
```

<!-- more -->

**解决**：动态生成的路径中加上require，可成功加载图片。

```js
<div>
    <img :src="imgUrl"/>
    <p>{{info}}</p>
</div>
computed: {
    imgUrl: function () {
        return require('./imgs/'+this.status+'.png')
    }
}
```

