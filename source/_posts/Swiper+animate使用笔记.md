---
title: Swiper+animate使用笔记
date: 2018/08/14
updated: 2018/08/14
tag: ['javascript', 'css']
summary: Swiper+animate使用笔记
---

Swiper+animate 使用笔记

<!--more-->

### Swiper 滑动插件使用

> 官方地址 [https://www.swiper.com.cn](https://www.swiper.com.cn)

##### 主要学习了 Swiper Animate 使用方法

地址[https://www.swiper.com.cn/usage/animate/index.html](https://www.swiper.com.cn/usage/animate/index.html)
Swiper Animate 是 Swiper 中文网提供的用于在 Swiper 内快速制作 CSS3 动画效果的小插件，适用于 Swiper2.x、Swiper3.x 和 Swiper4.x 。

### 学习路线

- 安装 vue 脚手架

```powershell
$ npm init webpack test
```

- 配置 scss

```powershell
$ npm install --save-dev sass-loader
//sass-loader依赖于node-sass
$ npm install --save-dev node-sass
```

- 修改配置文件

```JavaScript
{
   test: /\.scss$/,
   loaders: ['style', 'css', 'sass']
}
```

- 安装 swiper

```powershell
$ npm install swiper
```

- 按Swiper 官方 Api 引入

```html
<link rel="stylesheet" href="./static/animate.min.css" />
...
<script src="./static/swiper.animate.min.js"></script>
```

```javascript
//App.vue
import Swiper from 'swiper' //在app.vue引入

new Swiper('.swiper-container', {}) //实例化
```

注:实例化后对象查看官方 API [https://www.swiper.com.cn/api/index.html](https://www.swiper.com.cn/api/index.html)

### 使用总结

1 实例化需要在 dom 渲染完成后执行, vue mounted (){} ;钩子函数中使用

2 注意需要包裹两个壳![pic](/images/Swiper+animate使用笔记/swiper_tip.png)

- **红色框** 表示最外层,他需要添加 css overflow: hidden; 注意不需要设置高度.

- **绿色框** 表示显示内容, 这里我设置高度为可视区高度 height:100vh;

- **粉色框** 表示子项内容 可以在里面任意撸 html 代码

  **注:** 上面的框分别是 app.vue 未最外层, 设置 index.vue 包裹所有子项

##### 添加动画

- 根据官方手册 我们需要引入 `swiper.animate.min.js` 和 `animate.min.css`

- 最后在 item 子项添加样式就好了

- 注意 需要添加 class 样式

```html
<h1
class="ani animated"
swiper-animate-effect="rotateIn"
swiper-animate-duration="1s"
swiper-animate-delay="0s"
>
Swiper
</h1>
```
- 特别需要注意 class="ani animated" 是一定要加的 缺一不可
- 动画使用 [https://daneden.github.io/animate.css/](https://daneden.github.io/animate.css/)
- **swiper-animate-effect** 表示执行的动画名称;
- **swiper-animate-duration** 表示动画执行时间
- **swiper-animate-delay** 动画延时执行 (以单个页面计算,就不需要去计算上一个页面的动画时间)
- 另外需要注意的是执行动画的元素不能是行内元素

#### 总结

> 对于个人来说又一阶段的实训已经结束，明显感觉到自己落下了很多东西没有深入挖掘，许多的知识点还是掌握的不太牢固。在以后的学习生活中，我会跟加努力的挖掘出更多的知识，课外知识是庞大的一个体系，我要努力进军其中，努力提高自己的能力。
