---
title: 使用 mint-ui+lazyload 问题
date: 2018-011-06 17:14:53
tags: ['css']
categories: javascript
summary: 'background-attachment: fixed 在iphone设备失效的解决'
---

在项目中如果遇到图片很多,大家一定会想到懒加载,我也遇到了；
我选择使用mint-ui lazyload
<!--more-->
关于mint-ui 大家可以去[官网](http://mint-ui.github.io/#!/zh-cn)了解一下
lazyload 文档地址 [文档地址](http://mint-ui.github.io/docs/#/en2/lazyload)

#### 项目使用

根据官方文档 创建的目录结构

```html
<div id="container">
  <ul>
    <li v-for="item in list">
      <img v-lazy.container="item.url">
    </li>
  </ul>
</div>
```

添加懒加载样式,自定义加载动画

```css
image[lazy=loading] {
  width: 40px;
  height: 300px;
  /* margin: auto; */
  background: url('~@/assets/images/loading.gif') no-repeat /* fixed */ center;
}
```

#### 注意:这里没有使用 `background-attachment`

一开始以为时懒加载出现问题,所以去查找了标签上 **v-lazy.container** 属性,发现并不是

然后把背景图片替换单独的纯色,发现问题解决了,然后再背景属性中寻找原因.

通过排障方法,找到了卡死的原因 . `background-attachment:fixed`

具体大家可以参考iphone会卡死原因[参考文档](http://www.ptbird.cn/css-background-attachment--fiexed-no-work.html)

所以我们给img标签添加背景的时候,注意不要使用fixed.因为项目中也很少遇到类似的问题