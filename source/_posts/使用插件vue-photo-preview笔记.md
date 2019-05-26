---
title: '使用插件vue-photo-preview笔记'
date: 2018-11-06 17:14:53
tags: ['css','js']
categories: javascript
summary: '项目过程中，需要实现点击查看大图，并实现手势操作，使用插件vue-photo-preview'
---

在开发项目过程中,遇到一个需求,能不能像微信朋友圈图片一样点击查看原图,并且放大查看
百度了解了一下  诸如此类的插件有 `vue-photo-preview` `PhotoSwipe.js` `PinchZoom.js`

<!-- more -->

我在vue项目中选择使用 ** vue-photo-preview**
vue-photo-preview [npm地址](https://www.npmjs.com/package/vue-photo-preview) 

#### 安装

```shell
npm i vue-photo-preview
```

#### 项目中安装
```javascript
// 移动端Vue.js图片预览插件
import preview from 'vue-photo-preview'
import 'vue-photo-preview/dist/skin.css'
Vue.use(preview, {
  fullscreenEl: false // 关闭全屏按钮
})
```

#####  如何使用
+ 在img标签添加preview属性 preview值相同即表示为同一组

```html
<img src="xxx.jpg" preview="0" preview-text="描述文字">
<img src="xxx.jpg" preview="1" preview-text="描述文字">
<img src="xxx.jpg" preview="1" preview-text="描述文字">
<img src="xxx.jpg" preview="2" preview-text="描述文字">
<img src="xxx.jpg" preview="2" preview-text="描述文字">
```

+ 添加对原插件photoswipe的事件响应，示例：
this.$preview.on('close',())=>{//close只是众多事件名的其中一个，更多请查看文档
    console.log('图片查看器被关闭')
})

+ 添加图片查看器实例--this.$preview.self 注意：此实例仅在图片查看器被打开时生效
this.$preview.on('imageLoadComplete',(e,item)=>{
    console.log(this.$preview.self)  //此时this.$preview.self拥有原插件photoswipe文档中的所有方法和属性
})

+ 本次更新后继承了原插件的所有事件、方法和属性，如需复杂使用请多多查看[原插件文档](http://photoswipe.com/documentation/api.html) 

+ 应性能要求 新增大图查看 large标签填写大图路径 （插件的思路是 img的src默认为缩略图），如不填写large，则展示src
```html
<img src="xxx.jpg" large="xxx_3x.jpg" preview="2" preview-text="描述文字">
```

+ 如果图片是异步生成的，在图片数据更新后调用：
this.$previewRefresh()

#### 总结

在移动端项目通常不会用到复杂的api,我也粗略的过了一下.

根据个人的需求进行文档阅读,并配置



因为自己的项目中图片时异步渲染的.用到了api

```javascript
sync mounted () {
    await this.$store.dispatch("getVoteList", this); // 获取图片列表
    this.$previewRefresh() ; // 图片更新后使用图片查看器
}
```

