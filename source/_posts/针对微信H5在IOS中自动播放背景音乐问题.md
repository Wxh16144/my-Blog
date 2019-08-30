---
title: 针对微信H5在IOS中自动播放背景音乐问题
date: 2018-09-03 13:53:21
tags: ['css','vue','javascript']
---

在开发微信公众号**H5**时，遇到**IOS**无法自动播放音乐

<!--more-->


### html

```html
<!--audioEl为自定义变量  bgmusic为页面audio ID  forceSafariPlayAudio为自定义方法-->
<!--可根据个人风格改变名称-->

<!--全文src为音频文件路径请根据个人需求更改-->
<!--音频文件默认放在static目录下-->
<!--如果想打包生成请改变webpack配置-->
<!--配置参见地址如下-->
<!--https://zhuanlan.zhihu.com/p/26038486?refer=think-in-vue-->

<!--HTML代码-->

<audio id="bgmusic" 
            autoplay
            controls
            preload
            src="/static/music.mp3" style="display: none;">
</audio>
```

### javascript

+ 可以自动播放时正确的事件顺序是 :
 > **loadstart**  -> **loadedmetadata**  -> **loadeddata**  -> **canplay**  -> **play**  -> **playing**

+ 不能自动播放时触发的事件是
  - iPhone5 **iOS 7.0.6**  **loadstart**
  - iPhone6s **iOS 9.1**   **loadstart** -> **loadedmetadata** -> **loadeddata** -> **canplay**

+ 执行播放时需要注意
  - *iOS 9*   还需要额外的 `load` 一下, 否则直接 `play` 无效
  -  *iOS 7/8* 仅需要 `play` 一下
+ 注意点
  - 由于 iOS Safari 限制不允许 audio autoplay, 必须用户主动交互(例如 click)后才能播放 audio,
  - 因此我们通过一个用户交互事件来主动 `play` 一下 audio.

```javascript
// data中定义变量

this.audioEl = null

// methods中定义方法

 forceSafariPlayAudio() {
  this.audioEl.load(); // iOS 9   还需要额外的 load 一下, 否则直接 play 无效
  this.audioEl.play(); // iOS 7/8 仅需要 play 一下
}
// mounted中调用

// 不能自动播放时触发的事件是
// iPhone5  iOS 7.0.6 loadstart
// iPhone6s iOS 9.1   loadstart -> loadedmetadata -> loadeddata -> canplay

let that = this;
this.$nextTick(()=>{
  this.audioEl = document.getElementById('bgmusic');
  
      document.addEventListener('WeixinJSBridgeReady', that.forceSafariPlayAudio, false)
      
      this.audioEl.addEventListener('play', function() {
      // 当 audio 能够播放后, 移除这个事件
      window.removeEventListener('touchstart', that.forceSafariPlayAudio, false);
  }, false);

  // 由于 iOS Safari 限制不允许 audio autoplay, 必须用户主动交互(例如 click)后才能播放 audio,
  // 因此我们通过一个用户交互事件来主动 play 一下 audio.
  window.addEventListener('touchstart',  that.forceSafariPlayAudio, false);

  this.audioEl.src = './static/music.mp3';
})
```