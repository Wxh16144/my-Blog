---
title: 一眼就能看懂的防抖和节流动画(还能学一点canvas)
date: 2019-08-19 17:54:59
tags: ['javascript', 'canvas']
---
用一组**canvas**动画来描述**防抖(debounce)**和**节流(throttle)**做了些什么;

<!--more-->

> 线上体验: [http://wxhboy.cn/on-line-demo/debounce_throttle.html](http://wxhboy.cn/on-line-demo/debounce_throttle.html)
灵感来源于: [一图秒懂函数防抖和函数节流](https://juejin.im/post/5c404329e51d4551e064a332)


图片预览:
![截图](/images/一眼就能看懂的防抖和节流动画/screenshot.jpg)
动画预览:
![防抖和节流动画](/images/一眼就能看懂的防抖和节流动画/demo.gif)

图: 一条竖线代表一次函数调用，**函数防抖**是间隔超过一定时间后才会执行，**函数节流**是一定时间段内只执行一次。

### 开篇废话

刚刚开始工作的时候自己还是搞不清楚， 防抖和节流到底是什么样一个情况;
每次遇到要写这种需求时, 都是上网百度找代码然后复制粘贴;
直到在掘金社区看到一篇文章[一图秒懂函数防抖和函数节流](ttps://juejin.im/post/5c404329e51d4551e064a332)后**恍然大悟**、**通俗易懂**, 果断的点了赞,收藏;
直到今天自己去面试的时候，让手写防抖和节流， 哈哈... 写了一个比较简易版本的方法; 

今天算是给自己一个总结咯; 用**canvas**把别人文章中的图片实现并动起来;



### Canvas部分
> 使用原型的方式编写canvas

```javascript
const Canvas = function(canvas) {
  const { round } = Math
  this.canvas = canvas || document.createElement('canvas') // canvas 元素
  this.ctx = this.canvas.getContext('2d') // 创建 context 对象
  this.font_content_width = 0 // 左边文本区域大小
  this.row_height = round(this.canvas.height / 3) //每一块区域的高度
  // 渲染页面的数据
  this.data = {
    option: {
      width: 2, // 每个方块的宽度
      height: 0.8, // 每个方块的高度占整行的百分比
      interval: 3 // 两个方块之间的空隙
    },
    // 函数的数据
    lists: {
      common_array_data: {
        desc: '普通(common)',
        list: /*[]*/ [...Array(1000).keys()].map(v => v % Number.MAX_SAFE_INTEGER) // 初始化数据
      },
      debounce_array_data: {
        desc: '防抖(debounce)',
        list: /*[]*/ [...Array(1000).keys()].map(v => v % 2) // 初始化数据
      },
      throttle_array_data: {
        desc: '节流(throttle)',
        list: /*[]*/ [...Array(1000).keys()].map(v => v % 4) // 初始化数据
      }
    }
  }
}
```
> 绘制左边文字部分

```javascript
Canvas.prototype.draw_text = function() {
  const ctx = this.ctx
  const { width, height } = this.canvas // 获取 canvas 宽高
  // 通过计算文本宽度，找到最长的确定为左边文本区域的宽度
  const _set_text_content_width = (text, x = 10) => {
    const { round } = Math
    // 文本两边加上X间距
    const font_width = round(ctx.measureText(text).width + x * 2)
    if (!this.font_content_width) {
      this.font_content_width = font_width
      return
    }
    if (this.font_content_width < font_width) {
      this.font_content_width = font_width
    }
  }
  // 设置字体
  ctx.font = '16px Arial'
  // 绘制左边标题文字
  Object.keys(this.data.lists).forEach((key, index) => {
    // 获取文本
    const text = this.data.lists[key].desc
    // 统计宽度
    _set_text_content_width(text, 10)
    // 计算文本顶部
    const top = this.row_height * (index + 0.5)
    // 绘制文字
    ctx.fillText(text, 10, top)
  })
}
```
> 实现绘制小方块方法

```javascript
Canvas.prototype.fillRect = function() {
  const ctx = this.ctx
  const { min } = Math
  ctx.fillStyle = '#999' // 填充的颜色
  let line_length_arr = [] // 存储三个已经绘制的长度
  // 遍历数据
  Object.keys(this.data.lists).forEach((key, index) => {
    // 获取方法的数据
    const { desc, list } = this.data.lists[key]
    // 每个方法存储的数据
    list.forEach((v, i) => {
      // 获取每个方块的配置属性
      const { width = 2, height = 0.8, interval = 2 } = this.data.option
      // 计算每个方块X坐标  x = 左边文字标题区域 + (方块宽+间隙) * 索引(几个)
      const x = this.font_content_width + i * (width + interval)
      // 计算每个方块的Y坐标 y =每行高度 * ( 剩余百分百对半 + 索引(外循环) )
      const y = this.row_height * ((1 - height) / 2 + index)
      // 如果数组值不为0 则绘制正方形
      if (v) {
        ctx.fillRect(x, y, width, this.row_height * height)
      }
      // 把最后一个正方形的数据收集起来
      if (i === list.length - 1) {
        line_length_arr.push(x + width + interval)
      }
    })
  })
  // 如果没有数据需要默认为空数组；
  line_length_arr = line_length_arr.length ? line_length_arr : [0]
  // 取出最短的值
  const min_line_length = min(...line_length_arr)
  // 判断最短的是否超出右边渲染区域
  if (min_line_length > this.canvas.width) {
    // 擦除右边区域
    ctx.clearRect(
      this.font_content_width,
      0,
      Number.MAX_SAFE_INTEGER,
      Number.MAX_SAFE_INTEGER
    )
    const lists = this.data.lists
    Object.keys(lists).forEach(key => (lists[key].list = []))
  }
}
```

> 实现canvas动画`init`

```javascript
Canvas.prototype.init = function() {
  this.draw_text()
  const _start = () => {
    this.fillRect()
    // console.log('运行中...')
    window.requestAnimationFrame(_start)
  }
  window.requestAnimationFrame(_start)
}
```


------


### 防抖和节流部分

**函数防抖(debounce)**
> 当持续触发事件时，一定时间段内没有再触发事件，事件处理函数才会执行一次;
  如果设定的时间到来之前，又一次触发了事件，就重新开始延时;

```javascript
const debounce = (fn, t) => {
  let time
  return function(...arg) {
    clearTimeout(time)
    let that = this
    time = setTimeout(function(...arg) {
      fn.apply(that, arg)
    }, t)
  }
}
```
**函数节流(throttle)**
> 当持续触发事件时，保证一定时间段内只调用一次事件处理函数;

```javascript
const throttle = (fn, t) => {
   let now = Date.now()
   return function(...arg) {
     if (Date.now() - now > t) {
       fn.apply(this, arg)
       now = Date.now()
     }
   }
}
```

在**Demo**中实现
> 创建一个`example_fn`方法, 并往实例对象中添加数据

```javascript
const example_fn = (time = 300, living, log = false) => {
  const debounce_fn = debounce(() => {
    log && console.error('防抖')
    living.data.lists.debounce_array_data.list.push(1)
  }, time)
  const throttle_fu = throttle(() => {
    log && console.warn('节流')
    living.data.lists.throttle_array_data.list.push(1)
  }, time)
  return function() {
    log && console.log('普通')
    living.data.lists.common_array_data.list.push(1)
    throttle_fu()
    living.data.lists.throttle_array_data.list.push(0)
    debounce_fn()
    living.data.lists.debounce_array_data.list.push(0)
  }
}
```

**创建实例 并 初始化**
```javascript
let C = new Canvas(document.querySelector('canvas'))
C.init()
```

**创建使用实例的方法**
```javascript
const fn = example_fn(300, C, false)
```

**Example**
1. 绑定输入框输入事件

```javascript
document.querySelector('input').addEventListener('input', fn)
```
![输入框输入](/images/一眼就能看懂的防抖和节流动画/input.gif)

2. 绑定canvas鼠标移动事件

```javascript
document.querySelector('canvas').addEventListener('mousemove', fn)
```
![鼠标移动](/images/一眼就能看懂的防抖和节流动画/demo.gif)



3. 绑定浏览器改变窗体大小事件

```javascript
window.addEventListener('resize', fn)
```
![改变窗体大小](/images/一眼就能看懂的防抖和节流动画/window.gif)


### 总结

**函数防抖**：
将几次操作合并为一此操作进行。
原理是维护一个计时器，规定在delay时间后触发函数，但是在delay时间内再次触发的话，就会取消之前的计时器而重新设置。这样一来，只有最后一次操作能被触发。

**函数节流**：
使得一定时间内只触发一次函数。
原理是通过判断是否到达一定时间来触发函数。


感谢阅读
都**9102**年了，我还在讨论这个(手动捂脸)，自己能力有限写的不好，希望大家多多指正
<span style="font-size:12px;color:red">互联网生存不易，请勿狙击</span>
