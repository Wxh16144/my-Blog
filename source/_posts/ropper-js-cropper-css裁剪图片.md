---
title: 'ropper.js裁剪图片'
date: 2018-10-07 17:18:25
tags: ['css']
categories: javascript
summary: 用户选择头像或者图片，需要进行裁剪，使用ropper.js
---

用户选择头像或者图片，需要进行裁剪，使用ropper.js
<!-- more -->

#### croper.js 图片裁剪
> + Author:Wuxiaohong
> + 官方地址:[官方文档(en)](https://github.com/fengyuanchen/cropperjs/blob/master/README.md)
> + 中文文档参考地址:[中文参考地址](https://blog.csdn.net/weixin_38023551/article/details/78792400)

#### 1.安装
```html
     <link rel="stylesheet" href="./cropper.css">
     ...
     <script src="./cropper.js"></script>
```

#### 2.初始化html

```html
<!--使用块级元素包裹-->
<div class="box">
    <img id="img" src="./a.jpg" alt=""> <!--注意这里的地址不能是网络地址-->
</div>
```
#### 3.实例化对象
```Javascript
    const my = new Cropper(document.querySelector('#img'),{
        ...
    })
```

#### 初始化配置:
> + **`aspectRatio`** 裁剪图片的比例 默认自由 你可以任意比例;
> + **`responsive`** 响应窗口大小, 默认开启
> + **`model`**裁剪的时候裁剪框外层时候为黑色,默认开启;
> + **`guides`** 是否显示裁剪框中的九宫格虚线,默认开启;
> + **`center`**是否显示裁剪框中间准星 默认:开启
> + **`highlight`** 裁剪框中间是否显示一层高亮, 默认开启;
> + **`background`** 图片大小不够时,背景是否显示Photoshop类似的网格底图 默认开启;
> + **`autoCrop`** 初始化的时候是否显示已经裁剪框 默认开启;
> + **`autoCropArea`**。 一个介于0和1之间的数字。定义自动裁剪面积大小(百分比)。 默认80%,建议开启`autoCrop` 查看效果.
> **`cropBoxMovable`** 能够通过拖动移动裁剪框。 默认开启 如果false用户名称拖动都是拉出一个裁剪框;
> + `minContainerWidth` `minContainerHeight` `minCanvasWidth` `minCanvasHeight` `minCropBoxWidth` `minCropBoxHeight` 这几个属性都是设置最小值的,比较有用的是前两个和后两个,前俩个控制整个舞台的最小宽高,后两个控制 裁剪图片的时候,指定用户裁剪的最小尺寸. 中的属性,控制画布中的宽高;

#### 方法和事件
> + `ready()` 因为加载项目需要一定时间,ready就是准备就绪执行的方法
> + `cropstart` `cropmove` `cropend`  `crop` 裁剪开始 持续 结束 触发事件 event;
> + `reset()` 重置裁剪框
> + `clear()` 清除裁剪框;
> + `replace(url,hasSameSize)` 替换图片  url图片地址, 第二个参数如果新映像的大小与旧映像相同，则不会重新构建cropper，只更新所有相关映像的url。这可以用于应用过滤器。不填,默认fase;
> + `enable` `disable` 禁用裁剪框  解除禁用   ;
> + `move(x,y)` 移动要裁剪的图片 如果只有一个参数,第二个参数默认和第一个一样;
> + `moveTo(x,y)` 和 `move` 功能一样,一个相对自己之前本身,一个相对于容器;
> + `zoom()` 和 `zoomTo()` 和上面一样 放大缩小
> + `rotate(90deg)` 和 `rotateTo(90deg)` 2D旋转 前者左右旋转 后者旋转绝对角度;
> + `getCroppedCanvas([options])` 画一张剪裁的图片。如果没有剪裁，则返回一个绘制整个im的画布（这个感觉很有用）;


#### 获取裁剪后的图片资源
``` javascript
let res = my.getCroppedCanvas();
console.log(res.toDataURL()); //输出base64编码
res.toBlob(function (e) {
    console.log(e); //生成Blob的图片格式
})
```